---
layout: post
title: Vault Basic Debug
---

[HashiCorp Vault](https://www.hashicorp.com/) is a good off-the-shelf solution for secrets
management and data protection.

At its core, its about setting some rules encoded in
[policies](https://www.vaultproject.io/intro/getting-started/policies#policies) and
getting a token that adheres to those rules.

At times, it could turn out a black box when things don't work as
expected.

Below workflow should help debug the problem and [fix](https://www.vaultproject.io/intro/getting-started/policies#testing-the-policy) it with bare vault
infrastructure in simple steps:

- vault policy write my-policy my-policy.hcl
- vault policy read my-policy
- vault token create -policy=my-policy
- vault login <TOKEN-FROM-ABOVE-STEP>
- STEP WHICH LED YOU HERE
- Make changes in the policy `my-policy`
- Repeat from start until you get the right policy
- You might have to login using root_token so keep it handy somwhere
  - vault login <ROOT-TOKEN>

As vault is about setting right policies as desired for the use-case in
hand, we will always have a policy which we are investigating for the
missing or undesired rules.

## My Usecase

There is a approle, say, `secret-service` which we want to allow make
policies for another service and read them

We are able to create policies but not able to read them :(

So, we need a policy for secret-service which allows access to
creating/updating policies.

### Secret Service policy

```
# Filename: secret-service-policy.hcl

# Manage auth methods broadly across Vault
path "auth/*"
{
  capabilities = ["create", "read", "update", "delete", "list", "sudo"]
}

# Create and manage ACL policies via CLI
path "sys/policy/*"
{
  capabilities = ["create", "read", "update", "delete", "list", "sudo"]
}
```

**Root Login might be needed**

```
vault login <ROOT-TOKEN>
```

**Create Secret Service Policy**

```
vault policy write secret-service secret-service-policy.hcl
```

**Read Secret Service Policy**

```
vault policy read secret-service
```

**Create Token using secret-service policy**

```
vault token create -policy=secret-service
```

**Login using the token as adhered for secret-service policy**

```
vault login <TOKEN-FROM-ABOVE-STEP>
```

**TRY Creating a policy**

```
# Filename: another-service-policy.hcl

# Manage auth methods broadly across Vault
path "auth/*"
{
  capabilities = ["create", "read", "update", "delete", "list", "sudo"]
}

# Create and manage ACL policies via CLI
path "sys/policy/*"
{
  capabilities = ["create", "update", "delete", "sudo"]
}
```

```
vault policy write another-service another-service-policy.hcl
```

- Above should create the another-service policy sucessfully as it has `create|update` capabilities on `sys/policy` path

**Experiment to fix the failing policy read command**

- Try reading the policy, it should fail

```
## Fails as no *read* capability yet
vault policy read another-service
```

- Add `read` capability on `sys/policy` path in `secret-service-policy.hcl`

```
# Filename: another-service-policy.hcl

# Manage auth methods broadly across Vault
path "auth/*"
{
  capabilities = ["create", "read", "update", "delete", "list", "sudo"]
}

# Create and manage ACL policies via CLI
## Note the change from above of addition of *read*
path "sys/policy/*"
{
  capabilities = ["create", "read", "update", "delete", "sudo"]
}
```

```
vault policy write another-service another-service-policy.hcl
```

```
## Succeeds as *read* capability added now
vault policy read another-service
```

### Back to Usecase

**Problem to debug**

Approle `secret-service` not able to read policy.

**Create App Role** [link](https://www.vaultproject.io/guides/identity/authentication#cli-command-1)

```
vault write auth/approle/role/secret-service policies="secret-service"
```

**Get Role Id and Secret Id** [link](https://www.vaultproject.io/guides/identity/authentication#cli-command-2)

```
vault read auth/approle/role/secret-service/role-id
```

```
vault write -f auth/approle/role/secret-service/secret-id
```

**Login and get token**

```
vault write auth/approle/login role_id="675a50e7-cfe0-be76-e35f-49ec009731ea" \
 secret_id="ed0a642f-2acf-c2da-232f-1b21300d5f29"
```

The token returned here was not able to `read` policy as intended and above workflow helped fix it.
Above debug workflow helped fixed it, we simulated the token in debug workflow as `vault token create -policy=secret-service` whereas here it comes as part of `secret-service` approle login
