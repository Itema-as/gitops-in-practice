# Publisering av iSig på Kubernetes

[iSig](https://github.com/Itema-as/isig) er en liten Spring Boot basert applikasjon som benyttes til å generere e-post signaturer for Itema-ansatte. Den gjør foreløpig veldig lite som krever Java, men planen er å integrere med Google Apps sin adressebok.

## Deklarasjon av applikasjonen

I Git-repoet hvor denne øvelsen utvikles ligger en mappe med navn [isig-kubernetes](https://github.com/Itema-as/gitops-in-practice/tree/master/isig-kubernetes), den inneholder flere filer:

`isig-svc.yaml` beskriver _tjenesten_ som applikasjonenen publiserer. Legg merke til at vi også her benytter port **8080**. Denne kommer ikke i konflikt med Argo CD som vi jo også har satt opp til å bruke port 8080. Dette fordi vi kjører på en klynge hvor hver pod får sin egen IP-adresse.

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
      - image: ghcr.io/itema-as/isig:1.0.0
        name: isig
        ports:
        - containerPort: 8080
```

I `spec.template.spec.containers` angis det hvilket image som skal benyttes, altså `ghcr.io/itema-as/isig:1.0.0`. Så beskrivelsen av applikasjonen ligger i de to filene beskrevet ovenfor. Mens den ferdigbygde applikasjonen ligger i GitHub Container Registry.

Sist har vi `kustomization.yaml` som beskriver hvordan [Kustomize](https://kustomize.io) skal håndtere applikasjonsdeklarasjonen og lister også opp andre filer som inngår i deklarasjonen.

```yaml
namePrefix: kustomize-

resources:
- isig-deployment.yaml
- isig-svc.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
```

Merk at vi kunne rett og slett brukt `kubectl apply -k .` i den mappen hvor filene ovenfor ligger og Kubernetes kunne nesten ha gjort hele jobben, bortsett fra at det mangler autentisering mot GitHub. Men siden vi også skal lære oss Argo CD gjør vi det litt mer omstendelig.

## Konfigurere GitHub PAT

For at Argo CD skal kunne hente ut data fra dette repoet, som er et _privat_ GitHub repo, må applikasjonen autentiseres. Dette gjøres ved at du først lager et GitHub *Personal Access Token*. Dette gjøres via **Settings > Developer settings > Personal access tokens** Lag en ny med egenskapene:

- `read:packages`
- `repo`

Nå kan vi logge inn med:

```shell
argocd repo add https://github.com/itema-as/gitops-in-practice \
  --username <github-login> \
  --password <github-pat>
```

Du finner igjen denne konfigurasjone om du går inn i Argo CD under **Settings > Repositories**.

## Instansiere applikasjonen

For å lage en instans av applikasjonen i Argo CD er det lettest å bruke kommandolinja:

```shell
argocd app create isig --repo https://github.com/itema-as/gitops-in-practice \
  --path isig-kustomize --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```

Neste operasjon blir å synkronisere applikasjonen og dernest kontrollere at den faktisk er oppe og kjører. Nå kan det være en god idé å følge med på brukergrensesnittet til Argo CD – da det gir et fint bilde av hva som skjer. Det finner du på https://localhost:8080. Åpne dette og kjør så fra kommandolinja.

```shell
argocd app sync isig && kubectl get svc
```

Nå dukker det opp en god del tekst som viser status på applikasjonen og synkroniseringsstatusen. Den siste linja viser tjenesten i Kubernetes og tilstanden på denne.

```
NAME                     TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
kubernetes               ClusterIP   10.96.0.1      <none>        443/TCP    8d
kustomize-isig-service   ClusterIP   10.111.84.62   <none>        8080/TCP   4s
```

Det kan ta noen sekunder for applikasjonen har startet så kjør `kubectl get pods` for å se om status er `Running`. Hvis så er tilfelle kan vi redirigere trafikken slik at vi når den på vertsmaskina:

```shell
kubectl port-forward svc/isig-service 8081:8080 2>&1 >/dev/null &
```

…og så åpne http://localhost:8081

Hvis du har litt tid er det nå en god idé å utforske brukergrensenittet til Argo CD. Klarer du å finne loggen til iSig?