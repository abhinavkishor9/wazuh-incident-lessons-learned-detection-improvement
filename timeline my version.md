# Timeline

## Activity Timeline

### 1. Lab Workspace Preparation

**Activity:** Created the Lab 146 directory structure.

**Workspace:**

```text
C:\DFIR-Lab146
```

Created directories:

- incident-review
- baseline
- detection-tests
- detection-improvements
- validation
- evidence
- logs

**Result:** Workspace created successfully.

### 2. Simulated Incident Documentation

**Activity:** Created the incident summary and lessons-learned documents.

**Incident ID:** IR-146-001

**Scenario:** Simulated suspicious PowerShell activity on `DESKTOP-9MMM37V`.

**Result:** Initial review objectives and improvement areas documented.

### 3. Detection Baseline Preparation

**Activity:** Created the baseline document for Sysmon Event ID 1.

**Review questions:**

- PowerShell process visibility
- Command-line availability
- Parent-process availability
- User context
- Alert generation
- Manual investigation requirements

**Result:** Baseline questions documented; evidence review pending.

### 4. Harmless Test Script Creation

**Activity:** Created:

```text
C:\DFIR-Lab146\detection-tests\lab146-test.ps1
```

**Contents:**

```powershell
Write-Output "LAB146_DETECTION_TEST"
Write-Output "Controlled detection validation activity"
```

**Result:** Test file created and its contents verified.

### 5. PowerShell Execution-Policy Restriction

**Activity:** Attempted to execute the test script directly.

**Command:**

```powershell
powershell.exe -NoProfile -File "$BasePath\detection-tests\lab146-test.ps1"
```

**Result:** Execution was blocked by the existing PowerShell execution policy.

**Resolution:** The policy was not changed. The inspected file contents were executed using `Invoke-Expression`.

### 6. WMI-Related PowerShell Event Review

**Observed Wazuh timestamp:** 07:32:57.942 IST

**Event:** Sysmon Event ID 1

**Process:** powershell.exe

**Parent:** WmiPrvSE.exe

**User:** NT AUTHORITY\SYSTEM

**Command line included:**

```text
-ExecutionPolicy Bypass -File "C:\WMIPermanentEventLab\Payload\wmi-payload.ps1"
```

**Result:** WMI-launched PowerShell process activity was documented as an additional investigation example.

### 7. Lab 146 Script-File Execution Attempt Recorded

**Observed Wazuh timestamp:** 07:45:54.889 IST

**Event:** Sysmon Event ID 1

**Process:** powershell.exe

**Parent:** pwsh.exe

**Command line included:**

```text
-NoProfile -File C:\DFIR-Lab146\detection-tests\lab146-test.ps1
```

**Result:** The process-creation attempt was found in Wazuh archives. The script itself had been blocked by the execution policy.

### 8. Controlled Positive Test Recorded

**Observed Wazuh timestamp:** 07:46:25.746 IST

**Event:** Sysmon Event ID 1

**Process:** powershell.exe

**Parent:** pwsh.exe

**User:** DESKTOP-9MMM37V\Dell

**Command line marker:**

```text
LAB146_DETECTION_TEST
```

**Result:** The controlled test completed successfully, and the corresponding process-creation event was found in Wazuh archives.

### 9. Detection Improvement Tracker Created

**Activity:** Created `improvement-tracker.csv`.

**Improvement items:**

- DI-146-001 – PowerShell detection coverage
- DI-146-002 – Alert context
- DI-146-003 – Repeatable detection testing
- DI-146-004 – False-positive evaluation

**Result:** Four improvement items recorded with open status and pending validation.

### 10. Ordinary PowerShell Negative Test Executed

**Activity:** Executed a normal PowerShell command.

**Command:**

```powershell
powershell.exe -NoProfile -Command "Write-Output 'LAB146_NORMAL_ACTIVITY'"
```

**Result:** Expected console output observed. Corresponding Wazuh event and alert status remain pending verification.

### 11. Detection Quality Assessment Prepared

**Activity:** Created `detection-quality-assessment.txt`.

**Assessment areas:**

- Detection coverage
- Alert context
- False-positive assessment
- Repeatability
- Analyst investigation value

**Result:** Assessment prepared with validation items pending.

