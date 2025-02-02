# Horizontal Pod Autoscaling

In Kubernetes, a Horizontal Pod Autoscaler (HPA) automatically updates a workload resource (such as a Deployment or a StatefulSet), with the aim of automatically scaling the workload to match demand.

Horizontal scaling means that the response to increased load is to deploy more pods. This is different from vertical scaling, which for Kubernetes would mean assigning more resources (for example: memory or CPU) to the pods that are already running in the workload.

Ocean CD has support for HPA resources that target our SpotDeployments. Use one of the templates below to set HPA when using Ocean CD.

## SpotDeployment YAML  

```
apiVersion: spot.io/v1beta1

kind: SpotDeployment

metadata:

  labels:

    app: nginx

  name: nginx-deployment

spec:

  replicas: 2

  selector:

    matchLabels:

      app: nginx

  template:

    metadata:

      labels:

        app: nginx

    spec:

      containers:

      - name: nginx

        image: nginx:1.21.0

        ports:

        - name: http

          containerPort: 8080

          protocol: TCP

        resources:

          requests:

            memory: 32Mi

            cpu: 5m
```

## HPA YAML

To set HPA using Kubernetes metric server, use the following template:

```
apiVersion: autoscaling/v1

kind: HorizontalPodAutoscaler

metadata:

 name: nginx-hpa

spec:

 maxReplicas: 6

 minReplicas: 2

 scaleTargetRef:

   apiVersion: spot.io/v1beta1

   kind: SpotDeployment

   name: nginx

 targetCPUUtilizationPercentage: 20
 ```

## Scaled Object

To set HPA using Prometheus query, use the following template:

```
apiVersion: keda.sh/v1alpha1

kind: ScaledObject

metadata:

name: nginx-hpa-prometheus-scale

spec:

scaleTargetRef:

  apiVersion: spot.io/v1beta1

  kind: SpotDeployment

  name: nginx

minReplicaCount: 2

maxReplicaCount: 6

cooldownPeriod: 30

pollingInterval: 1

triggers:

- type: prometheus

  metadata:

    serverAddress: http://prometheus-kube-prometheus-prometheus.prometheus.svc.cluster.local:9090

    metricName: pod_cpu_with_keda

    query: |

     sum (rate (container_cpu_usage_seconds_total{pod=~"nginx.*"}[1m]))

    threshold: "0.1"  
```

## What’s Next

Learn how to overwrite a (strategy)[link]. 
