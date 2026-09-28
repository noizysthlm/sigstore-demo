## App Deployment
### Set up a virtual machine
The container in [docker.io/noizysthlm/sigstore-demo](docker.io/noizysthlm/sigstore-demo) runs on amd64 linux.
- Set up a virtual machine on AWS or any other environment.
- Install Docker in rootless mode, Minikube, HELM, and kubectl if not installed by Minikube.
- Make sure that port 8080 (TCP) is accessible.
### Deploy the app
`kubectl apply -f ./<webapp-deployment.yml>` and then `kubectl apply -f ./<webapp-service.yml>`.

Port forward with `kubectl port-forward --address 0.0.0.0 service/dummy-webapp-service 8080:8080` and access the webpage should be accessible on port 8080.

## The structure of this repo

### Workflows
There is currently one workflow in this repo

#### container-build-push
Builds and pushes, the image to `ghcr.io`. Image name will be noizysthlm/sigstore-demo, available tags are `latest` and the commit hashes.