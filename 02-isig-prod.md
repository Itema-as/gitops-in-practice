# Produksjonssetting med Argo CD

[iSig](https://github.com/Itema-as/isig) er en liten Spring Boot basert applikasjon som benyttes til å generere e-post signaturer for Itema-ansatte. Den gjør foreløpig veldig lite som krever Java, men planen er å integrere med Google Apps sin adressebok.

## Deklarering av applikasjonen

I Git-repoet hvor denne øvelsen utvikles ligger en mappe med navn [argocd-applications](./argocd-applications). Den inneholder flere filer som vi ikke skal endre på – kun gå kort gå i gjennom hva de gjør.

```
argocd-applications
├── argocd-notifications
│   ├── _base
│   │   ├── configmap.yaml
│   │   ├── deployment.yaml
│   │   ├── kustomization.yaml
│   │   ├── role.yaml
│   │   ├── rolebinding.yaml
│   │   ├── secret.yaml
│   │   ├── service-account.yaml
│   │   ├── service.yaml
│   │   └── templates.yaml
│   └── minikube
│       ├── configmap.yaml
│       └── kustomization.yaml
└── isig
    ├── _base
    │   ├── kustomization.yaml
    │   └── service.yaml
    ├── develop
    │   ├── application.yaml
    │   ├── deployment.yaml
    │   └── kustomization.yaml
    └── prod
        ├── application.yaml
        ├── deployment.yaml
        └── kustomization.yaml
```

I `isig/_base/service.yaml` beskrives tjenesten som applikasjonenen publiserer. Legg merke til at vi også her benytter port **8080**. Denne kommer ikke i konflikt med Argo CD som vi jo også har satt opp til å bruke port 8080. Dette fordi vi kjører på en klynge hvor hver *deployment* får sin egen IP-adresse. Denne filen er felles for de to miljøene vi skal opprette og i mappen `_base` ligger forøvrig alt som er felles.

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

Dernest har vi `isig/prod/deployment.yaml` som beskriver produksjonssettingen av det Docker-imaget som er bygget og som utgjør distribusjonen av applikasjonen til "produksjon". Her sies det også hvor dette imaget skal hentes fra og hvilken port tjenesten kjører på. Legg merke til at vi for eksemplet har låst oss til versjon 1.0.0 her. Det kommer ikke til må produksjonssatt en ny versjon av denne applikasjonen før man har endret på dette versjonsnummeret.

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

Så har vi `isig/prod/kustomization.yaml` som beskriver hvordan [Kustomize](https://kustomize.io) skal håndtere applikasjonsdeklarasjonen og lister også opp andre filer som inngår i deklarasjonen. Tilsvarende gjelder for `isig/develop/kustomization.yaml` og `isig/_base/kustomization.yaml`.

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - application.yaml
```

I filen [argocd-applications/isig/prod/application.yaml](./argocd-applications/isig/prod/application.yaml) finner vi applikasjonsdeklarasjonen til bruk for Argo CD. Her ligger det en peker til mappen hvor selvsamme fil ligger i – det vanlige er å skille Argo CD og Kustomize deklarasjonene, men vi har lagt alt i samme filstruktur for å gjøre det litt enklere å håndtere i denne øvelsen.

Merk at vi kunne rett og slett klonet repoet og kjørt `kubectl apply -k .` i miljømappene og Kubernetes kunne nesten ha gjort hele jobben. Men siden vi også skal lære oss Argo CD gjør vi det litt mer omstendelig.

## Instansiering av applikasjonen

For å lage en instans av applikasjonen i Argo CD er det lettest å bruke kommandolinja. Alternativet er å gjøre dette i Argo CD sitt brukergrensesnitt:

```shell
argocd app create isig --repo https://github.com/itema-as/gitops-in-practice \
  --path ./argocd-applications/isig/prod --dest-server https://kubernetes.default.svc \
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

Det kan ta noen minutter for applikasjonen har startet så kjør `kubectl get pods` for å se om status er `Running`. Hvis så er tilfelle kan vi kjøre…

```shell
minikube service isig-service
```

…for å kjapt få åpnet nettleseren på tjenesten.

---

Hvis du har litt tid er det nå en god idé å utforske brukergrensenittet til Argo CD. Klarer du å finne loggen til iSig?

👉 I [neste øvelse](./03-isig-develop.md) produksjonssetter vi applikasjonen kontinuerlig for utviklingsmiljøet.
