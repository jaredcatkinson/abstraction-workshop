+++
title = "Silk ETW"
menuTitle = "Silk ETW"
chapter = false
weight = 330
pre = "<i class='fab fa-leanpub'></i> "
+++

---
## Goals

Understand Event Tracing for Windows.

Identify Running Traces Sessions and query ETW Providers.

Use SilkETW to generate and events for consumption. 

__Scenario__

You have been tasked with logging Windows RPC events, as they could correlate with possible `DCSync` attacks in your environment. You will utilize ETW to capture these events, specifically a tool called `SilkETW`. You will then run a `DCSync` attack to make sure that logs are being propagated accordingly. 

!!! Info
    The DCSync technique is used to retrieve and dump credentials of a specified account. In order to do this, the user you are using to ‘sync’ to the Domain Controller (DC), must be a part of a high privileged group: Administrators, Domain Admins, Enterprise Admins, or a Domain Controller computer accounts. This is done by impersonating the Domain Controller, while doing so it requests user account credentials from the targeted DC.

___
### Requirements
- Windows 10 VM
- Windows Server 2016 (AD/DC)
- SilkETW
- Mimikatz

### Step 1: Unzip Mimikatz and Prepare SilkETW: 
- Open `File Explorer`
- Navigate to the `C:\tools` folder
- `Right-Click` on `mimikatz_trunk.zip`. 
- Choose `Extract All...` and extract to `C:\tools\mimikatz`, which is the default. 
- Click `Extract` and enter in the password of `infected`.

### Step 2: Query ETW sessions and providers with `logman`.
1. Click on the Windows Start Menu

2. In the search bar, type "powershell"

3. Right click on "Windows PowerShell" and select "Run as Administrator"

![understanding_sysmon_1](/img/understanding_sysmon_1.png)

To start, we want to take a look at all of the running trace sessions: 
```
logman query -ets
```

Next, let's take a look at the provider that the `EventLog-System` trace is subscribed to:
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
