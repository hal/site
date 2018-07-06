---
title: "Authentication on Keycloak SSO"
date: 2018-06-19T11:29:20+01:00
description: "Describes how HAL authentication works with Keycloak SSO"
icon: "/img/shield.png"
toc: true
weight: 30
---
When user configures Wildfly to authenticate user to Keycloak SSO, HAL supports the authentication the following way:

- HAL redirects the login attempt to keycloak server, if successful redirect back to HAL.
- Provide the logout option in HAL to call the logout page on keycloak. This is displayed under the username in the header section.
- Displays basic Keycloak settings in the Access Control page.
- Displays a link to user profile on Keycloak server.
- Point the Access Control link in the Homepage to the Keycloak server URL.

The Wildfly Access Control mechanism to manage users and roles should be disabled, because they are managed in Keycloak server. However RBAC continues to works in Wildfly to fine tune permission settings.

**How to setup Wildfly to use Keycloak authentication**

- [How to install Keycloak Wildfly Adapter binaries](https://www.keycloak.org/docs/latest/securing_apps/index.html#jboss-eap-wildfly-adapter)
- [How to configure Wildfly to enable keycloak](https://docs.jboss.org/author/display/WFLY/Protecting+Wildfly+Adminstration+Console+With+Keycloak)


**Warning**

- The authentication only works in standalone mode, as the wildfly keycloak adapter doesn't support domain mode yet.


# Login attempt

{{< imgflow src="/img/documentation/keycloak-auth.png" float="right" >}}
When user wants to access HAL management console, it redirects to Keycloak server and presents the authentication form.


{{</ imgflow >}}

# Access control

{{< imgflow src="/img/documentation/access-control-sso.png" float="left" >}}

The Access Control top level menu, shows basic Keycloak settings:

- User information
- Keycloak server url
- Keycloak account profile
- Realm name
- Realm public key

If RBAC is not set, there is a warning at the top warning the user, there is no role based access control mechanism, and there is an option to enable it.

Also, there is the "Logout" option in the header section.

{{</ imgflow >}}


# Account Profile

{{< imgflow src="/img/documentation/keycloak-acct-mgmt.png" float="right" >}}

When user wants to view and edit basic user information on keycloak server, there is a link in the Access Control page, that redirects to Keycloak server.
The page enables users to:

- Edit name and email.
- Change password.
- List all login sessions.
- Logout from all sessions.
- List keycloak application the user is entitled to log in.

{{</ imgflow >}}

# Keycloak Adapter Subsystem

{{< imgflow src="/img/documentation/keycloak-subsystem.png" float="left" >}}

The Keycloak adapter subsystem may be managed using hal model browser. It provides full management capabilities to keycloak adapter subsystem.

{{</ imgflow >}}

