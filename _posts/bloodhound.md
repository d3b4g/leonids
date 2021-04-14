#### Active Directory Enumeration - BloodHound

In this post i'm going to take to you through demostration of working with bloodhound to enumerate active directory. Bloodhound automates enumerating complex attack paths we used to perform with powershell or anyother tool. 

#### BloodHound

BloodHound uses graph theory to reveal the hidden and often unintended relationships within an Active Directory environment. Attackers can use BloodHound to easily identify highly complex and often unintended attack paths that would otherwise be impossible to identify.

#### Graph Theory
BloodHounds true power lies withing the Neo4j database that it uses. Neo4j is a graph database that can easily discover relationships and calculate the shortest paths between objects using its links.

#### Getting Started
Using BloodHound is straight forward and the tool is well docummented. So i will skip whole installation and configuration part.


#### Data Collection 

> Installation pip3 install bloodhound

Bloodhound collect data by using ingestors.

### Execution
Once data is ingested, we can querry or graph the data to find attack paths

#### Chypher Querries 



#### concluson

