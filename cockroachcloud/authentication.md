---
title: Authenticating to CockroachDB Cloud
summary: Learn about the authentication features for CockroachDB Cloud clusters.
toc: true
docs_area: manage
cloud: true
---

Users may connect with {{ site.data.products.db }} at two levels, the organization and the cluster, both of which are covered in this page.

Refer to: [Overview of the {{ site.data.products.db }} two-level authorization model](authorization.html#overview-of-the-cockroachdb-cloud-two-level-authorization-model).

## Overview of {{ site.data.products.db }} authentication

**Organization level** functions can be performed through three different interfaces, each with its own authentication flows:

- The [{{ site.data.products.db }} Console UI](https://cockroachlabs.cloud/) provides a user with an overview of their {{ site.data.products.db }} account, and offers functionality for administering an organization and the clusters within it.

	Refer to: [Console authentication](#console-ui-authentication)

- The `ccloud` utility allows users to execute cloud functions from the command line.

	Refer to: [`ccloud` authentication](#ccloud-authentication)

- The {{ site.data.products.db }} API allows [service accounts](../{{site.versions["stable"]}}/architecture/glossary.html#service-account) to perform many organization and cluster administration functions.

	Refer to: [Cloud API authentication](#cloud-api-authentication)

**Cluster level functions**, i.e. SQL statements, are executed by SQL clients.

	Refer to: [SQL client authentication](#sql-client-authentication)

## Console UI authentication

You may log in to the [{{ site.data.products.db }} Console](https://cockroachlabs.cloud/) with a username and password, or by using [Single Sign-on (SSO) for {{ site.data.products.db }}](cloud-org-sso.html).

{% include cockroachcloud/prefer-sso.md %}


## `ccloud` authentication

The `ccloud` utility offers Organization users a way to script many functions that must otherwise be performed in the console interface.

For more information, refer to: [Get Started with the `ccloud` CLI
](ccloud-get-started.html)

To authenticate `ccloud` to a particular organization, run the following command. `ccloud` will open up your machine's default browser, attempt to authenticate to the console and export a temporary auth token.

{% include_cached copy-clipboard.html %}
~~~shell
ccloud auth login --org <organization label>
~~~

Refer to: [Log in to CockroachDB Cloud using ccloud auth](ccloud-get-started.html#log-in-to-cockroachdb-cloud-using-ccloud-auth)

## Cloud API authentication

The [Cloud API](cloud-api.html) allows automated execution of organization functions. Unlike `ccloud` and the console UI, however, the API can not be authenticated by human users, but by [service accounts](../{{site.versions["stable"]}}/architecture/glossary.html#service-account).

Service accounts authenticate to the API using API keys, which are provisioned through the console UI.

Refer to: [Managing Service Accounts: API access](managing-access.html#api-access)

## SQL client authentication

To execute SQL statements or perform database administration functions on a cluster, you must connect to the cluster with a SQL client. CockroachDB clients include the CockroachDB CLI, and numerous [drivers and object-relational mapping (ORM) tools](../{{site.current_cloud_version}}/install-client-drivers.html).

### The connection string

You can obtain a connection string or CLI command for your cluster, to use with your choice of SQL client, by visiting your cluster's overview page, and clicking **Connect** button, and following the instructions.

`https://cockroachlabs.cloud/cluster/< your cluster UUID >/`

Clients can authenticate in two ways. Your connection string must be modified depending on which you are using:

- [Username and password](#username-and-password)
- [PKI security certificate](#pki-security-certificate)

Note that the [TLS public root certificate authority (CA) certificate of the cluster](../{{site.current_cloud_version}}/security-reference/transport-layer-security.html#certificates-signing-trust-and-authority) is also required for authenticating the cluster server against the SQL client. This certificate can be downloaded by following the instructions in the **Connect** UI.

### Username and password

A user can authenticate to a cluster by providing their username and password in the connection string.

[The connection string](#the-connection-string) UI will indicate how to modify the string to user credentials.

For information on managing SQL user credentials, including provisioning passwords, refer to [Manage SQL users on a cluster](managing-access.html#manage-sql-users-on-a-cluster).

### PKI security certificate

SQL clients may authenticate to {{ site.data.products.dedicated }} clusters using PKI security certificates.

Refer to [Transport Layer Security (TLS) and Public Key Infrastructure (PKI)](../{{site.versions["stable"]}}/security-reference/transport-layer-security.html) for an overview of PKI certificate authentication in general and its use in CockroachDB.

Refer to [Certificate Authentication for SQL Clients in Dedicated Clusters](client-certs-dedicated.html) for procedural information on administering and using client certificate authentication.

{{site.data.alerts.callout_info}}
{% include_cached feature-phases/limited-access.md %}
{{site.data.alerts.end}}

## Node identity verification

The [connection string](connect-to-your-cluster.html) generated to connect to your application uses the `verify-full` [SSL mode](#ssl-mode-settings) by default to verify a node’s identity. This mode encrypts the data in-flight as well as verifies the identity of the CockroachDB node, thus ensuring a secure connection to your cluster. Using this mode prevents MITM (Machine in the Middle) attacks, impersonation attacks, and eavesdropping.

To connect securely to your cluster using the `verify-full` mode:

1. Download the CA certificate and place it in the `certs` directory. The Certificate Authority (CA) certificate is the file that the client uses to verify the identity of the CockroachDB node.
1. When connecting to the cluster, specify the path to the `certs` directory in the connection string. See [Connect to your cluster](connect-to-your-cluster.html) for more details.

You can also use the `require` SSL mode, although we do not recommend using it since it can make the cluster susceptible to MITM and impersonation attacks. For more information, see the "Protection Provided in Different Modes" section in PostgreSQL's [SSL Support](https://www.postgresql.org/docs/9.4/libpq-ssl.html) document.

For more information about creating SQL users and passwords, see [User Authorization](managing-access.html).

## SSL mode settings

The table below lists the `sslmode` settings you can use to [connect to your cluster](connect-to-your-cluster.html) and their associated security risks. Other settings are not recommended.

`sslmode` | Eavesdropping protection | MITM protection | Description
-------------|------------|------------|------------
`require` | Yes | No | 	Force a secure connection. An error occurs if the secure connection cannot be established. This is less secure than using a CA certificate and is only recommended for testing or unimportant data.
`verify-full` | Yes | Yes | Force a secure connection, verify that the server certificate is signed by a known CA, and verify that the server address matches that specified in the certificate.

## See also

- [Cloud Organization SSO](cloud-org-sso.html)
- [Configure Cloud Organization SSO](configure-cloud-org-sso.html)
- [Client Connection Parameters](../{{site.current_cloud_version}}/connection-parameters.html)
- [Connect to Your {{ site.data.products.dedicated }} Cluster](connect-to-your-cluster.html)
