# Secrets and Credentials Management
[back](../README.md)

Vault is a tool for securely accessing secrets. A secret is anything that you want to tightly control access to, such as API keys, passwords, or certificates. Vault provides a unified interface to any secret, while providing tight access control and recording a detailed audit log.

The key features of Vault are:

Secure Secret Storage: Arbitrary key/value secrets can be stored in Vault. Vault encrypts these secrets prior to writing them to persistent storage, so gaining access to the raw storage isn't enough to access your secrets. Vault can write to disk, Consul, and more.

Dynamic Secrets: Vault can generate secrets on-demand for some systems, such as AWS or SQL databases. For example, when an application needs to access an S3 bucket, it asks Vault for credentials, and Vault will generate an AWS keypair with valid permissions on demand. After creating these dynamic secrets, Vault will also automatically revoke them after the lease is up.

Leasing and Renewal: All secrets in Vault have a lease associated with them. At the end of the lease, Vault will automatically revoke that secret. Clients are able to renew leases via built-in renew APIs.

For our use case vault can be used to manage credentials in order to not have them stored in git repositories and we would use:

**The SQL secrets engine** it would generate database credentials dynamically based on configured roles, this means that services that need to access a database no longer need to hardcode credentials avoiding to have them in repositories and configurations files, instead services can request them from Vault.

**The RabbitMQ secrets engine** it would generate credentials dynamically based on configured roles, this means that services that need to access to the RabbitMQ Cluster no longer need to hardcode credentials avoiding to have them in repositories and configurations files, instead services can request them from Vault.






