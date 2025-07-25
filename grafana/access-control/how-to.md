# How to

**Goal:** Manage user access in Grafana using **organizations, roles, and teams**

***

### 📌 1. Concepts Summary

<table><thead><tr><th width="142">Term</th><th>Meaning</th></tr></thead><tbody><tr><td><strong>Org</strong></td><td>Logical tenant (separate dashboards, datasources, teams)</td></tr><tr><td><strong>User</strong></td><td>An individual login account</td></tr><tr><td><strong>Role</strong></td><td>What the user can do in the org (Viewer, Editor, Admin)</td></tr><tr><td><strong>Team</strong></td><td>Group of users inside an org, used for folder-level access control</td></tr></tbody></table>

***

### ✅ 2. Create a New Organization

#### Via UI:

1. Go to **Administration → General → Organization**
2. Name it (e.g., `Engineering`, `DevOps`)
3. Click **New org**

#### Via CLI:

```bash
grafana-cli admin create-org --name="Engineering"
```

***

### ✅ 3. Create Users (or Invite)

#### Local Auth:

1. Go to **Administration → General → Users and access → Users**
2. Click **+ New User**
3. Fill in:
   * Login
   * Email
   * Password
   * Org (optional — default is "Main Org.")

#### External Auth (OAuth, LDAP, etc.)

* Users are auto-created at first login.
* Their org and role can be auto-assigned (see section 6).

***

### ✅ 4. Assign Users to an Org

#### UI:

1. Go to **Administration → General → Users and access → Users**
2. Click the user
3. Use **"Add user to organization"**
4. Assign a **role** (Viewer, Editor, Admin)

***

### ✅ 5. Create a Team and Assign Permissions

#### Step-by-step:

1. Go to **Configuration → Teams**
2. Click **New Team**
   * Example: `dev-team`
3. Add users to the team
4. Optionally link to external group (e.g., `GitHub:devs`)

***

#### Grant Folder Permissions to a Team

1. Go to **Configuration → Folders**
2. Select a folder (e.g., `Dev Logs`)
3. Click **Permissions**
4. Add your team
5. Assign access level:
   * **View**
   * **Edit**
   * **Admin**

***

### 🔐 6. Optional: Set Default Org and Role for New Users

Edit `grafana.ini` (or via Helm values):

```ini
[auth]
auto_assign_org = true
auto_assign_org_role = Viewer   # or Editor/Admin
```

For external auth (e.g., OAuth, LDAP), you can also map users to orgs/teams.

***

### 📋 7. Example Access Plan

| Org         | Team      | Role   | Folder Access     |
| ----------- | --------- | ------ | ----------------- |
| Engineering | dev-team  | Editor | Dev Dashboards    |
| Security    | sec-team  | Viewer | Audit Dashboards  |
| Main Org.   | all-users | Viewer | Read-only general |

***

### 🧼 8. Cleanup & Tips

* Periodically audit orgs/users:\
  **Server Admin → Users / Orgs**
* Restrict unused "Main Org" access
* Use **Teams** instead of assigning per-user permissions

***

### 🛠️ Bonus: Provision Users/Teams via API

You can script user/team/org creation using the **Grafana HTTP API**:

* User API
* Org API
* Team API
