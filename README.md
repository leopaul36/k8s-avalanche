# Kubernetes Avalanche

This repository provides basic Kubernetes manifests to spin up a local Avalanche local network.

## Prerequisites

- A Kubernetes environment (e.g., [Minikube](https://minikube.sigs.k8s.io/docs/) 
- `git clone https://github.com/leopaul36/k8s-avalanche.git` 

## Avalanche bootstrap node

We will first a single Avalanche node from which our next nodes will bootstrap

```
kubectl create configmap avalanchego-config-bootstrap --from-file=manifests/node_bootstrap.json
kubectl apply -f manifests/network-bootstrap.yml
kubectl logs -l app=avalanche-network --tail=-1 | grep "node ID is" | awk '{ print $7}'
```

## Avalanche bootstrap service

We then need to expose the `http-port` and `staking-port` of our bootstrap node so the other nodes will be able to reach it.

```
kubectl apply -f manifests/network-bootstrap-svc.yml
kubectl get svc avalanche-network-bootstrap-svc -o custom-columns=CLUSTER-IP:.spec.clusterIP
```

## Avalanche cluster

Finally we can run another Avalanche node that will bootstrap from the previous one.

```
kubectl create configmap avalanchego-config-cluster --from-file=manifests/node_cluster.json
kubectl apply -f manifests/network-statefulset.yml
```

The bootstrapping went fine. Let's add some more replicas to have a bigger Avalanche network.

```
kubectl scale statefulset avalanche-network-statefulset --replicas=5
```

## Monitoring

### Prometheus

```
kubectl create configmap prometheus-config --from-file=manifests/monitoring/prometheus.yml
kubectl apply -f manifests/monitoring/prometheus-dep.yml
kubectl apply -f manifests/monitoring/prometheus-svc.yml
```

### Grafana

```
kubectl create configmap grafana-datasource-config --from-file=manifests/monitoring/prom.yml
kubectl apply -f manifests/monitoring/grafana-dashboards-cm.yml
kubectl apply -f manifests/monitoring/grafana-dep.yml
kubectl apply -f manifests/monitoring/grafana-svc.yml
```

## TODO

- [ ] Improve README.md
- [ ] Improve [Monitoring](https://docs.avax.network/build/tutorials/nodes-and-staking/setting-up-node-monitoring/) by adding node_exporter metrics and enabling [Kubernetes service discovery](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#kubernetes_sd_config)

## Related projects

- https://github.com/ava-labs/avalanche-network-runner
- https://github.com/ava-labs/avalanchego
- https://docs.avax.network/build/tutorials/nodes-and-staking/setting-up-node-monitoring/
