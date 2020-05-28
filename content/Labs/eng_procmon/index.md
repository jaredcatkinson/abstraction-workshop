+++
title = "Engineer a Detection - Procmon"
menuTitle = "Engineer a Detection - Procmon"
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

### By the end of this lab, you should be able to

* Understand how to use Procmon
* Understand how to perform dynamic analysis with Procmon

{{% notice warning %}}
If this is not the case, please ask for help!
{{% /notice %}}

## Requirements

- Access to the Windows 10 student system
- Sysinterals Procmon

{{%attachments title="Lab files" style="blue" /%}}

---

## Steps

## 1. Open Procmon: 

* Open File Explorer and navigate to `C:\tools\SysinteralsSuite`

![Procmon](images/procmon_9.png?width=50pc)

* Right click on `Procmon64.exe` and click "Run as administrator"
* Click on "Agree" on the License Agreement

Procmon should be open and ready to use: 

![Procmon](images/procmon_10.png?width=50pc)

## 2. Set up Procmon for dynamic analysis

Now that we have an example of a common method to install a service, we are going to use Procmon to dynamically analyze what happens when the `New-Service` powershell cmdlet creates a service.

Follow the steps below to set Procmon up:

*  With Procmon open, click on Filter

![Procmon 1](images/procmon_1.png?width=50pc)

*  Click on Filter again

![Procmon 1](images/procmon_2.png?width=50pc)

*  In the Filter popup, enter the following values as: `Process Name is powershell.exe`. See screenshot for reference. Once these values are added, click `Add`

![Procmon 1](images/procmon_3.png?width=50pc)

*  Once the filter appears in the list, click Apply

![Procmon 1](images/procmon_4.png?width=50pc)

*  Click on capture

![Procmon 1](images/procmon_5.png?width=50pc)

Procmon is now configured to show us events for only `powershell.exe` activity on our system

## 2. Run example using Procmon

Now that Procmon is configured, we need to install a service using `powershell.exe`

To do this:

*  Open an Administrative PowerShell window
*  Run the following command:

`New-Service -Name LabService -BinaryPathName C:\Windows\System32\notepad.exe`

![Procmon 1](images/procmon_6.png?width=50pc)

If the command completed successfully, you will see 

![Procmon 1](images/procmon_7.png?width=50pc)

## 3. Analyze results

Now that our service has been created, we need to see what clues are contained in the Procmon data.

![Procmon 1](images/procmon_8.png?width=50pc)

We can see in Procmon that there is an amount of activity, but reading through, we see no evidence of interaction with `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services`. Why is this? 

We know that the Service Control Manager database is stored in the registry. Is powershell.exe the process creating the service? No. However, we don't know which process is so lets do some more analysis to find out. 

## 4. Re-analyzing Service Creation

*  In your powershell.exe prompt, input: `sc.exe delete LabService`. This will remove our test service we just created. 

![Procmon](images/sc_delete.png)

*  In Procmon press the `Capture button` to stop capturing events. 

![Procmon 1](images/procmon_5.png?width=50pc)

*  With Procmon open, click on Filter

![Procmon 1](images/procmon_1.png?width=50pc)

*  Click on Filter again

*  In the Filter popup, enter the following values as: `Path begins with HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\ `. Then click `Add`

![Procmon](images/registrykey.png)

*  We want to remove powershell.exe from the capture list, to do this uncheck its box on the left of its column within the filter listing. Your filter should now look like this: 

![Procmon](images/filter.png)

*  Once the filter is removed click `Apply`

*  Click on capture

![Procmon 1](images/procmon_5.png?width=50pc)

*  Go back into your powershell window and run: `New-Service -Name LabService -BinaryPathName C:\Windows\System32\notepad.exe` to create our test service.

![Procmon 1](images/procmon_6.png?width=50pc)

If the command completed successfully, you will see this: 

![Procmon 1](images/procmon_7.png?width=50pc)

*  Go back over to Procmon and perform analysis

![procmon](images/result.png)

## Questions: 

- What is the process name that created the service? 
- Why is this process used? Does it relate to RPC? If so - how? 
- How is powershell calling this process to create this service? 
- What is the full path name of the registry key? 