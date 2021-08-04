# Vault_secret_to_CIC
Injecting citrix ADC Credential to Citrix Ingress Controller. 

# Vault configurations



export VAULT_ADDR=http://127.0.0.1:8200

## Enable KV engine and store your secrets
-------------
vault secrets enable kv-v2

vault policy write citrix-adc-kv-ro citrix-adc-kv-ro.hcl

vault kv put secret/citrix-adc/credential   username=nsroot password=Dynasty@jan2020

vault kv put secret/server-cert/cert cert=@server.crt key=@server.key

## Enable kubernetes authentication for Sidecar vault agent to access the vault



vault auth enable kubernetes

export VAULT_SA_NAME=$(kubectl get sa vault-auth -o jsonpath="{.secrets[*]['name']}")

export SA_JWT_TOKEN=$(kubectl get secret $VAULT_SA_NAME -o jsonpath="{.data.token}" | base64 --decode; echo)

export SA_CA_CRT=$(kubectl get secret $VAULT_SA_NAME -o jsonpath="{.data['ca\.crt']}" | base64 --decode; echo)

vault write auth/kubernetes/config \
issuer="https://kubernetes.default.svc.cluster.local" \
token_reviewer_jwt="$SA_JWT_TOKEN" \
kubernetes_host="https://10.102.33.32:6443" \
kubernetes_ca_cert="$SA_CA_CRT"

vault write auth/kubernetes/role/cic-vault-example \
bound_service_account_names=cic-k8s-role \
bound_service_account_namespaces=default \
policies=citrix-adc-kv-ro \
ttl=24h




