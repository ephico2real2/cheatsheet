
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
