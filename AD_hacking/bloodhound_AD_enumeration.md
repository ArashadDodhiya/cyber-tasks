# 🩸 BloodHound — Active Directory Enumeration & Attack Path Discovery

**BloodHound** is a powerful open-source tool used by penetration testers and red teamers to **map Active Directory relationships** and discover **attack paths** to Domain Admin and other high-value targets.

> BloodHound uses graph theory to reveal hidden and often unintended relationships in an AD environment that would be nearly impossible to find manually.

---

## 🧠 What BloodHound Does

BloodHound collects and visualizes:

* **User-to-group memberships**
* **Local admin rights**
* **Session data** (who is logged in where)
* **ACL/ACE permissions** (who can modify what)
* **Trust relationships** between domains
* **Kerberos delegation** configurations
* **GPO relationships**

It then uses **graph database (Neo4j)** to find shortest attack paths.

---

## 🏗 Architecture

```text
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  Collector   │────▶│   Neo4j DB   │────▶│  BloodHound  │
│ (SharpHound/ │     │ (Graph Data) │     │   (GUI)      │
│  bloodhound- │     └──────────────┘     └──────────────┘
│  python)     │
└──────────────┘
```

1. **Collector** gathers data from AD
2. **Neo4j** stores the data as a graph
3. **BloodHound GUI** visualizes and queries the graph

---

## 🛠 Data Collection

### Option 1: SharpHound (Windows — On Domain-Joined Machine)

SharpHound is the official collector.

#### Run All Collection Methods

```powershell
SharpHound.exe -c All
```

#### Stealth Mode (Slower but Quieter)

```powershell
SharpHound.exe -c All --stealth
```

#### Specific Collection

```powershell
SharpHound.exe -c Session,LocalAdmin,Group
```

Output: ZIP file containing JSON data.

---

### Option 2: bloodhound-python (Linux — Remote Collection)

For attackers with valid domain credentials but no domain-joined machine.

#### Install

```bash
pip install bloodhound
```

#### Run Collection

```bash
bloodhound-python -u user -p 'Password123' -d corp.local -ns 10.0.0.100 -c All
```

#### With NTLM Hash

```bash
bloodhound-python -u user --hashes :NTLM_HASH -d corp.local -ns 10.0.0.100 -c All
```

Output: JSON files ready for BloodHound import.

---

### Option 3: AzureHound (For Azure AD / Entra ID)

```bash
azurehound -u user@corp.onmicrosoft.com -p Password list --tenant corp.onmicrosoft.com -o azuredata.json
```

---

## 🖥 Setting Up BloodHound

### Step 1: Install Neo4j

```bash
# Linux
sudo apt install neo4j
sudo neo4j start
# Access: http://localhost:7474
# Default creds: neo4j/neo4j (change on first login)
```

### Step 2: Install BloodHound

Download from: [https://github.com/BloodHoundAD/BloodHound/releases](https://github.com/BloodHoundAD/BloodHound/releases)

Or use BloodHound Community Edition (CE):

```bash
docker compose up -d
```

### Step 3: Import Data

1. Open BloodHound GUI
2. Click **Upload Data**
3. Select the ZIP/JSON files from SharpHound
4. Wait for import to complete

---

## 🔍 Key Pre-Built Queries

BloodHound includes powerful pre-built queries:

| Query | Purpose |
| --- | --- |
| Find all Domain Admins | Lists DA group members |
| Find Shortest Paths to Domain Admins | Shows easiest attack paths |
| Find Principals with DCSync Rights | Who can perform DCSync |
| Find Computers where Domain Users are Local Admin | Low-hanging fruit |
| Find All Kerberoastable Users | SPN accounts to target |
| Find AS-REP Roastable Users | Pre-auth disabled accounts |
| Shortest Paths to High Value Targets | Fastest compromise route |
| Find GPOs that modify Local Group Memberships | GPO abuse paths |
| Find Computers with Unconstrained Delegation | Delegation attacks |

---

## 🧠 Custom Cypher Queries

BloodHound uses **Cypher** (Neo4j query language).

### Find All Domain Admins

```cypher
MATCH (u:User)-[:MemberOf*1..]->(g:Group {name:"DOMAIN ADMINS@CORP.LOCAL"})
RETURN u.name
```

### Find Shortest Path from Owned User to DA

```cypher
MATCH p=shortestPath(
  (u:User {owned:true})-[*1..]->(g:Group {name:"DOMAIN ADMINS@CORP.LOCAL"})
)
RETURN p
```

### Find Users with DCSync Rights

```cypher
MATCH (n)-[:DCSync|GetChanges|GetChangesAll]->(d:Domain)
RETURN n.name, labels(n)
```

### Find Kerberoastable Users with Path to DA

```cypher
MATCH (u:User {hasspn:true})
MATCH p=shortestPath((u)-[*1..]->(g:Group {name:"DOMAIN ADMINS@CORP.LOCAL"}))
RETURN u.name, p
```

### Find All Sessions on a Computer

```cypher
MATCH (c:Computer {name:"WS01.CORP.LOCAL"})<-[:HasSession]-(u:User)
RETURN u.name
```

---

## 🔥 Realistic Attack Scenario with BloodHound

### Scenario: From Help Desk to Domain Admin

1. **Attacker** compromises Help Desk user `jsmith`.
2. **Runs SharpHound** from compromised workstation.
3. **Imports data** into BloodHound.
4. **Runs query**: "Shortest Path to Domain Admin"

BloodHound shows:

```text
jsmith → Local Admin on WS05 → ITAdmin has session on WS05 
       → ITAdmin is member of Server Admins 
       → Server Admins → Local Admin on DC01
```

5. Attack path:

   * Dump credentials on WS05 → get ITAdmin hash
   * PTH to servers → escalate
   * Reach DC01 → Domain Admin

**Without BloodHound**, this path would take hours to discover manually.

---

## 📊 BloodHound Node Types

| Node | Description |
| --- | --- |
| User | Domain user account |
| Computer | Domain-joined computer |
| Group | Security group |
| Domain | AD domain |
| GPO | Group Policy Object |
| OU | Organizational Unit |

---

## 📊 BloodHound Edge Types (Relationships)

| Edge | Meaning |
| --- | --- |
| MemberOf | User/Group is member of Group |
| AdminTo | Principal has admin rights on Computer |
| HasSession | User has active session on Computer |
| CanRDP | Principal can RDP to Computer |
| CanPSRemote | Principal can use PS Remoting |
| GenericAll | Full control over object |
| GenericWrite | Can modify object attributes |
| WriteDacl | Can modify object's ACL |
| WriteOwner | Can change object's owner |
| ForceChangePassword | Can reset user's password |
| AddMember | Can add members to group |
| DCSync | Can perform DCSync attack |
| AllowedToDelegate | Kerberos delegation configured |

---

## 🛡 Using BloodHound for Defense (Blue Team)

BloodHound isn't just for attackers! Defenders should:

1. **Run BloodHound regularly** to audit attack paths
2. **Eliminate unnecessary admin rights**
3. **Review ACL/ACE permissions**
4. **Identify risky Kerberos delegation**
5. **Find and fix GPO abuse paths**
6. **Remove unnecessary DCSync rights**

### Defensive Queries

```cypher
-- Find users who can reach DA in 3 hops or less
MATCH p=shortestPath((u:User)-[*1..3]->(g:Group {name:"DOMAIN ADMINS@CORP.LOCAL"}))
WHERE NOT u.name STARTS WITH "ADMIN"
RETURN u.name, length(p)
```

---

## ⚠ Ethical Reminder

Everything discussed:

* Legal only in authorized labs
* Legal with written penetration testing authorization
* Illegal otherwise

---

## 📚 What to Learn Next

* 🔄 Combining BloodHound with lateral movement techniques
* 🧪 Building an AD lab for BloodHound practice
* 🛡 BloodHound for blue team hardening
* 📊 Advanced Cypher queries for red teaming
