# acm-ansible-collection-demo
A collection of playbooks that show of the capability of the ACM Ansible Collection 
https://galaxy.ansible.com/stolostron/core

## Setup and run the demo from Ansible Automation Controller

### Prerequisites
- an working deployment of ACM 2.5
- an working deployment of Ansible Automation Controller
- ansible

### Modification that you can do to the demo
- Modify the dynamic inventory grouping in `inventories/cluster-inventory-example.yml`
- Add your own cool scenario in the `roles/cool-things-you-do` role
- Modify `playbooks/cluster-mgmt.yml` to run `roles/cool-things-you-do`

### Setup
- fork this repository in GitHub
- clone the forked repo to your local machine
- change directory to the cloned repo
```bash
cd acm-ansible-collection-demo
```
- Setup environment variable for Ansible Automation Controller
```bash
export CONTROLLER_HOST=<ansible automation controller URL>
export CONTROLLER_USERNAME=<username>
export CONTROLLER_PASSWORD=<password>
export CONTROLLER_TOKEN=`awx login | jq -r .token`
export CONTROLLER_VERIFY_SSL=no
```
- Setup environment variable for ACM Ansible Collection modules
```bash
export K8S_AUTH_KUBECONFIG=<path to kubeconfig file>
```
- Run the demo setup playbook
```bash
ansible-playbook playbooks/demo-setup.yml
```
The demo setup playbook will:
- Enable "ClusterProxy" and "Managed ServiceAccount" feature on ACM
- Create the nessary credential for connection to ACM in Ansible Automation Controller
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
        - Setup "ManagedServiceAccount" and "ClusterProxy" addon on the selected clusters
        - Connect to the selected clusters using these ACM features
        - Create a namespace on of all selected clusters (this can be modify to do literally ANYTHING you want to do!)
    - You can launch the job again and modify `"state": "absent"` to remove the created namespace on the selected clusters

### Cleanup after demo
- clone this repo to your local machine
```bash
git clone https://github.com/TheRealHaoLiu/acm-ansible-collection-demo.git
```
- change directory to the cloned repo
```bash
cd acm-ansible-collection-demo
```
- Setup environment variable for Ansible Automation Controller
```bash
export CONTROLLER_HOST=<ansible automation controller URL>
export CONTROLLER_USERNAME=<username>
export CONTROLLER_PASSWORD=<password>
export CONTROLLER_TOKEN=`awx login | jq -r .token`
export CONTROLLER_VERIFY_SSL=no
```
- Setup environment variable for ACM Ansible Collection modules
```bash
export K8S_AUTH_KUBECONFIG=<path to kubeconfig file>
```
- Run the demo cleanup playbook
```bash
ansible-playbook playbooks/demo-cleanup.yml
```
The demo setup playbook will:
- Disable "Cluster Proxy" and "Managed ServiceAccount" feature on ACM
- Delete all resources created by demo setup from Ansible Automation Controller