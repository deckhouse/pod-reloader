---
title: "Модуль pod-reloader: примеры"
---

## Слежение за всеми изменениями во всех подключенных ресурсах: смонтированных как volume или используемых в переменных окружения

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

## Слежение за изменениями только в конкретных ресурсах

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

## Слежение за изменениями в ресурсах из списка

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

## Перезапуск только при изменении Secret'ов (игнорируя ConfigMap'ы)

Используйте `secret.pod-reloader.deckhouse.io/auto: "true"` вместо общей аннотации `pod-reloader.deckhouse.io/auto`, если нагрузка должна перезапускаться только при изменении связанного Secret, но не ConfigMap. Аналогично, `configmap.pod-reloader.deckhouse.io/auto: "true"` реагирует только на изменения ConfigMap.

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

## Пауза rollout при пакетном обновлении конфигурации

Используйте аннотацию `pod-reloader.deckhouse.io/pause-period`, чтобы избежать многократных перезапусков при одновременном изменении нескольких ConfigMap или Secret. Rollout произойдёт один раз — после истечения указанного периода ожидания.

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

## Перезапуск ресурса `Rollout` Argo Rollouts

Сначала включите параметр `enableArgoRollouts: true` в конфигурации модуля. После этого ресурс `Rollout` обрабатывается по тем же аннотациям, что и Deployment. Необязательная аннотация `pod-reloader.deckhouse.io/rollout-strategy` определяет способ обновления ресурса: `rollout` (значение по умолчанию) выполняет полный выкат с заданными canary- или blue-green-шагами, `restart` только перезапускает поды через `spec.restartAt`.

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
