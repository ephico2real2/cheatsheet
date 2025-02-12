Each story is focused on a specific documentation or configuration task, with clear deliverables and acceptance criteria.

---

### **JIRA-007: Create Confluence Documentation for Systemd Scripts & Timer Setup**

**Description:**  
Document the systemd configuration and technical decisions made for our IT Glue & integration services. This Confluence page should cover:
- The design and configuration of the systemd service files (using `Type=oneshot`) and timer files (with an explicit `Unit=` directive).
- Rationale for using these options (e.g., why we chose oneshot for one-time execution and the benefits of explicit linking).
- Logging setup details (e.g., use of `StandardOutput=journal+console`) and troubleshooting tips (e.g., checking logs with `journalctl`).
- Code snippets and examples from our IT Glue Export Backup, Zendesk IT Glue, Jamf Pro Notifications, and Zendesk Status.io integrations.

In addition, update the Git README file to include a prominent link to the Confluence page so that all team members can easily reference our documentation.

**Tasks:**  
- Compile all configuration details, decisions, and examples into a single Confluence page.  
- Include screenshots or excerpts of the systemd service and timer files.  
- Add troubleshooting steps and links to official systemd documentation.  
- Update the Git README with the link to the Confluence documentation.

**Estimate:** **3 Story Points**

**Acceptance Criteria:**  
- A comprehensive Confluence page exists that documents the systemd service and timer setup (including the use of `Type=oneshot` and explicit `Unit=` directives).  
- The documentation explains all technical decisions, includes code snippets, and links to official references.  
- The Git README is updated with a working, clearly visible link to the Confluence page.  
- The documentation is reviewed and approved by the team.

---

### **JIRA-008: Install and Configure AWS CloudWatch Agent for Journald Log Shipping**

**Description:**  
Install and configure the AWS CloudWatch Agent on the target EC2 instance to capture logs from `journald`/`journalctl` and ship them to AWS CloudWatch. This task is necessary to ensure that all output from our systemd services is centrally collected and monitored.

**Tasks:**  
- Install the AWS CloudWatch Agent on the EC2 instance.  
- Create or update the CloudWatch Agent configuration file to capture logs from the journal (for example, targeting `/var/log/journal` or using the journald output).  
- Start the agent and verify that logs from our systemd services (including our IT Glue Export Backup and other integration services) are visible in AWS CloudWatch.  
- Document the installation and configuration process in our internal documentation.

**Estimate:** **3 Story Points**

**Acceptance Criteria:**  
- The AWS CloudWatch Agent is installed and running on the EC2 instance.  
- The agent configuration successfully captures logs from `journald`/`journalctl` and ships them to AWS CloudWatch.  
- Verification steps (such as log entries in CloudWatch) are documented and confirmed.  
- The installation and configuration process is added to our internal documentation (or linked in the Confluence page referenced in JIRA-007).

---

### **JIRA-009: Configure AWS CloudWatch Alarms for Systemd Script Failure Alerts**

**Description:**  
Leverage AWS CloudWatch Logs to capture systemd service failure events (recorded via `journald`/`journalctl`) and configure CloudWatch Alarms to notify stakeholders when these failures occur. This approach removes the need for custom shell scripts on the EC2 instance to monitor systemd failures.

**Tasks:**  
- **Identify Log Patterns:**  
  Analyze journald logs to determine the key patterns or messages that indicate a systemd service failure (e.g., "failed", "exit code", etc.).

- **Create a Metric Filter:**  
  Set up a CloudWatch Logs metric filter that scans the systemd logs for these failure events.

- **Set Up CloudWatch Alarm:**  
  Configure a CloudWatch Alarm that uses the metric filter to trigger notifications (via SNS or another notification service) when the failure count exceeds a defined threshold.

- **Test and Validate:**  
  Simulate failure events to ensure that the metric filter detects them and the CloudWatch Alarm triggers the appropriate notifications.

- **Document the Process:**  
  Update the Confluence page with detailed instructions on how the metric filter and alarm are configured. Include any relevant code snippets, configuration files, and testing procedures. Ensure that the Git README is updated with a link to this documentation.

**Estimate:** **3 Story Points**

**Acceptance Criteria:**  
- **Log Capture:** AWS CloudWatch Logs are successfully capturing systemd failure events from journald logs.  
- **Metric Filter:** A metric filter is in place that accurately identifies failure events based on the defined log patterns.  
- **Alarm Configuration:** A CloudWatch Alarm is configured to trigger when the metric filter detects a threshold of failure events.  
- **Notifications:** Notifications (via SNS or an equivalent service) are sent upon alarm triggering.  
- **Documentation:** The configuration process and testing instructions are documented on Confluence, with a link updated in the Git README.

---

### **JIRA-010: Scheduled OS Update via Appleboy SSH Action with Conditional Trigger**

**Description:**  
Implement a GitHub Actions workflow (`.github/workflows/update-os.yml`) that remotely updates the EC2 OS using `appleboy/ssh-action@master`. The workflow should:

- Use existing SSH variables (`SSH_HOST`, `SSH_USERNAME`, `SSH_PRIVATE_KEY`) from `deploy.yaml`.
- Run the OS update commands:
  ```bash
  export NEEDRESTART_MODE=a
  export DEBIAN_FRONTEND=noninteractive
  export DEBIAN_PRIORITY=critical
  sudo -E apt-get -qy clean
  sudo -E apt-get -qy update
  sudo -E apt-get -qy -o "Dpkg::Options::=--force-confdef" -o "Dpkg::Options::=--force-confold" upgrade
  ```
- Trigger on a schedule (e.g., daily at 3 AM UTC) and via manual dispatch (`workflow_dispatch`).
- **Also trigger conditionally** when an environment variable (e.g., `RUN_OS_UPDATE`) is set to `true`.
- Update the Confluence documentation and add a link in the Git README.

**Tasks:**  
- Create the `update-os.yml` workflow with scheduled, manual, and conditional triggers (using an `if: ${{ env.RUN_OS_UPDATE == 'true' }}` condition).
- Integrate the SSH action using the existing SSH variables.
- Verify that the OS update commands execute correctly on the remote EC2 instance.
- Update internal documentation (Confluence) and link it in the Git README.

**Acceptance Criteria:**  
- The workflow file exists and correctly triggers on schedule, manual dispatch, and when `RUN_OS_UPDATE` is set.
- The remote EC2 instance is updated via SSH with the proper commands.
- Documentation is updated and linked in the Git README.

---
