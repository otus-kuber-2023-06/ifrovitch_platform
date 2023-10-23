

kubectl create ns  vault
helm repo add hashicorp https://helm.releases.hashicorp.com
helm install consul hashicorp/consul -n vault --set global.name=consul

 $ helm status consul --namespace vault
  $ helm get all consul --namespace vault

helm search repo hashicorp/vault


 helm install vault hashicorp/vault -n vault  -f .config/vault-values.yaml --version 0.24.0

 $ helm status vault
  $ helm get manifest vault

NAME: vault
LAST DEPLOYED: Wed Oct 11 16:49:24 2023
NAMESPACE: vault
STATUS: deployed
REVISION: 1

kubectl logs pod/vault-0 -n vault
2023-10-11T13:53:31.435Z [INFO]  core: seal configuration missing, not initialize

NAME                                               READY   STATUS    RESTARTS   AGE
pod/consul-connect-injector-5846b89796-qf6zd       1/1     Running   0          6h41m
pod/consul-server-0                                1/1     Running   0          6h41m
pod/consul-webhook-cert-manager-868477b5db-fl7tt   1/1     Running   0          6h41m
pod/vault-0                                        0/1     Running   0          4m31s
pod/vault-agent-injector-59fdd7cdf8-ssqbw          1/1     Running   0          4m32s

kubectl -n vault exec -it vault-0 -- vault operator init --key-shares=1 --key-threshold=1

Unseal Key 1: Y9WSTqEPnJrmICEZqUKxusRPrCCfxVx+UsslEmkrhEk=

Initial Root Token: hvs.vLgoEiVez0MrMEGOBOawEglL

kubectl -n vault exec -it vault-0 -- vault operator unseal $unsealKey

kubectl -n vault exec -it vault-0 -- vault status

kubectl -n vault exec -it vault-0 -- vault status
Key                Value
---                -----
Seal Type          shamir
Initialized        true
Sealed             true

kubectl -n vault  exec -it pod/vault-0 -- env | grep VAULT
VAULT_K8S_NAMESPACE=vault
VAULT_ADDR=http://127.0.0.1:8200
VAULT_API_ADDR=http://10.124.129.7:8200
VAULT_CLUSTER_ADDR=https://vault-0.vault-internal:8201
VAULT_K8S_POD_NAME=vault-0
VAULT_PORT=tcp://172.16.170.77:8200
VAULT_AGENT_INJECTOR_SVC_SERVICE_PORT_HTTPS=443
VAULT_AGENT_INJECTOR_SVC_PORT_443_TCP_PROTO=tcp
VAULT_AGENT_INJECTOR_SVC_SERVICE_HOST=172.16.152.138
VAULT_PORT_8201_TCP_PORT=8201
VAULT_PORT_8201_TCP_ADDR=172.16.170.77
VAULT_PORT_8200_TCP=tcp://172.16.170.77:8200
VAULT_SERVICE_PORT_HTTPS_INTERNAL=8201
VAULT_PORT_8201_TCP_PROTO=tcp
VAULT_SERVICE_PORT_HTTP=8200
VAULT_AGENT_INJECTOR_SVC_PORT_443_TCP=tcp://172.16.152.138:443
VAULT_PORT_8200_TCP_PROTO=tcp
VAULT_AGENT_INJECTOR_SVC_PORT=tcp://172.16.152.138:443
VAULT_SERVICE_PORT=8200
VAULT_PORT_8201_TCP=tcp://172.16.170.77:8201
VAULT_SERVICE_HOST=172.16.170.77
VAULT_AGENT_INJECTOR_SVC_SERVICE_PORT=443
VAULT_AGENT_INJECTOR_SVC_PORT_443_TCP_PORT=443
VAULT_PORT_8200_TCP_PORT=8200
VAULT_PORT_8200_TCP_ADDR=172.16.170.77
VAULT_AGENT_INJECTOR_SVC_PORT_443_TCP_ADDR=172.16.152.138

kubectl -n vault  exec -it pod/vault-0 -- vault auth list
Error listing enabled authentications: Error making API request.

URL: GET http://127.0.0.1:8200/v1/sys/auth
Code: 403. Errors:

* permission denied

kubectl -n vault  exec -it pod/vault-0 -- vault login
Token (will be hidden):
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                hvs.vLgoEiVez0MrMEGOBOawEglL
token_accessor       lD4QuDYVJX6W4InF08x9uU6o
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]

kubectl -n vault  exec -it pod/vault-0 -- vault auth list
Path      Type     Accessor               Description                Version
----      ----     --------               -----------                -------
token/    token    auth_token_f9fc0e49    token based credentials    n/a

kubectl -n vault  exec -it pod/vault-0 -- vault secrets enable --path=otus kv
Success! Enabled the kv secrets engine at: otus/

kubectl -n vault  exec -it pod/vault-0 -- vault secrets list --detailed
Path          Plugin       Accessor              Default TTL    Max TTL    Force No Cache    Replication    Seal Wrap    External Entropy Access    Options    Description                                                UUID                                    Version    Running Version          Running SHA256    Deprecation Status
----          ------       --------              -----------    -------    --------------    -----------    ---------    -----------------------    -------    -----------                                                ----                                    -------    ---------------          --------------    ------------------
cubbyhole/    cubbyhole    cubbyhole_368cbdba    n/a            n/a        false             local          false        false                      map[]      per-token private secret storage                           38807e38-832d-4a89-8a3e-3a0c49f213a6    n/a        v1.14.0+builtin.vault    n/a               n/a
identity/     identity     identity_2fb6714e     system         system     false             replicated     false        false                      map[]      identity store                                             f903e98a-72a7-5525-7816-5a9057b20f01    n/a        v1.14.0+builtin.vault    n/a               n/a
otus/         kv           kv_1b1a0fc8           system         system     false             replicated     false        false                      map[]      n/a
                                                  99202f16-7ba5-4f2b-069e-44e0d87666cc    n/a        v0.15.0+builtin          n/a               supported
sys/          system       system_f3064b29       n/a            n/a        false             replicated     true         false                      map[]      system endpoints used for control, policy and debugging    f17564f6-fc6a-4431-b8cc-a57fcc5bd96a    n/a        v1.14.0+builtin.vault    n/a               n/a

kubectl -n vault  exec -it pod/vault-0 -- vault kv put otus/otus-ro/config username='otus' password='asalklkahs'
Success! Data written to: otus/otus-ro/config

kubectl -n vault  exec -it pod/vault-0 -- vault read otus/otus-ro/config
Key                 Value
---                 -----
refresh_interval    768h
password            asalklkahs
username            otus


kubectl -n vault  exec -it pod/vault-0 -- vault kv get otus/otus-ro/config
====== Data ======
Key         Value
---         -----
password    asalklkahs
username    otus

kubectl -n vault  exec -it pod/vault-0 -- vault auth enable kubernetes
Success! Enabled kubernetes auth method at: kubernetes/


kubectl -n vault  exec -it pod/vault-0 -- vault auth list
Path           Type          Accessor                    Description                Version
----           ----          --------                    -----------                -------
kubernetes/    kubernetes    auth_kubernetes_90cfb238    n/a                        n/a
token/         token         auth_token_f9fc0e49         token based credentials    n/a

kubectl create serviceaccount vault-auth -n vault
serviceaccount/vault-auth created

kubectl apply --filename vault-auth-service-account.yml

kubectl apply -f .config/vault-auth-service-account.yaml

clusterrolebinding.rbac.authorization.k8s.io/role-tokenreview-binding created

export VAULT_SA_NAME=$(kubectl -n vault get sa vault-auth -o jsonpath="{.secrets[*]['name']}")
export SA_JWT_TOKEN=$(kubectl -n vault get secret $VAULT_SA_NAME -o jsonpath="{.data.token}" | base64 --decode; echo)
export SA_CA_CRT=$(kubectl -n vault get secret $VAULT_SA_NAME -o jsonpath="{.data['ca\.crt']}" | base64 --decode; echo)
export K8S_HOST=$(more ~/.kube/config | grep server |awk '/http/ {print $NF}')

 kubectl -n vault  exec -it pod/vault-0  -- vault write auth/kubernetes/config token_reviewer_jwt="$SA_JWT_TOKEN" kubernetes_host="$K8S_HOST" kubernetes_ca_cert="$SA_CA_CRT"
Success! Data written to: auth/kubernetes/config

tee otus-policy.hcl <<EOF
    path "otus/otus-ro/*" {
    capabilities = ["read", "list"]
    }
    path "otus/otus-rw/*" {
    capabilities = ["read", "create", "list"]
    }
> EOF

kubectl   cp otus-policy.hcl -n vault vault-0:/tmp

kubectl -n vault exec -it vault-0 -- vault policy write otus-policy /tmp/otus-policy.hcl
Success! Uploaded policy: otus-policy

kubectl -n vault exec -it vault-0 -- vault write auth/kubernetes/role/otus bound_service_account_names=vault-auth bound_service_account_namespaces=vault policies=otus-policy ttl=240h
Success! Data written to: auth/kubernetes/role/otus

kubectl run --generator=run-pod/v1 tmp --rm -i --tty --serviceaccount=vault-auth --image alpine:3.7

VAULT_ADDR=http://vault:8200
KUBE_TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
curl --request POST --data '{"jwt": "'$KUBE_TOKEN'", "role": "otus"}' $VAULT_ADDR/v1/auth/kubernetes/login | jq

TOKEN=$(curl -k -s --request POST --data '{"jwt": "'$KUBE_TOKEN'", "role": "otus"}' $VAULT_ADDR/v1/auth/kubernetes/login | jq '.auth.client_token' | awk -F\" '{print $2}')

curl --header "X-Vault-Token:$TOKEN" $VAULT_ADDR/v1/otus/otus-ro/config
curl --header "X-Vault-Token:$TOKEN" $VAULT_ADDR/v1/otus/otus-rw/config

    / # curl --request POST --data '{"bar": "baz"}' --header "X-Vault-Token:$TOKEN" $VAULT_ADDR/v1/otus/otus-rw/config
    {"errors":["1 error occurred:\n\t* permission denied\n\n"]}
    / # curl --request POST --data '{"bar": "baz"}' --header "X-Vault-Token:$TOKEN" $VAULT_ADDR/v1/otus/otus-ro/config
    {"errors":["1 error occurred:\n\t* permission denied\n\n"]}
    / # curl --request POST --data '{"bar": "baz"}' --header "X-Vault-Token:$TOKEN" $VAULT_ADDR/v1/otus/otus-rw/config1

kubectl -n vault exec -it vault-agent-example -c nginx-container -- cat /usr/share/nginx/html/index.html
    <html>
    <body>
    <p>Some secrets:</p>
    <ul>
    <li><pre>username: otus</pre></li>
    <li><pre>password: asalklkahs</pre></li>
    </ul>

    </body>
    </html>


kubectl -n vault exec -it vault-0 -- vault secrets enable pki
Success! Enabled the pki secrets engine at: pki/

kubectl -n vault exec -it vault-0 -- vault secrets tune -max-lease-ttl=87600h pki
Success! Tuned the secrets engine at: pki/

kubectl -n vault exec -it vault-0 -- vault write -field=certificate pki/root/generate/internal common_name="example.ru" ttl=87600h > CA_cert.crt

kubectl -n vault exec -it vault-0 -- vault write pki/config/urls issuing_certificates="http://vault:8200/v1/pki/ca" crl_distribution_points="http://vault:8200/v1/pki/crl"
Key                        Value
---                        -----
crl_distribution_points    [http://vault:8200/v1/pki/crl]
enable_templating          false
issuing_certificates       [http://vault:8200/v1/pki/ca]
ocsp_servers               []

kubectl -n vault exec -it vault-0 -- vault secrets enable --path=pki_int pki
Success! Enabled the pki secrets engine at: pki_int/

 kubectl -n vault exec -it vault-0 -- vault secrets tune -max-lease-ttl=87600h pki_int
Success! Tuned the secrets engine at: pki_int/

 kubectl -n vault exec -it vault-0 -- vault write -format=json pki_int/intermediate/generate/internal common_name="example.ru Intermediate Authority" | jq -r '.data.csr' > pki_intermediate.csr

 kubectl cp pki_intermediate.csr -n vault vault-0:/

 kubectl -n vault exec -it vault-0 -- vault write -format=json pki/root/sign-intermediate csr=@pki_intermediate.csr format=pem_bundle ttl="43800h" | jq -r '.data.certificate' > intermediate.cert.pem

kubectl cp intermediate.cert.pem vault-0:./
kubectl exec -it vault-0 -- vault write pki_int/intermediate/set-signed certificate=@intermediate.cert.pem

kubectl exec -it vault-0 -- vault write pki_int/intermediate/set-signed
certificate=@intermediate.cert.pem

kubectl exec -it vault-0 -- vault write pki_int/roles/example-dot-ru \
allowed_domains="example.ru" allow_subdomains=true max_ttl="720h"

