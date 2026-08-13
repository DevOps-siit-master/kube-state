# Loki + Alloy (local cluster)

Single-binary **Loki** with **Grafana Alloy** as the log shipper — the logging
half of the ShopHub observability stack (spec 4.1). Alloy runs as a DaemonSet,
discovers every pod through the Kubernetes API, and pushes their stdout/stderr
to Loki; Loki is queried from Grafana. Alloy replaces the now-deprecated
Promtail (see `../alloy/`).

## Grafana datasource (coordination)

Loki ships **no Grafana of its own** — it plugs into the Grafana that comes
with the `kube-prometheus-stack` release (owned by the observability/operator
work). Add Loki as a datasource in **that** release's values so Grafana can
query logs:

```yaml
grafana:
  additionalDataSources:
    - name: Loki
      type: loki
      uid: loki
      access: proxy
      url: http://loki-gateway.observability.svc.cluster.local
      isDefault: false
```

The ShopHub dashboard (`helm-charts/charts/shophub`) references this
datasource by the fixed uid `loki`, so keep the uid stable.
