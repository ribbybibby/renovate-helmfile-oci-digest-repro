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
  -e LOG_LEVEL=debug \
  ghcr.io/renovatebot/renovate:44.11.1 \
  ribbybibby/renovate-helmfile-oci-digest-repro
```

Renovate finds and extracts the release:

```
DEBUG: Found helmfile package files
DEBUG: Found 1 package file(s)
 INFO: Dependency extraction complete (baseBranch=main)
       "stats": {
         "managers": {"helmfile": {"fileCount": 1, "depCount": 1}},
         "total": {"fileCount": 1, "depCount": 1}
       }
```

The extracted dep carries the `@sha256:...` suffix in `depName`/`packageName`
and is marked as unsupported:

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

The run finishes with nothing outdated and no branches created:

```
DEBUG: Repository libYears
       "libYears": {"managers": {"helmfile": 0}, "total": 0},
       "dependencyStatus": {"outdated": 0, "total": 1}
...
 INFO: Repository finished
       "result": "done"
```

## Expected behaviour

Split the digest into `currentDigest`, keep `packageName` clean, propose combined
tag+digest updates. Helmfile itself supports the syntax — see
[`parseOCIChartRef`](https://github.com/helmfile/helmfile/blob/main/pkg/state/oci_parse_test.go).

## Reproduced with

Renovate `44.11.1` (upstream `ghcr.io/renovatebot/renovate`).

## Related issue

Filed as <https://github.com/renovatebot/renovate/discussions/45054>.