# Loki + Alloy (local cluster)

Single-binary **Loki** with **Grafana Alloy** as the log shipper — the logging
half of the ShopHub observability stack (spec 4.1). Alloy runs as a DaemonSet,
discovers every pod through the Kubernetes API, and pushes their stdout/stderr
to Loki; Loki is queried from Grafana. Alloy replaces the now-deprecated
Promtail (see `../alloy/`).

## Grafana datasource

Loki ships **no Grafana of its own** — it plugs into the Grafana that comes
with the `kube-prometheus-stack` release. That datasource is now wired up in
[`../kube-prometheus-stack/values.yaml`](../kube-prometheus-stack/values.yaml)
under `grafana.additionalDataSources` (alongside Tempo for traces).

The ShopHub dashboards (`helm-charts/charts/shophub`) and the per-Shop
dashboards the shop-operator generates reference this datasource by the fixed
uid `loki`, so keep the uid stable.
