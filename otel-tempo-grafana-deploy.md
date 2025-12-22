# OpenTelemetry Collector + Grafana Tempo + Grafana 배포 YAML

구성: Dapr / App → OpenTelemetry Collector → Grafana Tempo → Grafana UI\
Namespace: monitoring\
Tempo: 2.5.x\
Collector: otel-collector-contrib:0.142.x

------------------------------------------------------------------------

## 0. Namespace

``` yaml
apiVersion: v1
kind: Namespace
metadata:
  name: monitoring
```

------------------------------------------------------------------------

## 1. OpenTelemetry Collector

### ConfigMap

``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: otel-collector-config
  namespace: monitoring
data:
  otel-collector.yaml: |
    receivers:
      otlp:
        protocols:
          grpc:
          http:

    processors:
      batch:

    exporters:
      otlp/tempo:
        endpoint: tempo.monitoring.svc:4317
        tls:
          insecure: true
      logging:
        loglevel: info

    service:
      pipelines:
        traces:
          receivers: [otlp]
          processors: [batch]
          exporters: [otlp/tempo, logging]
```

### Deployment & Service

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: otel-collector
  namespace: monitoring
spec:
  replicas: 2
  selector:
    matchLabels:
      app: otel-collector
  template:
    metadata:
      labels:
        app: otel-collector
    spec:
      containers:
        - name: otel-collector
          image: otel/opentelemetry-collector-contrib:0.142.0
          args: ["--config=/conf/otel-collector.yaml"]
          ports:
            - containerPort: 4317
            - containerPort: 4318
          volumeMounts:
            - name: otel-config
              mountPath: /conf
      volumes:
        - name: otel-config
          configMap:
            name: otel-collector-config
---
apiVersion: v1
kind: Service
metadata:
  name: otel-collector
  namespace: monitoring
spec:
  selector:
    app: otel-collector
  ports:
    - name: otlp-grpc
      port: 4317
      targetPort: 4317
    - name: otlp-http
      port: 4318
      targetPort: 4318
```

------------------------------------------------------------------------

## 2. Grafana Tempo (2.5.x)

### ConfigMap

``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: tempo-config
  namespace: monitoring
data:
  tempo.yaml: |
    server:
      http_listen_port: 3100

    distributor:
      receivers:
        otlp:
          protocols:
            grpc:
            http:

    storage:
      trace:
        backend: local
        local:
          path: /tmp/tempo
        wal:
          path: /tmp/tempo/wal

    compactor:
      compaction:
        block_retention: 24h
```

### Deployment & Service

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tempo
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tempo
  template:
    metadata:
      labels:
        app: tempo
    spec:
      containers:
        - name: tempo
          image: grafana/tempo:2.5.4
          args: ["-config.file=/etc/tempo/tempo.yaml"]
          ports:
            - containerPort: 3100
            - containerPort: 4317
            - containerPort: 4318
          volumeMounts:
            - name: tempo-config
              mountPath: /etc/tempo
      volumes:
        - name: tempo-config
          configMap:
            name: tempo-config
---
apiVersion: v1
kind: Service
metadata:
  name: tempo
  namespace: monitoring
spec:
  selector:
    app: tempo
  ports:
    - name: http
      port: 3100
      targetPort: 3100
    - name: otlp-grpc
      port: 4317
      targetPort: 4317
    - name: otlp-http
      port: 4318
      targetPort: 4318
```

------------------------------------------------------------------------

## 3. Grafana

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
        - name: grafana
          image: grafana/grafana:10.4.0
          ports:
            - containerPort: 3000
          env:
            - name: GF_SECURITY_ADMIN_PASSWORD
              value: admin
---
apiVersion: v1
kind: Service
metadata:
  name: grafana
  namespace: monitoring
spec:
  selector:
    app: grafana
  ports:
    - port: 3000
      targetPort: 3000
```

------------------------------------------------------------------------

## 4. 적용

``` bash
kubectl apply -f otel-tempo-grafana.yaml
```

## 5. Dapr 설정 예

``` yaml
tracing:
  samplingRate: "1"
  otel:
    endpointAddress: "otel-collector.monitoring.svc:4317"
```

------------------------------------------------------------------------

Grafana Tempo DataSource: - URL: http://tempo.monitoring.svc:3100 -
Type: Tempo
