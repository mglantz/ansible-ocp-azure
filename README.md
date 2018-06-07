# OpenShift on Azure
This project automates the installation of OpenShift on Azure using ansible.  It follows the [OpenShift + Azure Reference Architecture](https://access.redhat.com/documentation/en-us/reference_architectures/2018/html-single/deploying_and_managing_openshift_3.9_on_azure/) closely. By default the following is deployed, 3 masters, 3 Infra nodes, 3 app nodes, 3 cns nodes, Logging (EFK), Metrics, Prometheus & Grafana.

## Topology

![enter image description here](https://access.redhat.com/webassets/avalon/d/Reference_Architectures-2018-Deploying_and_Managing_OpenShift_3.9_on_Azure-en-US/images/582f7bc50a94c64d5fbc330296a2697a/topology.png)

# Pre-Reqs

Reqs
A few Pre-Reqs need to be met and are documented in the Reference Architecture already.  **Ansible 2.5 is required**, the ansible control host running the deployment needs to be registered and subscribed to `rhel-7-server-ansible-2.5-rpms`.  Creating a [Service Principal](https://access.redhat.com/documentation/en-us/reference_architectures/2018/html-single/deploying_and_managing_openshift_3.9_on_azure/#service_principal) is documented as well as setting up the Azure CLI.  Currently the Azure CLI is setup on the ansible control host running the deployment using the playbook `azure_cli.yml` or by following instructions here, [Azure CLI Setup](https://access.redhat.com/documentation/en-us/reference_architectures/2018/html-single/deploying_and_managing_openshift_3.9_on_azure/#azure_cli_setup).

 1. Register the ansible control host used for this deployment with valid RedHat subscription thats able to pull down ansible 2.5 or manually install ansible 2.5. 

`sudo subscription-manager register --username < username > --password < password >` 

`sudo subscription-manager attach --pool < pool_id >` 

`sudo subscription-manager repos --enable rhel-7-server-ansible-2.5-rpms` 

`sudo yum install ansible`

  

 2.  Install Azure CLI,  using playbook included or manually following above directions.
 `ansible-playbook azure-cli.yml`
 4. Authenticate with Azure,  `az login`  described here, [Azure Authentication](https://docs.microsoft.com/en-us/cli/azure/authenticate-azure-cli?view=azure-cli-latest).
 5. Create a Service Principal outlined here, [Creating SP](https://access.redhat.com/documentation/en-us/reference_architectures/2018/html-single/deploying_and_managing_openshift_3.9_on_azure/#service_principal).
 6. Clone this repository
 `git clone https://github.com/hornjason/ansible-ocp-azure.git`
 7. Copy vars.yml.example to vars.yml
  `cd ansible-ocp-azure; cp vars.yml.example vars.yml `
 

## Required Variables
Most defaults are specified in `role/azure/defaults/main.yml`,  Sensitive information is left out and should be entered in `vars.yml`.  Below are required variables that should be filled in before deploying.

 - **location**:  - Azure location for deployment ex. `eastus`
 - **rg**:  - Azure Resource Group ex. `test-rg`
- **admin_user**: - SSH user that will be created on each VM ex. `cloud-user`
- **admin_pubkey**: - Copy paste the Public SSH key that will be added to authorized_keys on each VM ex.
 `ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAB`
 - **admin_privkey**: - Path to the private ssh key associated with the public key above.  ex. `'~/.ssh/id_rsa`
 - **sp_name**: - Service Principal name created in step 5.
 - **sp_secret**: - Service Principal secret 
 - **rhsm_user**: - If subscribing to RHSM using username / password, fill in username
 - **rhsm_pass**: - If subscribing to RHSM using username / password, fill in passowrd for RHSM 
 - **rhsm_key**: -  If subscribing to RHSM using activation key and orgId fill in activation key here.
 - **rhsm_org**: - If subscribing to RHSM using activation key and orgId fill in orgId here.
 - **rhsm_broker_pool**: - If you have a broker pool id for masters / infra nodes fill it in here.  This will be used to for all masters/infra nodes.  If you only have one pool id to use make this the same as `rhsm_node_pool`.
 - **rhsm_node_pool**: - If you have a application pool id for app nodes fill it in here.  This will be used for all application nodes.  If you only have one pool id to use make this the same as `rhsm_broker_pool`
 

Optional Variables:

 - **vnet_cidr**: - Can customize as needed, ex `"10.0.0.0/16"`
By Default the HTPasswdPasswordIdentityProvider is used but can be customized,  this will be templated out to the ansible hosts file.  By default htpasswd user is added.
- **openshift_master_htpasswd_users**: - Contains the user: < passwd hash generated from htpasswd -n user >
- **deploy_cns**: true
- **deploy_metrics**: true
- **deploy_logging**: true
- **deploy_prometheus**: true
- **metrics_volume_size**: '20Gi'
- **logging_volume_size**: '100Gi'
- **prometheus_volume_size**: '20Gi'

# Deployment
After all pre-reqs are met and required variables have been filled out the deployment consists of running the following:
`ansible-playbook deploy.yml -e @vars.yml`

The ansible control host running the deployment will be setup to use ssh proxy through the bastion in order to reach all nodes.  The openshift inventory `hosts` file will be templated into the project root directory and used for the Installation.  