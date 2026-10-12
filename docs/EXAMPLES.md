---
title: "Module pod-reloader: examples"
---

## Tracking all changes in all attached resources: mounted as volumes or used in environment values

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
  annotations:
    pod-reloader.deckhouse.io/auto: "true"
spec:
  template:
    spec:
      containers:
        - name: nginx
          env:
            - name: SECRET_WORD
              valueFrom:
                secretKeyRef:
                  name: nginx-secret-value
                  key: extra
          volumeMounts:
            - name: pages
              mountPath: "/usr/share/nginx/pages"
      volumes:
        - name: pages
          configMap:
            name: nginx-pages
---
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: nginx-secret-value
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-pages
```

## Tracking changes in specific resources

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  annotations:
    pod-reloader.deckhouse.io/search: "true"
spec:
  template:
    spec:
      containers:
        - name: nginx
          env:
            - name: SECRET_WORD
              valueFrom:
                secretKeyRef:
                  name: nginx-secret-value
                  key: extra
---
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: nginx-secret-value
  annotations:
    pod-reloader.deckhouse.io/match: "true"
```

## Tracking changes in resources from the list

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  annotations:
    pod-reloader.deckhouse.io/configmap-reload: "nginx-config,nginx-pages"
spec:
  template:
    spec:
      containers:
        - name: nginx
          volumeMounts:
            - name: pages
              mountPath: "/usr/share/nginx/pages"
            - name: config
              mountPath: "/etc/nginx/templates"
      volumes:
        - name: pages
          configMap:
            name: nginx-pages
        - name: config
          configMap:
            name: nginx-config
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-pages
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
```

## Reloading only when Secrets change (ignoring ConfigMaps)

Use `secret.pod-reloader.deckhouse.io/auto: "true"` instead of the general `pod-reloader.deckhouse.io/auto` annotation if you want the workload to restart only when a referenced Secret changes, not when a ConfigMap changes. Similarly, use `configmap.pod-reloader.deckhouse.io/auto: "true"` to react only to ConfigMap changes.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
  annotations:
    secret.pod-reloader.deckhouse.io/auto: "true"
spec:
  template:
    spec:
      containers:
        - name: nginx
          env:
            - name: SECRET_WORD
              valueFrom:
                secretKeyRef:
                  name: nginx-secret-value
                  key: extra
          volumeMounts:
            - name: pages
              mountPath: "/usr/share/nginx/pages"
      volumes:
        - name: pages
          configMap:
            name: nginx-pages
---
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: nginx-secret-value
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-pages
```

## Pausing rollouts during batch config updates

Use `pod-reloader.deckhouse.io/pause-period` to prevent repeated restarts when several ConfigMaps or Secrets are updated in quick succession. The workload will be rolled out only once, after the pause period expires.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  annotations:
    pod-reloader.deckhouse.io/auto: "true"
    pod-reloader.deckhouse.io/pause-period: "30s"
spec:
  template:
    spec:
      containers:
        - name: nginx
          envFrom:
            - configMapRef:
                name: nginx-config
            - secretRef:
                name: nginx-secret
```

## Rolling out an Argo Rollouts `Rollout`

Set `enableArgoRollouts: true` in the module configuration first. After that, a `Rollout` is handled by the same annotations as a Deployment. The optional `pod-reloader.deckhouse.io/rollout-strategy` annotation defines how the resource is updated: `rollout` (default) performs a full rollout with the configured canary or blue-green steps, `restart` only restarts the pods via `spec.restartAt`.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: nginx-rollout
  annotations:
    pod-reloader.deckhouse.io/auto: "true"
    pod-reloader.deckhouse.io/rollout-strategy: "restart"
spec:
  strategy:
    canary:
      steps:
        - setWeight: 25
        - pause: {duration: 30s}
  template:
    spec:
      containers:
        - name: nginx
          envFrom:
            - configMapRef:
                name: nginx-config
            - secretRef:
                name: nginx-secret
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
---
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: nginx-secret
```
