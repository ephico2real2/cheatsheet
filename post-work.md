```yaml


---image-registry-pvc.yaml---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: image-registry-pvc
  namespace: openshift-image-registry
  annotations:
    argocd.argoproj.io/sync-wave: "-1"
    argocd.argoproj.io/compare-options: IgnoreExtraneous
    argocd.argoproj.io/sync-options: Prune=false
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 200Gi
  storageClassName: ocs-storagecluster-cephfs
  
--- imageregistry.operator.yaml ---
  
apiVersion: imageregistry.operator.openshift.io/v1
kind: Config
metadata:
  name: cluster
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            docker-registry: default      
        namespaces:
        - openshift-image-registry
        topologyKey: kubernetes.io/hostname
  defaultRoute: true
  logLevel: Normal
  managementState: Managed
  nodeSelector:
    node-role.kubernetes.io/infra: ""
  operatorLogLevel: Normal
  proxy: {}
  replicas: 2
  requests:
    read:
      maxWaitInQueue: 0s
    write:
      maxWaitInQueue: 0s
  rolloutStrategy: Recreate
  storage:
    managementState: Unmanaged
    pvc:
      claim: image-registry-pvc
  tolerations:
  - effect: NoSchedule
    key: node-role.kubernetes.io/infra
    operator: Exists
  unsupportedConfigOverrides: null
  
--- kustomization.yaml --
namespace: openshift-image-registry

resources:
  - image-registry-pvc.yaml
  - imageregistry.operator.yaml
  
  
  
--- configmap-cluster-monitoring-config.yaml ---
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: cluster-monitoring-config
  namespace: openshift-monitoring
  annotations:
    argocd.argoproj.io/sync-wave: "0"
data:
  config.yaml: |
    enableUserWorkload: true
    alertmanagerMain:
      retention: 48h
      volumeClaimTemplate:
        metadata:
          name: ocs-alertmanager-claim
        spec:
          storageClassName: ocs-storagecluster-ceph-rbd
          resources:
            requests:
              storage: 75Gi
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      tolerations:
      - key: node-role.kubernetes.io/infra
        operator: Exists
        effect: NoSchedule
    prometheusK8s:
      retention: 72h
      volumeClaimTemplate:
        metadata:
          name: ocs-prometheus-claim
        spec:
          storageClassName: ocs-storagecluster-ceph-rbd
          resources:
            requests:
              storage: 300Gi
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      tolerations:
      - key: node-role.kubernetes.io/infra
        operator: Exists
        effect: NoSchedule
    prometheusOperator:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      tolerations:
      - key: node-role.kubernetes.io/infra
        operator: Exists
        effect: NoSchedule
    grafana:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      tolerations:
      - key: node-role.kubernetes.io/infra
        operator: Exists
        effect: NoSchedule
    k8sPrometheusAdapter:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      tolerations:
      - key: node-role.kubernetes.io/infra
        operator: Exists
        effect: NoSchedule
    kubeStateMetrics:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      tolerations:
      - key: node-role.kubernetes.io/infra
        operator: Exists
        effect: NoSchedule
    telemeterClient:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      tolerations:
      - key: node-role.kubernetes.io/infra
        operator: Exists
        effect: NoSchedule
    openshiftStateMetrics:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
      tolerations:
      - key: node-role.kubernetes.io/infra
        operator: Exists
        effect: NoSchedule
        
        
--- openshift-user-workload-monitoring.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: user-workload-monitoring-config
  namespace: openshift-user-workload-monitoring
data:
  config.yaml: |
  
---

-- kustomization.yaml
namespace: openshift-monitoring

bases:
  - ../dynatrace-control-plane-monitoring

resources:
  - configmap-cluster-monitoring-config.yaml
  - openshift-user-workload-monitoring.yaml
  
---



logging


---  cluster-logging.yaml

apiVersion: logging.openshift.io/v1
kind: ClusterLogging
metadata:
  name: "instance"
spec:
  managementState: "Managed"
  logStore:
    type: "elasticsearch"
    retentionPolicy:
      # application:
      #  maxAge: 3d
      infra:
        maxAge: 3d
      audit:
        maxAge: 3d
    elasticsearch:
      nodeCount: 0
      nodeSelector: 
        node-role.kubernetes.io/infra: ''
      tolerations:
      - effect: NoSchedule
        key: node-role.kubernetes.io/infra
        operator: Exists
      storage:
        storageClassName: "ocs-storagecluster-ceph-rbd"
        size: 280G
      resources:
        requests:
          memory: "16Gi"
        limits:
          memory: "24Gi"
      proxy: 
        resources:
          limits:
            memory: 256Mi
          requests:
             memory: 256Mi
      redundancyPolicy: "SingleRedundancy"
      #redundancyPolicy: "MultipleRedundancy"
  visualization:
    type: "kibana"
    kibana:
      replicas: 0
      nodeSelector: 
        node-role.kubernetes.io/infra: ''
      tolerations:
      - effect: NoSchedule
        key: node-role.kubernetes.io/infra
        operator: Exists
      resources:
        limits:
          memory: 2Gi
        requests:
          cpu: 100m
          memory: 1Gi
  collection:
    logs:
      type: "fluentd"
      fluentd:
#        nodeSelector: 
#          node-role.kubernetes.io/disable: ''
        tolerations:
        - effect: NoSchedule
          operator: Exists
        - effect: NoExecute
          operator: Exists
          
          
--kustomization.yaml

commonLabels:
  app: openshift-logging
  kind: cluster-logging
  master: argocd-application

namespace: openshift-logging

#patches:
#  - es-storage.yaml
#  - es-limits.yaml
#  - es-retention.yaml

resources:
  - cluster-logging.yaml
  - cluster-all-forward.yaml


```
