# wazuh-incident-lessons-learned-detection-improvement
## Overview
### Introduction

A security incident does not end when suspicious activity has been investigated or contained. A SOC must also review how effectively the incident was detected, investigated, documented, and handled.

This process is commonly known as a lessons-learned review or post-incident review.

The objective is to identify operational weaknesses and convert those findings into measurable improvements.

### 1. What Is a Lessons-Learned Review?

A lessons-learned review examines the incident-handling process to identify:

- Detection coverage gaps
- Missing or incomplete telemetry
- Insufficient alert context
- Delays in investigation or escalation
- Containment and recovery challenges
- Documentation and communication weaknesses
- Opportunities to improve analyst procedures

The purpose is not simply to identify mistakes. It is to improve the organization's ability to detect and respond to similar activity in the future.

### 2. Detection Coverage Assessment

Detection coverage determines whether relevant security activity is visible to the SOC and whether existing detection logic can identify meaningful suspicious behavior.

For PowerShell activity, useful telemetry includes:

- Process image
- Command line
- Parent process
- Executing user
- Process identifiers
- Integrity level
- Process creation timestamp

Sysmon Event ID 1 provides process-creation information that can support this assessment.

However, collecting an event does not automatically mean that a detection rule will generate an alert.

### 3. Telemetry and Alert Context

Telemetry provides the underlying evidence used during an investigation.

Alert context helps analysts understand why an event requires attention.

For example, a PowerShell process-creation event may contain:

- Executable path
- Command-line arguments
- Parent process image
- User identity
- Process GUID
- Integrity level

These fields allow analysts to investigate process relationships and execution context instead of relying on the process name alone.

A PowerShell process is not inherently malicious. Its purpose must be assessed using the surrounding evidence.

### 4. Process Ancestry and Investigation Context

Parent-child process relationships are important during endpoint investigations.

For example:

```text
WmiPrvSE.exe
      |
      v
powershell.exe
      |
      v
Script execution
```

PowerShell launched by WMI may deserve additional investigation, particularly when it runs under the SYSTEM account or uses unusual command-line arguments.

However, process ancestry and execution privileges are investigation clues, not proof of malicious activity.

Analysts should examine the associated WMI configuration, payload, command line, and supporting telemetry.

### 5. Detection Validation

Detection validation determines whether a detection behaves as intended.

A controlled testing approach should include:

#### Positive Testing

Generate known test activity that should match the intended detection logic.

Example:

```powershell
powershell.exe -NoProfile -Command "Write-Output 'LAB146_DETECTION_TEST'"
```

The objective is to verify whether the expected telemetry and detection behavior occur.

#### Negative Testing

Generate ordinary activity that should not satisfy the intended suspicious-behavior condition.

Example:

```powershell
powershell.exe -NoProfile -Command "Write-Output 'LAB146_NORMAL_ACTIVITY'"
```

The objective is to evaluate whether ordinary activity generates unnecessary alerts.

Both tests must be evaluated against the actual detection logic, not merely against whether a process-creation event exists.

### 6. Telemetry, Archives, and Alerts

A SIEM investigation must distinguish between several stages:

1. Endpoint activity occurs.
2. Sysmon records the activity.
3. The endpoint forwards telemetry.
4. Wazuh receives and indexes the event.
5. Detection rules evaluate the event.
6. A matching rule may generate an alert.

An event appearing in the Wazuh archives index confirms archive visibility. It does not independently confirm that a corresponding alert was generated.

Alert verification should include the relevant alert record and rule ID.

### 7. Detection Improvement Tracking

Identified improvement opportunities should be recorded and tracked.

A detection improvement tracker can include:

- Improvement ID
- Incident reference
- Finding
- Recommended action
- Priority
- Owner
- Status
- Validation result

Each improvement should be linked to an observed finding or a clearly documented assessment requirement.

### 8. False-Positive Assessment

A detection should identify meaningful activity without generating excessive unnecessary alerts.

False-positive evaluation helps determine whether a detection is too broad or lacks sufficient context.

For example, alerting on every PowerShell process creation may create unnecessary noise.

A more useful detection may consider additional context, such as command-line patterns, parent process, execution identity, and other supporting indicators.

### 9. Continuous Detection Improvement

Detection improvement is an ongoing cycle:

```text
Incident Review
       |
       v
Lessons Learned
       |
       v
Detection Gap Assessment
       |
       v
Improvement Planning
       |
       v
Controlled Testing
       |
       v
Positive and Negative Validation
       |
       v
Detection Quality Assessment
       |
       v
Continuous Improvement
```

Detection changes should be implemented only after the existing coverage and rule behavior have been understood.

### Conclusion

Incident lessons-learned reviews help SOC teams turn investigation experience into measurable improvements.

Effective detection engineering requires more than collecting logs or creating rules. It requires reliable telemetry, useful investigation context, controlled testing, alert validation, and continuous assessment of false-positive behavior.

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

## Lab Objectives

### Primary Objective

Evaluate the effectiveness of existing PowerShell process telemetry and establish a structured approach for improving SOC detection and investigation capabilities through a simulated post-incident review.

### Technical Objectives

1. **Review endpoint process telemetry**
   - Examine Sysmon Event ID 1 records.
   - Identify PowerShell process-creation activity.
   - Verify whether process details are available in Wazuh.

2. **Assess investigation context**
   - Examine process command lines and executable paths.
   - Identify parent-child process relationships.
   - Review user identity, integrity level, process identifiers, and timestamps.
   - Understand the investigation value of WMI-launched PowerShell activity.

3. **Establish a detection baseline**
   - Record existing telemetry visibility.
   - Determine which process attributes are available.
   - Separate confirmed telemetry capabilities from unverified alert behavior.

4. **Perform controlled validation**
   - Execute a harmless PowerShell positive test.
   - Execute an ordinary PowerShell negative test.
   - Verify whether the corresponding process events reach Wazuh.
   - Compare the results without assuming that every process event should generate an alert.

5. **Identify improvement opportunities**
   - Assess PowerShell detection coverage.
   - Review alert-context requirements.
   - Evaluate repeatable testing procedures.
   - Consider false-positive risks.

6. **Maintain detection accountability**
   - Create a detection improvement tracker.
   - Link findings to improvement identifiers.
   - Document validation status and outstanding work.

### Expected Learning Outcomes

By completing this lab, the learner should be able to:

- Conduct a structured post-incident lessons-learned review.
- Explain the difference between telemetry collection and alert generation.
- Interpret Sysmon process-creation events.
- Investigate PowerShell process ancestry and execution context.
- Perform controlled positive and negative detection tests.
- Recognize why a process-creation event does not automatically establish malicious execution.
- Document detection improvement opportunities using evidence.
- Assess detection quality without making unsupported claims.

### Completion Criteria

The lab can be considered complete when the following are documented:

- [x] Simulated incident review prepared.
- [x] Lessons-learned document created.
- [x] Detection baseline established.
- [x] Sysmon Event ID 1 records found in Wazuh archives.
- [x] Controlled positive test executed and its event verified.
- [x] PowerShell execution-policy issue documented and handled without changing policy.
- [x] WMI-related PowerShell event reviewed.
- [x] Detection improvement tracker created.
- [x] Ordinary PowerShell negative test executed.
- [ ] Negative-test event independently verified in Wazuh.
- [ ] Wazuh alert generation and rule IDs verified.
- [ ] Detection quality and false-positive behavior assessed.
- [ ] Final improvement recommendations validated.

### Scope Limitations

This is a simulated SOC learning exercise. It does not establish that an actual security incident occurred.

No custom Wazuh detection rule was implemented or modified during the documented work.


## Lab Scenario

### Scenario Background

A SOC team has completed an investigation involving suspicious PowerShell activity on a Windows endpoint. Following the investigation, the team initiates a lessons-learned review to evaluate whether existing monitoring and detection capabilities provided sufficient visibility and investigation context.

The review is conducted using a simulated incident associated with endpoint `DESKTOP-9MMM37V`.

The objective is to determine whether process-creation telemetry is being collected, whether important execution details are available to analysts, and whether existing detection coverage can be validated through controlled testing.

### Investigation Context

During the review, the analyst examines Sysmon Event ID 1 process-creation records available in Wazuh archives.

The review identifies multiple examples of PowerShell execution.

One event shows PowerShell launched by `WmiPrvSE.exe` under the SYSTEM account. Its command line references a payload in the WMI Permanent Event lab directory and includes the `-ExecutionPolicy Bypass` argument.

This event provides an opportunity to examine parent-child process relationships, execution identity, and command-line context. However, the event alone does not establish malicious activity.

The analyst also creates a harmless PowerShell test script containing a recognizable marker. Direct script execution is blocked by the endpoint's existing PowerShell execution policy. Rather than changing the policy, the analyst executes the inspected test content using `Invoke-Expression`.

The controlled test completes successfully, and its command-line marker is visible in a Sysmon Event ID 1 record indexed by Wazuh.

### Detection Assessment

The review confirms that relevant process-creation telemetry is reaching Wazuh archives. The available event records contain useful investigation attributes, including executable path, command line, parent process, user, process identifiers, integrity level, and hashes.

However, the team must distinguish between successful telemetry ingestion and successful alert generation.

The presence of a process event in the archives index does not prove that an existing Wazuh detection rule generated an alert.

To evaluate detection behavior further, the analyst executes an ordinary PowerShell command using a different marker. This acts as a negative comparison test.

The command completes successfully, but its corresponding Wazuh event and alert status still require independent verification.

### SOC Review Requirements

The analyst is expected to:

- Document the simulated incident and its review objectives.
- Establish a baseline for existing PowerShell process telemetry.
- Review relevant Sysmon and Wazuh event records.
- Examine process ancestry and execution context.
- Perform controlled positive and negative tests.
- Identify potential detection and alert-context improvements.
- Maintain an improvement tracker with clear validation status.
- Avoid changing existing detection rules before reviewing their current behavior.

### Expected Outcome

The review should produce a documented assessment of existing telemetry visibility, investigation context, and detection-validation requirements.

Potential improvements should be traceable to observed evidence or clearly identified assessment needs.

The final review must distinguish confirmed findings from pending validation. It should not claim that a detection failed, that an alert was generated, or that malicious activity occurred unless the supporting evidence establishes those conclusions.

The intended outcome is a repeatable, evidence-based process for turning incident lessons into better SOC detection and investigation practices.


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

