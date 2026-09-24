---
title: "The pod-reloader module"
---

The module utilizes [Reloader](https://github.com/stakater/Reloader).
It provides the ability for automatic rollout on ConfigMap or Secret changes.
The module uses annotations for operating. The module is running on **system** nodes.

{{< alert level="info" >}}
Reloader does not have HighAvailability mode.
{{< /alert >}}

All annotations are described here. You can find examples in the [Examples](examples.html) section of the documentation.

| Annotation                                   | Resource                           | Description                                                                                                  | Acceptable values                             |
| -------------------------------------------- |------------------------------------| ------------------------------------------------------------------------------------------------------------ | --------------------------------------------- |
| `pod-reloader.deckhouse.io/auto` | Deployment, DaemonSet, StatefulSet | Changes to associated (mounted or used as environment variables) ConfigMap or Secret will cause a restart of this controller's pods | `"true"`, `"false"` |
| `secret.pod-reloader.deckhouse.io/auto` | Deployment, DaemonSet, StatefulSet | Changes to associated Secrets only will cause a restart of this controller's pods | `"true"`, `"false"` |
| `configmap.pod-reloader.deckhouse.io/auto` | Deployment, DaemonSet, StatefulSet | Changes to associated ConfigMaps only will cause a restart of this controller's pods | `"true"`, `"false"` |
| `pod-reloader.deckhouse.io/search` | Deployment, DaemonSet, StatefulSet | If this annotation is present, a restart will only occur when ConfigMaps or Secrets with the annotation `pod-reloader.deckhouse.io/match: "true"` change | `"true"`, `"false"` |
| `pod-reloader.deckhouse.io/configmap-reload` | Deployment, DaemonSet, StatefulSet | Specifying a list of ConfigMaps that the controller depends on | `"some-cm"`, `"some-cm1,some-cm2"` |
| `pod-reloader.deckhouse.io/secret-reload` | Deployment, DaemonSet, StatefulSet | Specifying a list of secrets that the controller depends on | `"some-secret"`, `"some-secret1,some-secret2"` |
| `pod-reloader.deckhouse.io/ignore`    | Secret, ConfigMap | Changing ConfigMap or Secret with this annotation will not occur restarts                                                                                                                        | `"true"`, `"false"` |
| `pod-reloader.deckhouse.io/match` | Secret, ConfigMap | Annotation by which related resources are selected to track changes | `"true"`, `"false"` |
| `pod-reloader.deckhouse.io/pause-period` | Deployment | Pauses rollouts for the specified duration when several ConfigMaps or Secrets are updated in quick succession | `30s`, `5m` |
| `pod-reloader.deckhouse.io/rollout-strategy` | Rollout | Defines how an Argo Rollouts `Rollout` is updated: `restart` sets `spec.restartAt` (a pod restart without a new rollout), `rollout` performs a full rollout | `"restart"`, `"rollout"` |

{{< alert level="warning" >}}
Annotation `pod-reloader.deckhouse.io/search` cannot be used together with `pod-reloader.deckhouse.io/auto: "true"` because Reloader will ignore `pod-reloader.deckhouse.io/search` and `pod-reloader.deckhouse.io/match`. For the right behavior set `pod-reloader.deckhouse.io/auto` to `"false"` or delete it.
{{< /alert >}}

{{< alert level="warning" >}}
Annotations `pod-reloader.deckhouse.io/configmap-reload` and `pod-reloader.deckhouse.io/secret-reload` cannot be used together with `pod-reloader.deckhouse.io/auto: "true"` because Reloader will ignore `pod-reloader.deckhouse.io/configmap-reload` and `pod-reloader.deckhouse.io/secret-reload`. For the right behavior set `pod-reloader.deckhouse.io/auto` to `"false"` or delete it.
{{< /alert >}}

## Argo Rollouts support

The module can also roll out the `Rollout` resource of [Argo Rollouts](https://argoproj.github.io/argo-rollouts/). The support is disabled by default; enable it with the [`enableArgoRollouts`](configuration.html#parameters-enableargorollouts) parameter:

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: pod-reloader
spec:
  enabled: true
  version: 1
  settings:
    enableArgoRollouts: true
```

Argo Rollouts must be installed in the cluster (the `rollouts.argoproj.io` CRD must exist) before enabling the parameter, otherwise `Rollout` resources will not be watched.

Once enabled, a `Rollout` reacts to the same annotations as a Deployment (`pod-reloader.deckhouse.io/auto`, `pod-reloader.deckhouse.io/search`, `pod-reloader.deckhouse.io/configmap-reload`, etc.).

If the `pod-reloader.deckhouse.io/rollout-strategy` annotation is not set, the `rollout` strategy is used.
