GitHub/Markdown viewers.


---

🔹 Option A — ClusterRole + RoleBinding (scoped to connectedcargo)

# ClusterRole: can create/manage Roles and RoleBindings
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: rbac-author-admin
rules:
- apiGroups: ["rbac.authorization.k8s.io"]
  resources: ["roles", "rolebindings"]
  verbs: ["create","get","list","watch","update","patch","delete"]
---
# RoleBinding: limit the ClusterRole's power to the connectedcargo namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: rbac-author-admin
  namespace: connectedcargo
subjects:
- kind: User                # Or Group/ServiceAccount if needed
  name: alice@example.com   # <-- replace with your OpenShift username
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: rbac-author-admin
  apiGroup: rbac.authorization.k8s.io


---

🔹 Option B — Role + RoleBinding (namespace-scoped only)

# Role: namespace-scoped capability to manage Roles/RoleBindings
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: rbac-author-admin
  namespace: connectedcargo
rules:
- apiGroups: ["rbac.authorization.k8s.io"]
  resources: ["roles", "rolebindings"]
  verbs: ["create","get","list","watch","update","patch","delete"]
---
# RoleBinding: grant Role to the user
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: rbac-author-admin
  namespace: connectedcargo
subjects:
- kind: User
  name: alice@example.com   # <-- replace with your OpenShift username
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: rbac-author-admin
  apiGroup: rbac.authorization.k8s.io


---

🔧 Tips

Minimal verbs: if you only want “create”, set verbs: ["create"].

Groups instead of Users:

subjects:
- kind: Group
  name: app-ocp-rbac-team-ns-admin
  apiGroup: rbac.authorization.k8s.io



---

workflow — a reusable ClusterRole (Option A) or a strictly namespace-only Role (Option B)?

