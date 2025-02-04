Sample

---

## **🔹 EPIC: Automate Systemd Timers & GitHub Actions Deployment**  
**Objective:** Deploy Python-based services using systemd timers (configured as oneshot jobs) and GitHub Actions for fully automated deployments, logging, and notifications.

---

### **1️⃣ JIRA-001: Install & Validate Python Environment**  
**Description:**  
- Set up **Python3** and **pip** on the target AWS VM.  
- Install required dependencies (e.g., `google-api-python-client`).  
- Validate the installation by running a test script.  

**Tasks & Considerations:**  
- Ensure that `python3 --version` outputs a valid version.  
- Run `pip install google-api-python-client` without errors.  
- Execute a simple Python test script to confirm that the environment is working.

**Estimate:** **2 Story Points**  

**Acceptance Criteria:**  
- The system shows a valid Python3 version.  
- Dependencies install correctly via pip.  
- The test script executes without errors.

---

### **2️⃣ JIRA-002: Create Systemd Services & Timers for Each Service**  
**Description:**  
- Implement systemd **service (`.service`) and timer (`.timer`)** files for each of the following:  
  1. **Zendesk IT Glue Integration**  
     - `zendesk-it-glue.service` (configured with `Type=oneshot`)  
     - `zendesk-it-glue.timer` (includes an explicit `Unit=zendesk-it-glue.service` directive)  
  2. **IT Glue Export Backup**  
     - `it-glue-backup.service` (configured with `Type=oneshot`)  
     - `it-glue-backup.timer` (includes an explicit `Unit=it-glue-backup.service` directive)  
  3. **Jamf Pro Notifications**  
     - `jamf-pro-notifications.service` (configured with `Type=oneshot`)  
     - `jamf-pro-notifications.timer` (includes an explicit `Unit=jamf-pro-notifications.service` directive)  
  4. **Zendesk Status.io Integration**  
     - `zendesk-statusio.service` (configured with `Type=oneshot`)  
     - `zendesk-statusio.timer` (includes an explicit `Unit=zendesk-statusio.service` directive)  

- Ensure that each service logs output correctly using `StandardOutput=journal+console` and similar settings.  
- Configure a restart policy on failure if needed (or, if not, simply rely on the timer’s schedule to trigger the next run).  

**Estimate:** **5 Story Points**  

**Acceptance Criteria:**  
- All systemd service files reside in `/etc/systemd/system/` and use `Type=oneshot`.  
- Timer files include a `Unit=` option that explicitly links to the appropriate service file.  
- Each timer is enabled (`systemctl enable xyz.timer`), and the schedule is visible via `systemctl list-timers --all`.  
- Logs can be verified using `journalctl -u service-name`.

---

### **3️⃣ JIRA-003: Implement Log Rotation for Service Logs**  
**Description:**  
- Configure **logrotate** for logs located in `/usr/local/api/logs/*.log`.  
- Ensure the logrotate configuration file is placed under `/etc/logrotate.d/`.  
- Verify that log rotation occurs daily, retains only the last 7 logs, and compresses old logs.

**Estimate:** **3 Story Points**  

**Acceptance Criteria:**  
- Log files in `/usr/local/api/logs/*.log` are rotated daily.  
- The configuration retains only the last 7 logs and uses compression.  
- Running `logrotate -d /etc/logrotate.d/<your-config>` confirms that the rotation process will work as expected.

---

### **4️⃣ JIRA-004: Develop GitHub Actions Workflow for Deployment**  
**Description:**  
- Create a workflow file (e.g., `.github/workflows/deploy-systemd-timers.yml`) that automates deployment of the systemd service and timer files (with oneshot configuration and explicit Unit linking) to the AWS VM.  
- Ensure the workflow includes steps to back up current configurations before overwriting them.  
- Reload systemd on the target machine after deployment to activate new units.

**Estimate:** **5 Story Points**  

**Acceptance Criteria:**  
- The GitHub Actions workflow triggers on pushes to `main` or on version tags (e.g., `v*.*.*`).  
- The workflow successfully copies the service and timer files to `/etc/systemd/system/`.  
- A backup of previous configurations is created before deployment.  
- The deployment logs appear in GitHub Actions, and the systemd daemon is reloaded to pick up new changes.

---

### **5️⃣ JIRA-005: Verify Systemd Timer Execution & Logs**  
**Description:**  
- Manually trigger systemd timers to confirm they run as expected, given their oneshot configuration.  
- Validate that logs are correctly recorded and that the timer files include the explicit Unit option.  
- Confirm that the next execution times are correctly displayed by `systemctl list-timers --all`.

**Estimate:** **4 Story Points**  

**Acceptance Criteria:**  
- Manually starting a timer (e.g., `systemctl start it-glue-backup.timer`) triggers the associated oneshot service.  
- Logs for each service are available via `journalctl -u <service-name> -n 50`.  
- `systemctl list-timers --all` correctly shows the next execution schedule.

---

### **6️⃣ JIRA-006: Implement Deployment Notifications**  
**Description:**  
- Add notifications (e.g., via a webhook integration with Slack, Teams, or email) to inform stakeholders of deployment success or failure.  
- Utilize the systemd `OnFailure=` directive in service files to trigger a notification service when a job fails.  
- Create a templated notification service (e.g., `send_email_notification@.service`) that runs a notification script (which could use AWS SES, for example).

**Estimate:** **2 Story Points**  

**Acceptance Criteria:**  
- A deployment that succeeds triggers a “Deployment Success” notification.  
- A failed deployment triggers a “Deployment Failed” notification that includes a log excerpt.  
- The notification service uses the templated approach (e.g., `OnFailure=send_email_notification@it-glue-backup.service`), and the corresponding script is properly executed.

---

## **📌 Summary of Services & Timers**
| **Service Name**          | **Service File (`.service`)**              | **Timer File (`.timer`)**                         | **Execution Schedule**                      |
|---------------------------|--------------------------------------------|--------------------------------------------------|---------------------------------------------|
| **Zendesk IT Glue**       | `zendesk-it-glue.service` (Type=oneshot)   | `zendesk-it-glue.timer` (with `Unit=zendesk-it-glue.service`) | 15 minutes past every hour                  |
| **IT Glue Backup**        | `it-glue-backup.service` (Type=oneshot)    | `it-glue-backup.timer` (with `Unit=it-glue-backup.service`)   | Every Monday at 08:00 AM                      |
| **Jamf Pro Notifications**| `jamf-pro-notifications.service` (Type=oneshot) | `jamf-pro-notifications.timer` (with `Unit=jamf-pro-notifications.service`) | 30 minutes past every hour                  |
| **Zendesk Status.io**     | `zendesk-statusio.service` (Type=oneshot)    | `zendesk-statusio.timer` (with `Unit=zendesk-statusio.service`)   | Hourly                                      |

---

## **📌 Estimated Total LOE**
| **JIRA Story**                                        | **Estimate (Story Points)** |
|-------------------------------------------------------|-----------------------------|
| JIRA-001: Install & Validate Python Environment       | **2 SP**                    |
| JIRA-002: Create Systemd Services & Timers             | **5 SP**                    |
| JIRA-003: Implement Log Rotation                      | **3 SP**                    |
| JIRA-004: Develop GitHub Actions Workflow             | **5 SP**                    |
| JIRA-005: Verify Systemd Timer Execution & Logs       | **4 SP**                    |
| JIRA-006: Implement Deployment Notifications          | **2 SP**                    |
| **Total Estimate**                                    | **21 Story Points**         |

---

## **🚀 Next Steps & Considerations**
- **Logging & Monitoring:**  
  Confirm if additional logging or monitoring mechanisms (e.g., integration with AWS CloudWatch) are required.  
- **Rollback Mechanism:**  
  Define a rollback strategy if a deployment fails, possibly as an extension of JIRA-004.  
- **Automated Testing:**  
  Explore automated tests for service startup, timer scheduling, and log verification.  
- **Notification Service Details:**  
  Finalize the integration details (e.g., specific webhook endpoints or email configuration) for deployment notifications.

Let me know if any further refinements are needed or if additional details should be added!
