
# Lab 146 – Troubleshooting Notes

## 1. PowerShell Script Execution Blocked

### Issue

Attempting to execute the Lab 146 test script directly produced an execution-policy error.

### Command

```powershell
powershell.exe -NoProfile -File "$BasePath\detection-tests\lab146-test.ps1"
```

### Error

```text
File cannot be loaded because running scripts is disabled on this system.
CategoryInfo          : SecurityError
FullyQualifiedErrorId : UnauthorizedAccess
```

### Cause

The existing PowerShell execution policy prevented direct execution of the `.ps1` file.

### Resolution

The system execution policy was not changed.

Instead, the contents of the harmless test file were read and executed in the current PowerShell session.

```powershell
$BasePath = "C:\DFIR-Lab146"

$TestContent = Get-Content "$BasePath\detection-tests\lab146-test.ps1" -Raw

Invoke-Expression $TestContent
```

### Result

```text
LAB146_DETECTION_TEST
Controlled detection validation activity
```

The test completed successfully.

### Lesson Learned

A script execution-policy restriction should not automatically lead to a system-wide policy change.

For this controlled lab, executing the known harmless test content in the current session allowed the test to proceed without changing the existing policy.

`Invoke-Expression` executes supplied PowerShell content, so it should only be used with trusted, inspected content.

---

## 2. Wazuh Archive Verification

### Observation

Sysmon Event ID 1 process-creation records were found in:

```text
wazuh-archives-4.x-2026.11.26
```

### Relevant Fields

```text
agent.name
data.win.system.providerName
data.win.system.eventID
data.win.eventdata.image
data.win.eventdata.commandLine
data.win.eventdata.parentImage
data.win.eventdata.parentCommandLine
data.win.eventdata.user
data.win.eventdata.parentUser
data.win.eventdata.processId
data.win.eventdata.parentProcessId
data.win.eventdata.integrityLevel
```

### Finding

The reviewed events contained useful process and parent-process context.

### Lesson Learned

Confirm the actual indexed field names from the event JSON before constructing queries or writing custom detection rules.

---

## 3. Distinguishing Process Creation from Successful Script Execution

### Observation

The direct script-file execution attempt appeared in Wazuh even though PowerShell blocked the script.

### Explanation

Sysmon Event ID 1 records process creation. It does not independently confirm that the requested script completed successfully.

### Investigation Approach

Correlate process-creation telemetry with:

- PowerShell console output or other execution evidence
- Relevant PowerShell event logs, where configured
- Script file details
- Parent process context
- Additional endpoint telemetry

### Lesson Learned

A process-creation event containing a script path is not sufficient proof of successful script execution.

---

## 4. Archive Visibility versus Alert Generation

### Observation

The controlled PowerShell test was found in the Wazuh archives index.

### Limitation

The presence of an event in the archives index does not establish that a corresponding Wazuh alert was generated.

### Next Validation

Inspect the relevant Wazuh alerts index and compare event timestamps, process details, and rule IDs.

### Lesson Learned

Telemetry ingestion, archive indexing, rule matching, and alert generation are separate stages and must be validated independently.

---

## 5. Negative-Test Verification Pending

### Test Command

```powershell
powershell.exe -NoProfile -Command "Write-Output 'LAB146_NORMAL_ACTIVITY'"
```

### Observed Result

```text
LAB146_NORMAL_ACTIVITY
```

### Current Status

The command executed successfully, but its corresponding Sysmon event and Wazuh archive record have not yet been independently verified.

### Next Action

Search Wazuh archives using the actual indexed command-line field:

```text
agent.name: "DESKTOP-9MMM37V" AND
data.win.system.providerName: "Microsoft-Windows-Sysmon" AND
data.win.system.eventID: 1 AND
data.win.eventdata.commandLine: *LAB146_NORMAL_ACTIVITY*
```

Use an appropriate time range around the negative test execution.

### Lesson Learned

A successful console execution is not equivalent to confirmed SIEM visibility.

---

## 6. Detection Rule Changes Deferred

### Observation

The baseline confirmed useful Sysmon process-creation telemetry.

### Decision

No custom Wazuh rule was created or modified during this stage.

### Reason

Alert generation and existing rule coverage have not yet been verified. Creating a rule before reviewing existing detection behavior could introduce duplicate alerts or unnecessary noise.

### Next Action

Review existing alert records and rule IDs before proposing a targeted detection change.

---

## 7. Troubleshooting Summary

| Issue / Observation | Action Taken | Result |
|---|---|---|
| PowerShell script blocked | Inspected file and used `Invoke-Expression` | Test completed |
| Sysmon process telemetry | Reviewed Wazuh archives | Event records confirmed |
| Command-line visibility | Inspected raw event data | Confirmed |
| Parent-process visibility | Inspected raw event data | Confirmed |
| WMI-launched PowerShell | Reviewed process-creation event | Context documented |
| Alert generation | Not yet checked in alerts index | Pending |
| Negative-test archive visibility | Requires Wazuh verification | Pending |
| Custom rule implementation | Deferred | No configuration changes |

## 8. Final Troubleshooting Notes

The main technical issue was the PowerShell execution-policy restriction. It was resolved for the controlled test without changing system policy.

The Wazuh archive review confirmed that Sysmon process-creation events contained useful investigation fields.

The remaining work is focused on alert verification, negative-test telemetry confirmation, and detection-quality assessment rather than changing the existing endpoint or Wazuh configuration prematurely.
