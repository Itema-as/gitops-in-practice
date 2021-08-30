# Publisering av iSig på Kubernetes

[iSig](https://github.com/Itema-as/isig) er en liten Spring Boot basert applikasjon som benyttes til å generere e-post signaturer for Itema-ansatte. Den gjør foreløpig veldig lite som krever Java, men planen er å integrere med Google Apps sin adressebok.

I Git-repoet hvor denne utvikles ligger en mappe med navn "k8s", den inneholder flere filer:

`isig-svc.yaml` beskriver _tjenesten_ som applikasjonenen publiserer. Legg merke til at vi også her bruker port 8080. Denne kommer ikke i konflikt med Argo CD som vi jo også har satt opp til å bruke port 8080. Dette fordi vi kjører på en klynge hvor hver pod får sin egen IP-adresse.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: isig-service
spec:
  selector:
    app: isig
  ports:
  - port: 8080
    targetPort: 8080
```

Dernest har vi `isig-deployment.yaml` som beskriver produksjonssettingen av det Docker-imaget som er bygget og som utgjør distribusjonen av applikasjonen. Her sies det også hvor dette imaget skal hentes fra og hvilken port tjenesten kjører på.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: isig
spec:
  replicas: 1
  revisionHistoryLimit: 3
  selector:
    matchLabels:
      app: isig
  template:
    metadata:
      labels:
        app: isig
    spec:
      containers:
      - image: ghcr.io/itema-as/isig:latest
        name: isig
        ports:
        - containerPort: 8080
```

Her angis det hvilket image som skal benyttes, altså `ghcr.io/itema-as/isig:latest`. Så beskrivelsen av applikasjonen ligger i de to filene beskrevet ovenfor. Mens den ferdigbygde applikasjonen ligger i GitHub Container Registry.

```Shell
argocd app create isig --repo https://github.com/itema-as/isig.git \
  --path k8s --dest-server https://kubernetes.default.svc \
  --dest-namespace default --server localhost:8080 --insecure
```

Neste operasjon blir å synkronisere applikasjonen og dernest kontrollere at den faktisk er oppe og kjører. Nå kan det være en god idé å følge med på brukergrensesnittet til Argo CD – da det gir et fint bilde av hva som skjer. Det ligger på https://localhost:8080. Åpne dette og kjør så fra kommandolinja.

```Shell
argocd app sync isig && kubectl get svc
```

Nå dukker det opp en god del tekst som viser status på applikasjonen og synkroniseringsstatusen. Den siste linja viser tjenesten i Kubernetes og tilstanden på denne.

```
NAME                     TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
kubernetes               ClusterIP   10.96.0.1      <none>        443/TCP    8d
kustomize-isig-service   ClusterIP   10.111.84.62   <none>        8080/TCP   4s
```

For å få testet om iSig faktisk gjør det den skal kan vi redirigere trafikken slik at vi når den på vertsmaskina:

```Shell
kubectl port-forward svc/kustomize-isig-service 8081:8080 2>&1 >/dev/null &
```

…og så åpne http://localhost:8081

Hvis du har litt tid er det nå en god idé å utforske brukergrensenittet til Argo CD. Klarer du å finne loggen til iSig?