# GitOps i praksis

## Forberedelser:

Følg [oppskriften](https://github.com/Itema-as/itemacon-2021-forberedelser) for å installere **minikube** og **kubectl**. Pass på å konfigurere en editor som kan benyttes når man kjører `kubectl edit`.

### Start opp minikube

Vi trenger ingen spesielle parametre satt for denne øvelsen, så det er rett og slett bare å kjøre `minikube start`.

```
❯ minikube start
😄  minikube v1.22.0 on Darwin 12.0
✨  Using the docker driver based on existing profile
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🤷  docker "minikube" container is missing, will recreate.
🔥  Creating docker container (CPUs=2, Memory=5896MB) ...
🐳  Preparing Kubernetes v1.21.2 on Docker 20.10.7 ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```
Neste trinn er å [installere og starte Argo CD](./01-argocd.md).


https://kubernetes.io/docs/reference/kubectl/cheatsheet/
