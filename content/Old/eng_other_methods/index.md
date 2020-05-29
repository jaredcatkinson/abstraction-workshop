+++
title = "Engineer a Detection - Sysmon/Registry"
menuTitle = "Engineer a Detection - Sysmon/Registry"
chapter = false
weight = 308
pre = "<i class='fab fa-leanpub'></i> "
+++

## Description

You should have just completed guided instruction and labs for engineering a detection for the [MSHTA technique (T1170)](https://attack.mitre.org/techniques/T1170/). This next lab increases in difficulty allowing students to have structure but with less step-by-step guidance. Instructors will be available and walking around, so feel free to ask questions.

{{% notice tip %}}
Review previous labs for process similarities in order to accomplish the main objective(s)
{{% /notice %}}

## Goals

Create your own detection using the detection engineering model.

### By the end of this lab, you should:

- Be able to engineer a detection upon completion of this lab
- Be familiar with the methodology of selection and research of a technique to building a detection based on your findings

{{% notice warning %}}
If this is not the case, please ask for help!
{{% /notice %}}

## Requirements

- Access to the Windows network
- Access to the Windows 10 student system



{{%attachments title="Lab files" style="blue" /%}}

---
## Steps

### 1. Select Target Technique (Selected for you)

- Choose the technique that you will build your detection around

{{% notice info %}}

__Technique: Credential Dumping__ 

For this lab, the technique has been chosen for you. 
    
- Technique: __Credential Dumping__
- Method of Technique Selection: __Data Driven__
{{% /notice %}}

- Detections should be built at the most generic level possible, but many times a single technique will require multiple detections for completion
    - Input
        - MITRE ATT&CK Framework
        - Your organization’s data sources
        - Threat Intelligence
    - Output
        - A specific Technique that you will be targeting
        - Possibly, a specific implementation of that Technique (Procedure)

- Document the different variations of this technique

__Questions__

- In what ways can Credential Dumping occur? 
- Where would you find these variances?
- Who might have such documentation? 

{{%expand "Answers" %}}

In what ways can Credential Dumping occur or what are the variances of this technique? 
    
- Look at some of the different tools mentioned in MITRE ATT&CK. Example: `Cached Credentials`

Where would you find these variances?

- Overview of [MITRE ATT&CK Tactic ID: 1003](https://attack.mitre.org/techniques/T1003/)

Who might have such documentation?

- MITRE Documents information such as this across their ATT&CK Framework. This is always a good starting point to learn more about TTPs. See the bottom of the technique pages to research other sources of information

{{% /expand%}}

### 2. Research Underlying Technology

- Research the technology associated with the technique to help understand the use cases, related data sources, and detection opportunities
- Defenders often create superficial detections because they lack an understanding of the technology involved
    
__Input Documentation__

- What is your selected Technique?
- Where would you first look regarding information about a specific Tactic and Technique?

{{%expand "Input Documentation Answers" %}}

- What is your selected Technique? __Credential Dumping/Plain-Text Credentials__
- Where would you first look regarding information about a specific Tactic and Technique? __MITRE ATT&CK Wiki Page for selected technique__
- What is the tactic ID for this specific technique?
{{% /expand%}}

__Output Documentation__

- How the technique works (low level)?
- Why attackers would use this technique?
- What alternatives to this technique do attackers have?
- What is the list of potential data sources?

{{%expand "Output Documentation Answers" %}}
- Remember to document `Initial Questions` you have about the technique
    - You can review the MSHTA example
- You can always add additional questions as you learn more about the technique
{{% /expand%}}

__Useful Tools__

- Are there any useful tools that can aid you in your research? Remember the MSHTA example?
- What can you learn from these tools?
- Did any additional questions come up?

{{%expand "Useful Tools Answers" %}}
- What does MITRE ATT&CK Wiki page say about tools you could use to verify this technique?
- Examples: 
    - `Task Manager` 
    - [ProcDump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump)
    - [Out-Minidump](https://github.com/PowerShellMafia/PowerSploit/blob/master/Exfiltration/Out-Minidump.ps1)
    - [Windows Credential Editor](https://attack.mitre.org/software/S0005/)
    - [Mimikatz](https://attack.mitre.org/software/S0002/)
{{% /expand%}}

__Questions__

- Where are credentials stored when a user logs onto a system?
- What functions as a common interface to several Security Support Providers (SSPs)?
    - What are Security Support Providers?
- What is the preferred authentication protocol for mutual client-server domain authentication?

{{%expand "Answers" %}}
- Where are credentials stored when a user logs onto a system? `Local Security Authority Subsystem Service (LSASS)`
- What functions as a common interface to several Security Support Providers (SSPs)? `SSPI (Security Support Provider Interface)`
- What are Security Support Providers? `a dynamic-link library (DLL) that makes one or more security packages available to applications`
- What is the preferred authentication protocol for mutual client-server domain authentication? `Kerberos`
{{% /expand%}}

__Questions__

- What mimikatz command is used to dump credentials?
- Did you notice the command `sekurlsa::logonPasswords`? 
- What can you learn about what [mimikatz](https://github.com/gentilkiwi/mimikatz/blob/master/mimikatz/modules/sekurlsa/kuhl_m_sekurlsa.c#L174-L176) is doing here?
    - Open lsass.exe
    - Enumerate all LogonSessions
    - Copy lsass memory, and parse it for credentials for default Authentication Packages

{{%expand "Answers" %}}

What Function is being called here?

- OpenProcess
- [https://blog.3or.de/hunting-mimikatz-with-sysmon-monitoring-openprocess.html](https://blog.3or.de/hunting-mimikatz-with-sysmon-monitoring-openprocess.html)

![engineer_detection_1.png](images/engineer_detection_1.png?width=50pc)
{{% /expand%}}

- You should now have enough information about the technique to understand how __Credential Dumping - Plain-Text Credentials__ works

### 3. Proof of Concept Malware Sample

- Build or Find a benign malware sample to allow for data source evaluation and detection validation

__Input Documentation__

- Target technique/procedure
- Understanding of attack technique

{{%expand "Input Documentation" %}}
- Target technique/procedure: `Credential Dumping - Plain Text Credentials`
- Understanding of attack technique: `All the information you documented in __Step 2__`
{{% /expand%}}

__Output Documentation__

- Ability to execute the technique for validation purposes
- Command Line parameters
- Script
- Binary
        
{{%expand "Output Documentation" %}}

- Ability to execute the technique for validation purposes
    - Document validation steps you would take to run a sample of the technique in your environment
    - Example: What are some command line tests you can document to run?
- Command Line parameters
    - What are the different arguments you can pass into the command line examples you found above?
- Script
    - Are there any other scripting examples you can test with?
- Binary
    - What types of binaries might be used for this technique? 
    - Document the use of these binaries for testing
{{% /expand%}}


### 4. Identify Data Sources
- Evaluate what data sources are necessary to allow for detection of the technique
- This step is part of a cycle with the next step (Build Atomic Detection)
    
{{%expand "Input Documentation" %}}
- Selected Technique/Procedure: `Credential Dumping - Plain Text Credentials`
- Understanding of involved technology
- Understanding of attacker’s motivation: `To obtain credentials for other techniques such as privilege escalation, lateral movement, etc.`
- Proof of concept malware sample
{{% /expand%}}

{{%expand "Output Documentation" %}}

- List of selected data sources
    - API monitoring
    - Process monitoring
    - PowerShell logs
    - Process command-line parameters
- Necessary data sources are enabled and being centralized
    
- Are there any data sources in HELK that are missing?
- Are there any other data sources missing in MITRE ATT&CK that may be valuable?
{{% /expand%}}

### 5. Build the Detection

{{%expand "How would you naturally describe what you want to query?" %}}
![engineer_detection_2.png](images/engineer_detection_2.png?width=50pc)

Look for any processes that opened a handle to `lsass.exe`
{{% /expand%}}
    
{{%expand "What is the data index you would use (based on the data sensor)?" %}}
![images/engineer_detection_4.png](images/engineer_detection_4.png?width=50pc)

logs-endpoint-winevent-sysmon-*
{{% /expand%}}

{{%expand "What is the event identifier (EventID)?" %}}
#### Data Model

![images/engineer_detection_3.png](images/engineer_detection_3.png?width=50pc)

#### Look for any processes that opened a handle to `lsass.exe`

![images/engineer_detection_5.png](images/engineer_detection_5.png?width=50pc)

#### ProcessAccess reports when a process opens another process

![images/engineer_detection_6.png](images/engineer_detection_6.png?width=50pc)

#### Event Identifier (EventID)
Sysmon EventID: `10`
{{% /expand%}}

{{%expand "What are the event details that will allow you to narrow in on your detection, based on your research?" %}}

#### Specify the Target Process
```
event_id:10 AND process_target_name:lsass.exe
```

#### Processes that Opened Handles to `lsass.exe`
![engineer_detection_8.png](images/engineer_detection_8.png?width=50pc)

#### MimiKatz and OpenProcess
![engineer_detection_7.png](images/engineer_detection_7.png?width=50pc)

#### Process Granted Access
```
event_id:10 AND process_target_name:lsass.exe AND process_granted_access:(0x1010 OR 0x1410)
```
{{% /expand%}}
