
A trust is used to establish forest-forest or domain-domain authentication, allowing users to access resources in (or administer) another domain outside of the domain their account resides in.

`A trust creates a link between the authentication systems of two domains.`

Trusts can be transitive or non-transitive.

- A transitive trust means that trust is extended to objects which the child domain trusts.
- In a non-transitive trust, only the child domain itself is trusted.

Trusts can be set up to be one-way or two-way (bidirectional).

- In bidirectional trusts, users from both trusting domains can access resources.
- In a one-way trust, only users in a trusted domain can access resources in a trusting domain, not vice-versa. `The direction of trust is opposite to the direction of access.`

There are several trust types.

----


|**Trust Type**|**Description**|
|---|---|
|Parent-child|Domains within the same forest. The child domain has a two-way transitive trust with the parent domain.|
|Cross-link|A trust between child domains to speed up authentication.|
|External|A non-transitive trust between two separate domains in separate forests that are not already joined by a forest trust. This type of trust utilizes SID filtering.|
|Tree-root|A two-way transitive trust between a forest root domain and a new tree root domain. They are created by design when you set up a new tree root domain within a forest.|
|Forest|A transitive trust between two forest root domains.|

Often, domain trusts are set up improperly and provide unintended attack paths. Also, trusts set up for ease of use may not be reviewed later for potential security implications. M&A can result in bidirectional trusts with acquired companies, unknowingly introducing risk into the acquiring company’s environment. It is not uncommon to perform an attack such as Kerberoasting against a domain outside the principal domain and obtain a user with administrative access within the principal domain.

---

## Enumerating Trust Relationships
Aside from using built-in AD tools such as the Active Directory PowerShell module, `PowerView`/`SharpView` and `BloodHound` can be utilized to enumerate trust relationships, the type of trusts established, as well as the authentication flow.

BloodHound creates a graphical view of trust relationships, which helps both attackers and defenders understand potential trust-related vectors.

`PowerView` can be used to perform a domain trust mapping and provide information such as the type of trust (parent/child, external, forest), as well as the direction of the trust (one-way or bidirectional). All of this information is extremely useful once a foothold is obtained, and you are planning to compromise the environment further.

We can use the function `Get-DomainTrust` to quickly check which trusts exist, the type, and the direction of the trusts.

```powershell-session
PS C:\htb> Get-DomainTrust

SourceName      : INLANEFREIGHT.LOCAL
TargetName      : LOGISTICS.INLANEFREIGHT.LOCAL
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : WITHIN_FOREST
TrustDirection  : Bidirectional
WhenCreated     : 7/27/2020 2:06:07 AM
WhenChanged     : 7/27/2020 2:06:07 AM

SourceName      : INLANEFREIGHT.LOCAL
TargetName      : freightlogistics.local
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : FOREST_TRANSITIVE
TrustDirection  : Bidirectional
WhenCreated     : 7/28/2020 4:46:40 PM
WhenChanged     : 7/28/2020 4:46:40 PM
```

We can use the function `Get-DomainTrustMapping` to enumerate all trusts for our current domain and other reachable domains.

```powershell-session
PS C:\htb> Get-DomainTrustMapping


SourceName      : INLANEFREIGHT.LOCAL
TargetName      : LOGISTICS.INLANEFREIGHT.LOCAL
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : WITHIN_FOREST
TrustDirection  : Bidirectional
WhenCreated     : 7/27/2020 2:06:07 AM
WhenChanged     : 7/27/2020 2:06:07 AM

SourceName      : INLANEFREIGHT.LOCAL
TargetName      : freightlogistics.local
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : FOREST_TRANSITIVE
TrustDirection  : Bidirectional
WhenCreated     : 7/28/2020 4:46:40 PM
WhenChanged     : 7/28/2020 4:46:40 PM

SourceName      : freightlogistics.local
TargetName      : INLANEFREIGHT.LOCAL
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : FOREST_TRANSITIVE
TrustDirection  : Bidirectional
WhenCreated     : 7/28/2020 4:46:41 PM
WhenChanged     : 7/28/2020 4:46:41 PM

SourceName      : LOGISTICS.INLANEFREIGHT.LOCAL
TargetName      : INLANEFREIGHT.LOCAL
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : WITHIN_FOREST
TrustDirection  : Bidirectional
WhenCreated     : 7/27/2020 2:06:07 AM
WhenChanged     : 7/27/2020 2:06:07 AM
```

Depending on the trust type, there are several attacks that we may be able to perform, such as the `ExtraSids` attack to compromise a parent domain once the child domain has been compromised or cross-forest trust attacks such as `Kerberoasting` and `ASREPRoasting` and `SID History` abuse. Each of these attacks will be covered in-depth in later modules.

---
## Attacking Trusts
Organizations set up a trust for various reasons, i.e., ease of management, quickly "plugging in" a new forest obtained through a merger & acquisition, enabling communications between multiple branches of a company, etc. Managed service providers often set up trusts between their domain and those of their clients to facilitate administration.

Some examples for why an organization may set up a trust are:

- Keeping management local to regions. You may see `FLORIDA.INLANEFREIGHT.LOCAL`. By having the `FLORIDA` Domain, it is easy for administrators to ensure those users access resources in their LAN.
    
- Acquisitions - When a company acquires another company and wants a quick way to manage the new equipment without rebuilding anything. They may establish a trust. This can lead to issues, especially if the acquired company has not had regular security assessments performed, has legacy hosts in its environment, has different/no security monitoring controls in place, etc.
    
- Keeping development, testing, etc., logically separated. If `DEVELOPMENT.INLANEFREIGHT.LOCAL` has little privileges over `INLANEFREIGHT.LOCAL`, it is unlikely for beta code to have any adverse effects on production.
    

In all of these cases, Domain Trusts are set up to minimize the number of accounts required. It is much easier to manage multiple domains when you can reference adjacent domains' groups/users. If configured wrong, with lax permissions, etc., a trust relationship can be attacked to further our access, compromising one or many domains in the process.

In our example environment, the domain `INLANEFREIGHT.LOCAL` has a bidirectional trust with the `LOGISTICS.INLANEFREIGHT.LOCAL` domain and is set up as a parent-child trust relationship (both domains within the same forest with `INLANEFREIGHT.LOCAL` acting as the forest root domain.). If we can gain a foothold in either domain, we will be able to perform attacks such as `Kerberoasting` or `ASREPRoasting` across the trust in either direction because our compromised user would be able to authenticate to/from the parent domain, therefore querying any AD objects in the other domain.

Furthermore, if we can compromise the child domain `LOGISTICS.INLANEFREIGHT.LOCAL` we will be able to compromise the parent domain using the `ExtraSids` attack. This is possible because the `sidHistory` property is respected due to a lack of "SID Filtering" protection. Therefore, a user in a child domain with their `sidHistory` set to the Enterprise Admins group (which only exists in the parent domain) is treated as a member of this group, which allows for administrative access to the entire forest.

Our lab environment also shows a bidirectional forest trust between the `INLANEFREIGHT.LOCAL` and `freightlogistics.local` forests, meaning that users from either forest can authenticate across the trust and query any AD object within the partner forest. Aside from attacks such as `Kerberoasting` and `ASREPRoasting`, we may also be able to abuse `SID History` to compromise the trusting forest.

The `SID history` attribute is used in migration scenarios. If a user in one domain is migrated to another domain, a new account is created in the second domain. The original user's SID will be added to the new user's SID history attribute, ensuring that they can still access resources in the original domain.

`SID history` is intended to work across domains but can actually work in the same domain. Using `Mimikatz`, it is possible to perform SID history injection and add an administrator account to the SID History attribute of an account that they control. When logging in with this account, all of the SIDs associated with the account are added to the user's token.

This token is used to determine what resources the account can access. If the SID of a Domain Admin account is added to the SID History attribute of this account, this account will be able to perform DCSync and create golden tickets for further persistence.

This can also be abused across a forest trust. If a user is migrated from one forest to another and SID Filtering is not enabled, it becomes possible to add a SID from the other forest, and this SID will be added to the user's token when authenticating across the trust. If the SID of an account having administrative privileges in Forest A is added to the SID history attribute of an account in Forest B, assuming they can authenticate across the forest, this account will have administrative privileges when accessing resources in the partner forest.

Another common way to cross trust boundaries is by leveraging password re-use. Let's say we compromise the `INLANEFREIGHT.LOCAL` forest and find a user account named `BSIMMONS_ADM` that also exists in the `freightlogistics.local` forest. There is a good chance that this administrator re-uses their password across environments. Also, it is always worth checking for foreign users/foreign group membership. We may find accounts belonging to administrative (or non-administrative) groups in Forest A that are actually part of Forest B and can be used to gain a foothold in the partner forest.

Each of these attacks will be covered in-depth in later modules.


