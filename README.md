# Kubernetes playground


### Start Kubernetes cluster with one cmd

1. Run `vagrant up` to init the cluster (1 master + 2 nodes)
2. To copy files from definitions folder to the master node, run `ansible-playbook deploy-master-playbook -i inventory.yaml`


### Usefull resources

1. https://kubernetes.io/docs/reference/kubectl/cheatsheet/


### Usefull debugging commands and utils:

1. `netstat -ln` install with `apt-get install net-tools` - shows exposed ports and network interfaces
2. `nslookup <domain>` - shows informations about a domain DNS records
3. `for p in $(kubectl get pods --namespace=kube-system -l k8s-app=kube-dns -o name); do kubectl logs --namespace=kube-system $p; done` - check logs of the kube-dns pod in kubernetes
4. `kubectl apply -f https://k8s.io/examples/admin/dns/dnsutils.yaml` - deploy a debug pod to see how the rest of the pods are seen from inside it
5. `kubectl get all -o yaml >> all-definitions.yaml` - backup all the resources created in an imperative or declarative way


### Generate certificates

1. `openssl genrsa -out ca.key 2048`
2. `openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr`
3. `openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt`