# How to Renew Active Directory LDAPS Certificates on Windows: A Step-by-Step Guide

This guide explains how to renew and import certificates and private keys for Active Directory LDAPS (Lightweight Directory Access Protocol over SSL) on Windows Domain Controllers.

## Prerequisites
- Certificate file (e.g., `certificate.crt`)
- Private key file (e.g., `datajar.key`)
- Administrative access to the Windows server
- One of the following tools for OpenSSL access:
  - GitBash for Windows (includes built-in OpenSSL)
  - OpenSSL for Windows (separate installation)
  - macOS/Linux system for PFX creation before transferring to Windows

## Step 1: Convert Certificate and Private Key to PFX Format

The Windows certificate store requires certificates with private keys to be in PFX (PKCS#12) format. Here's how to convert your files using GitBash:

### Option A: Using GitBash on Windows Server (Recommended)

1. Install GitBash on your Windows AD server if not already installed
2. Transfer your certificate and private key files to the Windows server:
   - Start an RDP session to your Windows AD server
   - Drag and drop the certificate bundle and key files directly from your local machine to a folder on the server
   - If the files came in a zipped folder, extract them after transfer
   - Create a dedicated folder for certificate operations (e.g., C:\Certificates)
3. Open GitBash terminal
4. Navigate to the directory containing your certificate and key files
5. Run the following OpenSSL command:

```
openssl pkcs12 -in certificate.crt -inkey demo-ldap.key -export -out demo-ldap.pfx
```

6. You'll be prompted to create a password for the PFX file
7. Enter and confirm your password (remember this password for the import step)

**Note**: If you have intermediate or root certificates, you can include them in the command:
```
openssl pkcs12 -in certificate.crt -inkey demo-ldap.key -certfile ca-chain.crt -export -out demo-ldap.pfx
```

### Option B: Generate PFX on macOS/Linux and Transfer

Alternatively, you can create the PFX file on your macOS or Linux system and then transfer it to the Windows server:

1. On your macOS/Linux system, open Terminal
2. Navigate to the directory containing your certificate and key files
3. Run the same OpenSSL command:
   ```
   openssl pkcs12 -in certificate.crt -inkey demo-ldap.key -export -out demo-ldap.pfx
   ```
4. Enter and confirm a password when prompted
5. Transfer the resulting PFX file to your Windows AD server:
   - Start an RDP session to the Windows AD server
   - Drag and drop the demo-ldap.pfx file from your local machine to the server
   - Place it in a dedicated folder for certificate operations

## Step 2: Import the PFX File Using MMC Console

1. Open the Microsoft Management Console:
   - Press `Win+R` to open the Run dialog
   - Type `mmc` and press Enter

2. Add the Certificates snap-in:
   - Go to File → Add/Remove Snap-in
   - Select "Certificates" and click "Add"
   - Choose "Computer account" → "Next"
   - Select "Local computer" → "Finish"
   - Click "OK" to close the Add/Remove Snap-in dialog

3. Navigate to the Personal certificate store:
   - Expand "Certificates (Local Computer)"
   - Expand "Personal"
   - Select "Certificates"

4. Import the PFX file:
   - Right-click on "Certificates" → "All Tasks" → "Import"
   - Click "Next" on the Certificate Import Wizard
   - Browse to your PFX file location and select your `demo-ldap.pfx` file
   - Click "Next"

5. Enter the PFX password:
   - Type the password you created when generating the PFX file
   - Select "Mark this key as exportable" if you might need to export it later
   - Click "Next"

6. Select certificate store:
   - Choose "Place all certificates in the following store"
   - Verify "Personal" is selected (or select the appropriate store for your LDAPS implementation)
   - Click "Next"

7. Complete the import:
   - Review the import summary
   - Click "Finish"
   - You should see a success message

## Step 3: Verify the Certificate Import

1. In the MMC console, refresh the "Certificates" folder under "Personal"
2. Look for your imported certificate
3. Verify it has a small key icon, indicating that the private key was successfully imported
4. Compare the expiration date of the new certificate with the old one to confirm it has been extended

## Step 4: Configure Active Directory to Use the New Certificate

For Active Directory LDAPS, the certificate is automatically detected if:
- It's in the computer's Personal store
- It has Server Authentication in the Enhanced Key Usage section
- The Subject Common Name (CN) matches the Domain Controller's FQDN

Since this is a renewal, you should restart the AD DS service to ensure it picks up the new certificate:
```
net stop ntds && net start ntds
```

After restarting, verify that the previous certificate is no longer in use by checking the certificate being presented on port 636.

## Step 5: Test LDAPS Connectivity and MSP Authentication

After renewing the certificate, it's critical to verify LDAPS is working correctly:

1. Test basic LDAPS connectivity:
   - Open `ldp.exe` (LDAP utility, part of AD tools)
   - Go to Connection → Connect
   - Enter your server name and port 636
   - Check "SSL" option
   - Click "OK" to test the connection

2. Verify the new certificate is being presented:
   - Use OpenSSL to check the certificate details:
   ```
   openssl s_client -connect servername:636 -showcerts
   ```
   - Confirm the expiration date matches your newly renewed certificate

3. Coordinate with the MSP team:
   - As mentioned in the Jira task, confirm that the MSP team can still authenticate after the certificate renewal
   - Have them test authentication through port 636
   - Document their confirmation for your renewal documentation

## Troubleshooting

If you encounter issues:

1. **Certificate not trusted**: 
   - Ensure the root CA certificate is in the Trusted Root Certification Authorities store

2. **Certificate has no private key**:
   - Verify the key matches the certificate (public key in certificate must correspond to private key)
   - Ensure you used the correct files in the PFX creation step

3. **Connection fails**:
   - Check certificate name matches server name
   - Verify certificate has not expired
   - Ensure proper permissions on certificate private key

4. **Port not accessible**:
   - Check firewall settings to allow port 636
   - Verify LDAPS is enabled on the server

## Alternative Method: Using Command Line Tools

Instead of the MMC console, you can import using PowerShell:
```powershell
$pfxPassword = ConvertTo-SecureString -String "YourPfxPassword" -Force -AsPlainText
Import-PfxCertificate -FilePath C:\path\to\demo-ldap.pfx -CertStoreLocation Cert:\LocalMachine\My -Password $pfxPassword
```

Or using certutil:
```
certutil -importpfx -p "YourPfxPassword" C:\path\to\demo-ldap.pfx
```

## Completing the Renewal Process

After successful renewal, verification, and MSP confirmation:

1. Document the renewal:
   - New certificate expiration date
   - Certificate thumbprint
   - Date of renewal
   - Confirmation from MSP team that authentication still works

2. Update your certificate inventory and calendar:
   - Set reminders for the next renewal (typically 30-60 days before expiration)
   - Document the renewal process for future reference

3. Clean up:
   - If necessary, remove expired certificates from the certificate store
   - Securely delete any temporary files created during the process
   - Ensure any PFX files are stored securely or deleted if no longer needed

4. Close the Jira ticket:
   - Include validation evidence
   - Document the SSL check on port 636
   - Attach confirmation from MSP team
  


   #######

   # LDAPS Certificate Renewal: Active Directory Domain Controllers

## Summary
Renew LDAPS certificate on the Active Directory domain controller and validate authentication continues to work with MSP team.

## Description
The current Active Directory LDAPS certificate is approaching expiration and needs to be renewed. This task involves importing the new certificate on the domain controller, validating the SSL configuration on port 636, and coordinating with the MSP team to ensure their authentication processes continue to work after the renewal.

**Team Responsible:** Red Dawg

**Background:**
- LDAPS (Lightweight Directory Access Protocol over SSL) certificate secures directory service communications
- Current certificate will expire soon, requiring renewal to maintain secure operations
- MSP team relies on this certificate for their authentication processes

## Level of Effort (LOE)

| Task | Notes |
|------|-------|
| **Pre-work Assessment** | |
| Document current certificate details (expiration, issuer, etc.) | Use MMC Certificate console |
| Develop renewal plan with rollback options | Include maintenance window timing |
| **Certificate Preparation** | |
| Acquire new certificate from certificate authority | Submit request to internal/external CA |
| Transfer certificate files to the domain controller | Use RDP session for secure transfer |
| **Implementation** | |
| Convert certificate and key to PFX format | Using GitBash on the server |
| Import certificate to domain controller | Use MMC or command line |
| Restart necessary services | AD DS services |
| Validation testing | Verify LDAPS connectivity |
| **MSP Coordination** | |
| Schedule testing window with MSP team | Advance notice required |
| Support MSP authentication testing | Be available during their testing |
| Document confirmation from MSP | Get written verification |
| **Documentation** | |
| Update certificate inventory | Record new expiration date |
| Document renewal process | For future reference |
| Update monitoring for new expiration date | Set alert for next renewal |

**Total Estimated LOE: Small-Medium (4-8 hours)**

## Acceptance Criteria
- [ ] New certificate successfully installed on the domain controller
- [ ] SSL check validates successfully on port 636
- [ ] MSP team confirms they can authenticate through LDAPS
- [ ] Authentication logs show no errors related to certificate issues
- [ ] Certificate inventory updated with new expiration date
- [ ] Monitoring updated for new certificate expiration date

## Dependencies
- New certificate from certificate authority
- Availability of MSP team for authentication testing
- Maintenance window for AD DS service restart

## Risk Assessment
- **Impact:** High (Authentication failure would affect multiple systems)
- **Probability:** Low (With proper planning and testing)
- **Mitigation:** Create rollback plan and perform renewal during low-traffic hours

## Attachments
- Certificate renewal procedure document
- Current certificate inventory list
- Contact information for MSP team
