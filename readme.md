# HashiCorp Vault

The Vault is a software to centrally manage and control access to secrets that are shared between multiple services and applications. The idea is to offload all the overhead of storing, encrypting, decrypting and even refreshing secrets to the central vault server and provide limited (and even time limited) access to secrets required by applications.

## Introduction

1. Watch this offical video to learn more about the concept of Vault.

    [Introduction to Vault | Vault - HashiCorp Learn](https://learn.hashicorp.com/tutorials/vault/getting-started-intro?in=vault%2Fgetting-started&amp;wvideo=v8vcu8xcch)
    
    <a href="https://learn.hashicorp.com/tutorials/vault/getting-started-intro?in=vault%2Fgetting-started&amp;wvideo=v8vcu8xcch"><img src="https://embedwistia-a.akamaihd.net/deliveries/c0543132560615aabdf0628234a18c08.jpg?image_play_button_size=2x&amp;image_crop_resized=960x540&amp;image_play_button=1&amp;image_play_button_color=1563ffe0" width="400" height="225" style="width: 400px; height: 225px;"></a>

    or you can read the [Getting Started Guide](https://learn.hashicorp.com/collections/vault/getting-started) .

2. Important concepts in Vault:

    - **Secret Engines** : Secret engines are pluggable components that allow secret management for all kinds of backend services. by default only the `key-value` secret engine is enabled.
    - **Storage Backends** : Storage backends are pluggable components where all the secrets are stored in an encrypted format and are managed by the vault server.

3. Points to note

    - Vault runs as a server-client setup and only server ever accesses the secret engines and storage backends.
    - Vault needs to be initialized on first load and it needs to be unsealed after every restart.
    - By default, only the key-value the secret engine is enabled and vault can be configured to add in more secret engines and storage backends.
    - We are using the `Standalone` mode in this configuration, which required a persistent storage mounted to the server.
    - The `dev` mode should not be used as is only stored the data in-memory and is unsafe overall.

## Setting up vault to store secrets

<https://deepsource.io/blog/setup-vault-kubernetes/>

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

You can consult the [ACL Section](https://www.vaultproject.io/docs/secrets/kv/kv-v2#acl-rules) to write the policy.

To create a new `token` using the newly created `policy` use:

```bash
vault token create -field token -policy=my-policy
```

## Configuration

The configuration for our setup is available in `override-values.yml` file in the root directory of the project.

The configuration sets:

- `standalone` mode which needs a persistent storage mounted.
- `ui` which is accessible at `http://localhost:8200/ui`. For this to work `service` should be enabled and `ServiceType` should be set.
- `api` which is accessible at `http://localhost:8200`

TODOs

- [ ] TLS communication for vault

## References

1. [Getting Started with vault](https://learn.hashicorp.com/collections/vault/getting-started) - Very basics of Vault and setting it up.
2. [Vault on Kubernetes Deployment Guide](https://learn.hashicorp.com/tutorials/vault/kubernetes-raft-deployment-guide#setup-helm-repo) - Setup vault using vault helm (prebuilt helm chart).
3. [Vault Helm Configuration](https://www.vaultproject.io/docs/platform/k8s/helm/configuration) - Additional configurations of the vault helm chart.
