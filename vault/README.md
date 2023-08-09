# Vault configurations

Login to keswick ESXi node, the execute following cmds :

inf-cli kubectl -n kube-system get pods
```
NAME                                                                READY   STATUS    RESTARTS        AGE
inf-runtime-b7a14d56-7a24-796e-b167-f95727228438                    4/4     Running   0               2d5h
keswick-system-control-plane-b7a14d56-7a24-796e-b167-f95727228438   13/14   Running   37 (2d5h ago)   2d5h
```

Set Control plane ID :
export CP_ID=keswick-system-control-plane-b7a14d56-7a24-796e-b167-f95727228438

inf-cli kubectl -n kube-system exec -it  $CP_ID -c kube-apiserver  -- sh

### Now interact with vault pods to setup the raft server:

```
sh-5.0# export KUBECONFIG=/etc/kubernetes/admin.conf
sh-5.0# kubectl -n ns-hashicorp-vault get pods
NAME                                READY   STATUS    RESTARTS   AGE
vault-rn-injector-968fb766f-g9kzt   1/1     Running   0          3m40s
vault-rn-server-0                   0/1     Running   0          3m40s
```

## Unseal the said vault instance:

kubectl -n ns-hashicorp-vault exec vault-rn-server-0 -- vault operator init -key-shares=1 -key-threshold=1 -format=json > keys.json

Note: Next command will use `jq`, run tdnf install `jq` to install it for photon os.

```
export VAULT_UNSEAL_KEY=$(jq -r ".unseal_keys_b64[]" keys.json)

kubectl -n ns-hashicorp-vault exec vault-rn-server-0 -- vault operator unseal $VAULT_UNSEAL_KEY
```

Vault should be ready now :
```
kubectl -n ns-hashicorp-vault get pods                               
NAME                                READY   STATUS    RESTARTS   AGE
vault-rn-injector-f67dd7dcf-flzsj   1/1     Running   0          6m1s
vault-rn-server-0                   1/1     Running   0          6m1s
```

Note : We are using bitnami helm configurations which are different from hashicorps default helm config flavours.

References :
1. Seal/Unseal : https://developer.hashicorp.com/vault/docs/concepts/seal
2. Raft Setup  : https://developer.hashicorp.com/vault/tutorials/kubernetes/kubernetes-minikube-raft
3. Storing secrets : https://developer.hashicorp.com/vault/tutorials/kubernetes/kubernetes-minikube-raft#set-a-secret-in-vault