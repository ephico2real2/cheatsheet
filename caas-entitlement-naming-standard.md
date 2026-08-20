# CaaS OpenShift Entitlement Naming Standard

**Status:** Draft
**Owner:** KCS OpenShift Platform Team
**Mnemonic:** OCP

## Purpose

Defines the naming grammar and description format for OIM entitlement groups
that grant RBAC access to Container as a Service (CaaS) OpenShift clusters.
Group names must remain accurate independent of who holds them, and
descriptions must give an approver enough information to make a decision
without consulting the platform team.

## Grammar

```
OCP.Container as a Service Cluster <Access Level> User (<Env>)
```

| Segment | Values | Notes |
| --- | --- | --- |
| Mnemonic | `OCP` | Fixed. The only all-caps token permitted. |
| Service | `Container as a Service` | Spelled out. `CAAS` is not a mnemonic and may not be abbreviated here. |
| Access Level | `View`, `Edit`, `Administrator` | Names the RBAC role granted, not the job title of the holder. |
| Env | `Prod`, `UAT`, `QA`, `RND` | One group per environment. Parentheses are reserved for this segment. |

Longest resulting name is 60 characters, well inside the 100-character
guidance.

### Rules

1. **Only the mnemonic is capitalized.** `OCP` stays; `CAAS` expands to
   `Container as a Service`.
2. **Name the access, not the person.** `Cluster Edit User`, not
   `Cluster Support Engineer`. A DBA or an SRE outside the platform team may
   legitimately need edit; a job-title name makes the group lie and makes
   access reviews unauditable.
3. **No abbreviations in the display name.** Spell out `Administrator`, not
   `Admin`.
4. **Period is the only special character.** Parentheses only for the
   environment. No dashes.
5. **Descriptions may use `CaaS`,** since it is spelled out in the display
   name. Descriptions must be complete sentences ending in a period, and must
   not contain a percent sign or an apostrophe.
6. **Descriptions must name the literal ClusterRole** (view, edit,
   cluster-admin) and the environment. Descriptions must not be duplicated
   across environments.

## Entitlements

| Group Name | Description |
| --- | --- |
| OCP.Container as a Service Cluster View User (Prod) | Grants the OpenShift view role on CaaS Production clusters to CaaS team members. Read only access. Requires OCP approval and business justification. |
| OCP.Container as a Service Cluster View User (UAT) | Grants the OpenShift view role on CaaS UAT clusters to CaaS team members. Read only access. |
| OCP.Container as a Service Cluster View User (QA) | Grants the OpenShift view role on CaaS QA clusters to CaaS team members. Read only access. |
| OCP.Container as a Service Cluster View User (RND) | Grants the OpenShift view role on CaaS RND clusters to CaaS team members. Read only access. |
| OCP.Container as a Service Cluster Edit User (Prod) | Grants the OpenShift edit role on CaaS Production clusters to CaaS team members for workload support. Requires OCP approval and business justification. |
| OCP.Container as a Service Cluster Edit User (UAT) | Grants the OpenShift edit role on CaaS UAT clusters to CaaS team members for workload support. |
| OCP.Container as a Service Cluster Edit User (QA) | Grants the OpenShift edit role on CaaS QA clusters to CaaS team members for workload support. |
| OCP.Container as a Service Cluster Edit User (RND) | Grants the OpenShift edit role on CaaS RND clusters to CaaS team members for workload support. |
| OCP.Container as a Service Cluster Administrator User (Prod) | Grants the OpenShift cluster-admin role on CaaS Production clusters to CaaS team members. This is a privileged, unrestricted entitlement. Requires OCP approval, business justification, and management approval. |
| OCP.Container as a Service Cluster Administrator User (UAT) | Grants the OpenShift cluster-admin role on CaaS UAT clusters to CaaS team members. This is a privileged, unrestricted entitlement. Requires OCP approval and business justification. |
| OCP.Container as a Service Cluster Administrator User (QA) | Grants the OpenShift cluster-admin role on CaaS QA clusters to CaaS team members. This is a privileged, unrestricted entitlement. Requires OCP approval and business justification. |
| OCP.Container as a Service Cluster Administrator User (RND) | Grants the OpenShift cluster-admin role on CaaS RND clusters to CaaS team members. This is a privileged, unrestricted entitlement. |

All twelve entitlements are scoped to CaaS team members. If a consumer outside
the CaaS team requires cluster access, request a separate entitlement rather
than widening these.

## Privileged Flag (Column K)

| Access Level | Column K |
| --- | --- |
| View | N |
| Edit | N |
| Administrator | Y |

Confirm against the Terminology worksheet definition before submitting. If
that definition treats any write access as privileged, Edit becomes `Y` as
well.

## Approval Matrix

| Access Level | RND | QA | UAT | Prod |
| --- | --- | --- | --- | --- |
| View | Manager | Manager | Manager | OCP + justification |
| Edit | Manager | Manager | Manager | OCP + justification |
| Administrator | Manager | OCP + justification | OCP + justification | OCP + justification + management |

Gating increases monotonically with both privilege and environment. A
read-only entitlement must never require more approval than an administrative
one in the same environment.

## Open Items

- [ ] **Confirm cluster-admin vs. admin.** The Administrator rows assume
      cluster-scoped `cluster-admin`. If the intent is namespace-scoped
      `admin`, update all four descriptions. This is the one detail an
      approver cannot infer and must not guess.
- [ ] **Confirm the privileged definition** on the Terminology worksheet
      before setting column K for Edit.
- [ ] **Decide on per-cluster scoping.** As written, one Prod group grants
      access to every Prod cluster in the fleet. If per-cluster scoping is
      plausible, reserve the slot now:
      `OCP.Container as a Service <Cluster> Cluster Edit User (Prod)`. Adding
      a segment after a hundred groups exist means re-cutting all of them.

## Rejected Alternatives

| Rejected | Reason |
| --- | --- |
| `OCP.OpenShift Cluster Edit User (Prod)` | Substitutes rather than expands. OCP already stands for OpenShift Container Platform, so the platform is named twice and the CaaS service identity is lost. |
| `OCP.CAAS Cluster Edit User (Prod)` | CAAS is not the mnemonic, so it may not appear in all caps or unexpanded. |
| `Cluster Support Engineer (Env)` | Job title, not access level. Breaks when non-team members hold the entitlement. |
| `Cluster Audit User (Env)` | Audit implies a compliance function; the grant is plain view. Renamed to Cluster View User. |
| `Cluster Admin (Env)` | Abbreviated, and ambiguous between admin and cluster-admin, which differ by orders of magnitude in blast radius. |
