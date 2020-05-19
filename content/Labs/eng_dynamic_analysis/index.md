+++
title = "Engineer a Detection - Dynamic Analysis"
menuTitle = "Engineer a Detection - Dynamic Analysis"
chapter = false
weight = 306
pre = "<i class='fab fa-leanpub'></i> "
+++

## Description

Now that we have selected New Service as our technique, and used the detection guidance within MITRE ATT&CK to research common utilities. We have chosen to focus on PowerShell's New-Service cmdlet as one of the most common methods to install services. In this next step, we will monitor SC.EXE with Procmon to dynamically analyze how it creates a service, and continue filling out the abstraction map.

{{% notice tip %}}
Review previous labs for process similarities in order to accomplish the main objective(s)
{{% /notice %}}

## Goals

### By the end of this lab, you should

{{% notice warning %}}
If this is not the case, please ask for help!
{{% /notice %}}

## Requirements

- Access to the Windows network
- Access to the Windows 10 student system
- Windows Sysinternals

{{%attachments title="Lab files" style="blue" /%}}

---

## Steps

## 1. Set up Procmon for dynamic analysis

Now that we have an example of a common method to install a service, we are going to use Procmon to dynamically analyze what happens when New-Service creates a service.

Follow the steps below to set Procmon up:

1. Download/Navigate to "insert procmon path"
2. Run `procmon.exe`
3. Click on Filter

![Procmon 1](images/procmon_1.png?width=50pc)

4. Click on Filter again

![Procmon 1](images/procmon_2.png?width=50pc)

5. In the Filter popup, enter the following values as shown in the screenshot, and click Add

![Procmon 1](images/procmon_3.png?width=50pc)

6. Once the filter appears in the list, click Apply

![Procmon 1](images/procmon_4.png?width=50pc)

7. Click on capture

![Procmon 1](images/procmon_5.png?width=50pc)

Procmon is now configured to show us events for only `POWERSHELL.EXE` activity on our system

## 2. Run example using Procmon

Now that Procmon is configured, we need to install a service using `POWERSHELL.EXE`

To do this:

1. Open an Administrative PowerShell window
2. Run the following command:

`New-Service -Name LabService -BinaryPathName C:\Windows\System32\notepad.exe`

![Procmon 1](images/procmon_6.png?width=50pc)

If the command completed successfully, you will see 

```
Status   Name               DisplayName
------   ----               -----------
Stopped  LabService         LabService
```

![Procmon 1](images/procmon_7.png?width=50pc)

## 3. Analyze results

Now that our service has been created, we need to see what clues are contained in the Procmon data.

![Procmon 1](images/procmon_8.png?width=50pc)

We can see in Procmon that there is an amount of activity, but reading through, we see no evidence of interaction with `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services`. This may come as a suprise at first, but let's think about it. We know that the Service Control Manager is an RPC server, and it stores its database in the registry, this would indicate that PowerShell would contact the SCM in order to create the service.

We don't know which process this is, so we need to modify our approach a little bit. Using a different tool in our arsenal, Sysmon, we can monitor the registry path and see what is making the registry changes.

