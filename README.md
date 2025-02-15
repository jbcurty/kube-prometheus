# kube-prometheus

[![Build Status](https://github.com/prometheus-operator/kube-prometheus/workflows/ci/badge.svg)](https://github.com/prometheus-operator/kube-prometheus/actions)
[![Slack](https://img.shields.io/badge/join%20slack-%23prometheus--operator-brightgreen.svg)](http://slack.k8s.io/)
[![Gitpod ready-to-code](https://img.shields.io/badge/Gitpod-ready--to--code-blue?logo=gitpod)](https://gitpod.io/#https://github.com/prometheus-operator/kube-prometheus)

> Note that everything is experimental and may change significantly at any time.

This repository collects Kubernetes manifests, [Grafana](http://grafana.com/) dashboards, and [Prometheus rules](https://prometheus.io/docs/prometheus/latest/configuration/recording_rules/) combined with documentation and scripts to provide easy to operate end-to-end Kubernetes cluster monitoring with [Prometheus](https://prometheus.io/) using the Prometheus Operator.

The content of this project is written in [jsonnet](http://jsonnet.org/). This project could both be described as a package as well as a library.

Components included in this package:

* The [Prometheus Operator](https://github.com/prometheus-operator/prometheus-operator)
* Highly available [Prometheus](https://prometheus.io/)
* Highly available [Alertmanager](https://github.com/prometheus/alertmanager)
* [Prometheus node-exporter](https://github.com/prometheus/node_exporter)
* [Prometheus Adapter for Kubernetes Metrics APIs](https://github.com/DirectXMan12/k8s-prometheus-adapter)
* [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics)
* [Grafana](https://grafana.com/)

This stack is meant for cluster monitoring, so it is pre-configured to collect metrics from all Kubernetes components. In addition to that it delivers a default set of dashboards and alerting rules. Many of the useful dashboards and alerts come from the [kubernetes-mixin project](https://github.com/kubernetes-monitoring/kubernetes-mixin), similar to this project it provides composable jsonnet as a library for users to customize to their needs.

## Prerequisites

You will need a Kubernetes cluster, that's it! By default it is assumed, that the kubelet uses token authentication and authorization, as otherwise Prometheus needs a client certificate, which gives it full access to the kubelet, rather than just the metrics. Token authentication and authorization allows more fine grained and easier access control.

This means the kubelet configuration must contain these flags:

* `--authentication-token-webhook=true` This flag enables, that a `ServiceAccount` token can be used to authenticate against the kubelet(s). This can also be enabled by setting the kubelet configuration value `authentication.webhook.enabled` to `true`.
* `--authorization-mode=Webhook` This flag enables, that the kubelet will perform an RBAC request with the API to determine, whether the requesting entity (Prometheus in this case) is allowed to access a resource, in specific for this project the `/metrics` endpoint. This can also be enabled by setting the kubelet configuration value `authorization.mode` to `Webhook`.

This stack provides [resource metrics](https://github.com/kubernetes/metrics#resource-metrics-api) by deploying the [Prometheus Adapter](https://github.com/DirectXMan12/k8s-prometheus-adapter/).
This adapter is an Extension API Server and Kubernetes needs to be have this feature enabled, otherwise the adapter has no effect, but is still deployed.

## Compatibility

The following Kubernetes versions are supported and work as we test against these versions in their respective branches. But note that other versions might work!

| kube-prometheus stack                                                                      | Kubernetes 1.19 | Kubernetes 1.20 | Kubernetes 1.21 | Kubernetes 1.22 | Kubernetes 1.23 |
|--------------------------------------------------------------------------------------------|-----------------|-----------------|-----------------|-----------------|-----------------|
| [`release-0.7`](https://github.com/prometheus-operator/kube-prometheus/tree/release-0.7)   | ✔               | ✔               | ✗               | ✗               | ✗               |
| [`release-0.8`](https://github.com/prometheus-operator/kube-prometheus/tree/release-0.8)   | ✗               | ✔               | ✔               | ✗               | ✗               |
| [`release-0.9`](https://github.com/prometheus-operator/kube-prometheus/tree/release-0.9)   | ✗               | ✗               | ✔               | ✔               | ✗               |
| [`release-0.10`](https://github.com/prometheus-operator/kube-prometheus/tree/release-0.10) | ✗               | ✗               | ✗               | ✔               | ✔               |
| [`main`](https://github.com/prometheus-operator/kube-prometheus/tree/main)                 | ✗               | ✗               | ✗               | ✗               | ✔               |

## Quickstart

> Note: For versions before Kubernetes v1.21.z refer to the [Kubernetes compatibility matrix](#compatibility) in order to choose a compatible branch.

This project is intended to be used as a library (i.e. the intent is not for you to create your own modified copy of this repository).

Though for a quickstart a compiled version of the Kubernetes [manifests](manifests) generated with this library (specifically with `example.jsonnet`) is checked into this repository in order to try the content out quickly. To try out the stack un-customized run:
* Create the monitoring stack using the config in the `manifests` directory:

```shell
# Create the namespace and CRDs, and then wait for them to be available before creating the remaining resources
kubectl apply --server-side -f manifests/setup
until kubectl get servicemonitors --all-namespaces ; do date; sleep 1; echo ""; done
kubectl apply -f manifests/
```

We create the namespace and CustomResourceDefinitions first to avoid race conditions when deploying the monitoring components.
Alternatively, the resources in both folders can be applied with a single command
`kubectl apply --server-side -f manifests/setup -f manifests`, but it may be necessary to run the command multiple times for all components to
be created successfully.

* And to teardown the stack:

```shell
kubectl delete --ignore-not-found=true -f manifests/ -f manifests/setup
```

### minikube

To try out this stack, start [minikube](https://github.com/kubernetes/minikube) with the following command:

```shell
$ minikube delete && minikube start --kubernetes-version=v1.23.0 --memory=6g --bootstrapper=kubeadm --extra-config=kubelet.authentication-token-webhook=true --extra-config=kubelet.authorization-mode=Webhook --extra-config=scheduler.bind-address=0.0.0.0 --extra-config=controller-manager.bind-address=0.0.0.0
```

The kube-prometheus stack includes a resource metrics API server, so the metrics-server addon is not necessary. Ensure the metrics-server addon is disabled on minikube:

```shell
$ minikube addons disable metrics-server
```

## Getting started

Before deploying kube-prometheus in a production environment, read:

1. [Customizing kube-prometheus](docs/customizing.md)
2. [Customization examples](docs/customizations)
3. [Accessing Graphical User Interfaces](docs/access-ui.md)
4. [Troubleshooting kube-prometheus](docs/troubleshooting.md)

## Documentation

1. [Continuous Delivery](examples/continuous-delivery)
2. [Update to new version](docs/update.md)
3. For more documentation on the project refer to `docs/` directory.

## Contributing

To contribute to kube-prometheus, refer to [Contributing](CONTRIBUTING.md).

## Join the discussion

If you have any questions or feedback regarding kube-prometheus, join the [kube-prometheus discussion](https://github.com/prometheus-operator/kube-prometheus/discussions). Alternatively, consider joining [the kubernetes slack #prometheus-operator channel](http://slack.k8s.io/) or project's bi-weekly [Contributor Office Hours](https://docs.google.com/document/d/1-fjJmzrwRpKmSPHtXN5u6VZnn39M28KqyQGBEJsqUOk/edit#).

## License

Apache License 2.0, see [LICENSE](https://github.com/prometheus-operator/kube-prometheus/blob/main/LICENSE).

## Installation

# Etapa 1: clonar o projeto kube-prometheus
```
$ git clone https://github.com/jbcurty/kube-prometheus.git
```
# Navegue até o diretório kube-prometheus :
```
$ cd kube-prometheus
```

# Etapa 2: criar namespace de monitoramento, CustomResourceDefinitions e pod de operador
```
$ kubectl create -f manifests/setup
```

//O namespace criado com CustomResourceDefinitions é denominado monitoring ://

# Confirmar se o namespace foi craido e os pods:
```
$ kubectl get ns monitoring
$ kubectl get pods -n monitoring
```

# Etapa 3: implantar a pilha de monitoramento do Prometheus no Kubernetes
```
$ kubectl create -f manifests/
```

```
//Verificar se os pods subiram//
$ kubectl get pods -n monitoring

//Para listar todos os serviços criados//
$ kubectl get svc -n monitoring
```

# Etapa 4: acessar os painéis do Prometheus, Grafana e Alertmanager
```
$ kubectl --namespace monitoring port-forward svc/grafana 3000
$ kubectl --namespace monitoring port-forward svc/prometheus-k8s 9090
```
//Primeiro método: acessando a interface do usuário do Prometheus e os painéis do Grafana usando o proxy kubectl
Uma maneira fácil de acessar os painéis do Prometheus, Grafana e Alertmanager é usar kubectl port-forward quando todos os serviços estiverem em execução://


//Segundo método: acessando a interface do usuário do Prometheus e o painel do Grafana usando o NodePort (somente para clusters privados)
Para acessar os painéis do Prometheus, Grafana e Alertmanager usando um dos endereços IP de nós do trabalhador e uma porta, você deve editar os serviços e configurar o tipo para NodePort.//

```
$ kubectl --namespace monitoring patch svc prometheus-k8s -p '{"spec": {"type": "NodePort"}}'
$ kubectl --namespace monitoring patch svc alertmanager-main -p '{"spec": {"type": "NodePort"}}'
$ kubectl --namespace monitoring patch svc grafana -p '{"spec": {"type": "NodePort"}}'

$ kubectl -n monitoring get svc  | grep NodePort
```
# Destruindo/derrubando a pilha de monitoramento do Prometheus
```
$ kubectl delete --ignore-not-found=true -f manifests/ -f manifests/setup
```