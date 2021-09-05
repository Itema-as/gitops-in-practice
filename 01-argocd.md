# Installasjon av ArgoCD

Argo CD er den komponenten i systemet vårt som leser applikasjonsdeklarasjoner fra vårt Git-repo og instansierer applikasjonene i klyngen. Siden vi ønsker å kjøre den i et eget navnerom lager vi dette først. Deretter kan vi installere Argo CD fra en beskrivelse også fra Git.

```Shell
kubectl create namespace argocd
kubectl apply -n argocd -f \
  https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Du må også [installere kommandolinjeverktøyet til Argo CD](https://argo-cd.readthedocs.io/en/stable/cli_installation/). Her er tilnærmingen litt forskjellig alt etter hvilket operativsystem man kjører. Dette kan du gjøre mens Argo CD starter opp på klyngen, noe som kan ta litt tid. For å sjekke statusen på poddene kan du utføre `kubectl get deploy -w -n argocd`:

```
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
argocd-redis         1/1     1            1           51s
argocd-repo-server   1/1     1            1           61s
argocd-server        1/1     1            1           64s
argocd-dex-server    1/1     1            1           82s
```

Her må vi vente til alle er i tilstanden `Ready` og `Available`.

For å kunne nå Argo CD fra utsiden av Kubernetes-klyngen må vi dirigere noe trafikk. Argo CD kjører med SSL på innsiden over port 443. Denne ønsker vi å ha på port 8080 på utsiden.

```Shell
kubectl port-forward svc/argocd-server -n argocd 8080:443 2>&1 >/dev/null &

```

Nå må vi få tak i passordet som trengs for å kunne logge seg inn på Argo CD som administrator. Dette ligger lagret som en *secret* i Kubernetes. Med andre ord ren tekst kodet i [Base64](https://en.wikipedia.org/wiki/Base64). Grunnen til at det ligger som base 64 er rett og slett at man skal også kunne legge inn binærfiler. Selve sikkerheten må komme fra et annet sted. For eksempel ved å begrense tilgang til klyngen eller med verktøy som [Hashicorp Vault](https://www.vaultproject.io).

```Shell
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d ; echo
```

Vi skal senere i øvelsen benytte oss av kommandolinjeverktøyet til Argo CD for å legge til applikasjoner, så for at dette skal virke må vi nå logge oss inn. Kjør følgende og legg inn passordet hentet ut ovenfor når det blir spurt etter.

```Shell
argocd login localhost:8080 --insecure --username admin
```
## Konfigurere GitHub PAT

For at Argo CD skal kunne hente ut data fra private GitHub repo, må den kunne autentisere seg. Dette gjør man med et GitHub *Personal Access Token* som må lages via [**Settings > Developer settings > Personal access tokens**](https://github.com/settings/tokens). Opprett et nytt for din brukerkonto med egenskapene:

- `read:packages`
- `repo`

Nå kan vi logge inn med:

```shell
argocd repo add https://github.com/itema-as/gitops-in-practice \
  --username <github-login> \
  --password <github-pat>
```

Du finner igjen denne konfigurasjone om du går inn i Argo CD under **Settings > Repositories**.

---
Har du litt tid kan du nå lese [veiledningen](https://argo-cd.readthedocs.io/en/stable/getting_started/) om om hvordan man kommer i gang med Argo CD. Eller du kan logge deg inn på brukergrensesnittet og utforske dette på https://localhost:8080.

👉 I [neste øvelse](./02-isig-kustomize.md) installerer vi en applikasjon ved hjelp av Argo CD. 

