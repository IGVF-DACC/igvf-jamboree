### Software installation

Install `gcloud` SDK and `helm`.
```bash
sudo apt-get install google-cloud-sdk-gke-gcloud-auth-plugin
sudo snap install helm --classic
```

Create a new project on GCP and authenticate via `gcloud`.
```bash
GCP_PROJECT=igvf-jamboree

gcloud auth login
gcloud auth application-default login
gcloud config set project "$GCP_PROJECT"
````

### Persistent disk for shared data

Create a new persitent disk in GCP project on the console UI and attach it to any VM. SSH to the VM and format the disk and add shared data to `/mnt/shared`. All participants will have read access to this data disk:
```bash
DEVICE_NAME=sdb

sudo mkfs.ext4 -m 0 -E lazy_itable_init=0,lazy_journal_init=0,discard "/dev/$DEVICE_NAME"
sudo mkdir -p /mnt/shared
sudo mount -o discard,defaults "/dev/$DEVICE_NAME" /mnt/shared
```

Make sure to detach the disk (detach it on GCP console UI) before adding it to Kubernetes cluster. Check if `in use by` field on GCP web console is blank. Edit `data_pv.yaml` and `data_pvc.yaml` accordingly.


### Docker image for user's pod instance

Edit `docker/Dockerfile` and push to Dockerhub. It's recommended that the docker image is based on Jupyterhub's base image.


### Kubernetes cluster on GCP

Create a new Kubernetes cluster on GCP. `MACHINE_TYPE_MASTER` is GCP machine type for the load balancer + web server.
```bash
MACHINE_TYPE_MASTER=n1-standard-4
NUM_NODES=5
REGION=us-central1

gcloud container clusters create \
	--machine-type "$MACHINE_TYPE_MASTER" \
	--num-nodes "$NUM_NODES" \
	--region "$REGION" \
	--disk-size=50Gi \
	--cluster-version latest \
	--no-enable-ip-alias	
```

### Zero to Jupyterhub (Z2JH) configuration

It's based on Z2JH's [instruction](https://zero-to-jupyterhub.readthedocs.io/en/stable/index.html).

Install `helm`.
```bash
helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/
helm repo update
````

Create a namespace for Jupyterhub.
```bash
kubectl create -f namespace_jhub.yaml
kubectl get namespace
````

Add the persistent disk (for shared data) to the cluster.
```bash
kubectl --namespace jhub apply -f data_pv.yaml
kubectl --namespace jhub apply -f data_pvc.yaml
kubectl --namespace jhub get pvc
```

Make a copy of `template.config.yaml` and rename it to `config.yaml`.

Generate a secret token and add it to `config.yaml`.
```bash
openssl rand -hex 32
```

Edit `config.yaml` to define docker image for user's pod instance and persistent disk (for shared data) to be attached.


### Deployment

Define `--timeout` in the command line.

```bash
# new deploy
helm upgrade --install --cleanup-on-fail --namespace jhub  --version 1.2.0 --values config.yaml --set global.safeToShowValues=true jhub jupyterhub/jupyterhub --timeout 30m

# redeply after changing config.yaml
helm upgrade --cleanup-on-fail --namespace jhub  --version 1.2.0 --values config.yaml --set global.safeToShowValues=true jhub jupyterhub/jupyterhub --timeout 30m
````

Get a public IP address of the load balancer.
```bash
kubectl -n jhub get svc proxy-public
```

Connect to the load balancer UI with `http://PUBLIC_IP/`.


### Contribution

[Code](https://github.com/kundajelab/jamboree-toolkit) from Anna Shcherbina in Kundaje's lab
