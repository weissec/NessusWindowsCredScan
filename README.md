# Nessus Windows Credentialed Scan Preparation Script

The script automates some configuration changes necessary for Tenable Nessus to perform credentialed checks.  
More information about the necessary configuration changes and how to debug common errors can be found below.  
The script creates a backup of the original configuration/settings so that these can be reverted after the scans.

#### How to run:
Simply download the .bat files, right-click on them and select "Run as Administrator".  
To create a backup of the configuration and prepare the system for a scan, run: `Nessus-Pre-Scan.bat`  
To revert to the original configuration after a scan, run: `Nessus-Post-Scan.bat` 

#### Configuration Changes:
- Enables File and Printer Sharing
- Starts Remote Registry service
- Sets registry key for File and Printer services 
- Sets registry key for Remote and Local access
- Disables Internet Connection Firewall for local LAN or VPN connections
- Disables UAC (User Account Control)
- Enable WMI Service

## Additional Guide on Testing and Debugging Nessus Access

#### Test the IPC$ share:
```
net use \\<Target_IP>\ipc$ "" /user:""
```
Errors resolution steps:
- Ensure SMB is set up correctly
- Double-check firewall settings

#### SMB Log on Test:
```
net use \\<Target_IP>\ipc$ /user:<username> <password>
net use \\<Target_IP>\admin$ /user:<username> <password>
```
Errors resolution steps:
- Check the credentials.
- Check the account has sufficient privileges.

#### Remote Registry Test:
```
reg query \\x.x.x.x\hklm
```
Errors resolution steps:
- service must be enabled and started

#### WMI Troubleshooting and Test:
From another Windows host that can reach the scan target over the network:
- Run wbemtest from the Start Menu.
- Click 'Connect' in the upper-right corner.
- In the Namespace field, enter the target namespace as '\\target_host_ip\root\cimv2'. Thus, if the scan target is located at 10.10.0.63, enter '\\10.10.0.63\root\cimv2'.
- In the Credentials section, enter the credentials of the scanning account. Use 'domain\username' syntax in the User field.
- Click Connect in the upper-right corner.
- If successful, the wbemtest window should list the namespace as \\target_host_ip\root\cimv2. In the IWbemServices section below, a number of buttons should appear.
- Click Query... and enter the following query exactly in the popup, then click Apply: 'select DomainRole from Win32_ComputerSystem'
- A Query Result window with a single entry reading 'Win32_ComputerSystem=<no key>' should appear. Double-click that entry.
- In the Instance of Win32_ComputerSystem window, scroll down in the Properties list. A DomainRole entry should appear, with a value of 2, 3, 4 or 5.
- If the test above failed, do the following on the scan target:

Errors resolution steps:
- Ensure that the WMI service is enabled and running.
- Ensure the scan user has access to the root/CIMV2 namespace:
- Open wmimgmt.msc.
- In the left-hand panel, right-click WMI Control (Local) and choose Properties.
- Click the Security tab, expand the Root folder, and select the CIMV2 folder. Click the Security button.
- In the 'Security for ROOT/CIMV2' window, click the Advanced button.
- Confirm that the scanning account, or a group which it belongs to, is listed in this window. Click on the relevant entry and click the View button.
- Confirm that the permissions entry covering the scanning account has both the Enable Account and Remote Enable permissions set.
- Add the scanning account to the Distributed COM user group on the scan target.
- Alternatively, open Component Services (dcomcnfg) from the Start Menu.
- In the left panel, expand Component Services, then Computers, and right-click on My Computer. Select Properties.
- In the COM Security tab of the My Computer Properties window, click the Edit Limits button in the Access Permissions section. Ensure that the scanning account has all permissions.
- Repeat the previous step with the Edit Limits option under the Launch and Activation Permissions section.

#### Testing from a Linux Host
```
yum install samba-client
smbclient //<Target_IP>/IPC$ -U <username> <password>
```
Errors resolution steps:
Check the credentials.
Check that the account has sufficient privileges.

Other authentication errors might be resolved by using Kerberos authentication. Use your domain controller for the KDC on the Kerberos credential menu in the Nessus policy.

## Nessus Recommendations:
1. The Windows Management Instrumentation (WMI) service must be enabled on the target. For more information, please see: Introduction to WEBMTEST. Additionally, ensure that ports 49152 through 65535 are open between the scanner and the target, as WMI connections will choose one of these ports to target.
2. The Remote Registry service must be enabled on the target.
3. File & Printer Sharing must be enabled in the target's network configuration.
4. An SMB account must be used that has local administrator rights on the target.
Note: A domain account can be used as long as that account is a local administrator on the devices being scanned.
5. TCP ports 139 and 445 must be open between the Nessus Scanner and the target.
6. Ensure that there are no security policies are in place that blocks access to these services. This can include:
    - Windows Security Policies
    - Antivirus or Endpoint Security rules
    - IPS/IDS
7. The default administrative shares must be enabled.
      - These shares include:
        - IPC$
        - ADMIN$
        - C$
      - The setting that controls this is AutoShareServer (Windows Server) or AutoShareWks (Windows Workstation) which must be set to 1.
      - Windows 10 has the ADMIN$ disabled by default.
      - For all other operating systems, these shares are enabled by default and can cause other issues if disabled. For more information, see http://support.microsoft.com/kb/842715/en-us
