---
title: Single Sign-On (SSO) for CockroachDB Cloud organizations
summary: Learn about the options for Single Sign-On for your CockroachDB Cloud organization.
toc: true
docs_area: manage
---

{% include_cached cockroachcloud/sso-intro.md %}

This page describes Basic SSO and Cloud Organization SSO. To enable Cloud Organization SSO, refer to [Configure Cloud Organization SSO](configure-cloud-org-sso.html).

## Basic SSO

Basic SSO provides flexibility and convenience for your users, and is enabled by default for each {{ site.data.products.db }} organization. With no configuration required, members can sign in using an identity from GitHub, Google, or Microsoft instead of using a password.

Basic SSO has the following differences from [Cloud Organization SSO](#cloud-organization-sso):

  - Configuration is not possible.
  - New authentication methods cannot be added, and existing authentication methods cannot be modified, limited, or disabled.
  - It is not possible to enforce a requirement to use SSO rather than password authentication.
  - It is not possible to limit the email domains allowed to sign in using a given authentication method.
  - Autoprovisioning is not supported, and members must be invited before they can sign in.
  - A member may have only one active authentication method in an organization, but may change it at any time by logging in using a different method. However, for a member to switch back to using a password, they must be removed and re-invited to your {{ site.data.products.db }} organization.

If your organization needs more flexibility and customization to meet your security and compliance requirements, you can enable and configure [Cloud Organization SSO](#cloud-organization-sso).

## Cloud Organization SSO

Cloud Organization SSO allows you to customize your SSO configuration to meet your organization's security and business requirements:

- Members sign in using a custom URL that allows only the authentication methods that you have configured.
- Members can sign in using any enabled authentication method, to help reduce the impact of an IdP outage. If a member signs in using a new method for the first time, they are prompted to optionally update their default method. **This is possible only as long as the members are using the same email address to sign in through each method**.
- You can [enable multiple authentication methods](configure-cloud-org-sso.html#enable-or-disable-an-authentication-method) simultaneously. You can even add custom authentication methods that connect to IdPs such as Okta or ActiveDirectory through the [Security Access Markup Language (SAML)](https://en.wikipedia.org/wiki/Security_Assertion_Markup_Language) and [OpenID Connect (OIDC)](https://openid.net/connect/) identity protocols.
- You can disable any authentication method. To enforce a requirement to use SSO, you can enable only SSO authentication methods and disable password authentication. If you disable password authentication, passwords are not retained.
- You can [restrict the email domains](configure-cloud-org-sso.html#allowed-email-domains) that are allowed to sign in using an SSO authentication method. By default, any email domain is allowed.
- [Autoprovisioning](#autoprovisioning) can be enabled for SSO authentication methods, and automatically creates a {{ site.data.products.db }} organization account when a member successfully authenticates using an SSO authentication method for the first time, with no invitation required.
- [SCIM Provisioning](configure-scim-provisioning.html) automatically creates a {{ site.data.products.db }} organization account when a user is assigned to the SCIM application in your IdP that is connected to your {{ site.data.products.db }} organization.

To enable and configure Cloud Organization SSO, refer to [Configure Cloud Organization SSO](configure-cloud-org-sso.html). The following sections provide more details about the features of Cloud Organization SSO.

{{site.data.alerts.callout_success}}
If your organization includes members whose identity you don't manage, such as partners or consultants, you can leave password authentication enabled for those members, while instructing your internal users to sign in using SSO. Members who sign in using a password must be removed from your {{ site.data.products.db }} organization manually.
{{site.data.alerts.end}}

### Autoprovisioning

Autoprovisioning is a self-service mechanism that removes the need for a new user to be [invited by an Org Admin](managing-access.html#invite-team-members-to-an-organization). When it is enabled, the first time a user successfully authentications using that method, {{ site.data.products.db }} organization account is automatically created for them. Autoprovisioned accounts are initially assigned the [**Organization Member** role](authorization.html#organization-member), which grants no permissions to perform cluster or org actions. Additional roles can be granted by a user with the [**Org Administrator (legacy)** role](authorization.html#org-administrator-legacy).

Autoprovisioning is disabled by default, but can be enabled per SSO authentication method.

If you enable autoprovisioning, Cockroach Labs recommends that you also limit the [Allowed Email Domains](configure-cloud-org-sso.html#allowed-email-domains) for the SSO authentication method. This ensures that only your organization's members can access your {{ site.data.products.db }} organization, and that only new accounts for the specified domains can be autoprovisioned.

{{site.data.alerts.callout_danger}}
{{ site.data.products.db }} users are identified by their email address. To reduce the risk of duplicated users, ensure that users have unique email addresses before you enable autoprovisioning for an authentication method. If duplicate users result from enabling autoprovisioning, you must delete them manually. Refer to [Manage an Organization's Members](managing-access.html#manage-an-organizations-users).
{{site.data.alerts.end}}

Cockroach Labs recommends that you enable autoprovisioning on only a single SSO method at a time, and that you migrate your users gradually. Most organizations aim to manage users in a single centralized IdP. It may be necessary to temporarily enable autoprovisioning to migrate a group of users from your centralized IdP who have yet not been onboarded to your {{ site.data.products.db }} organization.

If you [disable password authentication and require SSO authentication](configure-cloud-org-sso.html#require-sso), then when you deprovision a member from your IdP, they can no longer access your {{ site.data.products.db }} UI and there is no need to manually remove them from your {{ site.data.products.db }} organization, but some organizations choose to do so.

If you require automated user management features, you can configure [SCIM provisioning](#scim-provisioning) instead of autoprovisioning.

### Migration of individual members to SSO

After you [enable Cloud Organization SSO](configure-cloud-org-sso.html#enable-cloud-organization-sso) and [enable an authentication method](configure-cloud-org-sso.html#enable-or-disable-an-authentication-method) for your organization, it will appear on your organization's custom URL. Your existing users can then sign in using that method, rather than the method they were using previously. When an existing member signs in using an SSO authentication method for the first time, they can optionally designate that authentication method as their new default.

After you enable Cloud Organization SSO, all members of your organization must sign in again, even if they were previously signed in using [Basic SSO](#basic-sso). After signing in, they retain the same organizational roles they had previously.

However, members of your organization who also belong to other {{ site.data.products.db }} organizations must be re-added to your organization. If they sign in using an authentication method with [autoprovisioning](#autoprovisioning) enabled, they are automatically added upon successful sign-in. Otherwise, you must re-invite them to your organization.

When you enable Cloud Organization SSO or when you enable or disable an authentication method, you are shown a list of the members who will be impacted and the action that must be taken for them to regain access. Those members are also notified about the change via email.

After all users have been migrated and signing in using SSO, you can [disable password authentication and enforce a requirement to sign in using SSO](configure-cloud-org-sso.html#require-sso).

## SCIM provisioning

[System for Cross-Domain Identity Management SCIM](https://www.rfc-editor.org/rfc/rfc7644) centralizes and automates provisioning and deprovisioning of {{ site.data.products.db }} organization users from your IdP.

Rather than using [invitations](managing-access.html#invite-team-members-to-an-organization) or self-service [autoprovisioning](#autoprovisioning), SCIM provisioning tasks are performed centrally by a team of IAM admins, who manage the assignment of your organization's users to your organization's applications. To learn more or configure SCIM provisioning, refer to [Configure SCIM Provisioning](configure-scim-provisioning.html).

To learn more, refer to [configuring SCIM provisioning](configure-scim-provisioning.html).

## Frequently Asked Questions (FAQ)

#### If a user already has an email address associated with an SSO provider such as Gmail, can they sign in with Basic SSO?

Yes, as long as the email address associated with the user's SSO provider is exactly the same as the one associated with the user's existing {{ site.data.products.db }} account. After successfully signing in, the user will be prompted to approve the updated authentication method for their account.

A user can view their current authentication method by clicking **My Account** in the [{{ site.data.products.db }} Console](https://cockroachlabs.cloud).

#### With Basic SSO, once a user changes their active login method to a new SSO provider, can they still sign in using an email/password combination or GitHub identity?

No. With Basic SSO, only one authentication method can be active for each {{ site.data.products.db }} Console user. To view or update their active authentication method, a user can click **My Account** in the [{{ site.data.products.db }} Console](https://cockroachlabs.cloud) .

#### Does this change to invite users?

The [workflow for inviting team members](managing-access.html#invite-team-members-to-an-organization) to your {{ site.data.products.db }} organization remains the same.

However, if Cloud Organization SSO is enabled for your {{ site.data.products.db }} organization and [autoprovisioning](#autoprovisioning) is enabled for the authentication method a member uses to sign in, then they can create an account and sign in without waiting for an invitation.

If [SCIM provisioning](#scim-provisioning) is enabled, the user's account is provisioned when an IdP admin assigns them to the SCIM application.

#### How do I deprovision a user's access to the {{ site.data.products.db }} organization if they leave the relevant project?

If Cloud Organization SSO is enabled, then deprovisioning a user at the level of the IdP also removes their access to the {{ site.data.products.db }} organization. However, their account is only automatically removed if you use [SCIM provisioning](#scim-provisioning).

To remove a user's access to {{ site.data.products.db }} manually (such as when a user changes teams but does not leave the organization entirely), you can [remove their {{ site.data.products.db }} user identity from your {{ site.data.products.db}} organization](managing-access.html#remove-a-team-member).

#### Can Org Administrators require a particular authentication method for their {{ site.data.products.db }} organization?

Yes. When Cloud Organization SSO is enabled for your {{ site.data.products.db }} organization, only the [authentication methods you have enabled](configure-cloud-org-sso.html#enable-or-disable-an-authentication-method) are displayed to your users.

#### Which SAML-based authentication flows are supported with Cloud Organization SSO?

The following flows are supported:

- The _service provider-initiated flow_, where you initiate configuration of Cloud Organization SSO through the {{ site.data.products.db }} Console.
- An _identity provider-initiated flow_, where you initiate configuration through an IdP such as Okta.

#### What default role is assigned to users when autoprovisioning is enabled in a {{ site.data.products.db }} organization?

Autoprovisioned accounts are initially assigned the [**Organization Member** role](authorization.html#organization-member), which grants no permissions to perform cluster or org actions. Additional roles can be granted by a user with the [**Org Administrator (legacy)** role](authorization.html#org-administrator-legacy).

## What's next?
- [Configure Cloud Organization SSO](configure-cloud-org-sso.html)
- Learn more about [authenticating to {{ site.data.products.db }}](authentication.html).
