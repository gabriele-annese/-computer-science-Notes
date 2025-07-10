Nodes are the objects we interact with using BloodHound. These represent Active Directory and Azure objects. However, this section will only focus on Active Directory objects.

When we run SharpHound, it collects specific information from each node based on its type. The nodes that we will find in BloodHound according to version 4.2 are:

|**Node**|**Description**|
|---|---|
|Users|Objects that represent individuals who can log in to a network and access resources. Each user has a unique username and password that allows them to authenticate and access resources such as files, folders, and printers.|
|Groups|Used to organize users and computers into logical collections, which can then be used to assign permissions to resources. By assigning permissions to a group, you can easily manage access to resources for multiple users at once.|
|Computers|Objects that represent the devices that connect to the network. Each computer object has a unique name and identifier that allows it to be managed and controlled within the domain.|
|Domains|A logical grouping of network resources, such as users, groups, and computers. It allows you to manage and control these resources in a centralized manner, providing a single point of administration and security.|
|GPOs|Group Policy Objects, are used to define and enforce a set of policies and settings for users and computers within an Active Directory domain. These policies can control a wide range of settings, from user permissions to network configurations.|
|OUs|Organizational Units, are containers within a domain that allow you to group and manage resources in a more granular manner. They can contain users, groups, and computers, and can be used to delegate administrative tasks to specific individuals or groups.|
|Containers|Containers are similar to OUs, but are used for non-administrative purposes. They can be used to group objects together for organizational purposes, but do not have the same level of administrative control as OUs.|

We will briefly describe each node type and discuss some essential elements to consider. We will share reference links to the official BloodHound documentation, where all the nodes' details, properties, and descriptions are available.

---

## Nodes Elements

The `Node Info` tab displays specific information for each node type, categorized into various areas. We will highlight similarities shared among different types of nodes.

## [Users](https://bloodhound.readthedocs.io/en/latest/data-analysis/nodes.html#users)

The following example shows the sections for the `user` node type:

![BloodHound interface showing node info for PETER@INLANEFREIGHT.HTB. Sections include Overview, Node Properties, Extra Properties, Group Membership, Local Admin Rights, Execution Rights, Outbound Object Control, and Inbound Control Rights.](https://academy.hackthebox.com/storage/modules/69/bh_node_users_elements.jpg)

|**Category**|**Description**|
|---|---|
|Overview|General information about the object.|
|Node Properties|This section displays default information about the node|
|Extra Properties|This section displays some other information about the node, plus all other non-default, string-type property values from Active Directory if you used the –CollectAllProperties flag.|
|Group Membership|This section displays stats about Active Directory security groups the object belongs to.|
|Local Admin Rights|Displays where the object has admin rights|
|Execution Rights|Refers to the permissions and privileges a user, group, or computer has to execute specific actions or commands on the network. This can include RDP access, DCOM execution, SQL Admin Rights, etc.|
|Outbound Object Control|Visualize the permissions and privileges that a user, group, or computer has over other objects in the AD environment|
|Inbound Control Rights|Visualize the permissions and privileges of other objects (users, groups, domains, etc.) over a specific AD object.|

Within the node info tab, we can identify where this user is an administrator, what other object it controls, and more.

![GIF showcasing the Node Info on an object.](https://academy.hackthebox.com/storage/modules/69/bh_node_user2.gif)

**Note:** For the following `node types`, we will only describe the sections that are unique for each one.

## [Computers](https://bloodhound.readthedocs.io/en/latest/data-analysis/nodes.html#Computers)

The following example shows the sections for the `computer` node type:

![BloodHound interface showing node info for WS01.INLANEFREIGHT.HTB. Sections include Overview, Node Properties, Extra Properties, Local Admins, Inbound Execution Rights, Group Membership, Local Admin Rights, Outbound Execution Rights, Inbound Control Rights, and Outbound Object Control.](https://academy.hackthebox.com/storage/modules/69/bh_node_computer.jpg)

|**Category**|**Description**|
|---|---|
|Local Admins|Refers to the users, groups, and computers granted local administrator privileges on the specific computer.|
|Inbound Execution Rights|Display all the objects (users, groups, and computers) that have been granted execution rights on the specific computer|
|Outbound Execution Rights|Refers to the rights of the specific computer to execute commands or scripts on other computers or objects in the network.|

## [Groups](https://bloodhound.readthedocs.io/en/latest/data-analysis/nodes.html#Groups)

The following example shows the sections for the `Groups` node type:

![BloodHound interface showing node info for ITSECURITY@INLANEFREIGHT.HTB. Sections include Overview, Node Properties, Extra Properties, Group Members, Group Membership, Local Admin Rights, Execution Rights, Outbound Object Control, and Inbound Control Rights.](https://academy.hackthebox.com/storage/modules/69/bh_node_groups.jpg)

|**Category**|**Description**|
|---|---|
|Group Members|Refers to the users, groups, and computer members of the specific group.|

## [Domains](https://bloodhound.readthedocs.io/en/latest/data-analysis/nodes.html#Domains)

The following example shows the sections for the `Domains` node type:

![BloodHound interface showing node info for INLANEFREIGHT.HTB. Sections include Overview, Node Properties, Extra Properties, Foreign Members, Inbound Trusts, Outbound Trusts, and Inbound Control Rights.](https://academy.hackthebox.com/storage/modules/69/bh_node_domain_new.jpg)

|**Category**|**Description**|
|---|---|
|Foreign Members|Are users or groups that belong to a different domain or forest than the one currently being analyzed|
|Inbound Trusts|Refers to trust relationships where another domain or forest trusts the current domain. This means that the users and groups from the trusted domain can access resources in the current domain and vice versa.|
|Outbound Trusts|Refers to trust relationships where the current domain or forest trusts another domain. This means that the users and groups from the current domain can access resources in the trusted domain and vice versa.|

In the Domain Overview section, we had an option named `Map OU Structure`. This one is helpful if we want a high-level overview of the Active Directory and how it is organized.

![GIF showcasing the Map OU Structure.](https://academy.hackthebox.com/storage/modules/69/bh_domain_map_ou2.gif)

## [Organizational Units (OUs)](https://bloodhound.readthedocs.io/en/latest/data-analysis/nodes.html#OUs)

The following example shows the sections for the `OU` node type:

![BloodHound interface showing node info for WORKSTATIONS@INLANEFREIGHT.HTB. Sections include Overview, Node Properties, Extra Properties, Affecting GPOs, and Descendant Objects.](https://academy.hackthebox.com/storage/modules/69/bh_node_ou.jpg)

|**Category**|**Description**|
|---|---|
|Affecting GPOs|Refers to the group policy objects (GPOs) affecting the specific OU or container.|
|Descendant Objects|Refers to all objects located within a specific OU or container in the Active Directory (AD) environment.|

## [Containers](https://bloodhound.readthedocs.io/en/latest/data-analysis/nodes.html#Containers)

The following example shows the sections for the `Containers` node type:

![BloodHound interface showing node info for ADMINSDHOLDER@INLANEFREIGHT.HTB. Sections include Overview, Node Properties, Extra Properties, Affecting GPOs, and Descendant Objects.](https://academy.hackthebox.com/storage/modules/69/bh_node_containers.jpg)

## [GPOs](https://bloodhound.readthedocs.io/en/latest/data-analysis/nodes.html#GPOs)

The following example shows the sections for the `GPO` node type:

![BloodHound interface showing node info for DEFAULT DOMAIN POLICY@INLANEFREIGHT.HTB. Sections include Overview, Node Properties, Extra Properties, Affected Objects, and Inbound Control Rights.](https://academy.hackthebox.com/storage/modules/69/bh_node_gpo.jpg)

|**Category**|**Description**|
|---|---|
|Affected Objects|Refers to the users, groups, and computers affected by the specific group policy object (GPO) in the Active Directory (AD) environment.|

If we want to know which objects are affected by a GPO, we can quickly do this from here, as we can see in the following image:

![GIF showcasing the User objects in a GPO.](https://academy.hackthebox.com/storage/modules/69/bh_gpo_affected_object2.gif)
