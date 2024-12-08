# devops

using this IaC, you can deploy a One-Click kubernetes Cluster including CRI-O as CRI, Cilium as CNI, Rook-Ceph as CSI.

# Monitoring & Logging

Node-exporter on each k8s nodes, and also Fluent Bit and Kube State Metrics are deployed as DaemonSet for achive Observability on Kubernets Resources such as Pods, PVC and etc.


# Ecternal Elemans

the Docker Registry, Gitlab, ELK and Prometheus/grafana are outside of Cluster as remote telementary

