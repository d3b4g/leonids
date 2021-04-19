#### Active Directory Enumeration - BloodHound

In this post i'm going to take to you through demostration of working with bloodhound to enumerate active directory. Bloodhound automates enumerating complex attack paths we used to perform with powershell or anyother tool. 

#### BloodHound

BloodHound uses graph theory to reveal the hidden and often unintended relationships within an Active Directory environment. Attackers can use BloodHound to easily identify highly complex and often unintended attack paths that would otherwise be impossible to identify.

#### Graph Theory
BloodHounds true power lies withing the Neo4j database that it uses. Neo4j is a graph database that can easily discover relationships and calculate the shortest paths between objects using its links.

![source-01](/img/2.png){: .align-left}

The Property Graph Model
Property graphs contain nodes, and relationships.

**Nodes**
Nodes are the main data points of interest in a property graph.

**Relationships**
Relationships are the directed connections between two nodes.

**Properties**
Properties are attributes that belong to nodes and/or edges.

**Labels**
Labels are used to group data together.

#### Getting Started
Using BloodHound is straight forward and the tool is well docummented. So i will skip whole installation and configuration part.


#### Data Collection 

> Installation pip3 install bloodhound

Bloodhound collect data from Active Directory by using ingestors. Ingesters queries are performed  via:
- LDAP
- SMB RPC

Data collected with the SharpHound ingestor, and are saved in json format in different files. Import these files to BloodHound UI for processsing 

### Execution
Once data is ingested, we can querry or graph the data to find attack paths

#### Chypher Querries 

Neo4J have its own queery language


#### concluson

BloodHound gives a graphical and easy to handle interface to visualize otherwise complex relationships in Active Directory. 
