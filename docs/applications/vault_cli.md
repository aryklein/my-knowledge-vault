---
tags:
  - hashicorp
  - vault
  - cheatsheet
last_updated: 20240423
---
# Hashicorp Vault CLI cheat sheet

This document will focus on providing a reference for commands
and shortcuts that I often find challenging to remember. Please note that it is
not intended to be a comprehensive tutorial on how to use Hashicorp Vault

_Note_: All the commands below are focused on the Key-Value (KV) secrets engine

Before you use Vault CLI, you need to export the following environment variables
in your shell:

- VAULT_ADDR
- VAULT_TOKEN

## List secrets

If you're using the Key-Value (KV) secrets engine, you can list secrets with:

```sh
vault kv list secret/path
```

## Read secrets

To read a secret in the Key-Value (KV) secrets engine you would use the `vault
kv get` command:

```sh
vault kv get secret/path/my_secret
```

This command will fetch the secret stored at `secret/path/my_secret`.

The output will include metadata as well as the actual secret data. If you want
to see the secret data only, you can use the `-field` option. For example, if
your secret data has a field called `password`, you could fetch just that field
with:

```sh
vault kv get -field=password secret/path/my_secret
```

## Write a secret

Use the `vault kv put` command to write a secret to the vault:

```bash
vault kv put secret/path foo=bar bar=foo
```

## Delete a secret

If you want to remove the secret:

## Duplicate a secret from one path to another

The CLI doesn't offer a built-in way to directly duplicate a secret from one
path to another. However, you can achieve this with a combination of the vault
get and vault put commands.

```sh
SECRET=$(vault kv get -format=json secret/orig | jq -r '.data.data')
echo $SECRET | vault kv put secret/dest -
```

## Modify a secret

If you want to modify a specific field of a secret in HashiCorp Vault's Key-Value (KV) secrets engine, it's important to know that the vault `kv put` command will overwrite the existing secret at the specified path. **This means if you only specify the field you want to change, all other fields in the secret will be lost.**

So to modify a specific field, you should first read the current secret, update the specific field, and then write the entire secret back.

```bash
# Read the original secret
original_secret=$(vault kv get -format=json secret/my_secret | jq -r '.data.data')

# Update the specific field (replace "my_field" and "new_value" with your field and value)
updated_secret=$(echo $original_secret | jq '.my_field = "new_value"')

# Write the updated secret back
echo $updated_secret | vault kv put secret/my_secret -
```
