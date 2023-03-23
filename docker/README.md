### Building a docker image for Jupyterhub pod instance

Edit `Makefile` to define dockerhub repository name and tag (version).
```
REGISTRY?=YOUR_ACCOUNT
IMAGE_NAME?=DOCKER_REPO_NAME
VERSION=TAG
```

Make sure that you authenticate with dockerhub.

```bash
docker login
```

Edit `custom software installation` section in `Dockerfile`.

```
########### custom software installation

...

################################################
```

Make it and push to dockerhub.
```bash
make build
make push
```
