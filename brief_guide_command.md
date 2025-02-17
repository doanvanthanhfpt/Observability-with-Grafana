## Setup Instructions

#### Install OpenTelemetry Demo application

1. Install the OpenTelemetry Demo Application
```console
helm install --version '0.26.0' --values OTEL-Demo.yaml owg-demo open-telemetry/opentelemetry-demo
```

2. Validate the applications are running
```console
kubectl get pods --selector=app.kubernetes.io/instance=owg-demo
```
The output of installed applications look like:
``
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
``

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
