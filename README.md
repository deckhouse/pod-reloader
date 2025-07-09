# Pod Reloader Module for Deckhouse

The module utilizes [Reloader](https://github.com/stakater/Reloader).
It provides the ability for automatic rollout on ConfigMap or Secret changes.  
The module uses annotations for operating. The module is running on **system** nodes.

**Reloader does not have HighAvailability mode.**

## 🔁 What is Reloader?
Reloader is a Kubernetes controller that automatically triggers rollouts of workloads (like Deployments, StatefulSets, and more) whenever referenced Secrets or ConfigMaps are updated.

In a traditional Kubernetes setup, updating a Secret or ConfigMap does not automatically restart or redeploy your workloads. This can lead to stale configurations running in production, especially when dealing with dynamic values like credentials, feature flags, or environment configs.

Reloader bridges that gap by ensuring your workloads stay in sync with configuration changes — automatically and safely.


## ⚡ Quick Start
### 1. Enable the Module
Add the pod-reloader module to your Deckhouse configuration:

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: pod-reloader
spec:
  enabled: true
  version: 1
```

### 2. Configure Automatic Reload
To enable automatic reload for a Deployment:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  annotations:
    reloader.stakater.com/auto: "true"
spec:
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: app
          image: your-image
          envFrom:
            - configMapRef:
                name: my-config
            - secretRef:
                name: my-secret
```
This tells Reloader to watch the ConfigMap and Secret referenced in this deployment. When either is updated, it will trigger a rollout.

## 📚 Documentation

- [Deckhouse Module Documentation](https://deckhouse.ru/products/kubernetes-platform/documentation/v1/modules/pod-reloader/)
- [Original Stakater Reloader Documentation](https://docs.stakater.com/reloader/)
