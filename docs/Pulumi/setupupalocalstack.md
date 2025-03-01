---
title: Setup Local Stack
---
This covers how to manage Pulumi locally.

Make sure you are the folder where Pulumi.yaml file is.

Login into azure on your machine:

```
az login --use-device-code
```
Login in Pulumi locally
```
pulumi login --local
```
Show stacks configured local on your machine. These are stored in folder called ".pulumi" stored in you users folder in C:\Users 
```
pulumi stack ls
```
To select a stack
```
pulumi stack select stackname
```

You can store these stack files in azure blob.

You can use managed identity or sas/storage keys to access the state files in the Azure Storage Account.

Use managed identity, no need for clientID, secret, SAS token etc:
```
$env:ARM_USE_MSI="true"
```
To use storage key:
```
$env:AZURE_STORAGE_ACCOUNT='saname'
$env:AZURE_STORAGE_KEY='placekeyhere'
```
Then tell Pulumi to use a storage account as a backend
```
pulumi login azblob://state-file?storage_account=saname
```
To create a new stack 
```
pulumi stack init alnew
```
This will create a new stack called 'alnew' it will ask for a password by default in encrypt using a local passphase

You can also you a Azure keyvault to encrypt the config, create a new stack if one 
```
$env:AZURE_KEYVAULT_AUTH_VIA_CLI="true"

 pulumi stack select 'stackname' --create --secrets-provider azurekeyvault://KeyVaultname.vault.azure.net/keys/pulumi
```
To passvaules in pulumi

```
pulumi config set --path 'ict:environment' '${{parameters.StackName}}' --config-file ${{parameters.ConfigurationFilePath}}
```
or
```
pulumi config set --path 'core.Environment.ResourceOwnership.BillingCode' 260
pulumi config set --path 'core.Environment.Name'  Redone 
```