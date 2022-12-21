
 (sur le srvprod, il y a un exempe : git clone  https://gitlab.com/kawsark/vault-agent-docker
 sudo docker-compose up -d
 sudo docker exec -it vault_dev sh
 cd vault/scripts;sh 00-secrets.sh)


======= lancement vault via docker sur la vm srvprod, repertoire vault
sudo docker-compose up -d
 
==== creation des policy et des secrets via le navigateur :  http://192.168.5.20:8200


============= configuration vault voir lien https://developer.hashicorp.com/vault/docs/auth/approle

# login
sudo docker exec -it vault_dev sh
export VAULT_ADDR='http://127.0.0.1:8200'
vault login 
token : root

 
# choix de la methode d'authentification par approle 
vault auth enable approle

curl \
    --header "X-Vault-Token:root" \
    --request POST \
    --data '{"type": "approle"}' \
    http://192.168.5.20:8200/v1/sys/auth/approle


# creation des 2 roles avec leur policy respectives
vault write auth/approle/role/env3-secrets token_policies=env3-secrets \ secret_id_ttl=10m \ token_num_uses=10 \ token_ttl=20m \ token_max_ttl=30m \ secret_id_num_uses=40
vault write auth/approle/role/env3-approle token_policies=env3-approle \ secret_id_ttl=10m \ token_num_uses=10 \ token_ttl=20m \ token_max_ttl=30m \ secret_id_num_uses=40^C

curl --header "X-Vault-Token:root" \
     --request POST \
	 --data '{"policies": "env3-secrets"}' \
     http://192.168.5.20:8200/v1/auth/approle/role/env3-secrets

curl --header "X-Vault-Token:root" \
     --request POST \
	 --data '{"policies": "env3-approle"}' \
     http://192.168.5.20:8200/v1/auth/approle/role/env3-approle

curl \
    --header "X-Vault-Token: ..." \
    --request POST \
    --data '{"policies": "dev-policy,test-policy"}' \
    http://127.0.0.1:8200/v1/auth/approle/role/my-role


# voir la liste des rolesvault list auth/approle/role
vault list auth/approle/role


# lecture du role id du premier role
vault read auth/approle/role/env3-secrets/role-id

curl \
    --header "X-Vault-Token:root" \
    http://192.168.5.20:8200/v1/auth/approle/role/env3-secrets/role-id

# generation du secret-id du premier role
vault write -f auth/approle/role/env3-secrets/secret-id

curl \
    --header "X-Vault-Token:root" \
    --request POST \
     http://192.168.5.20:8200/v1/auth/approle/role/env3-secrets/secret-id




