# acm-ansible-collection-demo
A collection of [Ansible](https://www.ansible.com/) playbooks that showcase the capability of the [Red Hat Advance Cluster Management (ACM)](https://www.redhat.com/en/technologies/management/advanced-cluster-management) Ansible Collection 
https://galaxy.ansible.com/stolostron/core

## Table of Contents
* [Set up and run the demo from Ansible Automation Controller](#set-up-and-run-the-demo-from-ansible-automation-controller)
* [Set up and run the demo without Ansible Automation Controller](#set-up-and-run-the-demo-without-ansible-automation-controller)
* [Modification that you can do to the demo (DEFINITELY TRY THIS!)](#modification-that-you-can-do-to-the-demo-definitely-try-this)

## Set up and run the demo from Ansible Automation Controller
### Prerequisites
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
- [AWX CLI](https://docs.ansible.com/automation-controller/latest/html/controllercli/index.html)
- A working deployment of [Ansible Automation Controller](https://www.ansible.com/products/controller)
- A working deployment of [Red Hat Advance Cluster Management](https://www.redhat.com/en/technologies/management/advanced-cluster-management) 2.5+
- Kubernetes clusters managed by Red Hat Advance Cluster Management

### Demo setup
#### Fork and Clone this repository to your local machine
#### Change directory to the cloned repo
```bash
cd acm-ansible-collection-demo
```

#### Set up environment variables for Ansible Automation Controller
```bash
export CONTROLLER_HOST=<ansible automation controller URL>
export CONTROLLER_USERNAME=<username>
export CONTROLLER_PASSWORD=<password>
export CONTROLLER_VERIFY_SSL=no
export CONTROLLER_TOKEN=`awx login | jq -r .token`
```

#### Set up environment variables for ACM Ansible Collection modules
```bash
export K8S_AUTH_KUBECONFIG=<path to kubeconfig file for ACM>
```

> NOTE: See [this](#clusterrole) `ClusterRole` for the minimal access to leverage this collection without being `cluster-admin`.

#### Run the demo setup playbook
```bash
ansible-playbook playbooks/aap-demo-setup.yml
```
The demo setup playbook will:
- Enable "ClusterProxy" and "ManagedServiceAccount" features on ACM
- Create the necessary credential for connection to ACM in Ansible Automation Controller
- Add this repository as a Project in Ansible Automation Controller
- Create cluster inventory with ACM dynamic inventory plugin in Ansible Automation Controller
- Create Job Template from the cluster-mgmt.yml playbook in Ansible Automation Controller

### Running the demo from Ansible Automation Controller
- Login to the Ansible Automation Controller UI
- Test out the ACM dynamic inventory plugin 
    - Click on the "Inventory" tab from left navigation menu
    - Click on the "ACM Cluster Inventory" item in the table
    - Click on the "Source" tab
    - Click sync button next to "ACM Dynamic Cluster Inventory Example" item in the table
    - Click on the "Groups" or the "Hosts" tab to see the clusters in the inventory
- Running the demo playbook
    - Click on the "Templates" tab from left navigation menu
    - Click the launch button next to the "K8S MultiCluster Management Demo" item in the table
    - (optional) Modify the extra_vars in the popup window
    - Click on the "Next" button
    - Click on the "Launch" button
    - Unmodified `playbooks/cluster-mgmt.yml` will:
        - Set up "ClusterProxy" and "ManagedServiceAccount"  addons on the selected clusters
        - Connect to the selected clusters using these ACM features
        - Create a namespace on of all selected clusters (this can be modified to do literally ANYTHING you want to do!)
    - You can launch the job again and modify `"state": "absent"` to remove the created namespace on the selected clusters

### Cleanup after demo
#### Clone this repo to your local machine
```bash
git clone https://github.com/TheRealHaoLiu/acm-ansible-collection-demo.git
```

#### Change directory to the cloned repo
```bash
cd acm-ansible-collection-demo
```

#### Set up environment variables for Ansible Automation Controller
```bash
export CONTROLLER_HOST=<ansible automation controller URL>
export CONTROLLER_USERNAME=<username>
export CONTROLLER_PASSWORD=<password>
export CONTROLLER_TOKEN=`awx login | jq -r .token`
export CONTROLLER_VERIFY_SSL=no
```

#### Set up environment variables for ACM Ansible Collection modules
```bash
export K8S_AUTH_KUBECONFIG=<path to kubeconfig file for ACM>
```
#### Run the demo cleanup playbook
```bash
ansible-playbook playbooks/aap-demo-cleanup.yml
```
The demo setup playbook will:
- Disable "ClusterProxy" and "ManagedServiceAccount" features on ACM
- Delete all resources created by demo setup from Ansible Automation Controller

## Set up and run the demo without Ansible Automation Controller
### Prerequisites
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
- [AWX CLI](https://docs.ansible.com/automation-controller/latest/html/controllercli/index.html)
- An working deployment of [Red Hat Advance Cluster Management](https://www.redhat.com/en/technologies/management/advanced-cluster-management) 2.5+
- Kubernetes cluster managed by [Red Hat Advance Cluster Management](https://www.redhat.com/en/technologies/management/advanced-cluster-management)

### Set up the demo on your laptop
#### Fork and Clone this repository to your local machine
#### Change directory to the cloned repo
```bash
cd acm-ansible-collection-demo
```

#### Install required collections
```bash
ansible-galaxy collection install -r collections/requirements.yml
```

#### Install required python libraries
```bash
pip install kubernetes
```

#### Set up environment variables for ACM Ansible Collection modules
```bash
export K8S_AUTH_KUBECONFIG=<path to kubeconfig file for ACM>
```

#### Run the demo setup playbook
```bash
ansible-playbook playbooks/local-demo-setup.yml
```
The demo setup playbook will:
- Enable "ClusterProxy" and "ManagedServiceAccount" features on ACM

### Run the demo on your laptop
#### Try out the dynamic inventory plugin
```bash
ansible-inventory -i inventories/cluster-inventory-example.yml --list
```

#### Try out generate kubeconfig files for the clusters
```bash
ansible-playbook playbooks/create-kubeconfig.yml -i inventories/cluster-inventory-example.yml -e target_hosts=all-managed-clusters
```

The playbook will:
- Set up "ClusterProxy" and "ManagedServiceAccount"  addons on the selected clusters
- Generate a kubeconfig file in the `kubeconfig` directory for each of the selected clusters
- The generated kubeconfig files will use "ClusterProxy" to connect to the clusters
- The generated kubeconfig files will use "ManagedServiceAccount" to authenticate to the clusters
- **SECURITY NOTE**: By default the created "ManagedServiceAccount" will have the `cluster-admin` ClusterRole and does not have expiration time set. The cleanup playbook will remove the created "ManagedServiceAccount" and render the credential in kubeconfig useless.

#### Try out the multicluster management demo playbook
To create a namespace named `cool-app` on all clusters managed by ACM
```bash
ansible-playbook playbooks/cluster-mgmt.yml -i inventories/cluster-inventory-example.yml -e target_hosts=all-managed-clusters -e state=present -e namespace=cool-app
```

To remove the namespace named `cool-app` on all clusters managed by ACM
```bash
ansible-playbook playbooks/cluster-mgmt.yml -i inventories/cluster-inventory-example.yml -e target_hosts=all-managed-clusters -e state=absent -e namespace=cool-app
```

This playbook will:
- Set up "ClusterProxy" and "ManagedServiceAccount"  addons on the selected clusters
- Connect to the selected clusters using these ACM features
- Create or delete a specified namespace on of all selected clusters (this can be modified to do literally ANYTHING you want to do!)

### Cleanup after demo
#### Clone this repo to your local machine
```bash
git clone https://github.com/TheRealHaoLiu/acm-ansible-collection-demo.git
```

#### Change directory to the cloned repo
```bash
cd acm-ansible-collection-demo
```

#### Set up environment variables for ACM Ansible Collection modules
```bash
export K8S_AUTH_KUBECONFIG=<path to kubeconfig file for ACM>
```

#### Run the demo cleanup playbook
```bash
ansible-playbook playbooks/local-demo-cleanup.yml
```

The demo cleanup playbook will:
- Disable "ClusterProxy" and "ManagedServiceAccount" features on ACM
- All ManagedServiceAccount created will be deleted and render the credentials in kubeconfig useless

## Modification that you can do to the demo (DEFINITELY TRY THIS!)
- Modify the dynamic inventory grouping in `inventories/cluster-inventory-example.yml`
- Add your own cool scenario in the `roles/cool-things-you-do` role
- Modify or add your RBAC configuration for your cool role in `k8s-rbac` directory (or use cluster-admin /shrug)
- Modify `playbooks/cluster-mgmt.yml` to run `roles/cool-things-you-do`

## Examples

### ClusterRole

```yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: stolostron:ansible-collection:core
rules:
- apiGroups:
  - cluster.open-cluster-management.io
  resources:
  - managedclusters
  - managedclusters/finalizers
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - clusterview.open-cluster-management.io
  resources:
  - managedclusters
  - managedclusters/finalizers
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - authentication.open-cluster-management.io
  resources:
  - managedserviceaccounts
  - managedserviceaccounts/status
  - managedserviceaccounts/finalizers
  verbs:
  - '*'
- apiGroups:
  - work.open-cluster-management.io
  resources:
  - manifestworks
  - manifestworks/finalizers
  verbs:
  - '*'
- apiGroups:
  - addon.open-cluster-management.io
  resources:
  - managedclusteraddons
  - managedclusteraddons/status
  - managedclusteraddons/finalizers
  - clustermanagementaddons
  - clustermanagementaddons/status
  - clustermanagementaddons/finalizers
  verbs:
  - '*'
- apiGroups:
  - proxy.open-cluster-management.io
  resources:
  - managedproxyconfigurations
  - managedproxyconfigurations/status
  - managedproxyconfigurations/finalizers
  verbs:
  - '*'
- apiGroups:
  - certificates.k8s.io
  resources:
  - certificatesigningrequests
  - certificatesigningrequests/approval
  - certificatesigningrequests/status
  verbs:
  - get
  - list
  - watch
  - update
  - patch
- apiGroups:
  - certificates.k8s.io
  resourceNames:
  - open-cluster-management.io/proxy-agent-signer
  - kubernetes.io/kube-apiserver-client
  resources:
  - signers
  verbs:
  - get
  - list
  - watch
  - update
  - patch
- apiGroups:
  - route.openshift.io
  resources:
  - routes
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - serviceaccounts
  - services
  - secrets
  verbs:
  - get
  - list
  - watch

```

### ClusterRoleBinding

```yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: stolostron-ansible-collection-crb
subjects:
  - kind: ServiceAccount
    name: lab-rbac-sa
    namespace: open-cluster-management-pipelines-rbac
roleRef:
  kind: ClusterRole
  name: stolostron:ansible-collection:core
  apiGroup: rbac.authorization.k8s.io
```