# Torq SSO Configuration – Azure AD and Okta

## 1. Summary

These notes capture how Torq handles Single Sign-On (SSO) and how to configure it with:

* Azure Active Directory (OpenID Connect / OIDC)
* Okta (SAML 2.0)

Focus areas:

* Supported SSO protocols and account types
* How Torq maps IdP groups/roles to Torq roles
* Configuration steps for Azure AD and Okta
* Operational gotchas and best practices
* How this pattern generalizes to other SOAR platforms (Shuffle, XSOAR, Splunk SOAR)

---

## 2. Torq SSO Capabilities – Overview

### 2.1 Protocols and IdPs

Torq supports:

* **OpenID Connect (OIDC)**

  * Code flow
  * Implicit grant
* **SAML 2.0**

Common IdPs:

* Microsoft Azure Active Directory
* Okta
* OneLogin
* Others that speak OIDC/SAML 2.0

### 2.2 Account types that can be bound to SSO

Torq can tie SSO to:

* Google accounts
* GitHub accounts
* Local Torq username/password accounts

You can mix SSO users and non-SSO users in the same tenant.

### 2.3 Important behavior and constraints

* **Domain binding**

  * Torq assumes the SSO “organization” domain is the same as the email domain of the admin setting it up.
  * Example: `admin@mycompany.com` can configure SSO for `mycompany.com`.
  * To configure SSO for a different domain, you need Torq Support.

* **Pre-SSO invited users**

  * Users invited via email before SSO is enforced can still log in with their local credentials.
  * If you want *all* access via SSO only, remove those invited users and rely solely on SSO.

* **Keep break-glass accounts**

  * Recommended: maintain **1–2 non-SSO accounts** with strong auth for recovery if the IdP is unavailable or misconfigured.

* **Claims/role changes**

  * If you want to change claims/roles:

    * First add the new claim mapping in Torq.
    * Only then remove or change the claim on the IdP side.
  * Otherwise you can lock yourself out.

---

## 3. Torq SSO Settings – General Layout

Inside Torq:

1. Go to **Settings → SSO**.
2. You’ll see:

   * **SSO Login Settings**: overall status, IdP configuration.
   * **IdP Connection**: whether the connection is enabled/disabled.
   * **Mapping rules**: how IdP attributes (roles, groups, email, etc.) map to Torq roles.

Key points:

* The IdP connection has an **enable/disable toggle**. If SSO suddenly “stops working,” verify this wasn’t left disabled in testing.
* Mapping rules are **evaluated top-down**:

  * First match wins.
  * Place the most privileged mapping (e.g., Owner) at the top, then Viewer and lower roles.

---

## 4. Azure Active Directory (OIDC) – Configuration Walkthrough

### 4.1 High-level flow

1. Create an **App Registration** in Azure AD.
2. Configure redirect URI(s) for Torq.
3. Enable OIDC flows and generate client secret.
4. Define **App Roles** in AAD and assign them to groups.
5. Grant **User.Read** permissions and admin consent.
6. Configure Torq SSO settings with:

   * Tenant/Issuer URL
   * Client ID
   * Client Secret
   * Scopes and claim mappings (roles → Torq roles).
7. Test with sample users and confirm role mappings.

### 4.2 Azure AD – App registration basics

In Azure AD:

1. **Create app registration**

   * Name: `Torq` (or similar).
   * Redirect URI:

     * Use the correct Torq base (US/EU) per documentation.
     * URI is provided by Torq’s SSO docs.

2. **Platform configuration**

   * Enable OIDC flow used by Torq:

     * Code or implicit flow (as per docs).
   * Under **Authentication**:

     * Enable **ID tokens** (for implicit/hybrid if required).

3. **Client credentials**

   * Note down:

     * **Application (client) ID**
     * **Directory (tenant) ID**
   * Generate a **Client Secret** and store it securely (used in Torq config).

### 4.3 Azure AD – App roles and permissions

1. **App roles**

   * Define roles that mirror Torq roles:

     * Example display names and values:

       * Display name: `TorqOwner` → Value: `TorqOwner`
       * Display name: `TorqViewer` → Value: `TorqViewer`
   * These values will be used as claims in the token.

2. **API permissions**

   * Add `User.Read` (or equivalent minimal graph permission required by Torq).
   * Click **Grant admin consent** for the organization.

3. **Enterprise application assignments**

   * Go to **Enterprise applications** → your Torq app.
   * Assign groups to app roles:

     * Group `TorqOwners` → Role `TorqOwner`
     * Group `TorqViewers` → Role `TorqViewer`
   * Add users to those AAD groups as needed.

### 4.4 Torq SSO – Azure AD settings

In Torq SSO settings:

1. **Connection details**

   * Provider type: OIDC / Azure AD.
   * Client ID: from app registration.
   * Client Secret: from app registration.
   * URLs based on tenant ID (issuer, authorization endpoint, token endpoint) per Torq docs.

2. **Claim mapping (roles to Torq roles)**

Example mappings:

* Mapping rule #1 (highest privilege):

  * Claim type: `roles`
  * Claim value: `TorqOwner`
  * Torq role: `owner`

* Mapping rule #2:

  * Claim type: `roles`
  * Claim value: `TorqViewer`
  * Torq role: `viewer`

Order matters: `TorqOwner` mapping should be above `TorqViewer` because first match wins.

3. **Test logins**

* Log in to Torq using **Single Sign-On** as:

  * User in `TorqViewers` group → should become Torq `viewer`.
  * User in `TorqOwners` group → should become Torq `owner`.
* In Torq, filter by SSO users and confirm roles match expectations.

---

## 5. Okta (SAML 2.0) – Configuration Walkthrough

### 5.1 High-level flow

1. In Okta, add a **Torq app** from the app catalog (SAML-based).
2. Configure SAML settings and decide which groups to send as attributes.
3. Use Okta’s **SAML Setup Instructions** to collect:

   * SSO URL
   * Issuer URL
   * Public certificate
4. Assign groups and users in Okta.
5. Configure Torq with SAML metadata from Okta and set claim mappings (e.g., groups → Torq roles).
6. Test with users and confirm viewer/owner mapping.

### 5.2 Okta app setup

In Okta:

1. **Create application**

   * From the app catalog, search for `Torq` and add integration.
   * Hide app from user dashboards if desired.

2. **SAML settings**

   * On **Single sign-on** tab:

     * Specify which **groups** are passed in the SAML assertion.

       * Example: send all groups where name starts with `Torq` (capital T).

3. **SAML setup instructions**

   * Use the “View SAML setup instructions” page in Okta:

     * SSO URL
     * Issuer URL
     * X.509 certificate
     * Screenshots and field labels for Torq-side mapping.

4. **Assignments**

   * Assign app to groups/users:

     * Group `TorqViewer`
     * Group `TorqOwner`
   * Ensure demo users are members:

     * `SSODemoUser3` in `TorqViewer`.
     * `SSODemoUser4` in `TorqOwner`.

### 5.3 Torq SSO – Okta settings

1. **Connection configuration**

   * Choose protocol: `SAML 2.0`.
   * Fill:

     * SSO URL (Okta SAML endpoint).
     * Issuer URL.
     * Public certificate.

2. **Claims / mapping**

Instead of roles claim, use **groups**:

* Rule #1 (owner):

  * Claim type: `groups`
  * Claim value: `TorqOwner`
  * Torq role: `owner`

* Rule #2 (viewer):

  * Claim type: `groups`
  * Claim value: `TorqViewer`
  * Torq role: `viewer`

As with Azure AD, order is important: owner mapping goes first.

3. **Testing**

* Log in to Torq via SSO as:

  * User 3 (group `TorqViewer`) → should map to role `viewer`.
  * User 4 (group `TorqOwner`) → should map to role `owner`.
* Confirm in Torq user list that the roles are correct.

---

## 6. Operational Best Practices and Gotchas

### 6.1 Invite behavior and SSO-only access

* Any user invited before SSO is enforced can still use non-SSO login.
* To enforce SSO-only:

  * Remove those invited users from Torq.
  * Rely on SSO-provisioned accounts and role mappings.

### 6.2 Break-glass local accounts

* Maintain 1–2 local (non-SSO) admin accounts with strong passwords and MFA if available.
* Use them only when:

  * IdP is down.
  * SSO config breaks and you need console access to fix.

### 6.3 Change management for claims

* When changing roles/groups/claims:

  * First add new mapping rules in Torq.
  * Then change the IdP (add users to new groups, deprecate old roles).
* Avoid removing a claim in the IdP before Torq understands the replacement, or you can lock out entire roles.

### 6.4 Multiple workspaces

* If you have multiple Torq workspaces, confirm:

  * SSO is enabled for the correct workspace.
  * You are not testing in a workspace where SSO is disabled.
* The on/off toggle is easy to forget during testing.

---

## 7. How this maps to other SOAR / enterprise tools

Although this is specific to Torq, the pattern is identical in other SOAR platforms.

### 7.1 Common pattern

All tools roughly implement:

1. **IdP app / integration** (Azure AD, Okta, etc.).
2. **Attribute mapping** from IdP to tool roles:

   * Groups → roles.
   * Claims → permissions.
3. **Top-down evaluation** of mappings (first match wins).
4. **Break-glass local accounts** kept outside SSO.

### 7.2 Shuffle SOAR

* Shuffle itself relies on external auth for SSO:

  * Common approach: front it with a reverse proxy (e.g., NGINX, Keycloak, Dex) that handles SSO.
* Roles and group mapping happen either:

  * In the proxy (groups → claims → RBAC).
  * Or inside Shuffle if it supports RBAC linked to headers/claims.
* Same principles apply:

  * One identity source, group-to-role mapping, break-glass account outside SSO.

### 7.3 Cortex XSOAR

* SSO via:

  * SAML (Okta, ADFS, Azure AD).
  * OIDC (in newer versions).
* Configuration steps:

  * Upload IdP metadata or configure endpoints.
  * Map SAML group attributes to XSOAR roles (analyst, admin, etc.).
  * Keep at least one local admin account.
* Takeaway: nearly identical to Torq’s group/role mapping flow.

### 7.4 Splunk SOAR (Phantom / Splunk SOAR)

* Supports:

  * SAML, LDAP, and sometimes OIDC via external auth systems.
* RBAC:

  * Roles defined in Splunk SOAR.
  * Group/attribute mapping from IdP to roles.
* Same operational considerations:

  * Careful with claim changes.
  * Keep a local admin credential for recovery.

---

## 8. Practical checklist for implementing SSO in Torq

When configuring SSO for a real tenant:

1. **Plan roles and groups**

   * Decide on Torq roles and matching IdP groups/roles.
   * Example: `TorqOwners`, `TorqViewers`, `TorqEditors`.

2. **Set up IdP app**

   * Create app (Azure AD / Okta).
   * Configure redirect URIs correctly based on Torq region.
   * Generate client ID/secret or SAML metadata.

3. **Define claims**

   * OIDC: roles claim (`roles`) or groups claim.
   * SAML: group attribute or role attribute.
   * Ensure app roles or group claims are actually present in the token.

4. **Assign users/groups**

   * Map groups to app roles.
   * Add test users to each group.

5. **Configure Torq**

   * Enter client ID/secret or SAML URLs/cert.
   * Create mapping rules:

     * Most privileged role first, then less-privileged roles.
   * Verify SSO is enabled.

6. **Test**

   * Log in as each role.
   * Confirm:

     * Access to Torq.
     * Correct Torq role assignment.
   * Verify SSO user list in Torq.

7. **Harden**

   * Remove legacy invited users that bypass SSO if required.
   * Ensure 1–2 local break-glass accounts exist with strong security.
   * Document the configuration (this file) in source control.