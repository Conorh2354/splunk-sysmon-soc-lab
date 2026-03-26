Detecting PowerShell Process Creation with Sysmon and Splunk


Objective: Configure Splunk to use Sysmon logs from a Windows endpoint to detect and alert on PowerShell process creation events (Sysmon EventCode 1).

Environment
Windows 10 endpoint with Sysmon installed and logging to Microsoft-Windows-Sysmon/Operational.
Splunk Enterprise 10.2.1 with an index called endpoint receiving Windows logs via a forwarder.
Sysmon events ingested with source XmlWinEventLog:Microsoft-Windows-Sysmon/Operational.

Step 1 – Verify Sysmon data in Splunk
Open the Search & Reporting app in Splunk.

Run:

index=endpoint | stats count by source | sort -count
Confirm that XmlWinEventLog:Microsoft-Windows-Sysmon/Operational appears with events, proving Sysmon telemetry is being indexed.

Step 2 – Inspect raw Sysmon events
Narrow the search to Sysmon:

index=endpoint source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" | head 10
Review a few events to see the XML structure and extracted fields, including EventCode, Image, User, Computer, and CommandLine.

Step 3 – Generate test PowerShell activity
On the Windows endpoint, open PowerShell.

Run some harmless commands, for example Get-Process or Get-Service, to generate process creation events.

Wait 30–60 seconds for events to reach Splunk.



Step 4 – Build the detection search
Back in Splunk, run:

index=endpoint source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=1 Image="*\\powershell.exe"
| stats count by User, Computer, CommandLine

Confirm that at least one row appears showing:

Your Windows User
The Computer (host)
The CommandLine for powershell.exe

This search is now a basic detection for PowerShell process creation using Sysmon EventCode 1.

Step 5 – Save the search as an alert
With the search still open, click Save As → Alert.

Configure:
Title: Sysmon – PowerShell Process Creation
Permissions: Private
Alert type: Scheduled
Run: Every hour
Time range: Last 60 minutes
Under Trigger Conditions:
Trigger alert when: Number of Results
Condition: is greater than 0
Trigger: Once

This makes Splunk evaluate the last hour of Sysmon data every hour and trigger the alert if any matching PowerShell executions are found.
​







Step 6 – Add a trigger action (Log event)
In the same alert window, under Trigger Actions, click Add Actions → Log event.

Set:
Event text: PowerShell process creation detected by Sysmon alert
Index: main (or another existing index)
Leave source, sourcetype, and host at defaults.
Save the alert.

Whenever the search condition is met, Splunk writes a small alert event into the chosen index, so you can search for, and audit when the detection fired.
​

Step 7 – Validate the alert
Generate another PowerShell command on the endpoint.

After the next scheduled run (up to one hour), search in Splunk:

index=main "PowerShell process creation detected by Sysmon alert"
Confirm that at least one alert event exists, proving the detection and alert pipeline works end‑to‑end.
​


