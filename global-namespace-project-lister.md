``  global-project-and-namespace-lister ``

```
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
  - list
  - get
- apiGroups:
  - project.openshift.io
  resources:
  - projects
  verbs:
  - list
  - get

```

`` global-jnk-svc-admin-binding ``

```

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

