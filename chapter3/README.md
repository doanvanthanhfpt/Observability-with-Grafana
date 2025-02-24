## Setup Instructions

### Chapter 3

#### OTEL-Creds.yaml

in sections `config.extensions` and `config.exporters`

1. Replace <TRACE_USERNAME>, <TRACE_PASSWORD> and <TRACE_ENDPOINT> with details from Tempo
2. Replace <METRIC_USERNAME>, <METRIC_PASSWORD> and <METRIC_ENDPOINT> with details from Prometheus
3. Replace <LOG_USERNAME>, <LOG_PASSWORD> and <LOG_ENDPOINT> with details from Loki

Username should be a number
Password is a long token that is set up via the `Access Policies` screen in the Grafana Cloud Portal
Endpoint will look similar to these

* `tempo-us-central1.grafana.net:443`
  * This is expected to look different to the other endpoints
* `https://prometheus-prod-10-prod-us-central-0.grafana.net/api/prom/push`
* `https://logs-prod3.grafana.net/loki/api/v1/push`

#### Install OpenTelemetry collector

1. Add the OpenTelemetry Helm repository:
```console
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
```

2. Install the OpenTelemetry Collector
```console
helm install --version '0.73.1' --values chapter3/OTEL-Collector.yaml --values OTEL-Creds.yaml owg open-telemetry/opentelemetry-collector
```

3. Validate the installation is running
```console
kubectl get pods
```
The output look like:
```console
NAME                                           READY   STATUS    RESTARTS   AGE
owg-opentelemetry-collector-6b8fdddc9d-4tsj5   1/1     Running   0          2s
```

#### Install OpenTelemetry Demo application

1. Install the OpenTelemetry Demo Application
```console
helm install --version '0.26.0' --values OTEL-Demo.yaml owg-demo open-telemetry/opentelemetry-demo
```
The output look like below
```console
NAME: owg-demo
LAST DEPLOYED: Sat Jan 18 03:04:34 2025
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
=======================================================================================


 ██████╗ ████████╗███████╗██╗         ██████╗ ███████╗███╗   ███╗ ██████╗
██╔═══██╗╚══██╔══╝██╔════╝██║         ██╔══██╗██╔════╝████╗ ████║██╔═══██╗
██║   ██║   ██║   █████╗  ██║         ██║  ██║█████╗  ██╔████╔██║██║   ██║
██║   ██║   ██║   ██╔══╝  ██║         ██║  ██║██╔══╝  ██║╚██╔╝██║██║   ██║
╚██████╔╝   ██║   ███████╗███████╗    ██████╔╝███████╗██║ ╚═╝ ██║╚██████╔╝
 ╚═════╝    ╚═╝   ╚══════╝╚══════╝    ╚═════╝ ╚══════╝╚═╝     ╚═╝ ╚═════╝


- All services are available via the Frontend proxy: http://localhost:8080
  by running these commands:
     kubectl --namespace default port-forward svc/owg-demo-frontendproxy 8080:8080

  The following services are available at these paths once the proxy is exposed:
  Webstore             http://localhost:8080/
  Grafana              http://localhost:8080/grafana/
  Feature Flags UI     http://localhost:8080/feature/
  Load Generator UI    http://localhost:8080/loadgen/
  Jaeger UI            http://localhost:8080/jaeger/ui/
```

2. Validate the applications are running
```console
kubectl get pods --selector=app.kubernetes.io/instance=owg-demo
```
The output look like below
```console
NAME                                              READY   STATUS    RESTARTS   AGE
owg-demo-emailservice-68d874d8dc-xf2lv            1/1     Running   0          5m
owg-demo-frontendproxy-7f647bccf9-fdf8t           1/1     Running   0          5m
owg-demo-quoteservice-6cd6cb9476-jvlff            1/1     Running   0          5m
owg-demo-productcatalogservice-dd9d84d67-5sbx8    1/1     Running   0          5m
owg-demo-paymentservice-79fcbf6888-fs7fd          1/1     Running   0          5m
owg-demo-redis-578cdb6447-84x7d                   1/1     Running   0          5m
owg-demo-loadgenerator-b9ffd5d4f-m7g69            1/1     Running   0          5m
owg-demo-ffspostgres-6695d47498-kr6m5             1/1     Running   0          5m
owg-demo-kafka-58cdf5dbbc-fkh2h                   1/1     Running   0          5m
owg-demo-shippingservice-66dd4f4b8c-2lndp         1/1     Running   0          5m
owg-demo-recommendationservice-c57c5dfbd-bpx8c    1/1     Running   0          5m
owg-demo-frontend-7465547d4d-kmv7d                1/1     Running   0          5m
owg-demo-currencyservice-58b65ccd89-rrxkv         1/1     Running   0          5m
owg-demo-adservice-5678b98bcd-jz82b               1/1     Running   0          5m
owg-demo-cartservice-98468fff7-h9dz8              1/1     Running   0          5m
owg-demo-featureflagservice-7847d8f7f7-7hnrq      1/1     Running   0          5m
owg-demo-accountingservice-8596699678-8t88f       1/1     Running   0          5m
owg-demo-checkoutservice-7585d7d45f-zgrpp         1/1     Running   0          5m
owg-demo-frauddetectionservice-555bcff644-b6ktm   1/1     Running   0          5m
```
Validate the URL with are running
```console
curl -I http://localhost:8080
```

3. Open port for frontendproxy access
```console
kubectl port-forward svc/owg-demo-frontendproxy 8080:8080 &
```

4. Open port for browser spans
```console
kubectl port-forward svc/owg-opentelemetry-collector 4318:4318 &
```

5. Check access to the OpenTelemetry Demo application
In your browser head to (http://localhost:8080)


#### Some notes for troubleshooting

1. Find the Namespace with helm
   ```console
   helm list --all --all-namespaces
   ```

2. Delete the Release by namespace with helm
   ```console
   helm delete owg-demo -n <namespace>
   ```
   for example, delete release "owg-demo" in a specific namespace "default":
   ```console
   helm delete owg-demo -n default
   ```

3. Verify running Pods from installation
   ```console
   kubectl get pods --all-namespaces
   ```
   or without --all-namespaces parameter to exclude namespace "kube-system"
   ```console
   kubectl get pods
   ```

4. Stop Pods in a Specific Namespace
   ```console
   kubectl delete pods --all -n <namespace>
   ```
   for example, to stop Pods in a Specific Namespace "default":
   ```console
   kubectl delete pods --all -n default
   ```

5. Scale Down Deployments/StatefulSets to Stop Pods
   ```console
   kubectl scale deployment --all --replicas=0 -n <namespace>
   ```
   for example: "kubectl scale deployment --all --replicas=0 -n default" stop by scaling down Pods to zero 

6. Scale Up Deployments/StatefulSets to Start Pods
   ```console
   kubectl scale deployment --all --replicas=1 --all-namespaces
   ```
   🔹 Explanation:
   --all            → Apply to all deployments
   --replicas=1     → Set 1 pod per deployment
   --all-namespaces → Apply across all namespaces

7. To kill a process that combined with a sepcific port
 - Specify port combined PID to kill
   ```console
   sudo lsof -i :<port_want_to_release>
   ```
   for example: "sudo lsof -i :8080" to point out port 8080 PID of process combined 
 - Kill process to release a specific combined port
   ```console
   sudo kill -9 <combined_PID>
   ```
   for example: "sudo kill -9 8507" to end specific PID of process
