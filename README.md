Enterprise Active Directory Infrastructure Lab

Target Role: IT Support / Junior System Administrator (Germany)
Core Technologies: Windows Server 2022, Active Directory (AD DS), DNS, SMB/NTFS, Group Policy (GPO), RBAC, Disaster Recovery, PowerShell.
Project Overview

A comprehensive simulation of a corporate IT infrastructure built from scratch. Designed to demonstrate hands-on experience with Microsoft Windows Server ecosystems, enterprise networking principles, and administrative automation required in modern enterprise environments.
Architecture & Environment

    Hypervisor: VMware Workstation Pro (Host: Dell XPS 9570, x86 architecture for native Windows Server compatibility)
    Network Topology: Isolated Host-Only Network (VMnet1: 192.168.19.0/24) to simulate a secure, enclosed corporate LAN.
    Nodes deployed:
        DC01 (192.168.19.10): Windows Server 2022 Standard (Active Directory Domain Controller, DNS Server, File Server)
        CLIENT01 (192.168.19.20): Windows 11 Enterprise (Domain-joined endpoint)

Key Implementations
1. Core Identity & Access Management (RBAC)

Deployed a new AD forest (corp.local) enforcing strict Role-Based Access Control (RBAC).

    Organizational Units (OUs): Established a hierarchical structure separating Users, Groups, Computers, and Servers.
    Security Groups: Created departmental (HR, IT) and functional (File_Read, File_Modify) Global Security Groups.
    Zero Direct Assignments: All user permissions are granted exclusively through group memberships to ensure scalability and auditability.

2. File Services & Storage Security

Configured a centralized file server featuring hidden administrative shares (Public$, HR$, IT$, UserProfiles$).

    Implemented a secure access model utilizing an enterprise combination of Share Permissions and strict NTFS Security.
    Access Verification Matrix:

Role	Public$	HR$	IT$
Standard User	Read	Deny	Deny
HR Specialist	Read/Write	Read/Write	Deny
IT Admin	Read/Write	Read/Write	Read/Write
3. Desktop Automation via Group Policy (GPO)

Automated endpoint configuration to provide a seamless user experience upon login.

    Drive Mapping (GPO_Map_Network_Drives): Automatically maps network drives (P:, H:, I:) dynamically based on the user's AD group membership utilizing Item-Level Targeting.
    Folder Redirection (GPO_Folder_Redirection): Redirected user Desktop and Documents folders to standard network shares (\\DC01\UserProfiles$\%username%) to centralize data storage, protect against endpoint failure, and streamline backups.

4. Disaster Recovery & Business Continuity

Tested critical administrative recovery procedures to ensure data resilience.

    Object Recovery: Enabled and successfully tested the Active Directory Recycle Bin (via Active Directory Administrative Center) to instantly restore accidentally deleted user objects without downtime.
    System State Restore via DSRM:
        Provisioned a dedicated backup drive. Bypassed SAN Policy blocks (Offline by Policy, Read-only) using the diskpart CLI utility.
        Booted the Domain Controller into Directory Services Restore Mode (DSRM) and successfully restored the AD database using the wbadmin start systemstaterecovery command.

5. PowerShell Automation

Developed scripts to move away from GUI-based (ADUC) manual administration.

    Built a bulk provisioning script utilizing Import-Csv to automatically read new employee data, generate AD accounts, set initial parameters, and map users to appropriate Security Groups.

Technical Deep Dive: Troubleshooting Scenarios

A critical part of this lab was solving real-world infrastructure problems by analyzing symptoms and system behavior:
Scenario A: Stale DNS Records & Caching Issues

The Symptom: After adjusting the DC's static IP to the final production subnet (192.168.19.10), nslookup attempts returned timeouts, despite ICMP ping succeeding perfectly. The Fix:

    Purged stale A-records mapped to the old IP via the DNS Manager GUI.
    Flushed the local resolver cache (ipconfig /flushdns).
    Forced a manual reregistration of the DC's DNS records (ipconfig /registerdns).
    Restarted the netlogon service to rebuild critical AD SRV records.

Scenario B: Folder Redirection "Access Denied" Errors

The Symptom: Folder Redirection GPOs applied successfully (verified via gpresult), but user folders failed to create on the server, generating "Access Denied" events. The Fix:

    Simplified the Share permissions of UserProfiles$ to Everyone -> Full Control.
    Delegated security enforcement entirely to NTFS ACLs (SYSTEM Full Control, CREATOR OWNER Full Control).

Key Lessons Learned

    Infrastructure Begins With Network Design: Ad-hoc IP assignment leads to cascading failures in DNS and AD replication. Establishing the isolated 192.168.19.0/24 subnet and verifying routing before promoting the Domain Controller is mandatory.
    NTFS vs. Share Permissions Boundary: The traditional rule of "Broad Share, Restrictive NTFS" is not merely a best practice; it is a functional necessity for system-level services like Folder Redirection that must dynamically modify DACLs and object ownership.
    CLI Precision During Outages: Operating in recovery environments like DSRM requires strict adherence to command-line syntax.
