---
title: Troubleshoot Common Problems
summary: How to troubleshoot problems and handle transaction retry errors during application development
toc: true
docs_area: develop
---

This page has instructions for handling errors and troubleshooting problems that may arise during application development.

## Troubleshoot query problems

If you are not satisfied with your SQL query performance, follow the instructions in [Optimize Statement Performance Overview][fast] to be sure you are avoiding common performance problems like full table scans, missing indexes, etc.

If you have already optimized your SQL queries as described in [Optimize Statement Performance Overview][fast] and are still having issues such as:

- [Hanging or "stuck" queries](query-behavior-troubleshooting.html#hanging-or-stuck-queries), usually due to [contention](performance-best-practices-overview.html#transaction-contention) with long-running transactions
- Queries that are slow some of the time (but not always)
- Low throughput of queries

Take a look at [Troubleshoot SQL Behavior](query-behavior-troubleshooting.html).

{% include {{ page.version.version }}/prod-deployment/check-sql-query-performance.md %}

## Transaction retry errors

Messages with the error code `40001` and the string `restart transaction` are known as [*transaction retry errors*](transaction-retry-error-reference.html). These indicate that a transaction failed due to [contention](performance-best-practices-overview.html#understanding-and-avoiding-transaction-contention) with another concurrent or recent transaction attempting to write to the same data. The transaction needs to be retried by the client.

{% include {{ page.version.version }}/performance/transaction-retry-error-actions.md %}

## Unsupported SQL features

CockroachDB has support for [most SQL features](sql-feature-support.html).

Additionally, CockroachDB supports [the PostgreSQL wire protocol and the majority of its syntax](postgresql-compatibility.html). This means that existing applications can often be migrated to CockroachDB without changing application code.

However, you may encounter features of SQL or the PostgreSQL dialect that are not supported by CockroachDB. For example, the following PostgreSQL features are not supported:

{% include {{page.version.version}}/sql/unsupported-postgres-features.md %}

For more information about the differences between CockroachDB and PostgreSQL feature support, see [PostgreSQL Compatibility](postgresql-compatibility.html).

For more information about the SQL standard features supported by CockroachDB, see [SQL Feature Support](sql-feature-support.html).

## Troubleshoot cluster problems

As a developer, you will mostly be working with the CockroachDB [SQL API](sql-statements.html).

However, you may need to access the underlying cluster to troubleshoot issues where the root cause is not your SQL, but something happening at the cluster level. Symptoms of cluster-level issues can include:

- Cannot join a node to an existing cluster
- Networking, client connection, or authentication issues
- Clock sync, replication, or node liveness issues
- Capacity planning, storage, or memory issues
- Node decommissioning failures

For more information about how to troubleshoot cluster-level issues, see [Troubleshoot Cluster Setup](cluster-setup-troubleshooting.html).

## Troubleshoot SQL client application problems

<a name="scram-client-troubleshooting"></a>

### High client CPU load, connection pool exhaustion, or increased connection latency when SCRAM Password-based Authentication is enabled

+ [Overview](#overview)
+ [Mitigation steps while keeping SCRAM enabled](#mitigation-steps-while-keeping-scram-enabled)
+ [Downgrade from SCRAM authentication](#downgrade-from-scram-authentication)

#### Overview

When [SASL/SCRAM-SHA-256 Secure Password-based Authentication](security-reference/scram-authentication.html) (SCRAM Authentication) is enabled on a cluster, some additional CPU load is incurred on client applications, which are responsible for handling SCRAM hashing. It's important to plan for this additional CPU load to avoid performance degradation, CPU starvation, and [connection pool](connection-pooling.html) exhaustion on the client. For example, the following set of circumstances can exhaust the client application's resources:

1. SCRAM Authentication is enabled on the cluster (the `server.user_login.password_encryption` [cluster setting](cluster-settings.html#setting-server-user-login-password-encryption) is set to `scram-sha-256`).
1. The client driver's [connection pool](connection-pooling.html) has no defined maximum number of connections, or is configured to close idle connections eagerly.
1. The client application issues [transactions](transactions.html) concurrently.

In this situation, each new connection uses more CPU on the client application server than connecting to a cluster without SCRAM Authentication enabled. Because of this additional CPU load, each concurrent transaction is slower, and a larger quantity of concurrent transactions can accumulate, in conjunction with a larger number of concurrent connections. In this situation, it can be difficult for the client application server to recover.

Some applications may also see increased connection latency. This can happen because SCRAM incurs additional round trips during authentication which can add latency to the initial connection.

For more information about how SCRAM works, see [SASL/SCRAM-SHA-256 Secure Password-based Authentication](security-reference/scram-authentication.html).

#### Mitigation steps while keeping SCRAM enabled

To mitigate against this situation while keeping SCRAM authentication enabled, Cockroach Labs recommends that you:

{% include_cached {{page.version.version}}/scram-authentication-recommendations.md %}

If the above steps don't work, you can try lowering the default hashing cost and reapplying the password as described below.

##### Lower default hashing cost and reapply the password

To decrease the CPU usage of SCRAM password hashing while keeping SCRAM enabled:

1. Set the [`server.user_login.password_hashes.default_cost.scram_sha_256` cluster setting](cluster-settings.html#setting-server-user-login-password-hashes-default-cost-scram-sha-256) to `4096`:

    {% include_cached copy-clipboard.html %}
    ~~~ sql
    SET CLUSTER SETTING server.user_login.password_hashes.default_cost.scram_sha_256 = 4096;
    ~~~

1. Make sure the [`server.user_login.rehash_scram_stored_passwords_on_cost_change.enabled` cluster setting](cluster-settings.html) is set to `true` (the default).

{{site.data.alerts.callout_success}}
When lowering the default hashing cost, we recommend that you use strong, complex passwords for [SQL users](security-reference/authorization.html#sql-users).
{{site.data.alerts.end}}

If you are still seeing higher connection latencies than before, you can [downgrade from SCRAM authentication](#downgrade-from-scram-authentication).

#### Downgrade from SCRAM authentication

As an alternative to the [mitigation steps listed above](#mitigation-steps-while-keeping-scram-enabled), you can downgrade from SCRAM authentication to bcrypt as follows:

1. Set the [`server.user_login.password_encryption` cluster setting](cluster-settings.html#setting-server-user-login-password-encryption) to `crdb-bcrypt`:

    {% include_cached copy-clipboard.html %}
    ~~~ sql
    SET CLUSTER SETTING server.user_login.password_encryption = 'crdb-bcrypt';
    ~~~

1. Ensure the [`server.user_login.downgrade_scram_stored_passwords_to_bcrypt.enabled` cluster setting](cluster-settings.html#setting-server-user-login-downgrade-scram-stored-passwords-to-bcrypt-enabled) is set to `true`:

    {% include_cached copy-clipboard.html %}
    ~~~ sql
    SET CLUSTER SETTING server.user_login.downgrade_scram_stored_passwords_to_bcrypt.enabled = true;
    ~~~

{{site.data.alerts.callout_info}}
The [`server.user_login.upgrade_bcrypt_stored_passwords_to_scram.enabled` cluster setting](cluster-settings.html#setting-server-user-login-upgrade-bcrypt-stored-passwords-to-scram-enabled) can be left at its default value of `true`.
{{site.data.alerts.end}}

## See also

### Tasks

- [Connect to a CockroachDB Cluster](connect-to-the-database.html)
- [Run Multi-Statement Transactions](run-multi-statement-transactions.html)
- [Optimize Statement Performance Overview][fast]

### Reference

- [Common Errors and Solutions](common-errors.html)
- [Transactions](transactions.html)
- [Client-side transaction retry handling](transaction-retry-error-reference.html#client-side-retry-handling)
- [SQL Layer][sql]

<!-- Reference Links -->

[sql]: architecture/sql-layer.html
[fast]: make-queries-fast.html
