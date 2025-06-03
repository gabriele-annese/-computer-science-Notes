

The [LocalSystem account](https://docs.microsoft.com/en-us/windows/win32/services/localsystem-account) `NT AUTHORITY\SYSTEM` is a built-in account in Windows operating systems, used by the service control manager. It has the highest level of access in the OS (and can be made even more powerful with Trusted Installer privileges). This account has more privileges than a local administrator account and is used to run most Windows services. It is also very common for third-party services to run in the context of this account by default. The SYSTEM account has the following [privileges](https://docs.microsoft.com/en-us/windows/win32/secauthz/privilege-constants):

|Privilege|Default State|
|---|---|
|SE_ASSIGNPRIMARYTOKEN_NAME|disabled|
|SE_AUDIT_NAME|enabled|
|SE_BACKUP_NAME|disabled|
|SE_CHANGE_NOTIFY_NAME|enabled|
|SE_CREATE_GLOBAL_NAME|enabled|
|SE_CREATE_PAGEFILE_NAME|enabled|
|SE_CREATE_PERMANENT_NAME|enabled|
|SE_CREATE_TOKEN_NAME|disabled|
|SE_DEBUG_NAME|enabled|
|SE_IMPERSONATE_NAME|enabled|
|SE_INC_BASE_PRIORITY_NAME|enabled|
|SE_INCREASE_QUOTA_NAME|disabled|
|SE_LOAD_DRIVER_NAME|disabled|
|SE_LOCK_MEMORY_NAME|enabled|
|SE_MANAGE_VOLUME_NAME|disabled|
|SE_PROF_SINGLE_PROCESS_NAME|enabled|
|SE_RESTORE_NAME|disabled|
|SE_SECURITY_NAME|disabled|
|SE_SHUTDOWN_NAME|disabled|
|SE_SYSTEM_ENVIRONMENT_NAME|disabled|
|SE_SYSTEMTIME_NAME|disabled|
|SE_TAKE_OWNERSHIP_NAME|disabled|
|SE_TCB_NAME|enabled|
|SE_UNDOCK_NAME|disabled|

The SYSTEM account on a domain-joined host can enumerate Active Directory by impersonating the computer account, which is essentially a special user account. If you land on a domain-joined host with SYSTEM privileges during an assessment and cannot find any useful credentials in memory or other data on the machine, there are still many things you can do. Having SYSTEM-level access within a domain environment is nearly equivalent to having a domain user account. The only real limitation is not being able to perform cross-trust Kerberos attacks such as Kerberoasting.

There are several ways to gain SYSTEM-level access on a host, including but not limited to:

- Remote Windows exploits such as EternalBlue or BlueKeep.
- Abusing a service running in the context of the SYSTEM account.
- Abusing SeImpersonate privileges using [RottenPotatoNG](https://github.com/breenmachine/RottenPotatoNG) against older Windows systems, [Juicy Potato](https://github.com/ohpe/juicy-potato), or [PrintSpoofer](https://github.com/itm4n/PrintSpoofer) if targeting [Windows 10/Windows Server 2019](https://itm4n.github.io/printspoofer-abusing-impersonate-privileges/).
- Local privilege escalation flaws in Windows operating systems such as the [Windows 10 Task Scheduler 0day](https://blog.0patch.com/2019/06/another-task-scheduler-0day-another.html).
- PsExec with the `-s` flag

By gaining SYSTEM-level access on a domain-joined host, we will be able to:

- Enumerate the domain and gather data such as information about domain users and groups, local administrator access, domain trusts, ACLs, user and computer properties, etc., using `BloodHound`, and `PowerView`/`SharpView`.
- Perform Kerberoasting / ASREPRoasting attacks.
- Run tools such as [Inveigh](https://github.com/Kevin-Robertson/Inveigh) to gather Net-NTLM-v2 hashes or perform relay attacks.
- Perform token impersonation to hijack a privileged domain user account.
- Carry out ACL attacks.