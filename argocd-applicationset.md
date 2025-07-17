## setup ideas

```yaml

apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: tenant-applicationset
spec:
  syncPolicy:
    preserveResourcesOnDeletion: true

  generators:
    - git:
        repoURL: https://gitlab.trmsaas.com/openshift/day2ops.git
        revision: main
        files:
          - path: "tenants-discovery/staging-joscor-sandclusters/**/config.json"

  template:
    metadata:
      name: '{{tenant.app_id}}'
      labels:
        type: '{{tenant.appType}}'
      annotations:
        argocd.argoproj.io/manifest-generate-paths: ".,.."

    spec:
      project: '{{tenant.argoProject}}'

      source:
        repoURL: https://gitlab.trmsaas.com/openshift/day2ops.git
        targetRevision: '{{tenant.gitBranch}}'
        path: '{{tenant.basepath}}/{{tenant.appfolder_env}}'

      destination:
        server: '{{k8s_cluster.targetClusterName}}'
        namespace: '{{k8s_cluster.targetNamespace}}'

      ignoreDifferences:
        - kind: "ServiceAccount"
          jsonPointers:
            - /imagePullSecrets
            - /secrets
        - kind: "ConfigMap"
          jsonPointers:
            - /data
        - kind: "SecurityContextConstraints"
          group: "security.openshift.io"
          jsonPointers:
            - /volumes

      syncPolicy:
        syncOptions:
          - ApplyOutOfSyncOnly=true
          - RespectIgnoreDifferences=true

  templatePatch: |
    spec:
      {{- if (ne (.autoSync.enabled | toString) "false") }}
      syncPolicy:
        automated:
          prune: {{ if (eq (.autoSync.prune | toString) "nil") }}true{{ else }}{{ .autoSync.prune }}{{ end }}
          selfHeal: {{ if (eq (.autoSync.selfHeal | toString) "nil") }}true{{ else }}{{ .autoSync.selfHeal }}{{ end }}

        retry:
          limit: {{ default 6 .retry.limit }}
          backoff:
            duration: {{ default "30s" .retry.backoff.duration }}
            factor: {{ default 2 .retry.backoff.factor }}
            maxDuration: {{ default "30m" .retry.backoff.maxDuration }}
      {{- end }}
```
```json
{
  "tenant": {
    "app_id": "pnc-machineconfigpool",
    "appType": "machineconfigpool",
    "argoProject": "day2-operations",
    "gitBranch": "main",
    "basepath": "charts",
    "appfolder_env": "env/acm-eng"
  },
  "autoSync": {
    "enabled": true,
    "prune": false,
    "selfHeal": true
  },
  "retry": {
    "limit": 5,
    "backoff": {
      "duration": "20s",
      "factor": 3,
      "maxDuration": "10m"
    }
  },
  "k8s_cluster": {
    "targetClusterName": "in-cluster",
    "targetNamespace": "default"
  }
}
```
