# Nessus Windows Credentialed Scan Preparation Script

Windows script to deploy the necessary configuration changes to allow Nessus authenticated scans.
The script automates some configuration changes necessary for Tenable Nessus to perform credentialed checks.

More information about the necessary configuration changes can be found here: https://docs.tenable.com/nessus/Content/CredentialedChecksOnWindows.htm

The script creates a backup of the original configuration/settings so that these can be reverted after the scans.

#### How to run:
Simply download the .bat files, right-click on them and select "Run as Administrator".
To create a backup of the configuration and prepare the system for a scan, run: Nessus-Pre-Scan.bat
To revert to the original configuration after a scan, run: Nessus-Post-Scan.bat

#### Requirements: 
The script must be run with administrative privileges.

#### Configuration Changes:
- Enables File and Printer Sharing
- Starts Remote Registry service
- Sets registry key for File and Printer services 
- Sets registry key for Remote and Local access
- Disables Internet Connection Firewall for local LAN or VPN connections
- Disables UAC (User Account Control)
- Enable WMI Service
