# Secrets and Credentials Management
[back](../README.md)

Vault is a tool for securely accessing secrets. A secret is anything that you want to tightly control access to, such as API keys, passwords, or certificates. Vault provides a unified interface to any secret, while providing tight access control and recording a detailed audit log.

The key features of Vault are:

Secure Secret Storage: Arbitrary key/value secrets can be stored in Vault. Vault encrypts these secrets prior to writing them to persistent storage, so gaining access to the raw storage isn't enough to access your secrets. Vault can write to disk, Consul, and more.

Dynamic Secrets: Vault can generate secrets on-demand for some systems, such as AWS or SQL databases. For example, when an application needs to access an S3 bucket, it asks Vault for credentials, and Vault will generate an AWS keypair with valid permissions on demand. After creating these dynamic secrets, Vault will also automatically revoke them after the lease is up.

Leasing and Renewal: All secrets in Vault have a lease associated with them. At the end of the lease, Vault will automatically revoke that secret. Clients are able to renew leases via built-in renew APIs.

For our use case vault can be used to manage credentials in order to not have them stored in git repositories and we would use:

**The SQL secrets engine** it would generate database credentials dynamically based on configured roles, this means that services that need to access a database no longer need to hardcode credentials avoiding to have them in repositories and configurations files, instead services can request them from Vault.

```
# Allow database/mysql/postgres tokens to be read.
path "/database/creds/mysqlrole" {
  capabilities = [ "read", "list" ]
}
```

**The RabbitMQ secrets engine** it would generate credentials dynamically based on configured roles, this means that services that need to access to the RabbitMQ Cluster no longer need to hardcode credentials avoiding to have them in repositories and configurations files, instead services can request them from Vault.

**Certificates secrets engine** we can also use vault to keep certificates, here we have 2 options, or we create our own certs and upload them to vault so that application can request them at run time, or we can integrate with letsencrypt so that certificates are dynamically generated.

```
# Enable key/value secret engine for the certificates path
path "sys/mounts/certificates" {
  capabilities = [ "update" ]
}

# To list the available secret engines
path "sys/mounts" {
  capabilities = [ "read" ]
}

# Write and manage secrets in key/value secret engine
path "certificates/*" {
  capabilities = [ "create", "read", "update", "delete", "list" ]
}

# Create policies to permit apps to read secrets
path "sys/policies/acl/*" {
  capabilities = [ "create", "read", "update", "delete", "list" ]
}

# Create tokens for verification & test
path "auth/token/create" {
  capabilities = [ "create", "update", "sudo" ]
}
```

***Using Vault policies***

At the hcl level and per job we can then define and set the proper acls so that nomad can fetch dynamically from vault the required credentials and gums them up inside containers or vm env variables.

```
driver = "docker"
config {
        image = "myregistry.com/webservice:1.0.0"
        force_pull = true
    }

    # vault auth tokens
    vault {
    policies = ["mysql-policy","certificates-policy"]
    change_mode   = "restart"
    }
```

Please note the change_mode setting, with this as soon as the credentials change nomad will trigger the container / raw_execution to be restarted so that it receives the new credentials.
