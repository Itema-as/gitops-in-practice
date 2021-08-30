# Kontinuerlig produksjonssetting

Nå skal vi lage et opplegg hvor vi produksjonssetter kontinuerlig, altså for hver enkelt endring som kommer inn på `master`-greina. For å få til dette må vi først installere en komponent i Argo CD som kan reagere på endringer i et *Docker Container Registry*; [Argo CD Image Updater](https://argocd-image-updater.readthedocs.io/en/latest/install/start/).

```shell
kubectl apply -n argocd -f \
  https://raw.githubusercontent.com/argoproj-labs/argocd-image-updater/stable/manifests/install.yaml
```

Kjører du `kubectl get pods -n argocd` skal du nå se at den nye podden er oppe og kjører. For at denne skal få kontakt med Argo CD API'et må vi gi den tilgang. Først må vi redigere konfigurasjonen til Argo CD. Denne finner vi inne i Kubernetes så da må vi bruke `kubectl`:

```shell
kubectl edit configmap argocd-cm -n argocd
```
Kjør koden ovenfor og legg til følgende:

```yaml
data:
  accounts.image-updater: apiKey
```

Nå må vi generere et token som vi kan benytte senere. Vi tar vare på verdien i `$ARGOCD_TOKEN`.

```shell
ARGOCD_TOKEN=$(argocd account generate-token --account image-updater --id image-updater)
```

Nå har vi laget en brukerkonto og hentet ut et token som kan brukes til autentisering, nå må vi gi denne brukerkontoen tilgang (*rbac* er en mye brukt forkortelse for *Role Based Access Control*):

```shell
kubectl edit configmap argocd-rbac-cm -n argocd
```

Kjør kommandoen ovenfor og legg til følgende:

```yaml
data:
  policy.csv: |
    p, role:image-updater, applications, get, */*, allow
    p, role:image-updater, applications, update, */*, allow
    g, image-updater, role:image-updater

```
Neste jobb er å lage en "secret" ut av det tokenet vi hentet ut tidligere slik at *Argo CD Image Updater* kan benytte seg av dette.

```shell
kubectl create secret generic argocd-image-updater-secret \
  --from-literal argocd.token=$ARGOCD_TOKEN --dry-run=client -o yaml |
  kubectl -n argocd apply -f -
kubectl -n argocd rollout restart deployment argocd-image-updater
```




```
argocd app create isig-cd --repo https://github.com/itema-as/gitops-in-practice \
  --path isig-cd --dest-server https://kubernetes.default.svc \
  --dest-namespace default --upsert
argocd app sync isig-cd
```


