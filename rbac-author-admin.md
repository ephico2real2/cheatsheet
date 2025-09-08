There are two options:

---

Option A — ClusterRole + RoleBinding (scoped to one namespace)

Create a ClusterRole that grants RBAC authoring verbs, then bind it in the target namespace to your OpenShift user.

# ClusterRole: can create/manage Roles and RoleBindings
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: rbac-author-admin
rules:
- apiGroups: ["rbac.authorization.k8s.io"]
  resources: ["roles", "rolebindings"]
  verbs: ["create","get","list","watch","update","patch","delete"]

Bind it only in connectedcargo to the user (replace the username below):

# RoleBinding: limit the ClusterRole's power to the connectedcargo namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: rbac-author-admin
  namespace: connectedcargo
subjects:
- kind: User                # or Group/ServiceAccount as needed
  name: alice@example.com   # <-- your OpenShift username
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: rbac-author-admin
  apiGroup: rbac.authorization.k8s.io

Why this works: A ClusterRole is reusable and can be bound with a namespaced RoleBinding to restrict its effect to just that namespace.


---

Option B — (Often simpler) Role + RoleBinding (namespaced only)

If you only ever need this in connectedcargo, you can skip the ClusterRole and just use a namespaced Role.

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
# RoleBinding: grant to the user
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: rbac-author-admin
  namespace: connectedcargo
subjects:
- kind: User
  name: alice@example.com   # <-- your OpenShift username
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: rbac-author-admin
  apiGroup: rbac.authorization.k8s.io


---

Minimal vs. full verbs

If you only want to allow creation, change verbs to ["create"].

In practice, teams usually need update/patch/delete to manage/fix RBAC they create, so I included the full set.


Binding to Groups instead of Users

If you manage access via LDAP/Groups, switch the subject:

subjects:
- kind: Group
  name: app-ocp-rbac-<team>-ns-admin   # for example
  apiGroup: rbac.authorization.k8s.io
