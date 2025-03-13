This epic explicitly references that the POC is intended to evaluate the replacement of our current solution—**splunk/splunk-connect-for-kubernetes Helm charts**—with the new Splunk OTEL Collector deployment.

---

### **Epic: POC – Evaluate Replacement of Splunk Connect for Kubernetes with Splunk OTEL Collector via Helm in an Isolated Environment**

**Description:**  
This epic covers the work required to build a proof-of-concept (POC) to evaluate replacing our current Splunk Connect for Kubernetes deployment (using splunk/splunk-connect-for-kubernetes Helm charts) with the Splunk OpenTelemetry Collector. The goal is to deploy and evaluate the OTEL Collector using Helm in our isolated (no-internet) environment, ensuring that all images and dependencies are locally available. In addition, telemetry data integrity and quality will be verified against our existing solution. Coordination with the Splunk team is essential throughout this process to align on configuration, image versioning, and telemetry output.

**Acceptance Criteria:**
- A fully functioning Splunk OTEL Collector deployment is running in the isolated environment.
- The new deployment is evaluated against the current Splunk Connect for Kubernetes solution.
- All Helm chart image references have been updated to point to local registries.
- Detailed documentation of installation steps, configuration changes, and test results is available.
- Regular coordination with the Splunk team is maintained and recorded.

---

### **Story 1: Prepare Local Repositories and Mirror Images**

**Description:**  
Set up local mirrors for all necessary container images (such as the Splunk OTEL Collector image and any related images like Fluentd, if referenced) and update the Helm chart to point to our local registry.

**Tasks:**
- Identify all image references in the Helm chart (e.g., `quay.io/signalfx/splunk-otel-collector`).
- Mirror these images to our local registry.
- Update the `values.yaml` file in the Helm chart to use local registry paths and version tags.
- Validate that the updated image references work in a test environment.

**Acceptance Criteria:**
- A local repository is created with all required images.
- The Helm chart now references local images (e.g., `myregistry.local/signalfx/splunk-otel-collector:0.120.1`).
- The images are successfully pulled in an isolated test environment.

---

### **Story 2: Update Helm Chart for Offline Deployment**

**Description:**  
Modify the Splunk OTEL Collector Helm chart to operate in an offline environment. Ensure that all dependencies, configuration files, and image references are available locally.

**Tasks:**
- Update chart dependencies to use local versions (including any subchart references).
- Remove or update any network-dependent post-install hooks.
- Configure the chart to use local paths for any additional configuration files or secrets.
- Test the rendered manifests to ensure they reference only local resources.

**Acceptance Criteria:**
- The rendered Kubernetes manifests (DaemonSet, ConfigMaps, Secrets, RBAC rules, etc.) reference only local resources.
- The offline installation process is fully documented.

---

### **Story 3: Deploy Splunk OTEL Collector POC Using Helm**

**Description:**  
Deploy the Splunk OTEL Collector using the updated Helm chart into our isolated Kubernetes cluster as part of the POC.

**Tasks:**
- Execute the Helm install command using the updated values file.
- Monitor the deployment process by checking pod readiness and verifying that no external internet connectivity is required.
- Validate that the Collector is running correctly by reviewing pod logs and probe endpoints.

**Acceptance Criteria:**
- The Splunk OTEL Collector is successfully deployed in the isolated environment.
- All pods reach a ready state without connectivity errors.
- The collector begins processing telemetry data (logs, metrics, and traces).

---

### **Story 4: Validate Telemetry Data and Collector Configuration**

**Description:**  
Ensure that the deployed collector is processing and exporting telemetry data correctly, meeting the POC requirements. Validate that the new configuration provides telemetry data that is comparable to or better than our existing Splunk Connect for Kubernetes solution.

**Tasks:**
- Configure test applications or simulate telemetry within the cluster.
- Validate the output by examining the logs, metrics, and traces collected by the new collector.
- Compare the collected telemetry data with expected results and with the data from the current solution.
- Adjust configuration files (e.g., pipeline settings in the Helm chart) as necessary.
- Create basic dashboards or queries locally to verify data integrity.

**Acceptance Criteria:**
- The telemetry data collected matches the expected format and content.
- Any discrepancies are documented, and necessary adjustments are made.
- Detailed test results are available for review and comparison with our existing solution.

---

### **Story 5: Coordinate with Splunk Team and Document POC Outcomes**

**Description:**  
Set up regular coordination sessions with the Splunk team to review progress, share configuration details, and obtain feedback on telemetry data quality and configuration best practices. Ensure the POC evaluation includes a comparison between the current Splunk Connect for Kubernetes solution and the new Splunk OTEL Collector deployment.

**Tasks:**
- Schedule regular meetings (for example, weekly check-ins) with the Splunk team.
- Prepare comprehensive POC documentation, including all configuration changes, Helm chart updates, and test results.
- Collect feedback and incorporate recommended changes.
- Finalize the POC documentation and outline next steps for a full-scale deployment, pending approval.

**Acceptance Criteria:**
- Meeting minutes and action items are documented.
- Complete POC documentation is produced, detailing configuration changes, steps taken, test results, and lessons learned.
- Approval is received from the Splunk team to proceed based on the POC outcomes.

---

### **Story 6: Research Automated Deployment Using ArgoCD**

**Description:**  
Investigate and prototype an automated deployment solution for the Splunk OTEL Collector using ArgoCD. This research will cover integrating the Helm chart into an ArgoCD application, including adapting it for our isolated environment. The study will also explore how to configure ArgoCD to reference our local registry and offline configurations.

**Tasks:**
- Evaluate ArgoCD’s capabilities for deploying Helm charts, particularly in an offline/isolated environment.
- Create a sample ArgoCD application configuration that points to our local Helm chart repository.
- Adapt the ArgoCD configuration to reference local image registries and update any network-dependent settings.
- Set up a test environment to validate the automated deployment and perform a sync operation using ArgoCD.
- Document the GitOps workflow, including the ArgoCD application specification, sync policies, and required Helm chart modifications.
- Schedule a review session with the Splunk team to present findings and obtain feedback.

**Acceptance Criteria:**
- A working ArgoCD application configuration is developed that deploys the Splunk OTEL Collector using the updated Helm chart.
- The configuration is verified in a test (isolated/offline) environment with successful sync and deployment.
- Detailed documentation of the GitOps workflow is produced.
- A review meeting is held with the Splunk team, and their feedback is documented for further refinement.

---

These stories collectively provide a clear plan for evaluating the new Splunk OTEL Collector deployment, comparing it with our existing Splunk Connect for Kubernetes solution, and eventually moving to an automated GitOps workflow using ArgoCD. Let me know if you need any further adjustments or additional details.
