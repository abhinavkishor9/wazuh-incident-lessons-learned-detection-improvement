# Lab 146 – Incident Lessons-Learned and Detection Improvement

## Overview

This lab focuses on conducting a post-incident lessons-learned review and identifying opportunities to improve SOC detection capabilities using Wazuh and Sysmon.

A simulated PowerShell incident scenario is used to assess existing process-creation telemetry, review detection context, document potential gaps, and perform controlled positive and negative validation.

The exercise emphasizes evidence-based detection engineering rather than immediately creating new rules.

## Lab Environment

| Component | Details |
|---|---|
| Endpoint | DESKTOP-9MMM37V |
| Operating System | Windows 11 Pro |
| SIEM | Wazuh |
| Endpoint Telemetry | Sysmon |
| Wazuh Agent ID | 001 |
| Agent IP | 192.168.174.1 |
| Event Under Review | Sysmon Event ID 1 – Process Creation |
| PowerShell | PowerShell 7.7 and Windows PowerShell 5.1 |
| Incident ID | IR-146-001 |

## Objectives

- Conduct a structured lessons-learned review of a simulated security incident.
- Establish a baseline for PowerShell process-creation telemetry.
- Verify whether Sysmon events are collected and indexed in Wazuh.
- Examine process command lines, parent processes, user context, and timestamps.
- Review WMI-launched PowerShell activity as an additional investigation example.
- Perform controlled positive and negative PowerShell tests.
- Identify detection and alert-context improvement opportunities.
- Document findings, troubleshooting observations, and outstanding validation tasks.

## Scenario

The SOC is conducting a post-incident review following a simulated suspicious PowerShell activity investigation on a Windows endpoint.

The review examines whether existing telemetry provides sufficient process context for investigation and whether detection coverage can be validated through controlled testing.

A previously observed WMI-related PowerShell execution event is also reviewed to understand the value of parent-child process relationships and execution context.

## Activities Performed

1. Created a dedicated Lab 146 workspace.
2. Prepared a simulated incident summary.
3. Documented initial lessons learned.
4. Created a detection baseline.
5. Created a harmless PowerShell test script.
6. Encountered a PowerShell execution-policy restriction when attempting direct script execution.
7. Executed the harmless test content using `Invoke-Expression` without changing execution policy.
8. Reviewed Sysmon Event ID 1 records in Wazuh archives.
9. Documented a WMI-related PowerShell process event.
10. Created a detection improvement tracker.
11. Executed a controlled positive test and an ordinary PowerShell negative test.
12. Prepared a detection quality assessment for further validation.

## Key Findings

### 1. Sysmon Process-Creation Visibility

Sysmon Event ID 1 records were observed in the Wazuh archives index:

`wazuh-archives-4.x-2026.11.26`

The reviewed records contained:

- Process image and command line
- Parent process image and command line
- Process and parent process identifiers
- Process GUIDs
- User and parent user
- Integrity level
- File hashes
- Event timestamps

### 2. WMI-Related PowerShell Execution

A process-creation event showed:

- Process: `powershell.exe`
- Parent: `WmiPrvSE.exe`
- User: `NT AUTHORITY\SYSTEM`
- Command line included `-ExecutionPolicy Bypass`
- Payload path: `C:\WMIPermanentEventLab\Payload\wmi-payload.ps1`

This demonstrates the value of reviewing process ancestry and execution context. The event alone does not establish malicious activity.

### 3. Controlled Positive Test

The harmless Lab 146 test produced the expected output:

`LAB146_DETECTION_TEST`

The corresponding Sysmon Event ID 1 record was found in Wazuh archives, and its command line contained the test marker.

### 4. Negative Test

The ordinary PowerShell command produced:

`LAB146_NORMAL_ACTIVITY`

The command was executed successfully. Its corresponding Wazuh event and alert status remain to be independently verified.

## Troubleshooting

Direct execution of the test script was blocked by the existing PowerShell execution policy.

The test content was subsequently executed using `Invoke-Expression`. No system-wide or user-level execution-policy changes were made.

## Detection Improvement Tracker

The following improvement areas were documented:

- PowerShell detection coverage validation
- Alert context assessment
- Repeatable detection testing
- False-positive evaluation

See `detection-improvements/improvement-tracker.csv`.

## Validation Status

| Validation Area | Status |
|---|---|
| Sysmon Event ID 1 collection | Confirmed |
| Wazuh archive indexing | Confirmed |
| Process command-line visibility | Confirmed |
| Parent process visibility | Confirmed |
| User context visibility | Confirmed |
| Positive test marker visibility | Confirmed |
| Negative test execution | Confirmed |
| Negative test Wazuh event | Pending verification |
| Wazuh alert generation | Pending verification |
| Detection rule effectiveness | Not established |
| Custom detection rule implementation | Not performed |

## Conclusion

The lab confirmed that Sysmon process-creation telemetry is being collected and indexed in Wazuh archives, with useful process and parent-process context available.

The controlled positive test demonstrated that a recognizable PowerShell command-line marker can be observed in the collected telemetry.

However, archive visibility does not establish that a Wazuh alert was generated or that a detection rule provides adequate coverage. Alert-index verification, negative-test event review, and detection-quality assessment remain outstanding.

The main lesson is to validate telemetry, detection logic, alert quality, and false-positive behavior separately before implementing detection changes.

