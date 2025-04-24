# JIRA Epic and Associated Stories for Auto-Merge Implementation

## Epic: Implement Automated Main-to-Dev Branch Synchronization and Deployment

**Epic Description:**
Set up an automated system to regularly synchronize the main branch to the dev branch and implement continuous deployment for the dev environment. This will ensure that our development environment stays up-to-date with production code while enabling automated testing and validation in the dev environment.

**Epic Acceptance Criteria:**
- Automated sync from main to dev branch every 2 hours
- Notifications for sync status (success/failure/no changes)
- ArgoCD configuration for automatic deployment of dev branch
- Documentation for the entire process

### Story 1: Create GitHub Action for Main-to-Dev Branch Synchronization
**Description:**
Implement a GitHub Action workflow that automatically creates and merges a PR from the main branch to the dev branch every 2 hours. The workflow should handle various scenarios, including when branches are already in sync.

**Acceptance Criteria:**
- GitHub Action runs every 2 hours
- Workflow checks for differences before creating unnecessary PRs
- Admin token configured to bypass branch protection rules
- Manual trigger option available for on-demand execution
- Proper error handling and exit codes

**Tasks:**
- Research GitHub Actions capabilities for branch management
- Create workflow YAML file with appropriate schedule trigger
- Implement branch comparison logic
- Configure temporary branch creation for PRs
- Test workflow for different scenarios

### Story 2: Implement Notification System for Sync Status
**Description:**
Create a notification system that alerts the development team about the status of branch synchronization through Slack, including successful syncs, failures, and scenarios where no changes were needed.

**Acceptance Criteria:**
- Slack notifications for successful merges with PR links
- Notifications when branches are already in sync
- Error notifications with links to workflow logs
- Appropriate visual indicators (emojis) for quick status recognition

**Tasks:**
- Leverage the existing global `GH_SLACK_TOKEN` for Slack integration
- Use the organization's standard `slackapi/slack-github-action@v2.0.0` action
- Create notification templates for different scenarios
- Configure conditional job execution based on merge status
- Test notifications for all possible outcomes

### Story 3: Configure ArgoCD for Automatic Dev Branch Deployment
**Description:**
Update the ArgoCD configuration to automatically deploy changes from the dev branch to the development environment whenever new commits are detected on the dev branch.

**Acceptance Criteria:**
- ArgoCD monitors the dev branch for changes
- New commits on dev branch trigger automatic deployment
- Clear visualization of deployment status in ArgoCD dashboard
- Configurable sync options (automatic vs. manual)

**Tasks:**
- Review current ArgoCD setup
- Update Application manifest to target dev branch
- Configure appropriate sync policy
- Set up health checks for deployment validation
- Test end-to-end deployment from main to dev to environment

### Story 4: Create Admin Token with Branch Protection Override
**Description:**
Create a secure GitHub Personal Access Token with sufficient permissions to bypass branch protection rules for the automated PR process while adhering to security best practices.

**Acceptance Criteria:**
- PAT created with minimum required permissions
- Token stored securely as a GitHub secret
- Token able to bypass branch protection rules
- Documentation for token renewal process

**Tasks:**
- Determine minimum required permissions
- Create PAT with appropriate repository access
- Add token to repository or organization secrets
- Document token creation and renewal process

### Story 5: Create Documentation for the Automated Workflow
**Description:**
Create comprehensive documentation explaining how the automated branch synchronization and deployment process works, including troubleshooting guidelines and manual intervention steps when needed.

**Acceptance Criteria:**
- Technical documentation for workflow components
- Troubleshooting guide for common issues
- Instructions for manual triggering and monitoring
- Security considerations documented

**Tasks:**
- Document GitHub Action workflow
- Create troubleshooting decision tree
- Document ArgoCD configuration changes
- Create monitoring guide for team members

### Story 6: Implement Monitoring and Logging for Workflow Health
**Description:**
Set up appropriate monitoring and logging to track the health of the automated workflows and identify any recurring issues with the sync process.

**Acceptance Criteria:**
- Regular workflow execution metrics captured
- Log retention policy established
- Dashboard for workflow health visualization
- Alert thresholds for repeated failures

**Tasks:**
- Configure GitHub Action workflow logs retention
- Set up workflow monitoring
- Create dashboard for workflow health
- Implement alerting for multiple consecutive failures

### Story 7: Research and Implement Branch Protection Bypass for Automated Merges
**Description:**
Investigate options for allowing automated merges from branches with the pattern "auto-merge-main-to-dev*" into the dev branch while maintaining appropriate branch protection rules. Determine the most secure and maintainable approach that balances automation needs with security requirements.

**Acceptance Criteria:**
- Document available options for bypassing branch protection for automated merges
- Evaluate security implications of each approach
- Recommend and implement the most appropriate solution
- Update GitHub Actions workflow to utilize the selected approach
- Test that automated merges work correctly with the implementation
- Document the implementation for future maintenance

**Tasks:**
1. Research GitHub's branch protection exemption capabilities
2. Investigate options for pattern-based exemptions (if available)
3. Explore dedicated service account approach
4. Evaluate custom GitHub App solution if necessary
5. Document pros/cons of each approach
6. Implement the selected solution
7. Update automated workflow to use the new approach
8. Create documentation for ongoing maintenance

### Story 8: Validate Automated Playwright Tests in Dev Environment After Auto-Merge Implementation
**Description:**
Collaborate with Sid to validate that the automated Playwright tests are running correctly in the dev environment after implementing the main-to-dev auto-merge process. Ensure that the tests are triggered appropriately when changes are deployed to dev and verify that test results are accurately reported.

**Acceptance Criteria:**
- Confirm with Sid that Playwright tests are executing after automated merges to dev
- Verify that test results are being properly captured and reported
- Ensure notifications for test results are working correctly
- Document any necessary adjustments to the testing process
- Validate end-to-end flow from main branch merge to test execution

**Tasks:**
1. Schedule a working session with Sid to review the test setup
2. Run a test merge from main to dev using the new automation
3. Monitor the Playwright test execution after the merge
4. Review test reporting and notification mechanisms
5. Troubleshoot any issues found during validation
6. Document the verified workflow for future reference
7. Create runbook for handling potential test failures after auto-merges