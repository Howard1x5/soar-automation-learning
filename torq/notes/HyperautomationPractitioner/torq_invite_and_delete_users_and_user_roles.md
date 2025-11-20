# Workspace & User Management – Invite/Delete Users & RBAC Roles

## 1. Summary

This section covers two foundational parts of Torq workspace administration:

1. How to invite and delete users directly in Torq
2. Understanding Torq’s RBAC model and built-in user roles

These concepts appear throughout the platform and tie directly into SSO integrations, access allocation, and workflow development permissions.

---

## 2. Inviting & Deleting Users

Torq supports several ways to add users:

* Manual invitation (email-based; one user at a time)
* SSO group mapping (Azure AD, Okta, OneLogin, etc.)
* Torq API (programmatic onboarding)

Manual invites are the simplest method and require no IdP integration.

### 2.1 Inviting Users (Email-Based)

1. Navigate to **Settings → Users**.
2. Click **Invite**.
3. Enter the user’s **email address**.
4. Assign a **Torq role** (viewer, operator, creator, contributor, owner).
5. Send the invite.

Notes:

* Torq does **not** support bulk invitations through the UI.
* Invited users authenticate with email-based login unless SSO is active and enforced.

### 2.2 Deleting Users

1. Go to **Settings → Users**.
2. Locate the user in the list.
3. Click the **three-dot menu** beside their name.
4. Select **Delete**.

Deleting a user removes their access but preserves workflow and audit history.

### 2.3 Required Permissions

Only users with the **Owner** role can:

* Invite users
* Delete users
* Modify users
* Configure or manage SSO

---

## 3. Torq RBAC – User Roles

Torq uses a role-based access control system backed by claim-based access policies. Each user is assigned one high-level role that defines their permissions across workflows, integrations, secrets, events, and user management.

Built-in roles:

* Viewer
* Operator
* Creator
* Contributor
* Owner

**Roles are scoped per account**—a user can have different roles in different Torq accounts.

### 3.1 Custom Roles

Custom roles can be created by contacting Torq Support. This allows highly granular access control for large or regulated organizations.

---

## 4. Role Definitions

### Viewer

Read-only access.

| Permission              | Scope              |
| ----------------------- | ------------------ |
| View existing workflows | `playbook.get`     |
| List workflows          | `playbook.list`    |
| View activity logs      | `event.read`       |
| View integrations       | `integration.read` |

---

### Operator

Viewer + ability to run workflows.

| Permission               | Scope                             |
| ------------------------ | --------------------------------- |
| View/list workflows      | `playbook.get` / `playbook.list`  |
| Run workflows            | `playbook.execute`                |
| View logs/integrations   | `event.read` / `integration.read` |
| View personal API keys   | `apikey.read`                     |
| Create personal API keys | `apikey.write`                    |

---

### Creator

Operator + ability to create and modify workflows and integrations.

| Permission               | Scope               |
| ------------------------ | ------------------- |
| All Operator permissions | –                   |
| Create workflows         | `playbook.write`    |
| Create integrations      | `integration.write` |
| View account member list | `user.read`         |
| Modify secrets           | `secret.write`      |

---

### Contributor

Creator + ability to publish workflows.

| Permission              | Scope              |
| ----------------------- | ------------------ |
| All Creator permissions | –                  |
| Publish workflows       | `playbook.publish` |

---

### Owner

Highest-level permission. Full administrative access.

| Permission                  | Scope           |
| --------------------------- | --------------- |
| All Contributor permissions | –               |
| Modify users                | `user.write`    |
| Create support tickets      | `support.write` |
| View audit logs             | `audit.read`    |
| Manage secrets              | `secret.write`  |

---

## 5. Operational Notes

* Maintain **1–2 break-glass local admin accounts** (non-SSO) for recovery if the IdP is unavailable.
* If enforcing SSO-only access, delete any pre-SSO invited users so they cannot bypass SSO.
* Claim mapping (roles or groups) in SSO is **order-sensitive**—the first match wins.
* Review RBAC assignments regularly, especially in environments with turnover or multiple developers.
* Assign roles carefully:

  * Operators for junior SOC analysts
  * Creators for automation development
  * Contributors and Owners for team leads and platform managers

---

## 6. Cross-Platform Comparison (SOAR Context)

Although these notes describe Torq, the same RBAC principles apply across other SOAR platforms.

### Shuffle SOAR

* Role assignment typically happens at the reverse proxy/identity layer (Keycloak, Dex, etc.).
* Common separation of Viewer / Operator / Editor / Admin.

### Cortex XSOAR

* SSO via SAML/OIDC, roles mapped from IdP groups.
* Built-in roles: Analyst, Read-Only, Administrator, etc.
* Supports custom fine-grained roles.

### Splunk SOAR (Phantom)

* Supports SAML/OIDC and local accounts.
* Roles: Analyst, Administrator, Approver, plus custom roles.
* Same pattern: groups → roles → permissions.

---

## 7. Quick Reference Summary (No Checkboxes)

When managing users in Torq:

* Use an **Owner** account for inviting, deleting, or modifying users.
* Assign the correct role at invitation time based on job function.
* Remove pre-SSO invited users if transitioning to SSO-only authentication.
* Maintain non-SSO break-glass accounts for reliability and recovery.
* Verify user roles via **Settings → Users** after changes.
* Review RBAC assignments periodically to prevent privilege drift.
* When using SSO group or role claims, test each mapping to ensure proper Viewer/Operator/Owner role assignment.