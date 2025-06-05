It is important to know how to build proper filter syntax for querying Active Directory using `PowerShell`. This knowledge gives us a deeper understanding of how our tools such as `PowerView` function under the hood and how we can further harness their power when enumerating Active Directory. It is also useful to understand how to formulate filters if you find yourself in a situation during an assessment without any of your tools available to you. Armed with this knowledge, you will be able to effectively "live off the land" and utilize built-in PowerShell cmdlets to perform your enumeration tasks (albeit slower than using many of the tools we will cover in this module).

---

## PowerShell Filters
Filters in PowerShell allow you to process piped output more efficiently and retrieve exactly the information you need from a command. Filters can be used to narrow down specific data in a large result or retrieve data that can then be piped to another command.

We can use filters with the `Filter` parameter. A basic example is querying a computer for installed software:

#### PowerShell - Filter Installed Software

  Active Directory Search Filters

```powershell
PS C:\htb> get-ciminstance win32_product | fl


IdentifyingNumber : {7FED75A1-600C-394B-8376-712E2A8861F2}
Name              : Microsoft Visual C++ 2017 x86 Additional Runtime - 14.12.25810
Vendor            : Microsoft Corporation
Version           : 14.12.25810
Caption           : Microsoft Visual C++ 2017 x86 Additional Runtime - 14.12.25810

IdentifyingNumber : {748D3A12-9B82-4B08-A0FF-CFDE83612E87}
Name              : VMware Tools
Vendor            : VMware, Inc.
Version           : 10.3.2.9925305
Caption           : VMware Tools

IdentifyingNumber : {EA8CB806-C109-4700-96B4-F1F268E5036C}
Name              : Local Administrator Password Solution
Vendor            : Microsoft Corporation
Version           : 6.2.0.0
Caption           : Local Administrator Password Solution

IdentifyingNumber : {2CD849A7-86A1-34A6-B8F9-D72F5B21A9AE}
Name              : Microsoft Visual C++ 2017 x64 Additional Runtime - 14.12.25810
Vendor            : Microsoft Corporation
Version           : 14.12.25810
Caption           : Microsoft Visual C++ 2017 x64 Additional Runtime - 14.12.25810

<SNIP>
```

The above command can provide considerable output. We can use the `Filter` parameter with the `notlike` operator to filter out all Microsoft software (which may be useful when enumerating a system for local privilege escalation vectors).

#### PowerShell - Filter Out Microsoft Software

```powershell
PS C:\htb> get-ciminstance win32_product -Filter "NOT Vendor like '%Microsoft%'" | fl


IdentifyingNumber : {748D3A12-9B82-4B08-A0FF-CFDE83612E87}
Name              : VMware Tools
Vendor            : VMware, Inc.
Version           : 10.3.2.9925305
Caption           : VMware Tools
```

---

## Operators