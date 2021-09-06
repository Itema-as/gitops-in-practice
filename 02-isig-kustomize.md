# Publisering av iSig på Kubernetes med Argo CD

[iSig](https://github.com/Itema-as/isig) er en liten Spring Boot basert applikasjon som benyttes til å generere e-post signaturer for Itema-ansatte. Den gjør foreløpig veldig lite som krever Java, men planen er å integrere med Google Apps sin adressebok.

## Deklarasjon av applikasjonen

I Git-repoet hvor denne øvelsen utvikles ligger en mappe med navn [./applications](./applications), den inneholder flere filer. Disse skal vi ikke endre på, følgende er bare en forklaring på hva de gjør.

`isig/_base/service.yaml` beskriver _tjenesten_ som applikasjonenen publiserer. Legg merke til at vi også her benytter port **8080**. Denne kommer ikke i konflikt med Argo CD som vi jo også har satt opp til å bruke port 8080. Dette fordi vi kjører på en klynge hvor hver *deployment* får sin egen IP-adresse. I mappen `_base` ligger forøvrig alt som er felles for de to forskjellige miljøene vi skal definere.

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

Dernest har vi `isig/prod/deployment.yaml` som beskriver produksjonssettingen av det Docker-imaget som er bygget og som utgjør distribusjonen av applikasjonen til "produksjon". Her sies det også hvor dette imaget skal hentes fra og hvilken port tjenesten kjører på.

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

Så har vi `isig/prod/kustomization.yaml` som beskriver hvordan [Kustomize](https://kustomize.io) skal håndtere applikasjonsdeklarasjonen og lister også opp andre filer som inngår i deklarasjonen.

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
```

I filen [./argocd-applications/isig/prod/application.yaml](./argocd-applications/isig/prod/application.yaml) finner vi applikasjonsdeklarasjonen til bruk av Argo CD. Her, i `spec.template.spec.containers` angis det hvilket image som skal benyttes, altså `ghcr.io/itema-as/isig:1.0.0`. I tilleg er der en peker til `./applications/isig/prod` hvor infrastruktur, miljøvariabeler og så videre vist ovenfor er definert.

Merk at vi kunne rett og slett klonet repoet og brukt `kubectl apply -k .` i den mappen hvor filene ovenfor ligger og Kubernetes kunne nesten ha gjort hele jobben. Men siden vi også skal lære oss Argo CD gjør vi det litt mer omstendelig.

## Instansiere applikasjonen

For å lage en instans av applikasjonen i Argo CD er det lettest å bruke kommandolinja. Alternativet er å gjøre dette i Argo CD sitt brukergrensesnitt:

```shell
argocd app create isig --repo https://github.com/itema-as/gitops-in-practice \
  --path argocd-applications/isig/prod --dest-server https://kubernetes.default.svc \
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

---

Hvis du har litt tid er det nå en god idé å utforske brukergrensenittet til Argo CD. Klarer du å finne loggen til iSig?

👉 I [neste øvelse](./03-isig-argocd.md) produksjonssetter vi applikasjonen kontinuerlig.
