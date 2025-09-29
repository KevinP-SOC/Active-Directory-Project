# Active Directory 2.0 Lab

## Objective
This project was built to stand up a safe, on-prem domain and practice identity management and security monitoring. The project focused on promoting a Windows Server to a domain controller with DNS, creating OUs, users, and groups, enforcing security with Group Policy (password/lockout, auditing, baseline hardening), and joining a Windows 10 workstation. Telemetry from Windows and Sysmon was ingested into Splunk to see how logons, lockouts, policy changes, and process activity appear in the SIEM and to develop practical detections around those events.



### Skills Learned
- Deeper understanding of identity and access management in Active Directory (users, groups, OUs, GPO scope).
- Proficiency interpreting Windows Security and Sysmon logs and correlating them in a SIEM (Splunk).
- Ability to recognize patterns in authentication activity (failed/successful logons, account lockouts, lateral-movement signals).
- Stronger grasp of Kerberos/NTLM, DNS dependencies, and how directory services impact security posture.
- Experience developing practical detections for AD-related events (policy changes, suspicious parent/child processes, RDP misuse).
- Improved troubleshooting, analytical reasoning, and incident triage skills in a domain environment.


### Tools Used
- **Windows Server** (Domain Controller: AD DS, DNS) and **Windows 10** (domain-joined client).
- **Group Policy Management Console (GPMC)** for OUs, GPOs, and security baselines.
- **Sysmon** (with a solid config) for deep endpoint telemetry.
- **Splunk** + **Splunk Universal Forwarder** (Windows Add-on for Sysmon/Windows) for SIEM ingestion and analysis.
- **PowerShell** for server promotion, IP/DNS settings, and validation.
- (Optional) **Slack/Email** webhook for Splunk alert notifications.

## Video Walkthrough
  *Full end-to-end video walkthrough: `[Video Link Here soon]`.*


## Steps
`[Images need to be uploaded]`
1) **Prepare Windows Server (Static IP + DNS)**
![Ref 1 – Server IP/DNS](LINK_TO_IMAGE_1)  
*Ref 1:* Set a static IPv4 and point Preferred DNS to the server’s own IP (will host DNS after promotion).

2) **Rename Server and Reboot**
![Ref 2 – Rename-Computer](LINK_TO_IMAGE_2)  
*Ref 2:* Rename the host to something readable (e.g., `DC01`), then reboot to apply.

3) **Install AD DS Role**
![Ref 3 – Add Roles/Features](LINK_TO_IMAGE_3)  
*Ref 3:* Add **Active Directory Domain Services** (DNS will be added during promotion).

4) **Promote to Domain Controller**
![Ref 4 – Promote to DC](LINK_TO_IMAGE_4)  
*Ref 4:* Create a new forest (e.g., `lab.local`). Set DSRM password and complete the wizard; the server will reboot as a DC with DNS.

5) **Verify AD + DNS Health**
![Ref 5 – AD/DNS Health](LINK_TO_IMAGE_5)  
*Ref 5:* Check **Server Manager** and **Event Viewer** for AD/DNS errors; `dcdiag`/`nslookup` basic sanity checks.

6) **Create OUs (Tiered Structure)**
![Ref 6 – OU Structure](LINK_TO_IMAGE_6)  
*Ref 6:* In **Active Directory Users and Computers**, create OUs for `Workstations`, `Servers`, `Users`, `Admins`, etc.

7) **Create Users and Groups**
![Ref 7 – Users/Groups](LINK_TO_IMAGE_7)  
*Ref 7:* Add a few test users and security groups (e.g., `Helpdesk`, `Workstation Admins`) with sensible display names and UPNs.

8) **Baseline GPO: Password + Account Lockout**
![Ref 8 – GPO Lockout Policy](LINK_TO_IMAGE_8)  
*Ref 8:* In **GPMC**, create a GPO (e.g., `SEC-AccountPolicy`) and set password/lockout thresholds (e.g., lockout after 5 invalid attempts, duration 15 min).

9) **Audit Policy + Advanced Logging**
![Ref 9 – Audit Policy](LINK_TO_IMAGE_9)  
*Ref 9:* Enable auditing for logon, account lockout, policy changes, and process creation (include CommandLine if available via GPO).

10) **Deploy Sysmon via GPO (or Manual)**
![Ref 10 – Sysmon Config](LINK_TO_IMAGE_10)  
*Ref 10:* Install Sysmon with a known-good config (process creation, network, registry). Store config on a share or deploy via script.

11) **Join Windows 10 Client to the Domain**
![Ref 11 – Domain Join](LINK_TO_IMAGE_11)  
*Ref 11:* On the client, set DNS to `DC01` → join `lab.local` → reboot.

12) **Move Computer into Workstations OU**
![Ref 12 – Move to OU](LINK_TO_IMAGE_12)  
*Ref 12:* In ADUC, move the client into the `Workstations` OU so it gets the right GPOs.

13) **Force Policy Update and Validate**
![Ref 13 – gpupdate/gpresult](LINK_TO_IMAGE_13)  
*Ref 13:* On the client: `gpupdate /force` then `gpresult /h report.html` to confirm the GPOs applied.

14) **Install Splunk (SIEM)**
![Ref 14 – Splunk Install](LINK_TO_IMAGE_14)  
*Ref 14:* Install Splunk on your SIEM host (or reuse an existing Splunk box). Create an index called `endpoint`.

15) **Install Splunk Universal Forwarder on Client + DC**
![Ref 15 – UF Install](LINK_TO_IMAGE_15)  
*Ref 15:* Install UF on both Windows 10 and DC. Configure forwarding to Splunk and enable Windows/Sysmon inputs.

16) **Install Splunk Add-ons (Windows + Sysmon)**
![Ref 16 – Splunk Add-ons](LINK_TO_IMAGE_16)  
*Ref 16:* Add the **Splunk Add-on for Microsoft Windows** and **Splunk Add-on for Sysmon** to parse and normalize fields.

17) **Verify Ingestion → `index=endpoint`**
![Ref 17 – index=endpoint](LINK_TO_IMAGE_17)  
*Ref 17:* Search `index=endpoint` and confirm events from both the client and DC (check `host`, `sourcetype`, timestamps).

18) **Test: Trigger Account Lockout**
![Ref 18 – Lockout Test](LINK_TO_IMAGE_18)  
*Ref 18:* On the client, attempt several failed logons for a test user to trigger lockout. Confirm behavior on DC.

19) **Hunt in Splunk: Lockouts + Logon Patterns**
![Ref 19 – Lockout Query](LINK_TO_IMAGE_19)  
*Ref 19:* Build searches to find lockouts, correlated logons, and potential brute force (e.g., repeated failures then success from same IP).

20) **Hunt in Splunk: Sysmon Process Trees**
![Ref 20 – Sysmon Process Tree](LINK_TO_IMAGE_20)  
*Ref 20:* Inspect EventCode 1 (process creation). Pivot on `ParentProcessName`/`ProcessGuid` to spot suspicious chains (e.g., `winword.exe` → `powershell.exe`).

21) **Alerting: Slack/Email (Optional)**
![Ref 21 – Alert Action](LINK_TO_IMAGE_21)  
*Ref 21:* Create an alert for rapid lockouts or suspicious parent→child processes. Add a webhook/email action for notifications.

22) **Document and Snapshot Baselines**
![Ref 22 – Documentation](LINK_TO_IMAGE_22)  
*Ref 22:* Export GPO reports, capture OU/GPO diagrams, and note what you’d harden next (e.g., LAPS, SMB signing, PowerShell transcript logging).

---

### Sample Splunk Searches (Optional to include)
```spl
# Account lockouts (Windows Security)
index=endpoint sourcetype=WinEventLog:Security (EventCode=4740 OR EventCode=4625)
| stats count by user, host, src_ip

# Sysmon suspicious parent-child (PowerShell spawned by Office)
index=endpoint sourcetype=XmlWinEventLog:Microsoft-Windows-Sysmon/Operational EventCode=1
| search (ParentImage="*winword.exe" OR ParentImage="*excel.exe") Image="*powershell.exe"
| table _time host User ParentImage CommandLine Image
