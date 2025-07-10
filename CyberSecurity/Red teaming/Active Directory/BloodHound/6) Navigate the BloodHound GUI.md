The BloodHound GUI is the primary location for analyzing the data we collected using SharpHound. It has a user-friendly interface to access the data we need quickly.

We will explore BloodHound graphical interface and see how we can take advantage of it.

**Note:** See [BloodHound Official Documentation](https://bloodhound.readthedocs.io/en/latest/data-analysis/bloodhound-gui.html#the-bloodhound-gui) page for more information.

---

## Getting Access to BloodHound Interface

In the section [BloodHound Setup and Installation](https://academy.hackthebox.com/module/69/section/2073), we discuss connecting to BloodHound GUI. We need to ensure the neo4j database is running, execute the BloodHound application, and log in with the username and password we defined.

When we log in successfully, BloodHound will perform the query: `Find all Domain Admins` and display the users that belong to the group.

![BloodHound interface showing a graph of user relationships. PETER@INLANEFREIGHT.HTB is a member of ITSECURITY@INLANEFREIGHT.HTB. Other users like DAVID@INLANEFREIGHT.HTB and JULIO@INLANEFREIGHT.HTB are members of DOMAIN ADMINS@INLANEFREIGHT.HTB.](https://academy.hackthebox.com/storage/modules/69/bh_access_interface.jpg)

This area is known as the `Graph Drawing Area`, where BloodHound displays nodes and their relationship using edges. We can interact with the graph, move objects, zoom in or zoom out (with the mouse scroll or the bottom right buttons), click on nodes to see more information, or right-click them to perform different actions.

![GIF showcasing the interaction with objects.](https://academy.hackthebox.com/storage/modules/69/bh_gif_test4.gif)

Let's describe those options:

|**Command**|**Description**|
|---|---|
|Set as Starting Node|Set this node as the starting point in the pathfinding tool. Click this, and we will see this node’s name in the search bar. Then, we can select another node to target after clicking the pathfinding button.|
|Set as Ending Node|Set this node as the target node in the pathfinding tool.|
|Shortest Paths to Here|This will perform a query to find all shortest paths from any arbitrary node in the database to this node. This may cause a long query time in neo4j and an even longer render time in the BloodHound GUI.|
|Shortest Paths to Here from Owned|Find attack paths to this node from any node you have marked as owned.|
|Edit Node|This brings up the node editing modal, where you can edit current properties on the node or even add our custom properties to the node.|
|Mark Group as Owned|This will internally set the node as owned in the neo4j database, which you can then use in conjunction with other queries such as “Shortest paths to here from Owned”.|
|Mark/Unmark Group as High Value|Some nodes are marked as “high value” by default, such as the domain admins group and enterprise admin group. We can use this with other queries, such as “shortest paths to high-value assets”.|
|Delete Node|Deletes the node from the neo4j database.|

The `Graph Drawing Area` lets us also interact with `Edges`. They represent the link between two nodes and help us understand how we will move from one object to another. As BloodHound has many edges is challenging to keep track of how we can abuse every single one. The BloodHound team included a `help menu` under edges, where we can see information, examples, and references on how to abuse every single edge.

![GIF showcasing the shortest path to the Domain Admins group.](https://academy.hackthebox.com/storage/modules/69/bh_edges_helpmenu2.gif)

## Search Bar

The search bar is one of the elements of BloodHound that we will use the most. We can search for specific objects by their name or type. If we click on a node we searched, its information will be displayed in the node info tab.

If we want to search for a specific type, we can prepend our search with node type, for example, `user:peter` or `group:domain`. Let's see this in action:

![GIF showcasing the search bar.](https://academy.hackthebox.com/storage/modules/69/bh_search_bar_node_info2.gif)

Here's the complete list of node types we can prepend to our search:

Active Directory

- Group
- Domain
- Computer
- User
- OU
- GPO
- Container

Azure

- AZApp
- AZRole
- AZDevice
- AZGroup
- AZKeyVault
- AZManagementGroup
- AZResourceGroup
- AZServicePrincipal
- AZSubscription
- AZTenant
- AZUser
- AZVM

## Pathfinding

Another great feature in the search bar is `Pathfinding`. We can use it to find an attack path between two given nodes.

For example, we can look for an attack path from `ryan` to `Domain Admins`. The path result is:

`Ryan > Mark > HelpDesk > ITSecurity > Domain Admins`

This path contains the edge `ForceChangePassword`; let's say we want to avoid changing users' passwords. We can use the filter option, uncheck `ForceChangePassword`, and search for the path without this edge. The result is:

`Ryan > AddSelf > Tech_Support > Testinggroup > HelpDesk > ITSecurity > Domain Admins`

![GIF showcasing the setting of a Start node and Target node.](https://academy.hackthebox.com/storage/modules/69/bh_pathfinding2.gif)

## Upper Right Menu

We will find various options to interact with in the top right corner. Let's explain some of these options:

![GIF showcasing the upper right menu and the various options.](https://academy.hackthebox.com/storage/modules/69/bh_upper_right_menu_3.gif)

- `Refresh`: reruns the previous query and shows the results
    
- `Export Graph`: Saves the current graph in JSON format so it can be imported later. We can also save the current graph as a picture in PNG format.
    
- `Import Graph`: We can display the JSON formatted graph we exported.
    

![GIF showcasing the Export Graph functionality.](https://academy.hackthebox.com/storage/modules/69/bh_export_png2.gif)

- `Upload Data`: Uploads SharpHound, BloodHound.py, or AzureHound data to Neo4j. We can select the upload data with the upload button or drag and drop the JSON or zip file directly into the BloodHound window.

![GIF showcasing the Upload functionality.](https://academy.hackthebox.com/storage/modules/69/bh_upload_data_3.gif)

**Note:** When we upload, BloodHound will add any new data but ignore any duplicated data.

**Note:** Zip files cannot be password protected from being uploaded.

- `Change Layout Type`: Changes between hierarchical or force-directed layouts.
    
- `Settings`: Allows us to adjust the display settings for nodes and edges, enabling query to debug mode, low detail mode, and dark mode.
    
    - `Node Collapse Threshold`: Collapse nodes at the end of paths that only have one relationship. 0 to Disable, Default 5.
    - `Edge Label Display`: When to display edge labels. If Always Display, edges such as MemberOf, Contains, etc., will always be said.
    - `Node Label Display`: When to display node labels. If Always Display, node names, user names, computer names, etc., will always be displayed.
    - `Query Debug Mode`: Raw queries will appear in Raw Query Box. We will discuss more on this in the [Cypher Queries](https://academy.hackthebox.com/module/69/section/2081) section.
    - `Low Detail Mode`: Graphic adjustments to improve performance.
    - `Dark Mode`: Enable Dark mode for the interface.

![Settings window with options: Node Collapse Threshold set to 5, Edge and Node Label Display set to "Always Display." Query Debug Mode and Low Detail Mode are unchecked. Dark Mode is checked.](https://academy.hackthebox.com/storage/modules/69/bh_settings.jpg)

- `About`: Shows information about the author and version of the software.

## Shortcuts

There are four (4) shortcuts that we can take advantage from:

|**Shortcut**|**Description**|
|---|---|
|`CTRL`|Pressing CTRL will cycle through the three different node label display settings - default, always show, always hide.|
|`Spacebar`|Pressing the spacebar will bring up the spotlight window, which lists all currently drawn nodes. Click an item in the list, and the GUI will zoom into and briefly highlight that node.|
|`Backspace`|Pressing backspace will return to the previous graph result rendering. This is the same functionality as clicking the Back button in the search bar.|
|`S`|Pressing the letter s will toggle the expansion or collapse of the information panel below the search bar. This is the same functionality as clicking the More Info button in the search bar.|

## Database Info
The BloodHound Database Info tab gives users a quick overview of their current BloodHound database status. Here, users can view the database's number of sessions, relationships, ACLs, Azure objects and relationships, and Active Directory Objects.

Additionally, there are several options available for managing the database.:

| **Option**               | **Description**                                                                                                                                                                              |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Clear Database`         | Lets users completely clear the database of all nodes, relationships, and properties. This can be useful when starting a new assessment or dealing with outdated data.                       |
| `Clear Sessions`         | Lets users clear all saved sessions from the database.                                                                                                                                       |
| `Refresh Database Stats` | Updates the displayed statistics to reflect any changes made to the database.                                                                                                                |
| `Warming Up Database`    | Is a process that puts the entire database into memory, which can significantly speed up queries. However, this process can take some time to complete, especially if the database is large. |