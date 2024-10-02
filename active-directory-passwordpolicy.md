
# Active Directory Fine-Grained Password Policy Script Documentation

```
# Import Active Directory module
Import-Module ActiveDirectory

# Create Fine-Grained Password Policy (PSO)
$psoName = "EnhancedPasswordPolicy"
$precedence = 10  # Lower number means higher priority

New-ADFineGrainedPasswordPolicy -Name $psoName -Precedence $precedence `
    -MinPasswordLength 15 `
    -PasswordHistoryCount 24 `
    -MinPasswordAge "1.00:00:00" `
    -MaxPasswordAge "90.00:00:00" `
    -LockoutThreshold 10 `
    -LockoutDuration "0.00:30:00" `
    -LockoutObservationWindow "0.00:30:00" `
    -ComplexityEnabled $true `
    -ReversibleEncryptionEnabled $false

# Apply the PSO to a group (e.g., "Domain Users")
Add-ADFineGrainedPasswordPolicySubject -Identity $psoName -Subjects "Domain Users"

```

## Overview
This PowerShell script creates and applies a Fine-Grained Password Policy (FGPP) in Active Directory. It's designed to enhance password security by implementing stricter password requirements and account lockout policies.

## Script Details

### Prerequisites
- Active Directory PowerShell module
- Appropriate permissions to create and modify FGPPs in the domain

### Policy Settings
The script creates an FGPP named "EnhancedPasswordPolicy" with the following settings:

1. **Precedence**: 10 (lower number means higher priority)
2. **Minimum Password Length**: 15 characters
3. **Password History**: 24 passwords remembered
4. **Minimum Password Age**: 1 hour
5. **Maximum Password Age**: 90 days
6. **Account Lockout Threshold**: 10 invalid attempts
7. **Account Lockout Duration**: 30 minutes
8. **Lockout Observation Window**: 30 minutes
9. **Complexity Enabled**: Yes
10. **Reversible Encryption**: Disabled

### Policy Application
The policy is applied to the "Domain Users" group, affecting all standard user accounts in the domain.

## Usage
1. Run the script on a machine with the Active Directory PowerShell module installed.
2. Ensure you have the necessary permissions to create and modify FGPPs.
3. The script will create the FGPP and apply it to the Domain Users group.

## Limitations
- Does not prevent the use of usernames, first names, or last names in passwords (requires custom password filter)
- Uses default AD complexity rules, which may not align with all modern password guidelines

## Considerations
- Test thoroughly in a sandbox environment before applying to production
- Consider user education and communication about the new password requirements
- Monitor for any issues or user feedback after implementation

## Customization
To modify the policy, adjust the parameter values in the `New-ADFineGrainedPasswordPolicy` cmdlet. To apply the policy to a different group, change the "Domain Users" value in the `Add-ADFineGrainedPasswordPolicySubject` cmdlet.


# How-to Guide: Creating and Applying a Fine-Grained Password Policy in Active Directory

## Prerequisites

1. Windows Server with Active Directory Domain Services installed
2. PowerShell 5.1 or later
3. Active Directory PowerShell module installed
4. Domain Admin rights or delegated permissions to create and manage Fine-Grained Password Policies

## Step-by-Step Guide

1. **Prepare the Script**
   - Copy the entire script into a new file with a `.ps1` extension (e.g., `Create-FGPP.ps1`).
   - Save the file in a location accessible from your PowerShell environment.

2. **Review and Customize the Script**
   - Open the script in a text editor.
   - Review the following settings and adjust if necessary:
     - `$psoName`: The name of your new policy (default: "EnhancedPasswordPolicy")
     - `$precedence`: Priority of the policy (default: 10, lower number means higher priority)
     - Password policy settings (e.g., MinPasswordLength, PasswordHistoryCount, etc.)
     - The group to which the policy will be applied (default: "Domain Users")

3. **Open PowerShell as Administrator**
   - Right-click on the Start menu and select "Windows PowerShell (Admin)".

4. **Set Execution Policy** (if not already set)
   - Run the following command:
     ```powershell
     Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
     ```
   - Type 'Y' and press Enter to confirm.

5. **Navigate to Script Location**
   - Use the `cd` command to navigate to the directory where you saved the script:
     ```powershell
     cd C:\Path\To\Script\Directory
     ```

6. **Run the Script**
   - Execute the script by typing its name:
     ```powershell
     .\Create-FGPP.ps1
     ```

7. **Verify Policy Creation**
   - After the script runs, verify that the policy was created successfully:
     ```powershell
     Get-ADFineGrainedPasswordPolicy -Filter *
     ```
   - Look for your policy name in the output.

8. **Verify Policy Application**
   - Check that the policy is applied to the intended group:
     ```powershell
     Get-ADFineGrainedPasswordPolicySubject -Identity "EnhancedPasswordPolicy"
     ```
   - You should see "Domain Users" (or your specified group) in the output.

## Important Notes

- Run this script on a domain controller or a machine with the AD PowerShell module installed.
- Ensure you have proper authorization before running this script, as it creates and applies domain-wide policies.
- The script will overwrite any existing policy with the same name.
- The policy will apply to all members of the specified group (default: Domain Users).
- Changes may take time to replicate across your domain.

## Troubleshooting

- If the script fails to run, ensure the Active Directory module is installed:
  ```powershell
  Import-Module ActiveDirectory
  ```
- If you get a "cannot be loaded because running scripts is disabled on this system" error, you may need to adjust your execution policy.
- For permission errors, ensure you're running PowerShell as an administrator and have Domain Admin rights.
- If the policy doesn't seem to apply immediately, force AD replication or wait for the next replication cycle.

## Reverting Changes

To remove the policy:
1. Remove it from the group:
   ```powershell
   Remove-ADFineGrainedPasswordPolicySubject -Identity "EnhancedPasswordPolicy" -Subjects "Domain Users"
   ```
2. Delete the policy:
   ```powershell
   Remove-ADFineGrainedPasswordPolicy -Identity "EnhancedPasswordPolicy"
   ```

By following this guide, you should be able to successfully create and apply a Fine-Grained Password Policy in your Active Directory environment.
