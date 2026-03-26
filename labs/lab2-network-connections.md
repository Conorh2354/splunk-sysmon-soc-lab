
Detecting Network Connections with Sysmon EventCode 3 in Splunk

Objective: Use Sysmon Event ID 3 (network connection) data in Splunk to detect and alert on outbound network connections from a Windows endpoint.

Environment
Windows 10 endpoint with Sysmon installed and configured to log network connections (Event ID 3).
Sysmon logs written to Microsoft-Windows-Sysmon/Operational.
Splunk Enterprise 10.2.1 with an index named endpoint receiving Windows/Sysmon logs from the endpoint.

Step 1 – Verify recent Sysmon EventCode 3 data
In the Search & Reporting app, set the Time range to Last 15 minutes.

Run:

index=endpoint source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
| stats count by EventCode
Confirm that EventCode=3 appears with a non‑zero count, showing that recent network connection events are being ingested.

Step 2 – Inspect EventCode 3 events
Still with Last 15 minutes selected, run:


index=endpoint source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=3
| table _time User Computer Image DestinationHostname DestinationIp DestinationPort

Verify that results show:
The process Image initiating the connection (e.g. Defender, browser).
DestinationIp and optional DestinationHostname.
DestinationPort, typically 443 for HTTPS and others as applicable.

This confirms you have visibility into which processes are connecting to which remote addresses using Sysmon network events.






Step 3 – Generate test network connections
On the Windows endpoint, generate traffic to ensure EventCode 3 is produced:

Open a browser and visit several websites.
In a command prompt or PowerShell, run:
ping 8.8.8.8 -n 5
Test-NetConnection google.com -Port 443

Wait 20–60 seconds for logs to be forwarded to Splunk.
Re‑run the search from Step 2 to confirm new events appear in the selected time range.
​

Step 4 – Final detection search
Use the following SPL as the detection that will back the alert:


index=endpoint source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=3
| table _time User Computer Image DestinationHostname DestinationIp DestinationPort
This query lists all Sysmon network connection events in the period searched, including the user account, host, process, and remote endpoint details.

Step 5 – Save the search as an alert
With the search from Step 4 loaded, click Save As → Alert.

Configure Settings:

Title: Sysmon – Network Connections (EventCode 3)
Alert type: Scheduled
Run: Every hour
Time range: Last 60 minutes

In Trigger Conditions:
Trigger alert when: Number of Results
Condition: is greater than 0
Trigger: Once

This makes Splunk evaluate the last hour of Sysmon network connection data every hour and fire the alert if any EventCode 3 events exist in that window.
​




Step 6 – Add a trigger action (Log event)
In the same alert definition, under Trigger Actions, click Add Actions → Log event.

Configure the log event action:

Event text: Sysmon network connection alert fired
Index: main (or another existing index)
Leave source, sourcetype, and host at defaults.
Save the alert.

When the alert condition is met, Splunk writes a small event with that text into the selected index, providing an audit trail of when the Network Connections alert has fired.
​

Step 7 – Validate the alert
Generate additional network activity on the endpoint (browse the web or run Test-NetConnection again).

After the next hourly schedule passes, search:


index=main "Sysmon network connection alert fired"
Confirm that at least one alert event is present, proving end‑to‑end operation of Sysmon logging, Splunk ingestion, detection search, and alert action.
