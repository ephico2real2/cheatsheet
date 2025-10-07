Perfect 👍 — let’s put it **all together** into a structured **OpenShift Administrator Interview Playbook**.
This will give you a **full list of questions with strong answers** (what a good candidate should say) and **red flags** (what a weak candidate might say).

---

# 📝 OpenShift Administrator Interview Playbook

---

## 🔹 1. OpenShift vs Kubernetes Basics

**Q:** What are the main differences between Kubernetes and OpenShift from an administrator’s standpoint?
✅ **Strong Answer:**

* OpenShift includes enterprise features out of the box (OAuth, SCCs, Routes, integrated monitoring/logging).
* Uses `oc` CLI in addition to `kubectl`.
* OperatorHub and OLM handle Operators.
* Adds security defaults (restricted SCC, image policies).
  ❌ **Red Flags:** “They are the same” / Only talks about Pods/Deployments.

---

## 🔹 2. Cluster Upgrades

**Q:** How do you check and manage cluster upgrades?
✅ **Strong Answer:**

* Use `oc adm upgrade` to check available versions and trigger upgrades.
* `oc get clusterversion` shows status.
* Can `--pause` / `--resume` upgrades if needed.
  ❌ **Red Flags:** “kubectl doesn’t do upgrades” without knowing OpenShift has upgrade tooling.

---

## 🔹 3. Node Maintenance: Drain & Cordon

**Q:** How do you prepare a node for maintenance?
✅ **Strong Answer:**

* `oc adm cordon <node>` to prevent new scheduling.
* `oc adm drain <node> --ignore-daemonsets --delete-emptydir-data` to evict workloads.
* `oc adm uncordon <node>` to allow scheduling again.
  ❌ **Red Flags:** Deletes the node directly, doesn’t mention cordon vs drain difference.

---

## 🔹 4. MachineSet & MachineConfig

**Q:** What is the difference between a MachineSet and a MachineConfig?
✅ **Strong Answer:**

* **MachineSet** = defines/scales worker nodes (like a Deployment for nodes).
* **MachineConfig** = applies OS/CRI-O/kernel configs to nodes (managed by MachineConfigPools).
* Scaling vs Configuration.
  ❌ **Red Flags:** Treats them as the same thing, or only talks about deployments/pods.

---

## 🔹 5. KubeletConfig

**Q:** How do you tune kubelet parameters across nodes?
✅ **Strong Answer:**

* Create a `KubeletConfig` CR targeting a `MachineConfigPool`.
* Example: adjusting eviction thresholds, `maxPods`, or CPU manager policy.
  ❌ **Red Flags:** “Just edit the kubelet on each node manually.”

---

## 🔹 6. Security Context Constraints (SCC)

**Q:** What are SCCs in OpenShift?
✅ **Strong Answer:**

* Pod-level security policies that define what containers can/cannot do (privileged, hostNetwork, volumes, SELinux contexts).
* Built-in ones: `restricted`, `anyuid`, `privileged`.
* Example: `oc adm policy add-scc-to-user privileged -z sa-name -n project`.
  ❌ **Red Flags:** Confuses SCCs with Kubernetes PSP or can’t name any SCC.

---

## 🔹 7. Logging & Observability (Loki, must-gather)

**Q:** How do you gather logs for troubleshooting in OpenShift?
✅ **Strong Answer:**

* `oc adm must-gather` for full cluster dump.
* `oc adm inspect` for specific operators/resources.
* `oc logs <pod>` and `oc logs -f <pod>` for streaming.
* LokiStack Operator + `ClusterLogForwarder` for central logging.
  ❌ **Red Flags:** Only knows `kubectl logs`.

---

## 🔹 8. Debugging

**Q:** How do you debug a failing pod or node?
✅ **Strong Answer:**

* `oc debug pod/<name>` to get a shell into a debug container.
* `oc debug node/<node>` for node-level troubleshooting.
* `oc rsh` and `oc exec` for container-level access.
  ❌ **Red Flags:** Only mentions `kubectl exec`.

---

## 🔹 9. RBAC & Policies

**Q:** How do you test and grant permissions in OpenShift?
✅ **Strong Answer:**

* Grant: `oc adm policy add-role-to-user admin user1 -n project`.
* Check: `oc auth can-i create pods --as user1 -n project`.
* Cluster-wide: `oc auth can-i --list --as user1`.
  ❌ **Red Flags:** Doesn’t know about impersonation or `--as`.

---

## 🔹 10. Impersonation

**Q:** How do you impersonate another user to test RBAC?
✅ **Strong Answer:**

* `oc get pods --as <user>` or `oc auth can-i get pods --as <user>`.
* Can also impersonate groups with `--as-group`.
  ❌ **Red Flags:** Suggests logging in as the user manually.

---

## 🔹 11. OAuth & SSO

**Q:** How does OpenShift integrate with external IdPs?
✅ **Strong Answer:**

* Via OAuth CR (`oc get oauth cluster -o yaml`).
* Supports LDAP, GitHub, GitLab, Azure AD, Keycloak, etc.
* Logs available in `openshift-authentication` namespace.
  ❌ **Red Flags:** Only mentions kubeconfig or static users.

---

## 🔹 12. Routes & Routers

**Q:** What is a Route in OpenShift?
✅ **Strong Answer:**

* An OpenShift resource that exposes a Service externally via the OpenShift Router (HAProxy).
* TLS options: `edge`, `passthrough`, `reencrypt`.
* Create with `oc expose service <svc>`.
  ❌ **Red Flags:** Thinks it’s just a Kubernetes Ingress.

---

## 🔹 13. Day-2 Operations

**Scenario Qs**

* A PVC is stuck in `Pending` with ODF storage. What do you check?
  👉 StorageClass, CSI driver logs in `openshift-storage`, PVC events.
* Logs aren’t appearing in Loki. What do you check?
  👉 `ClusterLogForwarder`, `ClusterLogging` CR, LokiStack pods.
* User can’t log in. What do you check?
  👉 `oc get oauth`, OAuth pod logs, impersonation test.

---

# ✅ Quick Reference Command List

* Upgrade: `oc adm upgrade`
* Logs: `oc logs`, `oc adm must-gather`, `oc adm inspect`
* Debug: `oc debug pod/node/deployment`
* Nodes: `oc adm cordon|uncordon|drain`
* Scaling: `oc scale machineset …`
* RBAC: `oc adm policy add-role-to-user`, `oc auth can-i`
* Routes: `oc expose service`
* SCCs: `oc get scc`, `oc adm policy add-scc-to-user`

---

👉 This playbook ensures you cover **setup, security, upgrades, logging, debugging, nodes, storage, routes, and RBAC** — everything that makes OpenShift *different* from plain Kubernetes.

Do you want me to also **make a scoring sheet** (e.g., 0–2 weak, 3–5 okay, 6–8 strong) so you can quickly rate candidates during interviews?


Excellent — let’s **expand this OpenShift Admin Interview Playbook** with **even more advanced questions and scenarios** so you can really separate out candidates who have only “used” OpenShift vs those who have **run and operated clusters in production**.

---

# 📝 Expanded OpenShift Administrator Interview Playbook

---

## 🔹 14. Operators & OLM

**Q:** What role does the Operator Lifecycle Manager (OLM) play in OpenShift?
✅ **Strong Answer:**

* Manages installation, upgrades, and lifecycle of Operators.
* OperatorHub provides catalog of Operators (Red Hat Certified, Community, etc).
* OLM handles CRDs, subscriptions, and automatic updates.
  ❌ **Red Flags:** Doesn’t know what OLM is, confuses Operators with Helm charts only.

---

## 🔹 15. Cluster Monitoring

**Q:** What monitoring tools are built into OpenShift?
✅ **Strong Answer:**

* Prometheus and Alertmanager deployed by default.
* Grafana dashboards available for cluster and project metrics.
* Metrics exposed via `oc get --raw /apis/metrics.k8s.io`.
* Custom metrics can be forwarded via ServiceMonitors/PodMonitors.
  ❌ **Red Flags:** Only mentions external monitoring like Datadog/NewRelic.

---

## 🔹 16. Resource Management

**Q:** How do you enforce resource limits in OpenShift?
✅ **Strong Answer:**

* `ResourceQuota` objects for namespace-wide limits.
* `LimitRange` for default container requests/limits.
* Cluster admins can enforce quota via templates.
  ❌ **Red Flags:** “Just tell developers to set CPU and memory.”

---

## 🔹 17. Quotas & Expiration

**Q:** How would you manage temporary namespaces that expire?
✅ **Strong Answer:**

* Use metadata labels/annotations (`expiration-date`) + automation/Kyverno to enforce cleanup.
* ResourceQuotas applied to limit abuse.
  ❌ **Red Flags:** Suggests deleting projects manually.

---

## 🔹 18. Networking

**Q:** What CNI plugins does OpenShift support?
✅ **Strong Answer:**

* OpenShiftSDN (deprecated), OVN-Kubernetes (default), Multus for secondary networks.
* OVN provides egress IPs, network policies.
* Multus allows attaching multiple networks to pods.
  ❌ **Red Flags:** Doesn’t know Multus or thinks CNI is Calico by default.

**Q:** How do you set egress IPs in OpenShift?
✅ **Strong Answer:**

* Assign via `egressIPs` in namespace annotation or via EgressIP CR when using OVN-Kubernetes.
  ❌ **Red Flags:** Suggests using NodePort/LoadBalancer directly.

---

## 🔹 19. Cluster Logging Advanced

**Q:** How do you forward logs to multiple backends (e.g., Loki + Splunk)?
✅ **Strong Answer:**

* Configure `ClusterLogForwarder` with multiple outputs.
* Each output can be Kafka, Loki, Elasticsearch, Splunk HEC, syslog.
  ❌ **Red Flags:** “Just run Fluentd sidecars.”

---

## 🔹 20. Must-Gather & Inspect

**Q:** What’s the difference between `oc adm must-gather` and `oc adm inspect`?
✅ **Strong Answer:**

* `must-gather` = full cluster state for Red Hat Support.
* `inspect` = targeted diagnostics for specific resources/operators.
  ❌ **Red Flags:** Doesn’t know must-gather is the first thing Red Hat asks for.

---

## 🔹 21. Cluster Roles vs Namespaced Roles

**Q:** What’s the difference between Role/RoleBinding and ClusterRole/ClusterRoleBinding?
✅ **Strong Answer:**

* **Role** = namespace-scoped permissions.
* **ClusterRole** = cluster-wide or reusable across namespaces.
* Bind with `RoleBinding` (namespace) or `ClusterRoleBinding` (global).
  ❌ **Red Flags:** Says they’re the same or confuses ClusterRole with cluster-admin.

---

## 🔹 22. Infra & Node Placement

**Q:** How do you run infra workloads on specific nodes?
✅ **Strong Answer:**

* Label nodes (`node-role.kubernetes.io/infra`).
* Taint them, then add tolerations to infra pods.
* Example: Routers, Monitoring, Logging often placed on infra nodes.
  ❌ **Red Flags:** Doesn’t mention taints/tolerations.

---

## 🔹 23. Disaster Recovery

**Q:** How would you back up and restore an OpenShift cluster?
✅ **Strong Answer:**

* ETCD backup via `oc adm cluster-backup`.
* Restores via `oc adm cluster-restore`.
* Application data via Velero, ODF snapshots, or CSI drivers.
  ❌ **Red Flags:** Suggests just snapshotting VMs without mentioning etcd.

---

## 🔹 24. Image Management

**Q:** How does OpenShift manage images differently from Kubernetes?
✅ **Strong Answer:**

* Uses **ImageStreams** for version tracking.
* Supports internal registry (`image-registry` operator).
* Policies: `ImageContentSourcePolicy` and `CatalogSource` for mirroring.
  ❌ **Red Flags:** Only mentions DockerHub.

---

## 🔹 25. Advanced Scenarios

* **Q:** A MachineConfig update is stuck. What do you do?
  ✅ Check MCP status (`oc get mcp`), review machine-config-daemon logs, see if nodes are rebooting.
* **Q:** An SCC is blocking a pod. How do you fix it?
  ✅ Use `oc get events`, inspect SCC assignments, grant SA an appropriate SCC (`oc adm policy add-scc-to-user`).
* **Q:** Cluster upgrade stuck. What do you check?
  ✅ `oc get clusterversion`, operator conditions, run `oc adm must-gather`.

---

# ✅ Expanded Quick Command Cheat Sheet

* **Upgrade:** `oc adm upgrade`
* **Debug:** `oc debug pod/<pod>` / `oc debug node/<node>`
* **Logs:** `oc logs`, `oc adm must-gather`, `oc adm inspect`
* **Drain/Cordon:** `oc adm drain node`, `oc adm cordon node`
* **Scale Machines:** `oc scale machineset <name> -n openshift-machine-api --replicas=N`
* **RBAC:** `oc adm policy add-role-to-user`, `oc auth can-i`
* **SCCs:** `oc get scc`, `oc adm policy add-scc-to-user`
* **Routes:** `oc expose service <svc>`
* **Cluster Backup:** `oc adm cluster-backup`
* **MCP/Kubelet:** `oc get mcp`, `oc get kubeletconfig`

---

✅ With this **expanded playbook**, you can now cover **core operations, upgrades, nodes, SCCs, logging, networking, Operators, DR, ImageStreams, quotas, and advanced debugging**.

Would you like me to now **convert this into a one-page scoring matrix** (question → strong answer → weak answer → score 1–5) so you can use it quickly in live interviews?


Perfect 👌 — let’s **push this playbook even deeper** into the **real OpenShift admin trenches**: advanced day-2 ops, cluster security, performance, networking, multi-cluster, and tricky scenarios Red Hat or a customer would expect an experienced admin to handle.

---

# 📝 Deep-Dive OpenShift Administrator Interview Playbook (Extended)

---

## 🔹 26. Cluster Security

**Q:** What tools does OpenShift provide for cluster security enforcement?
✅ **Strong Answer:**

* **SCCs** (Security Context Constraints) for pod security.
* **NetworkPolicies** (default deny, namespace-scoped).
* **Image policies**: `ImageContentSourcePolicy`, signature verification, Quay integration.
* **Audit logs** in API server.
  ❌ **Red Flags:** Only mentions Kubernetes RBAC and ignores SCCs/Image policies.

---

## 🔹 27. Network Policy & Isolation

**Q:** How do you enforce that only certain namespaces can talk to each other?
✅ **Strong Answer:**

* Apply `NetworkPolicy` to namespaces (`allow-from`, `deny-all`).
* OpenShift Project Config Operator can enforce default policies.
* OVN-Kubernetes supports fine-grained egress control.
  ❌ **Red Flags:** “Use firewalls” or “We don’t restrict traffic.”

---

## 🔹 28. Router Sharding

**Q:** What is Router Sharding in OpenShift?
✅ **Strong Answer:**

* Deploy multiple IngressControllers (HAProxy routers).
* Each handles Routes for specific namespaces, domains, or labels.
* Example: one router for internal traffic, one for external.
  ❌ **Red Flags:** Doesn’t know multiple routers are possible.

---

## 🔹 29. Performance & Tuning

**Q:** How do you troubleshoot cluster performance issues?
✅ **Strong Answer:**

* Use `oc adm top nodes/pods` for resource usage.
* Inspect cluster operators (`oc get co`).
* Review etcd health (`oc exec -n openshift-etcd etcd-pod -- etcdctl endpoint status`).
* Tune kubelet via `KubeletConfig`.
  ❌ **Red Flags:** Only suggests checking `kubectl get pods`.

---

## 🔹 30. ETCD Operations

**Q:** How do you back up and restore etcd in OpenShift?
✅ **Strong Answer:**

* Backup: `oc adm cluster-backup <dir>`.
* Restore: `oc adm cluster-restore <dir>`.
* Know etcd is the “single source of truth” for cluster state.
  ❌ **Red Flags:** Doesn’t mention etcd at all.

---

## 🔹 31. Cluster Autoscaling

**Q:** How does OpenShift handle autoscaling of nodes and workloads?
✅ **Strong Answer:**

* Workloads: Horizontal Pod Autoscaler (HPA), Vertical Pod Autoscaler (VPA), KEDA for event-driven.
* Nodes: Cluster Autoscaler via MachineSets.
* Example: MachineAutoscaler objects adjust replicas of a MachineSet.
  ❌ **Red Flags:** Confuses pod scaling with node scaling.

---

## 🔹 32. Multi-Cluster (ACM / GitOps)

**Q:** How would you manage multiple OpenShift clusters?
✅ **Strong Answer:**

* Red Hat Advanced Cluster Management (ACM) for policy, cluster lifecycle, governance.
* ArgoCD / OpenShift GitOps for app deployments across clusters.
* Uses placement rules, ApplicationSets.
  ❌ **Red Flags:** Says “just use kubeconfig contexts.”

---

## 🔹 33. Registry & Image Security

**Q:** How does OpenShift secure image pulls and registries?
✅ **Strong Answer:**

* Integrated registry (`image-registry` operator).
* Pull secrets configured at namespace or SA level.
* Supports ImageContentSourcePolicy for mirrored registries (air-gapped).
* Quay as enterprise registry with Clair security scanning.
  ❌ **Red Flags:** Mentions only DockerHub with no OpenShift details.

---

## 🔹 34. Logging Advanced (Rate Limiting / Loki)

**Q:** What happens if Loki returns `429 Too Many Requests`?
✅ **Strong Answer:**

* Caused by ingestion throttling.
* Fix by tuning LokiStack operator values (ingester limits, rate limits).
* Adjust ClusterLogForwarder to buffer/reduce logs.
  ❌ **Red Flags:** Doesn’t know Loki can be tuned, blames “network issues.”

---

## 🔹 35. Admission Controllers & Policies

**Q:** How do you enforce governance in OpenShift?
✅ **Strong Answer:**

* Built-in: LimitRanges, ResourceQuotas.
* Policy engines: Kyverno, Gatekeeper (OPA).
* Cluster admins often enforce labels, expiration dates, or restrict images.
  ❌ **Red Flags:** Only mentions RBAC.

---

## 🔹 36. Disaster Recovery & HA

**Q:** How do you recover from a complete control plane node failure?
✅ **Strong Answer:**

* Replace failed master via MachineSet.
* Restore etcd snapshot if quorum is lost.
* Rely on highly available load balancer and 3+ master design.
  ❌ **Red Flags:** Suggests rebooting nodes only.

---

## 🔹 37. Cluster Logging Forwarding

**Q:** How do you configure logs to go to Splunk and Loki at the same time?
✅ **Strong Answer:**

* Define multiple outputs in `ClusterLogForwarder`.
* One `output` for LokiStack, another for Splunk HEC.
  ❌ **Red Flags:** “Install Fluentd manually.”

---

## 🔹 38. Special Commands

* **Impersonation:**

  * `oc get pods --as user1`
  * `oc auth can-i list pods --as user1 --as-group devs`
* **Policies:**

  * `oc adm policy add-role-to-user admin alice -n dev`
* **Inspect Operators:**

  * `oc adm inspect clusteroperator/kube-apiserver`
* **Node Debug:**

  * `oc debug node/<node>` → drops into host root shell.

---

## 🔹 39. Common Scenarios (Ask “What would you do if…”)

* **MachineConfig stuck updating:**
  👉 Check MCP conditions, machine-config-daemon logs, node reboot.
* **App team needs hostNetwork:**
  👉 Grant SCC (privileged or custom).
* **Namespace deletion stuck in Terminating:**
  👉 `oc adm inspect ns <ns>`, check finalizers, force remove.
* **Cluster upgrade fails:**
  👉 `oc adm upgrade`, check clusterversion, run must-gather.

---

# ✅ Extended Quick Command Cheat Sheet

* **Upgrades:** `oc adm upgrade --to=X.Y.Z`
* **Must-gather:** `oc adm must-gather`
* **Debug:** `oc debug pod/foo` / `oc debug node/bar`
* **Drain/Cordon:** `oc adm cordon`, `oc adm drain --ignore-daemonsets`
* **Cluster Backup/Restore:** `oc adm cluster-backup`, `oc adm cluster-restore`
* **RBAC:** `oc auth can-i`, `oc adm policy add-role-to-user`
* **Networking:** `oc get ingresscontroller -n openshift-ingress-operator`
* **Logging:** `oc get clusterlogforwarder -n openshift-logging`
* **Storage:** `oc get sc`, `oc get pvc -n ns`

---

✅ With these **extra topics** — Operators, Monitoring, Quotas, Router Sharding, Autoscaling, Multi-Cluster, Registry Security, Disaster Recovery, and real failure scenarios — you now have a **comprehensive, 360° OpenShift admin interview guide**.

Would you like me to **package this into a scoring rubric** (e.g. 1–5 scale per category with “expected depth” notes), so you can quickly grade a candidate while interviewing?
