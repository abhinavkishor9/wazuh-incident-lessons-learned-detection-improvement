
# Lab 146 – Investigation Notes

## 1. Investigation Details

| Field | Value |
|---|---|
| Lab ID | 146 |
| Incident ID | IR-146-001 |
| Investigation Type | Incident Lessons-Learned Review |
| Endpoint | DESKTOP-9MMM37V |
| SIEM | Wazuh |
| Telemetry Source | Sysmon |
| Primary Event | Event ID 1 – Process Creation |
| Review Date | November 26, 2026 |
| Status | Detection validation in progress |

## 2. Investigation Purpose

The purpose of this review was to assess whether existing endpoint telemetry provides sufficient information to investigate PowerShell process activity and identify potential improvements to SOC detection and validation practices.

The exercise used a simulated incident scenario and controlled PowerShell activity. It did not represent an actual security incident.

## 3. Detection Baseline

The following questions were used to assess the existing telemetry:

1. Are PowerShell processes recorded?
2. Is the command line available?
3. Is the parent process available?
4. Is the user available?
5. Are alerts generated for the activity?
6. Does the activity require manual investigation?

### Baseline Findings

Sysmon Event ID 1 records were found in:

`wazuh-archives-4.x-2026.11.26`

The reviewed events contained process image, command line, parent process, user, process identifiers, integrity level, hashes, and timestamps.

Telemetry collection and archive indexing were confirmed. Alert generation remains unverified.

## 4. Observed Event 1 – WMI-Related PowerShell Execution

### Event Details

| Field | Observed Value |
|---|---|
| Wazuh timestamp | Nov 26, 2026, 07:32:57.942 |
| Event ID | 1 |
| Provider | Microsoft-Windows-Sysmon |
| Process | powershell.exe |
| Parent process | WmiPrvSE.exe |
| Process ID | 28692 |
| Parent Process ID | 8972 |
| User | NT AUTHORITY\SYSTEM |
| Parent User | NT AUTHORITY\SYSTEM |
| Integrity Level | System |

### Command Line

```text
powershell.exe -NoProfile -ExecutionPolicy Bypass -File "C:\WMIPermanentEventLab\Payload\wmi-payload.ps1"
```

### Parent Process

```text
C:\Windows\System32\wbem\WmiPrvSE.exe -Embedding
```

### Investigation Observations

- PowerShell was launched by the WMI Provider Host process.
- The process ran under the SYSTEM account.
- The command line contained the `-ExecutionPolicy Bypass` argument.
- The command line referenced a payload under the WMI Permanent Event lab directory.
- Sysmon recorded the process-creation event and Wazuh indexed it in archives.

### Assessment

This event provides useful evidence for investigating WMI-launched PowerShell activity.

The parent-child relationship, execution identity, command line, and payload path are relevant investigation pivots.

However, the event alone does not establish malicious execution. The associated WMI permanent event configuration and payload would need to be reviewed to establish the activity's purpose and context.

## 5. Observed Event 2 – Lab 146 Script-File Execution Attempt

### Event Details

| Field | Observed Value |
|---|---|
| Wazuh timestamp | Nov 26, 2026, 07:45:54.889 |
| Event ID | 1 |
| Process | powershell.exe |
| Parent process | pwsh.exe |
| User | DESKTOP-9MMM37V\Dell |
| Command line | `-NoProfile -File C:\DFIR-Lab146\detection-tests\lab146-test.ps1` |

### Investigation Observations

- The test script existed and contained two harmless `Write-Output` commands.
- Direct execution was attempted using `powershell.exe -NoProfile -File`.
- PowerShell blocked script execution because of the existing execution policy.
- Sysmon recorded the PowerShell process-creation attempt.
- The event was found in Wazuh archives.

### Assessment

The event establishes that a PowerShell process was created with the script path in its command line.

It does not establish that the script completed successfully. The console output confirmed that execution was blocked.

## 6. Observed Event 3 – Controlled Positive Test

### Test Script

```powershell
Write-Output "LAB146_DETECTION_TEST"
Write-Output "Controlled detection validation activity"
```

### Execution Method

The script content was read and executed in the current PowerShell session:

```powershell
$BasePath = "C:\DFIR-Lab146"

$TestContent = Get-Content "$BasePath\detection-tests\lab146-test.ps1" -Raw

Invoke-Expression $TestContent
```

### Expected and Actual Output

```text
LAB146_DETECTION_TEST
Controlled detection validation activity
```

### Event Details

| Field | Observed Value |
|---|---|
| Wazuh timestamp | Nov 26, 2026, 07:46:25.746 |
| Event ID | 1 |
| Process | powershell.exe |
| Parent process | pwsh.exe |
| Process ID | 25012 |
| Parent Process ID | 30424 |
| User | DESKTOP-9MMM37V\Dell |
| Integrity Level | High |
| Command line marker | LAB146_DETECTION_TEST |

### Investigation Observations

- The controlled test completed successfully.
- Sysmon Event ID 1 recorded the Windows PowerShell process.
- The command line contained the unique test marker.
- The parent process was PowerShell 7.7 (`pwsh.exe`).
- Wazuh indexed the event in its archives index.

### Assessment

The positive test confirms that the controlled PowerShell process activity and its command-line marker were visible in Wazuh archives.

It does not independently establish that an alert was generated.

## 7. Observed Event 4 – Ordinary PowerShell Activity

### Command Executed

```powershell
powershell.exe -NoProfile -Command "Write-Output 'LAB146_NORMAL_ACTIVITY'"
```

### Expected Output

```text
LAB146_NORMAL_ACTIVITY
```

### Observations

- The ordinary PowerShell command was executed.
- The expected console output was observed.
- The command was intended as a negative comparison test.
- Its corresponding Sysmon event and Wazuh archive record have not yet been independently verified.
- Alert generation has not been assessed.

### Assessment

The negative test execution is confirmed, but its telemetry visibility and alert outcome remain pending.

It must not yet be recorded as a successfully validated negative detection test.

## 8. Positive and Negative Validation Summary

| Test | Execution | Sysmon / Wazuh Archive | Alert Status |
|---|---|---|---|
| Script-file execution attempt | Blocked | Event confirmed | Unverified |
| Controlled positive test | Successful | Event confirmed | Unverified |
| Ordinary negative test | Successful | Pending verification | Unverified |

## 9. Detection Improvement Opportunities

### DI-146-001 – PowerShell Detection Coverage

Review existing PowerShell process detection coverage and validate whether meaningful suspicious patterns are identified.

### DI-146-002 – Alert Context

Assess whether alerts contain useful process context, including command line, user, endpoint, parent process, and timestamp.

### DI-146-003 – Repeatable Detection Testing

Establish a repeatable testing process with documented positive and negative test cases.

### DI-146-004 – False-Positive Evaluation

Compare controlled suspicious-pattern tests with ordinary PowerShell activity to assess unnecessary alert generation.

These are improvement opportunities, not confirmed detection failures.

## 10. Investigation Limitations

- Wazuh archive records were reviewed, but corresponding alert records were not verified.
- The ordinary negative-test event has not yet been independently located in Wazuh.
- No custom Wazuh detection rule was implemented.
- No rule-level positive or negative validation was performed.
- The WMI event was reviewed as an additional investigation example; its underlying WMI configuration was not re-investigated as part of this lab.
- No actual security incident was established.

## 11. Final Investigation Assessment

The lab confirmed that Sysmon Event ID 1 process-creation telemetry is reaching Wazuh archives with useful process and parent-process context.

The WMI-related event demonstrated the investigation value of process ancestry and execution identity. The controlled positive test demonstrated command-line marker visibility.

The next validation activities are to verify the ordinary negative-test event, inspect Wazuh alerts and rule IDs, and assess whether any targeted detection improvement is justified.

**Overall status:** Telemetry baseline established; detection and alert validation remain in progress.
