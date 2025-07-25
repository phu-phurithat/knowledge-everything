# Access Control

Grafana has a **flexible role and access control system** based on:

1. **Organizations**
2. **Users**
3. **Teams**
4. **Roles**

Here’s a clear breakdown of how they work together 👇

***

### 🏢 1. **Organization (Org)**

* A **logical container** for users, dashboards, data sources, etc.
* Users **belong to one org at a time** (but can be part of many)
* Each org has **separate resources** (dashboards, data sources, teams)
* Useful for **multi-tenant** setups (e.g., Company A and B use the same Grafana)

> 📝 When a user logs in, they land in their **default org**.

***

### 👤 2. **User**

* A user account (local or external auth: LDAP, OAuth, etc.)
* Can belong to **multiple orgs** but only one at a time
* Has a **role** within each org (Viewer, Editor, Admin)

#### Roles per user per org:

<table><thead><tr><th width="178">Role</th><th>Permissions</th></tr></thead><tbody><tr><td><strong>Viewer</strong></td><td>View dashboards, alerts, explore data</td></tr><tr><td><strong>Editor</strong></td><td>Create and edit dashboards, alerts</td></tr><tr><td><strong>Admin</strong></td><td>Manage users, teams, data sources, settings</td></tr></tbody></table>

> 🛡️ **Only Admins** can change settings or invite others in the org.

***

### 👥 3. **Team**

* A group of users **within an organization**
* Useful for applying **folder-level permissions** at scale

#### Example:

* Create team `dev-team` in Org A
* Add 5 users to the team
* Set dashboard folder permissions so `dev-team` can **edit**, others can only **view**

> 📁 Teams let you control access to **specific resources**, like dashboards or folders, in an efficient way.

***

### 🧠 How They Work Together

<table><thead><tr><th width="154">Concept</th><th>Description</th></tr></thead><tbody><tr><td><strong>User</strong></td><td>Belongs to org(s), may belong to team(s), has a role in each org</td></tr><tr><td><strong>Team</strong></td><td>Group of users, used for folder/dash-level access control</td></tr><tr><td><strong>Role</strong></td><td>Defines what the user or team can do within the org</td></tr><tr><td><strong>Org</strong></td><td>Scope container: separate dashboards, users, datasources</td></tr></tbody></table>

***

### 🛠️ Example Setup

Imagine you run monitoring for multiple teams:

#### 👇 DevOps Org

* Team: `infra-team`
  * Members: Alice, Bob
  * Role: **Editor**
  * Folder Access: Edit `infra/`, View others

#### 👇 Data Org

* Team: `data-team`
  * Members: Carol, Dave
  * Role: **Viewer**
  * Can only view dashboards

***

### 🚦Pro Tips

* You can **limit dashboard access** by folder:
  * Grafana UI → Configuration → Folders → Permissions
* Teams make this scalable: no need to configure per-user permissions
* External auth (like LDAP/OAuth) can **map groups to teams/roles** automatically
