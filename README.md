# amq-streams-kafka-metrics
Putting together the components and connecting the dots using Grafana Dashboards, Prometheus, Strimizi Podmonitors, AMQ Streams

![image](component.png)

## Prerequisites:

Install the following operators
* AMQ Streams
* Prometheus
* Grafana

To use the yaml examples in this lab:
* Replace <your-project> with your own project namespace
* Make sure kafka resources and cluster names match your project

External image registry and source repository access
* Make sure your environment allows your OpenShift cluster to reach out to external image registries and Maven repositories


## Apply the CRD resources in this order
```
Kafka/KafkaConnect:
oc -n <your-namespace> apply -f yaml/kafka/kafka-metrics-configmap.yaml
oc -n <your-namespace> apply -f yaml/kafka/kafka-cluster.yaml
oc -n <your-namespace> apply -f yaml/kafka/kafka-topics.yaml
oc -n <your-namespace> apply -f yaml/kafka-connect/kafka-connect-metrics-example.yaml
oc -n <your-namespace> apply -f yaml/kafka-connect/kafka-connect.yaml
oc -n <your-namespace> apply -f yaml/kafka-connect/kafka-connector-mongodb.yaml

helm repo add bitnami https://charts.bitnami.com/bitnami

helm install mongodb bitnami/mongodb \
--set podSecurityContext.fsGroup="",containerSecurityContext.enabled=false,podSecurityContext.enabled=false,auth.enabled=false --version 13.6.0 \
-n <your-namespace>

Prometheus:
oc -n <your-namespace> apply -f yaml/prometheus/prometheus-additional-scrape-configmap.yaml
oc -n <your-namespace> apply -f yaml/prometheus/prometheus.yaml
oc -n <your-namespace> apply -f yaml/prometheus/prometheus-strimzi-pod-monitor.yaml

Grafana:
oc -n <your-namespace> apply -f yaml/grafana/grafana.yaml
oc -n <your-namespace> expose service grafana-service
oc -n <your-namespace> apply -f yaml/grafana/grafana-datasource.yaml
oc -n <your-namespace> apply -f yaml/grafana/grafana-*-dashboard.yaml
Or
find . -type f -name "grafana*dashboard*.yaml" -exec sh -c 'oc apply -f "$0"' {} \;
```

## Post Installation Steps
* Use the grafana route to launch the Grafana console
* Test the Prometheus Data Source connection in Grafana
* If everything is setup and configured correctly following all the steps, you should see dashboards such as these:

![image](dash1.png)
