# Kubernetes on Vagrant practice

Use `vagrant up` to create the Kubernetes cluster.

# Set up kubectl to connect to the cluster

After you're run `vagrant up` and you have a copy of kubectl on your host machine, run these commands to connect to the cluster (changing the paths to the *kubepractice* folder):

```
kubectl config set-cluster local-cluster --server=https://192.168.2.2:6443 --insecure-skip-tls-verify=true

kubectl config set-credentials local-cluster-user --client-certificate=../projects/kubepractice/ca.crt --client-key=../projects/kubepractice/ca.key

kubectl config set-context local-cluster --cluster=local-cluster --user=local-cluster-user

kubectl config  use-context local-cluster
```