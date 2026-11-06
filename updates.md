Option 2 as an epic, ready to paste into Jira. The lab POC is the first phase, and Ansible runs the sync, the registry build and the updates.

---

## EPIC: Rebuild the OpenShift mirror on oc-mirror v2 (Option 2 – Fallback)

**Summary**
Replace the deprecated oc-mirror v1 process with new mirror registries and an automated oc-mirror v2 sync pipeline run by Ansible. This lets the 9 clusters on 4.18 (7 KCS shared, 2 MFTS) update along the path 4.18.17 → 4.18.54 → 4.19.47 → 4.20.39 before 4.18 extended support ends on **25 Feb 2027**.

**Problem**
- Our v1 process breaks past 4.18, and v1 can't be retrofitted to v2. v2 needs new configs, new mirror objects and a fresh mirror.
- Today's registries can't verify images and can't be upgraded.
- From 4.21, release signatures must be mirrored too.
- Image syncs run for days and hit registry errors, so we fall behind on vulnerability fixes.

**Approach**
1. **Phase 1, Lab POC:** build a new registry server, prove v2 mirroring end to end, automate it with Ansible, and update one lab cluster.
2. **Phase 2, production registries:** roll out to the EWD and CLT registry hosts.
3. **Phase 3, fleet updates:** convert each cluster to the new mirror, then run the three-hop update.

**In scope:** new registry hosts (Lab, EWD, CLT); v2 ImageSetConfigurations; operator catalog curation; signature mirroring; IDMS/ITMS migration; update graph or explicit-update handling; Ansible automation for sync, cluster-resource apply and updates; runbooks.
**Out of scope:** app workload proxying, and Option 1 (control plane proxy).

**Epic acceptance criteria**
- [ ] The new registries are live in Lab, EWD and CLT, are supported and upgradable, and verify images.
- [ ] The oc-mirror v2 sync runs on a schedule through Ansible, is repeatable and resumable, and alerts on failure.
- [ ] Release payloads, signatures and the curated operator catalogs are mirrored for 4.18.54, 4.19.47 and 4.20.39. The config is ready for 4.21+.
- [ ] All 9 clusters use v2-generated IDMS/ITMS and catalogs, and the legacy ICSPs are removed.
- [ ] All 9 clusters are on 4.20.39 before 25 Feb 2027.
- [ ] Runbooks and handover are done.

---

### Stories

**Phase 1: Lab POC**

1. **Design and decision record.** Choose the registry product (mirror-registry/Quay, Quay or Harbor), sizing, storage, TLS/CA and auth. Choose the workflow: mirror-to-mirror through the proxy, or mirror-to-disk then disk-to-mirror.
   *AC:* design is approved and the decision is recorded.
2. **Provision the lab registry server.** Ansible role covering the VM, OS hardening, storage, certificates, registry install, robot/pull accounts and backups.
   *AC:* the registry rebuilds from Ansible alone, and image push/pull and signature verification work.
3. **Write the v2 ImageSetConfiguration.** Platform channels and the min/max versions for the upgrade path, `graph: true` if we use an update service, and the additional images.
   *AC:* the config is version-controlled and a dry run passes.
4. **Curate the operator catalogs.** Build an inventory of installed operators per cluster, then list packages and channels with their dependencies resolved.
   *AC:* all installed operators and their dependencies come through the mirror with no missing bundles.
5. **Mirror release signatures.** Validate signature mirroring and apply the generated signature ConfigMaps.
   *AC:* a lab cluster verifies the release signature with no manual workaround (this is the gate for 4.21).
6. **Automate the sync with Ansible.** Scheduled job (AAP or cron) with cache/workspace handling, retries, logging, a delete/prune workflow and failure alerts.
   *AC:* an unattended sync finishes and reruns idempotently, and a delta sync takes hours rather than days.
7. **Automate the cluster-resources apply.** Apply the generated IDMS/ITMS, CatalogSources and signature ConfigMaps through Ansible or GitOps, and migrate existing ICSPs (`oc adm migrate icsp`).
   *AC:* the lab cluster pulls everything from the new registry and the ICSPs are gone.
8. **Choose how updates find their path.** Either a local OpenShift Update Service, or explicit `--to-image` updates by digest.
   *AC:* the chosen method is documented and tested.
9. **Update the lab cluster end to end.** Use the Ansible update playbook with pre-checks, MCP pause/unpause, health gates and rollback notes, and run 4.18.17 → 4.18.54 → 4.19.47 → 4.20.39.
   *AC:* the lab reaches 4.20.39 healthy, we capture timings, and the POC sign-off goes to the go/no-go.

**Phase 2: Production registries**

10. **Build the EWD registry** from the Lab roles.
11. **Build the CLT registry** from the Lab roles.
12. **Run the initial full v2 sync** to EWD and CLT, then switch to the scheduled delta syncs.

**Phase 3: Fleet conversion and updates**

13. **Convert each cluster to the new mirror** (one window per cluster): apply the IDMS/ITMS and catalogs, remove the ICSPs, and validate operators.
14. **Update the 7 KCS shared clusters.** Stage the windows and notify tenants.
15. **Update the 2 MFTS clusters.**
16. **Decommission the v1 mirror and registries.**
17. **Write the runbooks and hand over**: sync, troubleshooting, adding operators, onboarding new versions.

---

### Dependencies and risks
- **Schedule:** the Thanksgiving and December freezes cut the available windows, so the lab POC should finish before the December freeze. The **25 Feb 2027** EUS deadline is the hard stop.
- **Spend:** new registry hosts are needed for Lab, EWD and CLT, and procurement or VM lead time may sit on the critical path.
- **Residual risk:** the sync stays, so the long-standing problems (sync failures, lag on vulnerability fixes) are reduced, not removed. Track sync duration and failure rate as epic metrics.
- **Catalog drift:** each new operator means a config change. Tie it into the namespace and operator onboarding process.
- **Proxy access:** if we choose mirror-to-mirror, the registry hosts need the Red Hat URLs allowed on the proxy.

---

Labels I'd use: `oc-mirror-v2`, `upgrade-4.20`, `kcs`, `mfts`, `ansible`. I can't create this in Jira because no Jira connection is set up in this chat. If you connect one, I can create the epic and its 17 stories directly. I can also add story points or a phase timeline that works around the freezes.
