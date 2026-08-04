# renovate-helmfile-oci-digest-repro

Minimal reproduction for a Renovate bug in the `helmfile` manager.

## Current behaviour

Given a `helmfile.yaml` with an OCI chart pinned by both tag and digest, the
built-in `helmfile` manager bakes the `@sha256:...` suffix into `depName`
and `packageName` and drops the release with
`skipReason: unsupported-chart-type`.

Reproduce by running Renovate against this repo:

```
docker run --rm \
  -e RENOVATE_TOKEN=<your-github-token> \
  -e RENOVATE_DRY_RUN=lookup \
  -e LOG_LEVEL=debug \
  ghcr.io/renovatebot/renovate:44.11.1 \
  ribbybibby/renovate-helmfile-oci-digest-repro
```

Extraction result:

```
 INFO: Dependency extraction complete (repository=ribbybibby/renovate-helmfile-oci-digest-repro, baseBranch=main)
       "stats": {
         "managers": {"helmfile": {"fileCount": 1, "depCount": 1}},
         "total": {"fileCount": 1, "depCount": 1}
       }
```

The extracted dep:

```json
{
  "depName": "ghcr.io/grafana/helm-charts/grafana@sha256:3d75dc3173c3fb07eaa8853283f38558c7c27259f7ed00cf8c096173667426a1",
  "currentValue": "10.5.15",
  "datasource": "docker",
  "packageName": "ghcr.io/grafana/helm-charts/grafana@sha256:3d75dc3173c3fb07eaa8853283f38558c7c27259f7ed00cf8c096173667426a1",
  "skipReason": "unsupported-chart-type",
  "updates": []
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
