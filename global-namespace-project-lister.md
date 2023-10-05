``  global-project-and-namespace-lister.yaml ``

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: global-project-and-namespace-lister
rules:
- apiGroups:
  - ""
  resources:
  - namespaces
  verbs:
  - get
- apiGroups:
  - project.openshift.io
  resources:
  - projects
  verbs:
  - get

```

`` global-jnk-svc-admin-binding.yaml  ``

```yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: global-jnk-svc-admin-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: global-project-and-namespace-lister
subjects:
- kind: ServiceAccount
  name: jnk-svc-admin
  namespace: ocp-automation

```

`` oc get namespace -l demo.com/mnemonic=jnk -o jsonpath='{.items[*].metadata.name}' | xargs -I {} oc get pods -n {} ``


