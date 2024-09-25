### Full How-To Guide for Helm Chart with RoleBindings Based on Users

This guide will walk you through creating a Helm chart that generates Kubernetes `RoleBinding` resources based on user accounts and their associated roles in multiple namespaces.

### 1. **Create the Helm Chart Structure**

First, create the directory structure for your Helm chart. Run the following commands:

```bash
mkdir user-rolebinding-chart
cd user-rolebinding-chart
mkdir templates
touch values.yaml Chart.yaml templates/rolebinding.yaml
```

### 2. **Create the `Chart.yaml` File**

The `Chart.yaml` file contains metadata about your Helm chart. Add the following content:

```yaml
apiVersion: v2
name: user-rolebinding-chart
description: A Helm chart to create RoleBindings for users across multiple namespaces.
version: 1.0.0
```

### 3. **Create the `values.yaml` File**

The `values.yaml` file contains default values for your Helm chart. This is where you define the users and their respective role bindings.

```yaml
# List of OIM accounts and their respective role bindings
oimAccountName:
  - name: test1
    roleBindings:
      - roleName: edit
        namespace: demo-lab
      - roleName: view
        namespace: dsp-rnd
      - roleName: admin
        namespace: dev
  - name: test2
    roleBindings:
      - roleName: view
        namespace: qa
      - roleName: edit
        namespace: staging
```

### 4. **Create the `templates/rolebinding.yaml` File**

The `rolebinding.yaml` file is the main template that will generate the `RoleBinding` resources based on the users and role bindings defined in `values.yaml`.

```yaml
{{- range $user := .Values.oimAccountName }}
  {{- range $binding := $user.roleBindings }}
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: {{ $user.name }}-{{ $binding.roleName }}-binding
  namespace: {{ $binding.namespace }}
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: {{ $binding.roleName }}
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: {{ $user.name }}
  {{- end }}
{{- end }}
```

### 5. **How the Chart Works**

- The Helm chart uses a nested loop to iterate through each user in `oimAccountName` and through each `roleBinding` associated with the user.
- For each user and role binding, a Kubernetes `RoleBinding` resource is created.
- The `name` of the `RoleBinding` is generated by combining the user's name and the role name (e.g., `test1-edit-binding`).

### 6. **Deploy the Chart**

To deploy the chart, first ensure that you are in the directory of the Helm chart (`user-rolebinding-chart`), then run the following command:

```bash
helm install my-release .
```

This command will deploy the `RoleBinding` resources based on the default `values.yaml` file. The `RoleBinding`s will be created for the users and namespaces defined there.

### 7. **Customizing the Chart**

If you need to customize the chart, such as adding more users, changing roles, or applying the bindings to different namespaces, you can either:

- Edit the `values.yaml` file directly, or
- Pass custom values using the `--set` flag during installation.

#### Example of custom values using `--set`:

```bash
helm install my-release . --set oimAccountName[0].name=johndoe --set oimAccountName[0].roleBindings[0].roleName=view --set oimAccountName[0].roleBindings[0].namespace=dev
```

This command overrides the first user and their role bindings, changing them to a new user (`johndoe`) with a `view` role in the `dev` namespace.

### 8. **Upgrading the Chart**

If you want to make changes to the `values.yaml` or other parts of the chart after it’s already installed, you can upgrade the release using:

```bash
helm upgrade my-release .
```

This will apply any changes made to the Helm chart or values and update the resources in Kubernetes accordingly.

### 9. **Uninstalling the Chart**

To remove all resources created by the Helm chart, use the following command:

```bash
helm uninstall my-release
```

This will delete all `RoleBinding`s created by the chart for the specified users and roles.

### 10. **Verifying the Resources**

After installing or upgrading the chart, you can verify that the `RoleBinding`s were created successfully by running:

```bash
kubectl get rolebindings -n <namespace>
```

Replace `<namespace>` with the namespace you want to check. This will list all the `RoleBinding`s in that namespace.

For example, to check the `demo-lab` namespace:

```bash
kubectl get rolebindings -n demo-lab
```

### 11. **Additional Notes**

- **Multiple Releases**: If you want to deploy different sets of users with different role bindings in separate Helm releases, you can install the chart multiple times using different release names, like so:

  ```bash
  helm install first-release .
  helm install second-release .
  ```

- **Advanced Customization**: If you anticipate needing more advanced configuration options, such as varying the `RoleBinding` name format per user or applying different policies, you can extend the chart with additional values and logic in the templates.

---

### Conclusion

With this Helm chart, you can efficiently manage and automate the creation of Kubernetes `RoleBinding`s for users across different namespaces with various roles. The flexibility provided by Helm allows you to easily customize users, roles, and namespaces by adjusting the `values.yaml` or using Helm’s `--set` flag.