# Installasjon av ArgoCD

Vi må nå installere Argo CD. Siden vi ønsker å kjøre denne i et eget navnerom lager vi dette først. Deretter er det bare å angi hvor vi finner applikasjonsdeklarasjonen.

```
kubectl create namespace argocd
kubectl apply -n argocd -f \
  https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Har du litt tid kan du nå lese [veiledningen](https://argo-cd.readthedocs.io/en/stable/getting_started/) om om hvordan man kommer i gang med Argo CD. Du kan også [installere kommandolinjeverktøyet til Argo CD](https://argo-cd.readthedocs.io/en/stable/cli_installation/).

For å kunne nå Argo CD fra utsiden av Kubernetes-klyngen må vi dirigere noe trafikk. Argo CD kjører med SSL på innsiden over port 443. Denne ønsker vi å ha på port 8080 på utsiden.

```
kubectl port-forward svc/argocd-server -n argocd 8080:443 2>&1 >/dev/null &
```

Nå må vi få tak i passordet som trengs for å kunne logge seg inn på Argo CD som administrator. Dette ligger lagret som en "hemmelighet" i Kubernetes. Med andre ord ren tekst kodet i [Base64](https://en.wikipedia.org/wiki/Base64). Grunnen til at det ligger som base 64 er rett og slett at man skal også kunne legge inn binærfiler. Selve sikkerheten må komme fra et annet sted. For eksempel ved å begrense administratortilgang til klyngen eller ved verktøy som [Hashicorp Vault](https://www.vaultproject.io).

```
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d

```
Resultatet er en streng avsluttet med et prosenttegn. Tegnet er der for å indikere at strengen ikke er avsluttet med linjeskift, og er ikke en del av passordet.

Har du installert kommandolinjeverktøyet kan du teste innloggingen slik:

```
argocd login localhost:8080 --insecure --username admin
```

Eller du kan gå til https://localhost:8080/ og logge deg inn derfra.
