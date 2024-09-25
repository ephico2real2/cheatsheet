Here is a suggested Jira story to update the Active Directory (AD) password policy:

---

**Title:** Update Active Directory Password Policy in Sandbox Environment

**Description:**

As part of our security enhancement, we need to update the password policy in the Active Directory (AD) to align with Okta's policies and improve password security. The changes will first be implemented in the sandbox environment for testing before pushing to production.

The password policy will include the following:

- Minimum length of 15 characters.
- Must not contain part of the username.
- Must not contain the user's first name.
- Must not contain the user's last name.
- Enforce password history for the last 24 passwords.
- Minimum password age set to 1 hour.
- Lock the user after 10 unsuccessful attempts.
- Automatically unlock the account after 30 minutes.

**Acceptance Criteria:**

- [ ] Ensure the password policy is updated in the sandbox AD environment.
- [ ] Test the new password policy by creating users and validating compliance with the new rules.
- [ ] Verify the lockout behavior after 10 unsuccessful attempts and confirm automatic unlocking after 30 minutes.
- [ ] Document any issues encountered during the sandbox implementation and potential changes needed before applying to production.
- [ ] Review and confirm the password history and minimum password age settings are applied correctly.
- [ ] All changes to be reviewed and signed off before deployment to production.

**Priority:** Medium  
**Labels:** Active Directory, Security, Sandbox  

**Assignee:** (Assign yourself or the relevant team member)  
**Reporter:** (Your name)  
**Sprint:** (Add to the relevant sprint if applicable)

---

Feel free to adjust any of the details, including the priority or sprint, based on your team's workflow!
