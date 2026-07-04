# GreenboneLauncher
Bootstrap Windows and setup and launch greenbone scanner

## Windows Vulnerability Scanning Orchestration Script
This dual-mode script automates the configuration of a Windows host to prepare it for vulnerability scanning (via Nessus or OpenVAS) and completely rolls back those changes to secure the system once scanning is complete. It bridges host-level Windows configuration with containerized OpenVAS deployment inside Windows Subsystem for Linux (WSL).
## 🚢 Overview
Vulnerability scanners require highly privileged access, unencrypted local traffic paths, and specific authentication protocols to perform deep host-level configuration audits. Leaving these configurations permanently active creates severe security vulnerabilities.
This tool provides a one-click approach:

* ON mode: Spins up OpenVAS in WSL and opens the Windows host to be safely scanned by disabling UAC filters, configuring WinRM, opening firewalls, and enabling a dedicated scanning account.
* OFF mode: Closes all temporary security gaps, disables the scanning account, tears down WinRM endpoints, hardens the registry, stops container environments, and offers a compressed WSL system state backup.

------------------------------
## ⚡ Quick Start## Prerequisites

* Windows 10/11 or Windows Server (Run with Administrator privileges)
* WSL installed with a target distro (Default: Ubuntu)
* Docker and Docker Compose installed inside the WSL Linux environment

## Execution

   1. Save the script payload as a .bat file (e.g., ScanManager.bat).
   2. Double-click the file or run it via an elevated command prompt.
   3. The script will automatically self-elevate to Administrator mode and prompt you for an action.

------------------------------
## 🛠️ Configuration Variable Variables
Before running, modify these top-level script variables within the text editor to match your environment:

| Variable | Default Value | Description |
|---|---|---|
| $scanneruser | "scanneruser" | The dedicated local Windows user profile targeted by scanners. |
| $winrmrule | "windows remote management (http-in)" | The name of the built-in Windows firewall rule to toggle. |
| $wsldistro | "ubuntu" | The exact distribution string found via wsl -l. |
| $linuxuser | "user" | Your primary Linux non-root username inside the WSL distro. |
| $openvasdir | "/home/$linuxuser/greenbone" | Directory context hosting the Greenbone compose.yaml stack. |
| $backuppath | "c:\users\wsl_openvas_backup.tar.gz" | Destination path for system imaging. |

------------------------------
## ⚙️ Operation Workflow## 🟢 ON Mode (Preparation for Scans)
When you input on, the script executes the following sequence:

[+] Starting Services & Unlocking...
 ├── Enable scanning user account ($scanneruser)
 ├── Set Remote Registry startup to Automatic and start service
 ├── Initialize WinRM / PowerShell Remoting listeners
 ├── Configure WinRM Auth (Allow Basic, Allow Unencrypted HTTP traffic)
 ├── Bypass UAC Remote restrictions (LocalAccountTokenFilterPolicy = 1)
 ├── Open port rules in Windows Defender Firewall
 └── Wake WSL ➔ Pull latest Greenbone/OpenVAS Compose YAML ➔ Start Containers


* Final Action: Automatically fires up the native web browser to point directly to the OpenVAS localized deployment dashboard (http://127.0.0.1:9392).

## 🔴 OFF Mode (Post-Scan Hardening)
When you input off, the script prompts for a optional backup and locks down the host:

[-] Initiating Lockdown...
 ├── Terminate running docker containers gracefully
 ├── Shut down the WSL subsystem (`wsl --shutdown` to free RAM)
 ├── Optional: Compress and export full tarball backup of WSL state
 ├── Disable scanning user account
 ├── Stop and disable the Remote Registry service
 ├── Purge the LocalAccountTokenFilterPolicy registry item
 └── Re-harden WinRM Client/Service (Disable Basic, Block Unencrypted, clear TrustedHosts)

------------------------------
## 🔒 Security Implications Reference
This script intentionally creates the following temporary configurations during ON mode:

* Unencrypted WinRM: WinRM traffic uses Basic auth over HTTP. Ensure this tool is only utilized inside safe, isolated testing perimeters or air-gapped laboratory settings.
* LocalAccountTokenFilterPolicy: Setting this key to 1 allows members of the local Administrators group to retain full privileges during remote admin queries instead of being stripped by User Account Control (UAC).

The OFF routine aggressively targets these changes to completely close potential pivot vectors.
------------------------------
If you need any adjustments made to this structure, or would like to add custom troubleshooting or installation instructions to the placeholder spaces, let me know!

