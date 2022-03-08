# Ansible Automation Platform 2 with Hashicorp Vault

## AAP2

I expect you to have a running AAP2.

If not, check out the documentation at https://access.redhat.com/documentation/en-us/red_hat_ansible_automation_platform/2.1

You can install on bare metal, VM but also on OpenShift.

## Set up a Hashicorp Vault dev instance

On a Fedora 35 box:

```bash
podman run --cap-add=IPC_LOCK -e 'VAULT_DEV_ROOT_TOKEN_ID=myroot' -e 'VAULT_DEV_LISTEN_ADDRESS=0.0.0.0:1234' -p 1234:1234 vault
```

You should get something like:

```bash
==> Vault server configuration:

             Api Address: http://0.0.0.0:1234
                     Cgo: disabled
         Cluster Address: https://0.0.0.0:1235
              Go Version: go1.16.7
              Listener 1: tcp (addr: "0.0.0.0:1234", cluster address: "0.0.0.0:1235", max_request_duration: "1m30s", max_request_size: "33554432", tls: "disabled")
               Log Level: info
                   Mlock: supported: true, enabled: false
           Recovery Mode: false
                 Storage: inmem
                 Version: Vault v1.8.2
             Version Sha: aca76f63357041a43b49f3e8c11d67358496959f

==> Vault server started! Log data will stream in below:

2022-03-08T12:03:20.702Z [INFO]  proxy environment: http_proxy="" https_proxy="" no_proxy=""
2022-03-08T12:03:20.702Z [WARN]  no `api_addr` value specified in config or in VAULT_API_ADDR; falling back to detection if possible, but this value should be manually set
2022-03-08T12:03:20.703Z [INFO]  core: security barrier not initialized
2022-03-08T12:03:20.703Z [INFO]  core: security barrier initialized: stored=1 shares=1 threshold=1
2022-03-08T12:03:20.703Z [INFO]  core: post-unseal setup starting
2022-03-08T12:03:20.715Z [INFO]  core: vault is unsealed
[...]
2022-03-08T12:03:20.716Z [INFO]  expiration: revoked lease: lease_id=auth/token/root/h4afa6eed26d5421140a8956a38eea41bc5de28321f023cb53d0168d26ece4b0a
2022-03-08T12:03:20.717Z [INFO]  core: successful mount: namespace="" path=secret/ type=kv
2022-03-08T12:03:20.727Z [INFO]  secrets.kv.kv_5706626e: collecting keys to upgrade
2022-03-08T12:03:20.727Z [INFO]  secrets.kv.kv_5706626e: done collecting keys: num_keys=1
2022-03-08T12:03:20.727Z [INFO]  secrets.kv.kv_5706626e: upgrading keys finished
WARNING! dev mode is enabled! In this mode, Vault runs entirely in-memory
and starts unsealed with a single unseal key. The root token is already
authenticated to the CLI, so you can immediately begin using Vault.

You may need to set the following environment variable:

    $ export VAULT_ADDR='http://0.0.0.0:1234'

The unseal key and root token are displayed below in case you want to
seal/unseal the Vault or re-authenticate.

Unseal Key: UZdssOuEDFCcXXX
Root Token: myroot
`
Development mode should NOT be used in production installations!
```

In a web browser navigate to http://0.0.0.0:1234/

Token: `myroot`

Navigate to Secrets Engines > secret/ > Create secret

- Path for this secret: `root`
- Secret data: `password` with value `redhat`

Save

## Integration with Vault

In AAP2 navigate to Credentials and create a new entry

Name: `Hashicorp Vault`

Credential Type: `Hashicorp Vault Secret Lookup`

Server URL: `http://192.168.122.1:1234` (the IP of your host running podman that AAP should be able to reach)

Token: `myroot`

API version: `v2`

Click test

Path to secret: `secret/root`

Key name: `password`

Click Run

It should say "Hashicorp Vault Test passed"

## Credential

Create a credential called "root password stored in Hashicorp Vault"

Credential type: `Machine`

In the username type `root`

In the password click on the "key" icon. The icon will let you look up your credential in Vault.

Choose `Hashicorp Vault`

Path to secret: `secret/root`

Key name: `password`

When done you should see a label "Hashivault Kv: Hashicorp Vault".

Save your machine credential.

## Use your new Machine credential in a job

In a job, pick up the machine credential you just created.

Run the job and it should run.


