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

Validate the URL with are running
```console
curl -I http://localhost:8080
```
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
   or Show all running Pods with additional details
   ```console
   kubectl get pods --all-namespaces -o wide
   ```
   The output look like:
   ```console
   thanhdv@vm-2412272632:~$ kubectl get pods --all-namespaces -o wide
   NAMESPACE     NAME                                      READY   STATUS      RESTARTS       AGE     IP           NODE                    NOMINATED NODE   READINESS GATES
   kube-system   coredns-ccb96694c-zlxxc                   1/1     Running     1 (5d2h ago)   5d19h   10.42.0.71   k3d-poc-otel-server-0   <none>           <none>
   kube-system   helm-install-traefik-crd-2f6vd            0/1     Completed   0              5d19h   <none>       k3d-poc-otel-server-0   <none>           <none>
   kube-system   helm-install-traefik-zk8jm                0/1     Completed   1              5d19h   <none>       k3d-poc-otel-server-0   <none>           <none>
   kube-system   local-path-provisioner-5cf85fd84d-stndc   1/1     Running     1 (5d2h ago)   5d19h   10.42.0.72   k3d-poc-otel-server-0   <none>           <none>
   kube-system   metrics-server-5985cbc9d7-44qrc           1/1     Running     1 (5d2h ago)   5d19h   10.42.0.69   k3d-poc-otel-server-0   <none>           <none>
   kube-system   svclb-traefik-9c5eadef-spfvz              2/2     Running     2 (5d2h ago)   5d19h   10.42.0.68   k3d-poc-otel-server-0   <none>           <none>
   kube-system   traefik-5d45fc8cc9-mrq2k                  1/1     Running     1 (5d2h ago)   5d19h   10.42.0.70   k3d-poc-otel-server-0   <none>           <none>
   ```
   
   List all services with their ports
   ```console
   kubectl get svc --all-namespaces
   ```
   The output look like:
   ```console
      thanhdv@vm-2412272632:~$ kubectl get svc --all-namespaces
   NAMESPACE     NAME                            TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                   AGE
   default       kubernetes                      ClusterIP      10.43.0.1       <none>        443/TCP                                                   5d19h
   default       mdm-poc-adservice               ClusterIP      10.43.129.78    <none>        8080/TCP                                                  5d18h
   default       mdm-poc-cartservice             ClusterIP      10.43.167.197   <none>        8080/TCP                                                  5d18h
   default       mdm-poc-checkoutservice         ClusterIP      10.43.94.111    <none>        8080/TCP                                                  5d18h
   default       mdm-poc-currencyservice         ClusterIP      10.43.8.0       <none>        8080/TCP                                                  5d18h
   default       mdm-poc-emailservice            ClusterIP      10.43.51.20     <none>        8080/TCP                                                  5d18h
   default       mdm-poc-featureflagservice      ClusterIP      10.43.184.219   <none>        50053/TCP,8081/TCP                                        5d18h
   default       mdm-poc-ffspostgres             ClusterIP      10.43.248.241   <none>        5432/TCP                                                  5d18h
   default       mdm-poc-frontend                ClusterIP      10.43.192.203   <none>        8080/TCP                                                  5d18h
   default       mdm-poc-frontendproxy           ClusterIP      10.43.42.66     <none>        8080/TCP                                                  5d18h
   default       mdm-poc-kafka                   ClusterIP      10.43.63.73     <none>        9092/TCP,9093/TCP                                         5d18h
   default       mdm-poc-loadgenerator           ClusterIP      10.43.190.188   <none>        8089/TCP                                                  5d18h
   default       mdm-poc-paymentservice          ClusterIP      10.43.12.11     <none>        8080/TCP                                                  5d18h
   default       mdm-poc-productcatalogservice   ClusterIP      10.43.255.186   <none>        8080/TCP                                                  5d18h
   default       mdm-poc-quoteservice            ClusterIP      10.43.87.128    <none>        8080/TCP                                                  5d18h
   default       mdm-poc-recommendationservice   ClusterIP      10.43.187.227   <none>        8080/TCP                                                  5d18h
   default       mdm-poc-redis                   ClusterIP      10.43.111.112   <none>        6379/TCP                                                  5d18h
   default       mdm-poc-shippingservice         ClusterIP      10.43.191.11    <none>        8080/TCP                                                  5d18h
   default       owg-opentelemetry-collector     ClusterIP      10.43.129.104   <none>        6831/UDP,14250/TCP,14268/TCP,4317/TCP,4318/TCP,9411/TCP   5d19h
   kube-system   kube-dns                        ClusterIP      10.43.0.10      <none>        53/UDP,53/TCP,9153/TCP                                    5d19h
   kube-system   metrics-server                  ClusterIP      10.43.207.148   <none>        443/TCP                                                   5d19h
   kube-system   traefik                         LoadBalancer   10.43.212.113   172.20.0.2    80:32434/TCP,443:30380/TCP                                5d19h
   ```

5. Stop Pods in a Specific Namespace
   ```console
   kubectl delete pods --all -n <namespace>
   ```
   for example, to stop Pods in a Specific Namespace "default":
   ```console
   kubectl delete pods --all -n default
   ```

6. Scale Down Deployments/StatefulSets to Stop Pods
   ```console
   kubectl scale deployment --all --replicas=0 -n <namespace>
   ```
   for example: "kubectl scale deployment --all --replicas=0 -n default" stop by scaling down Pods to zero 

7. Scale Up Deployments/StatefulSets to Start Pods
   ```console
   kubectl scale deployment --all --replicas=1 --all-namespaces
   ```
   🔹 Explanation:
   --all            → Apply to all deployments
   --replicas=1     → Set 1 pod per deployment
   --all-namespaces → Apply across all namespaces

8. To kill a process that combined with a sepcific port
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

8.  Show Running Pods with Their Ports on Ubuntu 22.04 LTS
 - Run the following command:
   ```console
   kubectl get pods -o=custom-columns="POD:metadata.name,NAMESPACE:metadata.namespace,PORTS:spec.containers[*].ports[*].containerPort" --all-namespaces
   ```
   Explanation::
     POD:metadata.name → Displays the Pod name.
     NAMESPACE:metadata.namespace → Shows the namespace of each Pod.
     PORTS:spec.containers[*].ports[*].containerPort → Extracts exposed container ports.
 The output look like:
    ```console
     POD                                       NAMESPACE     PORTS
    coredns-ccb96694c-zlxxc                   kube-system   53,53,9153
    helm-install-traefik-crd-2f6vd            kube-system   <none>
    helm-install-traefik-zk8jm                kube-system   <none>
    local-path-provisioner-5cf85fd84d-stndc   kube-system   <none>
    metrics-server-5985cbc9d7-44qrc           kube-system   10250
    svclb-traefik-9c5eadef-spfvz              kube-system   80,443
    traefik-5d45fc8cc9-mrq2k                  kube-system   9100,9000,8000,8443
    ```

9.  Show Exposed Ports with Their Services on Ubuntu 22.04 LTS
 - Run the following command:
   ```console
   kubectl get svc -o=custom-columns="SERVICE:metadata.name,NAMESPACE:metadata.namespace,PORTS:spec.ports[*].port,TARGETPORT:spec.ports[*].targetPort" --all-namespaces
   ```
   Explanation::
     SERVICE:metadata.name → Shows the service name.
     NAMESPACE:metadata.namespace → Displays the namespace.
     PORTS:spec.ports[*].port → Lists exposed service ports.
     TARGETPORT:spec.ports[*].targetPort → Shows the actual port inside the container.
 The output look like:
    ```console
    SERVICE                         NAMESPACE     PORTS                             TARGETPORT
    kubernetes                      default       443                               6443
    mdm-poc-adservice               default       8080                              8080
    mdm-poc-cartservice             default       8080                              8080
    mdm-poc-checkoutservice         default       8080                              8080
    mdm-poc-currencyservice         default       8080                              8080
    mdm-poc-emailservice            default       8080                              8080
    mdm-poc-featureflagservice      default       50053,8081                        50053,8081
    mdm-poc-ffspostgres             default       5432                              5432
    mdm-poc-frontend                default       8080                              8080
    mdm-poc-frontendproxy           default       8080                              8080
    mdm-poc-kafka                   default       9092,9093                         9092,9093
    mdm-poc-loadgenerator           default       8089                              8089
    mdm-poc-paymentservice          default       8080                              8080
    mdm-poc-productcatalogservice   default       8080                              8080
    mdm-poc-quoteservice            default       8080                              8080
    mdm-poc-recommendationservice   default       8080                              8080
    mdm-poc-redis                   default       6379                              6379
    mdm-poc-shippingservice         default       8080                              8080
    owg-opentelemetry-collector     default       6831,14250,14268,4317,4318,9411   6831,14250,14268,4317,4318,9411
    kube-dns                        kube-system   53,53,9153                        53,53,9153
    metrics-server                  kube-system   443                               https
    traefik                         kube-system   80,443                            web,websecure
    ```
    
