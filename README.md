# renovate-helmfile-oci-digest-repro

Minimal reproduction for a Renovate bug in the `helmfile` manager.

## Current behaviour

Given a `helmfile.yaml` with an OCI chart pinned by both tag and digest, the
built-in `helmfile` manager bakes the `@sha256:...` suffix into `depName`
and `packageName` and drops the release with `skipReason: unsupported-chart-type`:

```json
{
  "depName": "ghcr.io/grafana/helm-charts/grafana@sha256:3d75dc31…",
  "currentValue": "10.5.15",
  "datasource": "docker",
  "packageName": "ghcr.io/grafana/helm-charts/grafana@sha256:3d75dc31…",
  "skipReason": "unsupported-chart-type"
}
```

## Expected behaviour

Split the digest into `currentDigest`, keep `packageName` clean, propose combined
tag+digest updates. Helmfile itself supports the syntax — see
[`parseOCIChartRef`](https://github.com/helmfile/helmfile/blob/main/pkg/state/oci_parse_test.go).

## Reproduced with

Renovate `44.11.1` (upstream `ghcr.io/renovatebot/renovate`).

## Related issue

TODO: link once filed on renovatebot/renovate.
