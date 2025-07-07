# Pod Reloader Module for Deckhouse

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Deckhouse](https://img.shields.io/badge/Deckhouse-Module-blue)](https://deckhouse.io/)

A Deckhouse Kubernetes Platform module that provides automatic pod reload functionality when ConfigMaps or Secrets are updated. This module is based on the popular [Stakater Reloader](https://github.com/stakater/Reloader) project and has been adapted for seamless integration with the Deckhouse ecosystem.

## 🔁 What is Pod Reloader?

Pod Reloader is a Kubernetes controller that automatically triggers rolling upgrades of workloads (Deployments, StatefulSets, DaemonSets, etc.) whenever referenced ConfigMaps or Secrets are updated. This ensures that your applications always run with the latest configuration without manual intervention.

In traditional Kubernetes setups, updating a ConfigMap or Secret doesn't automatically restart the pods that use them, leading to stale configurations in production. Pod Reloader bridges this gap by monitoring these resources and triggering rolling updates when changes are detected.

## 🚀 Why Use Pod Reloader in Deckhouse?

- **✅ Zero Manual Restarts**: Eliminates the need to manually restart workloads after configuration changes
- **🔒 Security-First**: Ensures applications always use the most up-to-date credentials and certificates
- **🛠️ Flexible Configuration**: Works with all major Kubernetes workload types
- **⚡ CI/CD Ready**: Perfect for GitOps workflows where configurations change frequently
- **🎯 Deckhouse Integration**: Seamlessly integrates with Deckhouse's module system and conventions
- **📊 Observability**: Built-in metrics and alerting capabilities compatible with Deckhouse monitoring

## 🔧 How It Works

1. **Configuration Sources**: External secrets operators, sealed secrets, cert-manager, or manual ConfigMaps/Secrets
2. **Monitoring**: Pod Reloader watches for changes in ConfigMaps and Secrets
3. **Automatic Rollout**: When changes are detected, it triggers rolling updates of associated workloads
4. **Zero Downtime**: Uses Kubernetes' built-in rolling update mechanism to ensure availability

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

Annotate your workloads to enable automatic reloading:

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
          image: my-app:latest
          envFrom:
            - configMapRef:
                name: my-config
            - secretRef:
                name: my-secret
```

This configuration tells Pod Reloader to monitor the referenced ConfigMap and Secret, automatically triggering a rolling update when either is modified.

## 🧩 Usage Patterns

### Automatic Reload (Recommended)

Use these annotations to automatically restart workloads when referenced resources change:

| Annotation | Description |
|------------|-------------|
| `reloader.stakater.com/auto: "true"` | Reloads when any referenced ConfigMap or Secret changes |
| `secret.reloader.stakater.com/auto: "true"` | Reloads only when referenced Secrets change |
| `configmap.reloader.stakater.com/auto: "true"` | Reloads only when referenced ConfigMaps change |

### Specific Resource Monitoring

Target specific resources regardless of pod references:

| Annotation | Description |
|------------|-------------|
| `secret.reloader.stakater.com/reload: "my-secret"` | Monitors specific Secret by name |
| `configmap.reloader.stakater.com/reload: "my-config"` | Monitors specific ConfigMap by name |

### Targeted Reload with Match Pattern

For fine-grained control in multi-tenant environments:

```yaml
# On the workload
metadata:
  annotations:
    reloader.stakater.com/search: "true"

# On the ConfigMap/Secret
metadata:
  annotations:
    reloader.stakater.com/match: "true"
```

## ⚙️ Module Configuration

Configure the pod-reloader module through Deckhouse's ModuleConfig:

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: pod-reloader
spec:
  enabled: true
  version: 1
  settings:
    # Reload behavior configuration
    reloadOnCreate: true
    reloadOnDelete: true
    autoReloadAll: false
    reloadStrategy: "env-vars"  # or "annotations"
    
    # Resource filtering
    resourceLabelSelector: ""
    namespacesToIgnore: ["kube-system", "d8-system"]
    
    # Alerting configuration
    alerting:
      enabled: true
      webhook:
        url: "https://hooks.slack.com/services/your/webhook/url"
        sink: "slack"  # slack, teams, gchat, or webhook
        additionalInfo: "Pod Reloader triggered in production"
    
    # Resource limits
    resources:
      requests:
        cpu: "10m"
        memory: "128Mi"
      limits:
        cpu: "150m"
        memory: "512Mi"
    
    # Advanced configuration
    logFormat: "json"
    namespaceSelector: ""
```

## 📊 Monitoring and Observability

The module provides built-in integration with Deckhouse's monitoring stack:

### Metrics

Pod Reloader exposes Prometheus metrics that are automatically scraped by Deckhouse:

- `reloader_reload_executed_total`: Total number of reloads executed
- `reloader_reload_errors_total`: Total number of reload errors
- `reloader_watched_resources_total`: Total number of watched resources

### Alerts

Pre-configured alerts for common issues:

- **PodReloaderDown**: Pod Reloader controller is not running
- **PodReloaderErrors**: High error rate in reload operations
- **PodReloaderResourcesHigh**: High resource consumption

### Grafana Dashboard

A dedicated Grafana dashboard is included showing:
- Reload activity over time
- Resource consumption
- Error rates and patterns
- Watched resources distribution

## 🔐 Security Considerations

### RBAC

The module creates minimal RBAC permissions:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-reloader
rules:
- apiGroups: [""]
  resources: ["configmaps", "secrets"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments", "statefulsets", "daemonsets"]
  verbs: ["get", "list", "patch"]
```

### Network Policies

When Deckhouse network policies are enabled, Pod Reloader is automatically configured with appropriate network access rules.

## 🛠️ Advanced Configuration

### GitOps Integration

For GitOps workflows (ArgoCD, Flux), use the annotations strategy to prevent configuration drift:

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: pod-reloader
spec:
  settings:
    reloadStrategy: "annotations"
```

### Multi-Cluster Setup

In multi-cluster Deckhouse deployments, configure per-cluster settings:

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: pod-reloader
spec:
  settings:
    namespacesToIgnore: ["kube-system", "d8-system", "deckhouse"]
    resourceLabelSelector: "environment=production"
```

## 🔄 Migration from Stakater Reloader

If you're migrating from the original Stakater Reloader:

1. **Remove existing Reloader**: Uninstall the standalone Reloader installation
2. **Enable the module**: Add the pod-reloader module to your Deckhouse configuration
3. **Update annotations**: No changes needed - all existing annotations are compatible
4. **Review settings**: Migrate any custom configurations to the ModuleConfig

## 🐛 Troubleshooting

### Common Issues

**Pod Reloader not triggering reloads:**
- Check if the workload has the correct annotations
- Verify ConfigMap/Secret names match exactly
- Ensure the module is enabled and running

**High resource consumption:**
- Adjust resource limits in module configuration
- Consider using namespace filtering to reduce scope
- Check for resource leaks in logs

**Permission denied errors:**
- Verify RBAC permissions
- Check if security policies are blocking operations

### Debug Mode

Enable debug logging:

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: pod-reloader
spec:
  settings:
    logFormat: "json"
    logLevel: "debug"
```

## 📚 Documentation

- [Deckhouse Module Development](https://deckhouse.io/products/kubernetes-platform/documentation/v1/module-development/)
- [Original Stakater Reloader Documentation](https://docs.stakater.com/reloader/)
- [Kubernetes Configuration Management Best Practices](https://kubernetes.io/docs/concepts/configuration/)

## 🤝 Contributing

Contributions are welcome! Please read our [Contributing Guide](CONTRIBUTING.md) and follow the Deckhouse module development guidelines.

### Development Setup

1. Clone the repository
2. Set up a local Deckhouse development environment
3. Enable the module in development mode
4. Make your changes and test thoroughly
5. Submit a pull request

## 📄 License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.

## 🏗️ Based On

This module is based on the excellent [Stakater Reloader](https://github.com/stakater/Reloader) project, adapted for seamless integration with the Deckhouse Kubernetes Platform.

## 🆘 Support

- **Documentation**: [Deckhouse Documentation](https://deckhouse.io/documentation/)
- **Community**: [Deckhouse Community](https://github.com/deckhouse/deckhouse/discussions)
- **Issues**: [GitHub Issues](https://github.com/deckhouse/pod-reloader/issues)
- **Enterprise Support**: Available through Deckhouse Enterprise subscriptions

---

Made with ❤️ for the Deckhouse community