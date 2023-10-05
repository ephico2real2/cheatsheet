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


`` ## commands to check ``

`` oc get namespace -l demo.com/mnemonic=jnk -o jsonpath='{.items[*].metadata.name}' | xargs -I {} oc get pods -n {} ``

`` # Iterate over namespaces with the matching label and get pods in each ``

```
for ns in $(oc get namespace -l demo.com/mnemonic=jnk -o jsonpath='{.items[*].metadata.name}'); do
   oc get pods -n $ns
done

```


####




