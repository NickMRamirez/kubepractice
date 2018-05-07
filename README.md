# Kubernetes on Vagrant practice

Use `vagrant up` to create the Kubernetes cluster.

# Set up kubectl to connect to the cluster

After you're run `vagrant up` and you have a copy of kubectl on your host machine, `cd` into the project folder and run these commands to connect to the cluster. The `ca.crt` and `ca.key` files should have been generated for you:

```
kubectl config set-cluster local-cluster --server=https://192.168.2.2:6443 --insecure-skip-tls-verify=true

kubectl config set-credentials local-cluster-user --client-certificate=ca.crt --client-key=ca.key

kubectl config set-context local-cluster --cluster=local-cluster --user=local-cluster-user

kubectl config  use-context local-cluster
```

This will update your context in ~/.kube, or on Windows %USERPROFILE%/.kube, to point to the cluster running in Vagrant.