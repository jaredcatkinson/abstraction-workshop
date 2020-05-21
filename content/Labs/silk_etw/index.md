+++
title = "Silk ETW"
menuTitle = "Silk ETW"
chapter = false
weight = 330
pre = "<i class='fab fa-leanpub'></i> "
+++

---
## Goals

* Understand Event Tracing for Windows.
* Identify Running Traces Sessions and query ETW Providers.
* Use SilkETW to generate and events for consumption. 

__Scenario__

In a previous lab we discovered that New-Service leverages RPC as part of the service creation routine. Until now we have relied on [Microsoft's Service Control Manager Remote Protocol documentation](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-scmr/705b624a-13de-43cc-b8a2-99573da3635f) to make inferences about the implementation details for New-Service's RPC interactions. Inferences are a good start, but we generally like to confirm these details through dynamic or static analysis. As a lead in to this lab, we introduced the idea of Event Tracing for Windows or ETW. This is a logging feature that is available by default on all modern versions of Windows which provides a mechanism for monitoring numerous aspects of the operating system. You are likely very familiar with at least one technology that is the result of ETW, the Windows Security Event Log. The cool thing is that ETW already feeds the Windows Security Event Log, but it also has numerous other events that remain dormant until specifically asked for.

This lab will take you on a tour of one way to explore ETW natively, explain ETW concepts as they become relevant, and show how you can enable targetted logging for testing and validation all within the context of sc.exe and PowerShell's New-Service.

!!! Info
    The DCSync technique is used to retrieve and dump credentials of a specified account. In order to do this, the user you are using to ‘sync’ to the Domain Controller (DC), must be a part of a high privileged group: Administrators, Domain Admins, Enterprise Admins, or a Domain Controller computer accounts. This is done by impersonating the Domain Controller, while doing so it requests user account credentials from the targeted DC.

___
### Requirements
- Windows 10 VM

### Step 1: Query ETW sessions and providers with `logman`.
1. Click on the Windows Start Menu

2. In the search bar, type "powershell"

3. Right click on "Windows PowerShell" and select "Run as Administrator"

![understanding_sysmon_1](/img/understanding_sysmon_1.png)

To start, we want to take a look at all of the running trace sessions: 

!!! Note
    Describe Trace Session HERE
    
```
logman query -ets
```

Notice that there are already a few trace sessions running on the system by default. As mentioned earlier in the lab, the trace session where the "Data Collector Set" is set to EventLog-System is the trace session that is collecting on behalf of the Windows Security Event Log.

Let's take a look to see the specific provider that the `EventLog-System` trace is subscribed to:


!!! Note
    Describe Providers HERE
    
```
logman query "EventLog-System" -ets
```

Make note of the `File-Mode` field at the top of the output and `Name` & `Provider GUID` fields for each subsection. 

Now let's take a look at all of the providers availible: 
```
logman query providers
```

Because we want to filter on all of the possible providers, let's store the providers into a variable with this command:
```
$ETW = logman query providers
```

!!! Note
    This command will take a long time to finish.

To view all of the providers related to RPC run the following command:
```
$ETW | Where-Object { $_ -Like “*RPC*”}
```

![/img/silketw_1.png](/img/silketw_1.png)

### Step 3: Execute the attack and collect the data using SilkETW

1. Click on the Windows Start Menu

2. In the search bar, type "cmd"

3. Right click on "Command Prompt" and select "Run as Administrator"

- To go to the SilkETW Directory run the following: 
```
cd C:\tools\SilkETW_v8\v8\SilkETW
```

- To start a trace for Microsoft RPC run: 
```
.\SilkETW.exe -t user -pn Microsoft-Windows-RPC -ot file -p C:\Users\da\rpc.json
```

The command is stating that we want to peform `user` collection while subscribed to the `Microsoft-Windows-RPC` provider, sending the resulting events to a file which is `C:\Users\da\rpc.json`.

### Open another command prompt as Administrator.

1. Click on the Windows Start Menu

2. In the search bar, type "cmd"

3. Right click on "Command Prompt" and select "Run as Administrator"

To carry out the DCSync Attack, we will use Mimikatz.

- Run the command: 
```
C:\tools\mimikatz_trunk\mimikatz_trunk\x64\mimikatz.exe
```

- To perform the DCSync attack and collect the `krbtgt` credentials, run the command: 
```
lsadump::dcsync /domain:corp.local /user:krbtgt
```

- Wait 30-45 seconds then on the SilkETW console press `Ctrl-C` to stop the collection process. 


### Step 4: View the data
Open up File Explorer and navigate to `C:\Users\da\rpc.json`.

Double click on the file. You will see data from the Microsoft-RPC provider will be present here: 

![/img/silketw_4.png](/img/silketw_4.png)


!!! Note "Conclusion"
    You can now collect data from ETW providers. Use this to start building relationships between known attack techniques to data sensors, data sources, and ETW providers.
