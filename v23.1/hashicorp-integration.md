---
title: CockroachDB - HashiCorp Vault Integration
summary: Overview of uses cases for integrating CockroachDB with HashiCorp Vault
toc: true
docs_area: reference.third_party_support
---

This pages reviews the supported integrations between CockroachDB and [HashiCorp's Vault](https://www.vaultproject.io/).

Vault is an identity-based secrets and encryption management service, which can either be self-hosted or accessed as a software as a service (SaaS) product through HashiCorp Cloud Platform (HCP). Vault's tooling can complement CockroachDB's data security capabilities to significantly bolster your organizational security posture.

Cockroach Labs supports the following integrations between Vault and CockroachDB:

- [Using Vault's Key Management Secrets (KMS) Engine to manage & distribute encryption keys to AWS or GCP KMS for {{ site.data.products.dedicated }}'s customer-managed encryption key (CMEK) functionality.](#use-vaults-kms-secrets-engine-to-manage-a-cockroachdb-dedicated-clusters-customer-managed-encryption-key)
- Public Key Infrastructure (PKI):
	- [Using Vault's PKI Secrets Engine to manage a {{ site.data.products.dedicated }} cluster's certificate authority (CA) and client certificates](#use-vaults-pki-secrets-engine-to-manage-a-cockroachdb-dedicated-clusters-certificate-authority-ca-and-client-certificates)
	- [Using Vault's PKI Secrets Engine to manage a {{ site.data.products.core }} cluster's certificate authority (CA), server, and client certificates](#use-vaults-pki-secrets-engine-to-manage-a-cockroachdb-self-hosted-clusters-certificate-authority-ca-server-and-client-certificates)
- [Using Vault's PostgreSQL Database Secrets Engine to manage CockroachDB SQL user credentials](#use-vaults-postgresql-database-secrets-engine-to-manage-cockroachdb-sql-users-and-their-credentials).
- [Using Vault's Transit Secrets Engine to generate the store key for {{ site.data.products.enterprise }} Encryption At Rest for a {{ site.data.products.core }} cluster.](#use-vaults-transit-secrets-engine-to-manage-a-cockroachdb-self-hosted-clusters-enterprise-encryption-at-rest-store-key)

## Use Vault's KMS secrets engine to manage a {{ site.data.products.dedicated }} cluster's customer-managed encryption key
{{ site.data.products.dedicated }} supports the use of customer-managed encrypted keys (CMEK) for the encryption of data at rest.

[Vault's Key Management secrets engine](https://www.vaultproject.io/docs/secrets/key-management) allows customers to manage encryption keys on external key management services (KMS) such as those offered by Google Cloud Platform (GCP) or Amazon Web Services (AWS).

CockroachDB customers can integrate these services, using Vault's KMS secrets engine to handle the full lifecycle of the encryption keys that {{ site.data.products.dedicated }} uses to protect their data.

Resources:

- [CMEK overview](../cockroachcloud/cmek.html)
- [CMEK FAQ](../cockroachcloud/cmek-faq.html)
- [Manage Customer-Managed Encryption Keys (CMEK) for CockroachDB Dedicated](../cockroachcloud/managing-cmek.html)
- [Provisioning GCP KMS Keys and Service Accounts for CMEK](../cockroachcloud/cmek-ops-gcp.html)
- [Provisioning AWS KMS Keys and IAM Roles for CMEK](../cockroachcloud/cmek-ops-aws.html)

## Use Vault's PKI Secrets Engine to manage a {{ site.data.products.dedicated }} cluster's certificate authority (CA) and client certificates.

{{ site.data.products.dedicated }} customers can use Vault's public key infrastructure (PKI) secrets engine to manage PKI certificates for client authentication to the cluster. Vault's PKI Secrets Engine greatly eases the security-critical work involved in maintaining a certificate authority (CA), generating, signing and distributing PKI certificates.

By using Vault to manage certificates, you can use only certificates with short validity durations, an important component of PKI security.

Refer to [Transport Layer Security (TLS) and Public Key Infrastructure (PKI)](security-reference/transport-layer-security.html) for an overview.

Refer to [Certificate Authentication for SQL Clients in Dedicated Clusters](../cockroachcloud/client-certs-dedicated.html) for procedures in involved in adminstering PKI for a {{ site.data.products.dedicated }} cluster.

## Use Vault's PKI Secrets Engine to manage a {{ site.data.products.core }} cluster's certificate authority (CA), server, and client certificates

{{ site.data.products.core }} customers can use Vault's public key infrastructure (PKI) secrets engine to manage PKI certificates for internode as well as client-cluster authentication. Vault's PKI Secrets Engine greatly eases the security-critical work involved in securely maintaining a certificate authority (CA), generating, signing and distributing PKI certificates.

By using Vault to manage certificates, you can use only certificates with short validity durations, an important component of PKI security.

Refer to [Transport Layer Security (TLS) and Public Key Infrastructure (PKI)](security-reference/transport-layer-security.html) for an overview.

Refer to [Manage PKI certificates for a CockroachDB deployment with HashiCorp Vault](manage-certs-vault.html) for procedures in involved in adminstering PKI for a {{ site.data.products.core }} cluster.

## Use Vault's PostgreSQL Database Secrets Engine to manage CockroachDB SQL users and their credentials

CockroachDB users can use Vault's PostgreSQL Database Secrets Engine to handle the full lifecycle of SQL user credentials (creation, password rotation, deletion). Vault is capable of managing SQL user credentials in two ways:

- As [Static Roles](https://www.vaultproject.io/docs/secrets/databases#static-roles), meaning that a single SQL user/role is mapped to a Vault role.

- As [Dynamic Secrets](https://www.vaultproject.io/use-cases/dynamic-secrets), meaning that credentials are generated and issued on demand from pre-configured templates, rather than created and persisted. Credentials are issued for specific clients and for short validity durations, further minimizing both the likelihood of a credential compromise, and the possible impact of any compromise that might occur.

Try the tutorial: [Using HashiCorp Vault's Dynamic Secrets for Enhanced Database Credential Security in CockroachDB](vault-db-secrets-tutorial.html)

## Use Vault's Transit Secrets Engine to manage a {{ site.data.products.core }} cluster's {{ site.data.products.enterprise }} Encryption At Rest store key

When deploying {{ site.data.products.enterprise }}, customers can provide their own externally managed encryption keys for use as the *store key* for CockroachDB's [{{ site.data.products.enterprise }} Encryption At Rest](security-reference/encryption.html#encryption-at-rest-enterprise).

Vault's [Transit Secrets Engine](https://www.vaultproject.io/docs/secrets/transit) can be used to generate suitable encryption keys for use as your cluster's store key.
