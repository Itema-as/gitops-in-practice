# Installasjon av ArgoCD

Vi må nå installere Argo CD. Siden vi ønsker å kjøre denne i et eget navnerom lager vi dette først. Deretter er det bare å angi hvor vi finner applikasjonsdeklarasjonen.

```Shell
kubectl create namespace argocd
kubectl apply -n argocd -f \
  https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Du må også [installere kommandolinjeverktøyet til Argo CD](https://argo-cd.readthedocs.io/en/stable/cli_installation/). Her er tilnærmingen litt forskjellig alt etter hvilket operativsystem man kjører. Dette kan du gjøre mens Argo CD starter opp på klyngen, noe som kan ta litt tid. For å sjekke statusen på poddene kan du utføre `kubectl get pods -n argocd`:

```
NAME                                 READY   STATUS              RESTARTS   AGE
argocd-application-controller-0      0/1     ContainerCreating   0          59s
argocd-dex-server-69599588c6-ktjbt   0/1     PodInitializing     0          60s
argocd-redis-5b6967fdfc-8zdwk        1/1     Running             0          60s
argocd-repo-server-779d955bc-84fgn   0/1     ContainerCreating   0          60s
argocd-server-c45f45758-qgfr7        0/1     Running             0          59s
```

Her må vi vente til alle er i tilstanden `Running` og `Ready`.

For å kunne nå Argo CD fra utsiden av Kubernetes-klyngen må vi dirigere noe trafikk. Argo CD kjører med SSL på innsiden over port 443. Denne ønsker vi å ha på port 8080 på utsiden.

```Shell
kubectl port-forward svc/argocd-server -n argocd 8080:443 2>&1 >/dev/null &
```

Nå må vi få tak i passordet som trengs for å kunne logge seg inn på Argo CD som administrator. Dette ligger lagret som en "hemmelighet" i Kubernetes. Med andre ord ren tekst kodet i [Base64](https://en.wikipedia.org/wiki/Base64). Grunnen til at det ligger som base 64 er rett og slett at man skal også kunne legge inn binærfiler. Selve sikkerheten må komme fra et annet sted. For eksempel ved å begrense administratortilgang til klyngen eller ved verktøy som [Hashicorp Vault](https://www.vaultproject.io).

```Shell
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d
```
Resultatet er en streng avsluttet med et prosenttegn. Tegnet er der for å indikere at strengen ikke er avsluttet med linjeskift, og er ikke en del av passordet.

Vi skal senere i øvelsen benytte oss av kommandolinjeverktøyet til Argo CD, så for at dette skal virke må vi nå logge oss inn. Kjør følgende og legg inn passordet hentet ut ovenfor når det blir spurt etter.

```Shell
argocd login localhost:8080 --insecure --username admin
```

Har du litt tid kan du nå lese [veiledningen](https://argo-cd.readthedocs.io/en/stable/getting_started/) om om hvordan man kommer i gang med Argo CD. 

