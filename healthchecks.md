
```yaml

readinessProbe:
  httpGet:
    path: /health
    port: 80
    scheme: HTTP
  initialDelaySeconds: 60
  failureThreshold: 3
  periodSeconds: 10
  timeoutSeconds: 5

```
