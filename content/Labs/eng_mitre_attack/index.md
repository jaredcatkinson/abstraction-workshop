+++
title = "Engineer a Detection - MITRE ATT&CK"
menuTitle = "Engineer a Detection - MITRE ATT&CK"
chapter = false
weight = 305
pre = "<i class='fab fa-leanpub'></i> "
+++

## Description

You should have just completed guided instruction and labs for engineering a detection for the [MSHTA technique (T1170)](https://attack.mitre.org/techniques/T1170/). We will be following similar procedures, and filling out an abstraction map on a different technique.

{{% notice tip %}}
Review previous labs for process similarities in order to accomplish the main objective(s)
{{% /notice %}}

## Goals

Utilize Mitre ATT&CK to research a given technique and begin the development of an abstraction map.

### By the end of this lab, you should:

- Have researched this technique?

{{% notice warning %}}
If this is not the case, please ask for help!
{{% /notice %}}

## Requirements

- Access to the Windows network
- Access to the Windows 10 student system

{{%attachments title="Lab files" style="blue" /%}}

---

## Steps

## 1. Select Target Technique (Selected for you)

- Choose the technique that you will build your detection around

{{% notice info %}}

#### Technique: Service Installation (T1050)

For this lab, the technique has been chosen for you.

- Technique: __New Service (T1050)__
- Method of Technique Selection: __TTP Driven__
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

This leaves us with an abstraction map in this state:

![Empty Abstraction](images/mitre_attack_1.png?width=50pc)

## 2. Research common tools and methods

When reading the MITRE ATT&CK page on T1050, we see this wording under the Detection guidance:

`Monitor service creation through changes in the Registry and common utilities using command-line invocation`

This raises a few questions:

1. What are common utilities to install a service?
2. Why would we monitor the registry for service creations?

In order to answer question number 1, we can put the following into Google:

`how to install a windows service`

The first result is [this](https://docs.microsoft.com/en-us/dotnet/framework/windows-services/how-to-install-and-uninstall-services) article:

![Google](images/mitre_attack_2.png?width=50pc)

We see that one of the methods to create a service is using the PowerShell cmdlet `New-Service` with the following syntax:

`New-Service -Name "YourServiceName" -BinaryPathName <yourproject>.exe`

![Google](images/mitre_attack_3.png?width=50pc)

Now we have a partial answer to Question 1: What are common utilities to install a service?

However, we have a new question to answer, which is "How many ways are there to install a service?"

Our abstraction map has the first level filled out - but we need to continue investigating the current tool before we can expand our research.

![Tool Abstraction](images/mitre_attack_4.png?width=50pc)

Here are our current set of questions:

1. What are common utilities to install a service?
    - New-Service is one of them, but are there more?
2. Why would we monitor the registry for service creations?
    - Unknown

This is the end of what we can glean from the Mitre ATT&CK Matrix - in the next lab we will continue researching utilizing Microsoft documentation and open source code.
