Absolutely! Here's the updated version using **`config.yaml`** instead of `config.json`, along with the corresponding **`ApplicationSet`**.

---

## ✅ `config.yaml` (Tenant Configuration File)

Save this as:
`tenants-discovery/staging-joscor-sandclusters/<some-tenant>/config.yaml`

```yaml
tenant:
  app_id: pnc-machineconfigpool
  appType: machineconfigpool
  argoProject: day2-operations
  gitBranch: main
  basepath: charts
  appfolder_env: env/acm-eng

autoSync:
  enabled: true
  prune: false
  selfHeal: true

retry:
  limit: 5
  backoff:
    duration: "20s"
    factor: 3
    maxDuration: "10m"

k8s_cluster:
  targetClusterName: in-cluster
  targetNamespace: default
```

---

## ✅ `ApplicationSet.yaml`

This will process all `config.yaml` files in matching folders and apply conditional sync automation and retry logic.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: tenant-applicationset
spec:
  goTemplate: true
  goTemplateOptions: ["missingkey=error"]
  syncPolicy:
    preserveResourcesOnDeletion: true

  generators:
    - git:
        repoURL: git@gitlab.trmsaas.com:openshift/day2ops.git
        revision: main
        files:
          - path: "tenants-discovery/staging-joscor-sandclusters/**/config.yaml"

  template:
    metadata:
      name: '{{ tenant.app_id }}'
      labels:
        type: '{{ tenant.appType }}'
      annotations:
        argocd.argoproj.io/manifest-generate-paths: ".,.."

    spec:
      project: '{{ tenant.argoProject }}'

      source:
        repoURL: git@gitlab.trmsaas.com:openshift/day2ops.git
        targetRevision: '{{ tenant.gitBranch }}'
        path: '{{ tenant.basepath }}/{{ tenant.appfolder_env }}'

      destination:
        server: '{{ k8s_cluster.targetClusterName }}'
        namespace: '{{ k8s_cluster.targetNamespace }}'

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
    {{- if (ne (.autoSync.enabled | toString) "false") }}
    spec:
      syncPolicy: {{
        dict
          "automated" (dict
            "prune" (default true .autoSync.prune)
            "selfHeal" (default true .autoSync.selfHeal)
          )
          "retry" (dict
            "limit" (default 6 .retry.limit)
            "backoff" (dict
              "duration" (default "30s" .retry.backoff.duration)
              "factor" (default 2 .retry.backoff.factor)
              "maxDuration" (default "30m" .retry.backoff.maxDuration)
            )
          )
        | toJson
      }}
    {{- end }}

###
  templatePatch: |
    {{- if (ne (.autoSync.enabled | toString) "false") }}
    spec:
      syncPolicy:
        automated:
          prune: {{- if hasKey .autoSync "prune" }}{{ .autoSync.prune }}{{ else }}true{{ end }}
          selfHeal: {{- if hasKey .autoSync "selfHeal" }}{{ .autoSync.selfHeal }}{{ else }}true{{ end }}
        retry:
          limit: {{ default 6 .retry.limit }}
          backoff:
            duration: {{ default "30s" .retry.backoff.duration }}
            factor: {{ default 2 .retry.backoff.factor }}
            maxDuration: {{ default "30m" .retry.backoff.maxDuration }}
    {{- end }}


```

---

### ✅ Summary:

| File                  | Purpose                                                                     |
| --------------------- | --------------------------------------------------------------------------- |
| `config.yaml`         | Per-tenant settings (YAML-based, clean, override-friendly)                  |
| `ApplicationSet.yaml` | Git generator using YAML files, supports dynamic syncPolicy with defaulting |

Let me know if you'd like Helm `valueFiles`, extra annotations, or full project structure scaffolding.
