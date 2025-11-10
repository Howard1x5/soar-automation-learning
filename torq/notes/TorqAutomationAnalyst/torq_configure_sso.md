Configure SSO in Torq (Azure AD & Okta/SAML/OpenID)
1. Overview

    Torq supports multiple SSO providers:

        Azure Active Directory (AAD)

        Okta (SAML / OpenID Connect)

        Other SAML 2.0 / OpenID providers

    SSO in Torq controls authentication and role-based authorization.

    Common use cases: centralized identity management, group-based role mapping, and cross-workspace single sign-on.

2. Azure Active Directory SSO Setup
Preparation

    Check Documentation First

        Torq provides provider-specific setup guides (US vs EU tenants differ in endpoints).

    Verify SSO is Enabled in Torq UI

        If disabled, users cannot log in via SSO.

Azure AD Steps

    Create New App Registration in Azure Portal:

        Select correct platform (US/EU).

        Note Client ID and Directory/Tenant ID.

    Configure Authentication:

        Enable Implicit Grant for ID tokens.

        Create a Client Secret (note value — shown only once).

    App Roles Setup:

        Define Display Name + Value (used for mapping in Torq).

    Permissions:

        Add User.Read permission.

        Admin consent at organization level.

    Enterprise Application Setup:

        Assign user groups to the enterprise app.

        Map Azure AD groups → Torq roles.

    Torq SSO Settings:

        Add Client ID, Client Secret, Authority URL.

        Create Mappings (most permissive role first — top-down evaluation).

            Example: torq_owner → Owner role
            torq_viewer → Viewer role

3. Okta SSO (SAML / OpenID Connect) Setup
Okta Preparation

    Decide SAML vs OpenID Connect (OIDC).

    Create new Okta app for Torq or use App Catalog.

    Configure end-user display (sidebar/tab name).

Okta Steps

    App Setup in Okta:

        Include Redirect URL from Torq.

        Download certificate & note SSO URL.

    Group Creation & Assignment:

        Create torq_owner and torq_viewer groups in Okta.

        Assign users accordingly.

    Torq Configuration:

        Enter Okta SSO URL, Certificate, and redirect info.

        Role mapping (most permissive first).

    Testing:

        Test with a user in each role group.

        Confirm proper role mapping in Torq upon login.

4. Role Mapping Logic (Important!)

    Torq evaluates mappings top-down.

    First match wins — no further matches processed.

    Always place Owner/Admin mappings above Viewer/Restricted mappings.

5. Tool-Agnostic SSO Configuration Principles
Step	Torq	Shuffle	XSOAR	Splunk SOAR
Create App in IdP	App registration in AAD/Okta	Same as Torq	Same	Same
Record IDs & Secrets	Client ID, Secret, Tenant ID	Same	Same	Same
Map Roles	Azure/Okta roles → Torq roles	Azure/Okta roles → Shuffle roles	Azure/Okta roles → XSOAR roles	Azure/Okta roles → Splunk SOAR roles
Configure Redirects	Use platform-specific redirect URL	Platform URL	Platform URL	Platform URL
Test with Multiple Users	Verify role mapping	Same	Same	Same
6. Cross-Platform Practice Lab (Tool-Independent)

Since Torq isn’t available for hands-on right now, you can recreate this in:

    Shuffle (free to host)

    Okta Developer Tenant (free)

    Azure AD Developer Account (free sandbox)

    Local mock SAML/OIDC IdP (e.g., SSO Circle or Keycloak)

Lab Steps

    Set up a dummy app in Okta/Azure AD.

    Create two groups (owner, viewer) and assign test users.

    Spin up Shuffle/XSOAR CE/Splunk SOAR CE locally or in cloud.

    Configure SSO integration using same redirect/metadata principles.

    Map roles in the SOAR platform just like in Torq.

    Test:

        Log in as viewer user — confirm limited access.

        Log in as owner user — confirm full access.

    Extra:

        Change group membership in IdP and re-login to confirm role changes propagate.

7. Key Takeaways

    Always document IdP app config and SOAR mapping together.

    Keep role mappings least-to-most permissive in a clear hierarchy.

    Test with multiple roles and multiple users.

    SSO setup process is ~80% identical across SOAR tools; only UI labels and endpoint formats differ.

8. SSO Cross-Platform Cheatsheet
Step	Torq	Shuffle	XSOAR (Palo Alto)	Splunk SOAR
IdP App Registration	Azure AD / Okta app with correct US/EU Torq redirect	Azure AD / Okta app with Shuffle redirect	Azure AD / Okta app with XSOAR redirect	Azure AD / Okta app with Splunk SOAR redirect
Redirect URL (FQDN)	https://<torq_workspace>.torq.io/auth/callback	https://<shuffle_instance>/callback	https://<xsoar_instance>/sso	https://<splunksoar_instance>/sso/callback
Metadata Source	From Azure AD/Okta (download XML or copy OpenID config endpoint)	Same	Same	Same
Client ID Placeholder	<TORQ_CLIENT_ID>	<SHUFFLE_CLIENT_ID>	<XSOAR_CLIENT_ID>	<SPLUNK_CLIENT_ID>
Client Secret Placeholder	<TORQ_CLIENT_SECRET>	<SHUFFLE_CLIENT_SECRET>	<XSOAR_CLIENT_SECRET>	<SPLUNK_CLIENT_SECRET>
Tenant / Directory ID Placeholder	<AAD_TENANT_ID>	<AAD_TENANT_ID>	<AAD_TENANT_ID>	<AAD_TENANT_ID>
Role Mapping Field	roles claim in ID token	roles claim in ID token	Group attribute or roles claim	Group attribute or roles claim
Example Group → Role Mapping	torq_owner → Owner, torq_viewer → Viewer	shuffle_owner → Admin, shuffle_viewer → Read-Only	xsoar_owner → Administrator, xsoar_analyst → Analyst	soar_admin → Admin, soar_user → User
Testing Tip	Use two accounts in different IdP groups	Same	Same	Same