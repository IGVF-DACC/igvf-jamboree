### Software installation

Install `gcloud` SDK and `helm`.
```bash
sudo apt-get install google-cloud-sdk-gke-gcloud-auth-plugin
sudo snap install helm --classic
```

Create a new project on GCP web console and authenticate with `gcloud` via command line.
```bash
GCP_PROJECT=som-igvf-jamboree

gcloud auth login --no-launch-browser
gcloud auth application-default login --no-launch-browser
gcloud config set project "$GCP_PROJECT"
```

Enable APIs on the web console.
- Kubernetes Engine API


### Persistent disk for shared data

Create a new `Zonal Standard Persistent Disk` in GCP project on the web console.

Note that `Zonal/Regional Balanced Persistent Disk` can be attach to at most 10 instances in read-only mode. So `Zonal Standard Persistent Disk` is recommended for a jamboree with >10 participants.

Make sure that it's created on the same zone as `ZONE` defined below. Attach it to any VM. SSH to the VM and format the disk and add shared data to `/mnt/shared`. All participants will have read access to this data disk:
```bash
# get device name of the attached disk
lsblk

# define device name
DEVICE_NAME=sdb

# format the disk
sudo mkfs.ext4 -m 0 -E lazy_itable_init=0,lazy_journal_init=0,discard "/dev/$DEVICE_NAME"

# mount it
sudo mkdir -p /mnt/shared
sudo mount -o discard,defaults "/dev/$DEVICE_NAME" /mnt/shared
```

Make sure to detach the disk (detach it on GCP web console) before adding it to Kubernetes cluster. Check if `in use by` field on GCP web console is blank. Edit `data_pv.yaml` to add persistent disk's name to it.


### Docker image for user's pod instance

See [README](docker/README.md).


### Kubernetes cluster on GCP

Set your admin role. Use your gmail account for env var `ADMIN`.
```bash
ADMIN=leepc12@stanford.edu

kubectl create clusterrolebinding cluster-admin-binding \
  --clusterrole=cluster-admin \
  --user="$ADMIN"
```

Create a new Kubernetes cluster on GCP.
```bash
CLUSTER_NAME=igvf-jamboree-2023
ZONE=us-central1-a

gcloud container clusters create \
	--machine-type n1-standard-8 \
	--num-nodes 1 \
	--zone "$ZONE" \
	--disk-size=110Gi \
	--cluster-version latest \
	--enable-autoscaling \
	--min-nodes=1 \
	--max-nodes=100 \
	--no-enable-ip-alias \
	"$CLUSTER_NAME"
```

Close the cluster later after the jamboree.
```bash
gcloud container clusters delete --zone "$ZONE" "$CLUSTER_NAME"
```

Note that deleting a cluster does not delete all persistent disks for individual users. You need to manually delete all  `pvc-*` disks on the web console.


### Zero to Jupyterhub (Z2JH) configuration

It's based on Z2JH's [instruction](https://zero-to-jupyterhub.readthedocs.io/en/stable/index.html).

Install `helm`.
```bash
helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/
helm repo update
````

Create a namespace for Jupyterhub and add shared disk's name to `data_pv.yaml` and run the followings.
```bash
kubectl create -f namespace_jhub.yaml
kubectl get namespace

kubectl --namespace jhub apply -f data_pv.yaml
kubectl --namespace jhub apply -f data_pvc.yaml
kubectl --namespace jhub get pvc
```

Generate a secret token and add it to `config.yaml`.
```bash
openssl rand -hex 32
```

Make a copy of `template.config.yaml` and rename it to `config.yaml`. Edit `config.yaml` to define docker image for user's pod instance and persistent disk (for shared data) to be attached.


### Deployment

To install a new deploy:
```bash
helm upgrade --install --cleanup-on-fail --namespace jhub  --version 1.2.0 --values config.yaml --set global.safeToShowValues=true jhub jupyterhub/jupyterhub --timeout 30m
```

Get a public IP address of the load balancer. It takes about a minute to get an external IP address.
```bash
kubectl -n jhub get svc proxy-public
```

Users/admins can connect to the load balancer UI with `http://SERVER_PUBLIC_IP/`.

To re-deploy after changing `config.yaml`.
```bash
helm upgrade --cleanup-on-fail --namespace jhub  --version 1.2.0 --values config.yaml --set global.safeToShowValues=true jhub jupyterhub/jupyterhub --timeout 30m
````

### Login password

No password is needed for the first login. Users input their own password for the first login.


### Contribution

[Code](https://github.com/kundajelab/jamboree-toolkit) from Anna Shcherbina in Kundaje's lab
