# simple-http

This is a Golang example for metrics development guidance

## Reference

- [ServiceMonitor](https://github.com/prometheus-operator/prometheus-operator/blob/main/Documentation/api.md#servicemonitor)
- [PrometheusRule](https://github.com/prometheus-operator/prometheus-operator/blob/main/Documentation/api.md#prometheusrule)
- promtool official unit test [doc](https://prometheus.io/docs/prometheus/latest/configuration/unit_testing_rules/)
- Robust Perception prometheus alert rules unit test [post](https://www.robustperception.io/unit-testing-alerts-with-prometheus)
- Complex [test.yaml](https://github.com/kubernetes-monitoring/kubernetes-mixin/blob/master/tests.yaml) by kubernetes-mixin

## Metrics

Metrics exposed by application are listed below:

- myapp_processed_ops_total

## ServiceMonitor

ServiceMonitor is defined to configure Prometheus to scrape the application metrics endpoints. Its selector will select a Kubernetes Service and Prometheus will get the pod information from there and scrape those pod's metrics endpoint. You may refer to this [servicemonitor.yaml](./helm-chart/templates/servicemonitor.yaml)

## PrometheusRule

PrometheusRule is defined to configure Prometheus for the alert rules. You may refer to this [prometheusrule.yaml](./helm-chart/templates/prometheusrule.yaml)

## How to test alert rules

### promtool binary

promtool binary is required to test the alert rule. It is available in every prometheus [release](https://github.com/prometheus/prometheus/releases)

### Step to test

1. In this example, the alert rules is written in [alertrules.yaml](./helm-chart/files/alertrules.yaml) so that this file can be used for both prometheusrules.yaml and alertrules-test.yaml
2. Write an [alertrules-test.yaml](./helm-chart/files/alertrules-test.yaml). You may refer to the simple post by Robust Perception or a much more complex example by kubernetes-mixin when writing this
3. Run command `promtool test rules ./helm-chart/files/alertrules-test.yaml`

## How to develop Grafana dashboard

### Prerequisites

- Helm 3+
- kubectl

### Step to develop

1. Create a kubernetes cluster with kube-prometheus-stack based on the [simple cluster setup](https://github.com/automata-network/devops/tree/feature/grafana-iac/kube-prometheus-stack-v30#simple-cluster-setup)
2. Docker build your image by running command `docker build -t simple-http:$(git rev-parse --short HEAD) .`
3. Load image into kubernetes cluster by running command `kind load docker-image simple-http:$(git rev-parse --short HEAD)`
4. Helm install by running command `helm install app --set image.tag=$(git rev-parse --short HEAD) ./helm-chart`
5. Port forward to grafana service by running command `kubectl port-forward svc/prom-grafana -n monitoring 3000:80 &`
6. Connect to grafana with `http://localhost:3000`, use `admin` as username, `prom-operator` as password to login 
7. Develop the dashboard based on your application metrics
8. When it is done, save the dashboard json
9. Automata developed dashboard is save in this [folder](https://github.com/automata-network/devops/tree/feature/kube-prometheus-stack-v30.1.0/kube-prometheus-stack-v30/templates/automata/dashboards), open a PR merge your dashboard