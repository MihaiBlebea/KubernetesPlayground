# Kubernetes playground

https://mihaiblebea.github.io/kubernetes-playground/

### Index
- [ ] Resources
- [ ] Security & Roles
- [ ] Secrets & Config
- [ ] [Network](/docs/network.md)
- [ ] Storage & Volumes

### Start Kubernetes cluster with one cmd

1. Run `vagrant up` to init the cluster (1 master + 2 nodes)
2. To copy files from definitions folder to the master node, run `ansible-playbook deploy-master-playbook -i inventory.yaml`


### Usefull resources

1. https://kubernetes.io/docs/reference/kubectl/cheatsheet/ - Cheatsheet with all the commands available in the cli
2. https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/ - API docs references


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
4. `openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout` - Get information about a certificate


### Find the kube-apiserver.yaml

Run command `sudo cat /etc/kubernetes/manifests/kube-apiserver.yaml`


### TLS Certificates - Generate certificate for a new admin user

Step by step flow:
1. Generate a key:
    - `openssl genrsa -out john.key 2048`
2. Generate a certificate signing request:
    - `openssl req -new -key john.key -subj "/CN=john" -out john.csr`
3. Create a certificate signing request object definition file and deploy it with `kubectl create -f`

```yaml
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
    name: john
spec:
    groups:
        - system:authenticated
    usages:
        - digital signature
        - key encipherment
        - server auth
    request:
        <user request base64 encoded>
```
4. Encode base64 a key:
    - `cat john.csr | base64`
5. View all the requests with a kubectl command:
    - `kubectl get csr`
6. Approve a request with command:
    - `kubectl certificate approve john`
7. View the certificate and return to the user
    - `kubectl get csr john -o yaml`
    - copy from `status.certificate`
    - decode from base64 with command `echo "<encoded-certificate>" | base64 --decode >> john.crt`
8. Query the kube api with the new certs:
```js
curl https://192.168.10.10:6443/api/v1/pods \
    --key john.key
    --cert john.crt
    --cacert ca.crt
```
- or you can create a KubeConfig file
- and then call `kubectl get pods --kubeconfig <name-of-file>`
- if you don't want to specify the file everytime, then copy the config file to `$HOME/.kube/config`


### Create roles for users RBAC

- Create a definition file and specify what the role has access to based on the API groups and verbs
Ex:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
    namespace: default
    name: developer
rules:
    - apiGroups: [""] # "" indicates the core API group
      resources: ["pods"]
      verbs: ["get", "watch", "list"]
```

- Now it's time to bind the role to a particular user. Create another definition file for RoleBinding:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
    name: read-pods
    namespace: default
subjects:
    - kind: User
      name: jane # "name" is case sensitive
      apiGroup: rbac.authorization.k8s.io
roleRef:
    kind: Role
    name: developer
    apiGroup: rbac.authorization.k8s.io
```

## Install Nginx Ingress Controller to a bare metal Kubernetes cluster

- Run this command `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.34.1/deploy/static/provider/baremetal/deploy.yaml`