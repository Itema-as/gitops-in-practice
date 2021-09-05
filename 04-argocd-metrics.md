## Overvåkning med Prometheus

Denne gangen skal vi bruke et såkalt *Helm Chart* til å installere en relativt kompleks applikasjon, [Prometheus](https://prometheus.io).


Først må vi legge til repoet hvor vi finner "kartet". Dernest installerer vi applikasjonen og eksponerer porten den kjører på.

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/prometheus
kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-np
```

Prometheus klarer av en eller annen grunn ikke løse DNS innenfor klyngen, så når vi nå skal legge til metrikkene til Argo CD må vi gjøre dette på den vanskelige måten. Først må vi finne IP-adressen til Argo CD Metrics tjenesten:

```
get service argocd-metrics -n argocd
```
Du skal nå få noe som dette. `CLUSTER-IP` er den adressen vi er ute etter.

```
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
argocd-metrics   ClusterIP   10.107.133.238   <none>        8082/TCP   14m
```

Nå må vi legge til denne informasjonen i Prometheus-konfigurasjonen.

```
kubectl edit cm prometheus-server -o yaml
```
Finn `data.scrape_configs` og legg denne snutten rett under. Bytt ut IP-adressen med den du faktisk fikk. Du kan like gjerne endre `scrape_interval` med det samme, siden vi ønsker rask tilbakemelding.

```
    - job_name: argocd-metrics
      static_configs:
      - targets:
        - 10.107.133.238:8082
```

Kjør følgende kode for å tilgjengeliggjøre tjenesten.

```
export POD_NAME=$(kubectl get pods --namespace default -l "app=prometheus,component=server" -o jsonpath="{.items[0].metadata.name}")
kubectl --namespace default port-forward $POD_NAME 9090 2>&1 >/dev/null &
```

Nan vi åpne [Prometheus > Targets](http://localhost:9090/targets) og kontrollere at vi har tilgang til disse metrikkene. 

![](./prometheus-target.png)

Nå som vi har fått litt data kan vi bruke Prometheus sin spørrefunksjon til å tegne et fint diagram over hvor ofte hver enkelt applikasjon vi har installert har blitt synkronisert.

![](./prometheus-graph.png)

