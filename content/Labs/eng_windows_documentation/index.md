+++
title = "Engineer a Detectin - Windows Documentation"
menuTitle = "Engineer a Detection - Windows Documentation"
chapter = false
weight = 307
pre = "<i class='fab fa-leanpub'></i> "
+++

## Description

Now that we have determined that `New-Service` can be used to install a Windows service, we need to continue investigating to complete our abstraction map.

{{% notice tip %}}
Review previous labs for process similarities in order to accomplish the main objective(s)
{{% /notice %}}

## Goals

Use source code and Windows Documentation to research service installation.

### By the end of this lab, you should:

- Be familiar with researching source code
- Be familiar with researching using Microsoft Documentation

{{% notice warning %}}
If this is not the case, please ask for help!
{{% /notice %}}

## Requirements

- Access to the Windows network
- Access to the Windows 10 student system
- Google Chrome

{{%attachments title="Lab files" style="blue" /%}}

---

## Steps

## 1. Research source code

This is the current state of our Abstraction Map:

![Tool Abstraction](images/mitre_attack_4.png?width=50pc)

We need to continue our research, and we know that PowerShell is open sourced. This can give an excellent starting point in determining how this PowerShell cmdlet actually creates a service.

The source code for PowerShell is stored in [this](https://github.com/PowerShell/PowerShell) GitHub repository.

Once we navigate there, we want to utilize the search function to speed our research.

![Tool Abstraction](images/windows_docs_1.png?width=50pc)

We are going to search the name of the function we are trying to investigate: `New-Service`. Make sure you select "In this repository" when searching:

![Tool Abstraction](images/windows_docs_2.png?width=50pc)

Scrolling through the results, none really stand out until you reach this one:

![Tool Abstraction](images/windows_docs_3.png?width=50pc)

It states `This class implements the New-Service command.` This sounds exactly like what we need. After opening the file and finding the section we are interested in on line `2120` - we can see this:

![Tool Abstraction](images/windows_docs_4.png?width=50pc)

Line `2120` shows that we are in the function that creates the service.

Line `2138` mentions the "service controller".

Line `2145` shows an interesting function called `OpenSCManagerW`. This function shares the naming convention of a Win32 API function.

A quick Google search supports this theory:

![Tool Abstraction](images/windows_docs_5.png?width=50pc)

Opening [this](https://docs.microsoft.com/en-us/windows/win32/api/winsvc/nf-winsvc-openscmanagerw) article from Microsoft, we can see in the description what the purpose of the function is:

`Establishes a connection to the service control manager on the specified computer and opens the specified service control manager database.`

We are starting to see a pattern of the "Service Controller" or "Service Control Manager (SCM)". According to Microsoft, however, this function only establishes a connection to the SCM and does not actually create the service.

If we continue reading on GitHub, there is another line in the PowerShell source code that looks relevant to service creation:

![Tool Abstraction](images/windows_docs_6.png?width=50pc)

Line `2211` also says "Create the service".

Line `2212` has another function that matches the Win32 API scheme, but looks much more specific to service creation: `CreateServiceW`

Returning to Google, we see [another Microsoft doc](https://docs.microsoft.com/en-us/windows/win32/api/winsvc/nf-winsvc-createservicew) for this function.

![Tool Abstraction](images/windows_docs_7.png?width=50pc)

This doc has the following description: 

`Creates a service object and adds it to the specified service control manager database.`

We have now arrived at the Win32 API function that creates a service. Here is where the abstraction map currently stands, given what we know:

![Tool Abstraction](images/windows_docs_8.png?width=50pc)

Here are our questions from before:

1. What are common utilities to install a service?
    - New-Service is one of them, but are there more?
2. Why would we monitor the registry for service creations?
    - Unknown

Given our research, we should add two more questions:

1. What are common utilities to install a service?
    - New-Service is one of them, but are there more?
2. Why would we monitor the registry for service creations?
    - Unknown
3. What is the Service Control Manager?
    - Unknown
4. What is the Service Control Database?
    - Unknown

You may have guessed the next step...Google `Service Control Manager` to see if there is any Microsoft documentation on the subject.

The first result is an [article](https://docs.microsoft.com/en-us/windows/win32/services/service-control-manager) that may provide clarity:

![Tool Abstraction](images/windows_docs_9.png?width=50pc)

According to Microsoft, the Service Control Manager is: 

`The service control manager (SCM) is started at system boot. It is a remote procedure call (RPC) server, so that service configuration and service control programs can manipulate services on remote machines.`

Now we know that the next layer of the Abstraction Map will be RPC, but this article doesn't provide much more context. We will return to Google to search `service control manager rpc`. Unfortunately this has a lot of help site articles, so we need to add something to our search to only include Microsoft docs: 

`service control manager rpc site:docs.microsoft.com`

![Tool Abstraction](images/windows_docs_10.png?width=50pc)

We see a few relevant results at the bottom. First reading through [Services and RPC/TCP - Win32 apps | Microsoft Docs](https://docs.microsoft.com/en-us/windows/win32/services/services-and-rpc-tcp) - we see that:

`Starting with Windows Vista, the service control manager (SCM) supports remote procedure calls over both Transmission Control Protocol (RPC/TCP) and named pipes (RPC/NP). Client-side SCM functions use RPC/TCP by default.`

We see that the SCM supports both RPC over both TCP and Named Pipe (NP). Another link in the results, [[MS-SCMR]: Overview | Microsoft Docs](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-scmr/d5bd5712-fa64-44bf-9433-3651f6a5ce97)  doesn't offer too much clarify, but one sub-page caught my eye:

![Tool Abstraction](images/windows_docs_11.png?width=50pc)

Upon clicking, we see that this contains exactly what we need - the UUID of [MS-SCMR] and the name of the [MS-SCMR] named pipe:

![Tool Abstraction](images/windows_docs_12.png?width=50pc)

This allows us to add to our Abstraction Map:

![Tool Abstraction](images/windows_docs_13.png?width=50pc)

We have also answered one of our questions:

1. What are common utilities to install a service?
    - New-Service is one of them, but are there more?
2. Why would we monitor the registry for service creations?
    - Unknown
3. What is the Service Control Manager?
    - The SCM is a service that supports RPC/TCP and RPC/NP for remote service management.
4. What is the Service Control Database?
    - Unknown

We still don't know how the registry fits into the puzzle, but we have yet to research the Service Control Database - so let's do that. Googling `scm database` has some non-Microsoft results, so once again we will add `site:docs.microsoft.com`:

![Tool Abstraction](images/windows_docs_14.png?width=50pc)

It looks like we have a relevant results that we haven't seen yet. Opening the doc reveals this:

`The SCM maintains a database of installed services in the registry. The database is used by the SCM and programs that add, modify, or configure services. The following is the registry key for this database: HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services.`

This information allows us to both complete our Abstraction Map, and answer some questions that we had:

![Tool Abstraction](images/windows_docs_15.png?width=50pc)

1. What are common utilities to install a service?
    - New-Service is one of them, but are there more?
2. Why would we monitor the registry for service creations?
    - The SCM maintains a database of installed services in the registry.
3. What is the Service Control Manager?
    - The SCM is a service that supports RPC/TCP and RPC/NP for remote service management.
4. What is the Service Control Database?
    - The SC Database is a database of installed services that resides in the registry.
    