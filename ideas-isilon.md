Here are two Jira stories based on your requirements:

---

### **Jira Story 1: Install Dell Cert-CSI Tools and Configure for Disconnected Cluster**

**Summary:**  
Install and configure Dell Cert-CSI tools for CSM 2.6.1 in a disconnected OpenShift 4.14 environment using local registry images.

**Description:**  
We need to install the Dell Cert-CSI tools for CSM Isilon drivers on the existing OpenShift 4.14 cluster. Since this is a disconnected cluster, the tool will need to be configured to pull images from the local registry instead of external sources.

**Acceptance Criteria:**
- Dell Cert-CSI tools installed successfully in the OpenShift 4.14 environment.
- The Cert-CSI tool is configured to pull all required images from the local registry due to the disconnected nature of the cluster.
- Validation that the tool configuration works as expected and is able to run in the disconnected environment.

**Tasks:**
1. Download Dell Cert-CSI tool binaries and dependencies.
2. Modify configuration to point to the local image registry.
3. Install the Cert-CSI tool in OpenShift 4.14.
4. Validate the installation by checking tool logs and performing a test run.
5. Document the steps and configurations for future reference.

---

### **Jira Story 2: Run Dell Cert-CSI Tool in LAB 2 and Export Report**

**Summary:**  
Run the Dell Cert-CSI tool in the LAB 2 OpenShift environment and export the certification report to the MFTS team.

**Description:**  
We need to run the Dell Cert-CSI tool on the existing LAB 2 OpenShift environment and generate a report for the MFTS team. The report should detail the certification status and any issues found during the certification process.

**Acceptance Criteria:**
- Dell Cert-CSI tool is executed in LAB 2 OpenShift environment without errors.
- Certification report is generated and exported successfully.
- Report is shared with the XXXX team for review.

**Tasks:**
1. Set up the LAB 2 environment to run the Dell Cert-CSI tool.
2. Execute the Cert-CSI tool to perform certification checks.
3. Capture and export the certification report.
4. Share the report with the MFTS team.
5. Document the findings and any issues identified during the process.

---

