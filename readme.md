# HashiCorp Vault

## Resources to know what is Vault

1. <p><a href="https://learn.hashicorp.com/tutorials/vault/getting-started-intro?in=vault%2Fgetting-started&amp;wvideo=v8vcu8xcch"><img src="https://embedwistia-a.akamaihd.net/deliveries/c0543132560615aabdf0628234a18c08.jpg?image_play_button_size=2x&amp;image_crop_resized=960x540&amp;image_play_button=1&amp;image_play_button_color=1563ffe0" width="400" height="225" style="width: 400px; height: 225px;"></a></p><p><a href="https://learn.hashicorp.com/tutorials/vault/getting-started-intro?in=vault%2Fgetting-started&amp;wvideo=v8vcu8xcch">Introduction to Vault | Vault - HashiCorp Learn</a></p>

2. Important concepts in Vault:

    - Secret Engines: Secret engines are pluggable components that allow secret management for all kinds of backend services.
    - Vault runs as a server- and client configurations and only server ever accesses the secret engines and storage backends


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
