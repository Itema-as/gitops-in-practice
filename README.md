# GitOps i praksis

![](./slides/oversikt.png)

Dette repoet inneholder et sett med øvelser for å gjøre seg kjent med GitHub Actions og kontinuerlig produksjonssetting av applikasjoner i Kubernetes ved hjelp av [Argo CD](https://argo-cd.readthedocs.io/en/stable/). Vi ser også litt på uthenting av metrikker for denne produksjonssettinga med Prometheus, samt visualsering med Grafana.

Nødvendig kode for å instansiere *Argo CD Notifications* i klyngen er også med, men vi skal ikke rigge opp dette da det blir i overkant mange instanser som skal varsle på samme tid. Se heller på Itema Slack, kanalen `#torkildtest` – der vil det bli varslet når det er bygget et nytt *image* som da også vil bli produksjonssatt i din klynge.

## Forberedelser:

Følg [oppskriften](https://github.com/Itema-as/itemacon-2021-forberedelser) for å installere `minikube`, `helm` og `kubectl`. Pass på å også konfigurere en editor som kan benyttes når man kjører `kubectl edit`.

Hvis du har en del kjørende i Minikube fra før av, kan det nå være en god ide å rydde unna slik at du har plass til alle konteinerne som vi skal starte i dette settet med øvelser.

```
❯ minikube delete
🔥  Deleting "minikube" in docker ...
🔥  Deleting container "minikube" ...
🔥  Removing /Users/torkild/.minikube/machines/minikube ...
💀  Removed all traces of the "minikube" cluster.
```

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
## Øvelser

1.  [Installere og starte Argo CD](./01-argocd.md) (omtrent 10 minutter)
2.  [Publisere en applikasjon fra et Git-repo](./02-isig-prod.md) (omtrent 10 minutter)
3.  [Kontinuerlig produksjonssetting med Argo CD](./03-isig-develop.md) (omtrent 15 minutter)
4.  [Samle inn metrikker med Prometheus](./04-argocd-metrics.md) (omtrent 15 minutter)


---
Copyright © 2021, Torkild U. Resheim, Itema AS
