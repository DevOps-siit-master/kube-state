# Discord ops alerts (local cluster)

Provisions the Discord channel + webhook that cluster-level alerts (node
CPU/memory, pod crash-looping, Prometheus operator health, etc.) are routed
to. Uses the `shophub-discord` chart from `helm-charts`, which is a generic 
chart that creates a single `DiscordChannel` resource —
this is one install of it, scoped to ops/cluster alerts specifically.

## What this actually deploys

A `shop-operator.devops-siit.io/v1 DiscordChannel` resource. `shop-operator`'s
`DiscordChannelReconciler` picks it up, creates the real Discord channel +
webhook via the bot API, and writes the webhook URL into a Secret named
`<discordChannelObjectName>-webhook` in this namespace (`observability`).

## Dependencies (must already exist before this installs cleanly)

- `shop-operator` must be running and watching this namespace — the
  `DiscordChannel` CRD and its reconciler live there.
- The operator's own bot-token Secret (`discord-bot-credentials`, in
  `shop-operator-system` by default) must already exist — see
  `shop-operator`'s own README/TESTING.md for how to create it. This is
  separate from the webhook Secret this release produces.

## Coordination

- `cluster-alerts.yaml` (`PrometheusRule`) and `alertmanager-routing.yaml`
  (`AlertmanagerConfig`) are **not** part of this chart — they're static,
  single-use manifests with no real parameterization, applied directly
  (`kubectl apply -f`), not installed via Helm. They live in
  `../cluster-alerting/` and reference the webhook Secret this release creates by name
  (`discord-ops-webhook`, matching `discordChannelObjectName` below).
- If `discordChannelObjectName` or the Discord channel's display name ever
  changes here, update `alertmanager-routing.yaml`'s `apiURL.name`
  reference to match — nothing enforces this automatically.

## Local usage

```
helm install shophub-discord ../path/to/helm-charts/repo/locally/charts/shophub-discord \
  -n observability \
  -f values.yaml
```
