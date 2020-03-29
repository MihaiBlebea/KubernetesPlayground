# Start Kubernetes cluster with one cmd

1. Run `vagrant up` to init the cluster (1 master + 2 nodes)
2. To copy files from definitions folder to the master node, run `ansible-playbook deploy-master-playbook -i inventory.yaml`