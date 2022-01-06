# HashiCorp Vault

## Resources to know what is Vault

1. <p><a href="https://learn.hashicorp.com/tutorials/vault/getting-started-intro?in=vault%2Fgetting-started&amp;wvideo=v8vcu8xcch"><img src="https://embedwistia-a.akamaihd.net/deliveries/c0543132560615aabdf0628234a18c08.jpg?image_play_button_size=2x&amp;image_crop_resized=960x540&amp;image_play_button=1&amp;image_play_button_color=1563ffe0" width="400" height="225" style="width: 400px; height: 225px;"></a></p><p><a href="https://learn.hashicorp.com/tutorials/vault/getting-started-intro?in=vault%2Fgetting-started&amp;wvideo=v8vcu8xcch">Introduction to Vault | Vault - HashiCorp Learn</a></p>

2. Important concepts in Vault:

    - Secret Engines: Secret engines are pluggable components that allow secret management for all kinds of backend services.

3. Points to note

    - Vault runs as a server-client setup and only server ever accesses the secret engines and storage backends.
    - Vault needs to be initialized on first load and it needs to be unsealed after every restart.
    - By default, only the key-value the secret engine is enabled and vault can be configured to add in more secret engines and storage backends.
    - We are using the `Standalone` mode in this configuration, which required a persistent storage mounted to the server.
    - The `dev` mode should not be used as is only stored the data in-memory and is unsafe overall.

## Setting up vault to store secrets

https://deepsource.io/blog/setup-vault-kubernetes/

```bash
helm repo add hashicorp https://helm.releases.hashicorp.com

helm install vault hashicorp/vault
```

to run in dev mode we can run:

```bash
helm install vault \
    --set "server.dev.enabled=true" \
    --set "ui.externalPort=8200"
    hashicorp/vault

```

or if we need to change more configurations we can use a file.

```bash
helm install vault hashicorp/vault \
    -f override-values.yml
```

For more indepth configurations have a look at [Vault Helm Configuration](https://www.vaultproject.io/docs/platform/k8s/helm/configuration) page.

After the vaut is run it needs to be initialized with the following:

```bash
kubectl exec -it vault-o -- vault operator init
```

By default this generates `5` secret keys and `1` root token and to unseal the default key you need to provide(`threshold`) is `3`. You can change this by providing additional parameters to above command.

The vault is `sealed` after initialization and we need to `unseal` it to be able to use it. This is done by running the following command `3`( default threshold) times and providing different keys each time.

```bash
vault operator unseal
# or via kubectl
kubectl exec -it vault-o -- vault operator unseal
```

> This can also be done via UI if you enable that in the configuration.

After the vault is unsealed we need to login into the vault using the `root token`. The command for the same is:

```bash
vault login <Initial_Root_Token>
```

The `root` user has all the capabilities so it is not a recommended way to access the vault. You should create a new user for general services, and even a new one for every service that wants to access the vault.

For that, first create a new `policy` and then create a new `token` that uses the specified `policy`.
To create a new `policy` use:

```bash
vault policy write my-policy - << EOF
# Dev servers have version 2 of KV secrets engine mounted by default, so will
# need these paths to grant permissions:
path "secret/data/*" {
  capabilities = ["create", "update"]
}

path "secret/data/foo" {
  capabilities = ["read"]
}
EOF
```
> Note: We are passing the policy in `hcl` format directly into the command.

You can consult the [ACL Scetion](https://www.vaultproject.io/docs/secrets/kv/kv-v2#acl-rules) to write the policy.

To create a new `token` using the newly created `policy` use:

```bash
vault token create -field token -policy=my-policy
```
