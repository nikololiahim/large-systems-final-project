# Large System Final project

## Deployment Steps

These steps are now compiled into Python and shell scripts in the [tools](/tools) directory. 

0. Generate TLS certificates and keys. The following files should be in [`charts/vault/tls`](/charts/vault/tls). 
    * `ca-key.pem` (with private key)
    * `ca.pem` (with cert)
    * `vault-key.pem` (with private key)
    * `vault.pem` (with cert)
For the sake of convenience of testing and debugging, sample files with certificates were already pushed to the repository. These certificates are not used anywhere else.

1. Deploy Vault
```shell
helm install vault vault --values prod-values.yaml
```

2. Initialize and unseal Vault, run a script to create basic policies
```shell
kubectl -n vault exec -it vault-0 -- sh
vault operator init -n 1 -t 1 // will create two tokens: unseal and root 
vault operator unseal <unseal token goes here>
vault login <root token goes here>
sh /home/create-policies.sh
```

3. Deploy MongoDB
```shell
helm install mongo mongo --namespace=vault --values prod-values.yaml
```

4. Create policies for MondoDB credentials rotation
```shell
kubectl -n vault exec -it vault-0 -- sh
vault login <root token goes here>
sh /home/create-mongo-policies.sh
```

5. Create application secrets in Vault
```shell
vault kv put secret/path/is/in/vault/values apiKey=***
```

6. Deploy the application
```shell
helm install app manual-chart --namespace=vault --values prod-values.yaml
```

Wait 10-15 seconds after this step.

7. Go to the application service and query its endpoint:
```
kubectl -n vault port-forward svc/manual-service 32343:3000
```

8. ...

9.  
<details>
<summary>I don't like vault</summary>

![kid named vault](https://i.imgur.com/t8413Ag.png)

</details>
