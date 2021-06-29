OpenShift vSphere IPI Lab with Static IPs
===========================================

## Description
------------

The goal of this repository is to provide a simple, reproducible way to deploy an OpenShift Container Platform lab using the vSphere IPI method with Static IP Addresses and an internally hosted Load Balancer. Once completed the Cluster will be deployed with 3 Masters with Static IPs, 3 Workers from a MachineSet with DHCP issued IPs, and an optional CoreDNS server to host the DNS entries for the API and Ingress VIPs.

***NOTE: This repository is meant for a Lab only. This method is not supported by Red Hat whatsoever.***

## Quickstart
This is a concise summary of everything you need to do to use the repo. The rest of the document goes into details of every step.
1. Edit `group_vars/all.yml`, the following must be changed while the rest can remain the same
   * pull secret
   * ip and host/domain names
   * vcenter details
     * datastore name
     * datacenter name
     * cluster name
     * username and passwords of admin accounts
2. If you wish to run a specific Channel and Version modify the following in `group_vars/all.yml`:
   * download.channel
   * download.version
3. For the CoreDNS VM to be able to pull the image from Quay.io you must specify an `coredns_vm.upstream_dns`. It cannot have itself as a primary DNS Server.

## Infrastructure Prerequisites

1. vSphere ESXi and vCenter 6.7U3 or 7.0 installed.
   * 6.5 is not supported by this repository due to HW Version 15.
2. A datacenter created with a vSphere host added to it, a datastore exists and has adequate capacity
3. Ansible (preferably latest) on the machine where this repo is cloned.
   * Before you install Ansible, install the `epel-release`, run `yum -y install epel-release`
4. Your DNS Provider (PiHole, AdGuard, etc) should be configured to lookup your `base_domain` from your `coredns_vm.ipaddr`
   * Optionally, you configure the `coredns_vm.upstream_dns` to be your primary DNS Server and then configure your workstation or bastion host to use the CoreDNS Server as your primary DNS Server.
   * If you wish to use the CoreDNS as your primary DNS Server see the deploy-ipi-lab.yml Extra Variables section below.
5. You must be running an OS with a glibc version higher than 2.32 such as Fedora 33 or higher or you can deploy from the [ocp4-vsphere-deploy-container](https://github.com/cptmorgan-rh/ocp4-vsphere-deploy-container)

   ***NOTE: If you are going to use the CoreDNS vm as your primary DNS Server you must specify your vcenter in group_vars/all.yml as an IP address as no A Record will exist.***

 ## Installation Steps

 ### Set Global Variables
 > Pre-populated entries in **group_vars/all.yml** are ready to be used unless you need to customize further. Any updates described below refer to [group_vars/all.yml](group_vars/all.yml) unless otherwise specified.
 1. Get the ***pull secret*** from [here](https://cloud.redhat.com/OpenShift/install/vsphere/user-provisioned). Update the file on the line with location of your `pull_secret`. ex. ~/openshift/pull-secret.json  
 2. Get the vCenter details:
    1. IP address
    2. Admin account username
    3. Admin account password
    4. Datacenter name *(created in the prerequisites mentioned above)*
    5. Cluster name
    6. Datastore name
 3. Downloadable link to `govc` (vSphere CLI, *pre-populated*)
 4. OpenShift cluster
    1. base domain *(pre-populated with **example.com**)*
    2. cluster name *(pre-populated with **ocp4**)*

USAGE
------------

### Run Deploy Playbook
```sh
# Deploy the Lab and all components
ansible-playbook deploy-ipi-lab.yml
```
### deploy-ipi-lab.yml Extra Variables

`config_local_dns=true` - Configures /etc/resolv.conf or systemd-resolved to use CoreDNS as primary DNS after CoreDNS has been deployed.

`skip_dns=true` - Skips deploying a DNS server if proper DNS is already configured.

`specific_version=4.6.z` - Deploys a specific version of OpenShift. Must be in 4.x.z format.

### Run Destroy Playbook
```sh
# Destroy the Lab and all components
ansible-playbook destroy-ipi-lab.yml -e cluster=true

# Destroy the Lab and all components and revert DNS Configuration
ansible-playbook destroy-ipi-lab.yml -e cluster=true -e config_local_dns=true
```

### Expected Outcome

1. Necessary Linux packages installed for the installation
3. Necessary folders [bin, downloads, install-dir] created
4. OpenShift client, install and .ova binaries downloaded to the **downloads** folder
5. Unzipped versions of the binaries installed in the **bin** folder
6. In the **install-dir** folder:
   1. master.ign and worker.ign
   2. Copy of the install-config.yaml
7. A folder is created in the vCenter under the mentioned datacenter and the template is imported
8. The template file is edited to carry certain default settings and runtime parameters common to all the VMs
9. VMs (coredns, bootstrap, master0-2, 3 workers) are generated in the designated folder and (in state of) **poweredon**

```sh
# In the root folder of this repo run the following commands
export KUBECONFIG=$(pwd)/install-dir/auth/kubeconfig
export PATH=$(pwd)/bin:$PATH

# OpenShift Client Commands
oc whoami
oc get co
```
## Special Thanks:

Vijay Chintalapati, Mike Allmen, and all the contributors to [ocp4-vsphere-upi-automation Repo](https://github.com/RedHatOfficial/ocp4-vsphere-upi-automation) that inspired this repository.

AUTHOR
------
Morgan Peterman
