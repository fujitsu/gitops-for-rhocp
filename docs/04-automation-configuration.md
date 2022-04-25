# Automation Configuration and Usage

Given that we have each component ready, next we go through an example to illustrate how to configure and use it.

## Configuration Description

A Network administrator and a cluster administrator will configure the GitOps respectively, then merge their configuration files in turn to deploy a RHOCP 4.9.5 cluster consisting of 3 master nodes and 1 worker node via IPI.

IPI stands for installer-provisioned infrastructure, which is a method to install OpenShift Container Platform on bare metal. Please refer to [RHOCP official documentation](https://docs.openshift.com/container-platform/4.10/installing/installing_bare_metal/preparing-to-install-on-bare-metal.html#choosing-a-method-to-install-ocp-on-bare-metal-installer-provisioned) for more information.

### Overall composition

![overall composition](images/overall-composition.png)

This example includes both configuration and usage sections. The green part shows what needs to be configured, the network administrator and the cluster administrator can do it at the same time. The red part shows the files that need to be merged into the repository when using it, the cluster administrator need to wait for the network configuration to complete before deploying the cluster. These files are inventory, playbook.yaml and install-config.yaml, which respectively represent your PSWITCH connection information, network configuration content and RHOCP cluster installation configuration.

### Server connections on the PSWITCH

![server connections](images/server-connections.png)

This diagram shows how the iRMC servers are connected to the switch and which ports are going to be added to which VLAN. The iRMC server here refers to the server from Fujitsu vendor such as PRIMERGY, which has a Baseboard Management Controller (BMC) with integrated LAN connection and extended functionality. The PSWITCH to which these servers are connected uses a format of slot/number to represent a port interface, e.g.: interface 0/30 means the physical port 30 in slot 0. The baremetal network and the provisioning network respectively represent the network that makes up the RHOCP cluster and the network required to install the cluster via iPXE boot. Please refer to [RHOCP official documentation](https://docs.openshift.com/container-platform/4.10/installing/installing_bare_metal_ipi/ipi-install-overview.html) for more information.

## GitLab Configuration

### For Network Administrator

1. Click on the user icon in the upper right corner then click on "Preferences".

![user preferences](images/gitlab-1.png)

2. Click on "Access Tokens" on the left menu bar.

![access tokens](images/gitlab-2.png)

3. Add a personal access token, configure as follows then click on "Create personal access token".

    - Enter "Token name".
    - Leave "Expiration date" blank to set it permanent.
    - In "Select scopes" we choose "api".

![add token](images/gitlab-3.png)

4. The access token is for configuring Jenkins project later, make sure you save it, you won't be able to access it again. Then click on the GitLab icon in the upper left corner to return to the welcome page.

![save token](images/gitlab-4.png)

5. Click on "Create a project".

![create a project](images/gitlab-5.png)

6. Create new project the way you want. In this example, we just create a blank project".

![create blank project](images/gitlab-6.png)

7. Enter project information and create the project. In "Visibility Level" we choose "Private" for security purpose.

![enter project info](images/gitlab-7.png)

8. Get the HTTP URL for configuring Jenkins project.

![copy http url](images/gitlab-8.png)

9. Hover over "Settings" on the left menu bar then click on "Webhooks" in the submenu.

![click webhooks](images/gitlab-9.png)

10. Configure the webhook as follows then click on "Add webhook".
    - "URL" refers to the step 11 of Jenkins configuration.
    - "Secret token" refers to the step 12 of Jenkins configuration.
    - "Push events" should be the target branch for automation.

![configure webhook](images/gitlab-10.png)

### For Cluster Administrator

All steps are exactly the same as "For Network Administrator" section, please use the cluster administrator information to complete those steps.

## Jenkins Configuration

### For Network Administrator

1. Click on "Manage Jenkins" on the left menu bar then click on "Manage Credentials".

![manage credentials](images/jenkins-1.png)

2. Click on the drop-down menu next to "(global)" then click on "Add credentials".

![add credentials](images/jenkins-2.png)

3. Configure the credential as follows then click on "OK".

    - Enter "Username" and "Password" with network administrator account information.
    - Name "ID".
    - "Description" is optional.

![username credential](images/jenkins-3.png)

4. Click on "Add credentials" to add another credential, configure it as follows then click on "OK".

    - In "Kind" we choose "GitLab API token".
    - "API token" refers to the step 4 of GitLab configuration.
    - Name "ID".
    - "Description" is optional.

![token credential](images/jenkins-4.png)

5. Back to "Manage Jenkins" then click on "Configure System"

![configure system](images/jenkins-5.png)

6. Set "Gitlab"

    - Turn on "Enable authentication for '/project' end-point".
    - Click on "Add" to add a new "GitLab connections" item.
    - Name "Connection name".
    - Enter "Gitlab host URL".
    - In "Credentials", use the access token credential we have created.

![gitlab connections](images/jenkins-6.png)

7. Click on "Save" to save the settings.

![save settings](images/jenkins-7.png)

8. Click on "Dashboard" to back to the welcome page, then click on "Create a job".

![create a job](images/jenkins-8.png)

9. Enter an item name and choose "Freestyle project" then click on "OK".

![freestyle project](images/jenkins-9.png)

10. Set "Source Code Management".

    - Choose "Git".
    - "Repository URL" refers to the step 8 of GitLab configuration.
    - In "Credentials", use the username credential we have created.
    - "Branch Specifier" should be the target branch for automation.

![source code management](images/jenkins-10.png)

11. Set "Build Triggers".

    - Turn on "Build when a change is pushed to GitLab".
    - Copy the GitLab webhook URL for configuring the webhook of GitLab repository.
    - Turn off other triggers and leave only "Push Events".
    - Click on "Advanced" to expand some advanced options.

![build triggers](images/jenkins-11.png)

12. Click on "Generate" to generate "Secret token" for configuring the webhook of GitLab repository.

![advanced options](images/jenkins-12.png)

13. Set "Build", click on "Add build step" and choose "Execute shell".

![execute shell](images/jenkins-13.png)

14. Create shell commands to execute network configuration.

![configure PSWITCH](images/jenkins-14.png)

Command explanation

```
echo "Start configuring PSWITCH"

# Copy the inventory file to Ansible node.
scp /var/lib/jenkins/workspace/network-auto/inventory username@192.168.1.103:~

# Copy the playbook.yaml file to Ansible node.
scp /var/lib/jenkins/workspace/network-auto/playbook.yaml username@192.168.1.103:~

# Remotely execute ansible-playbook command to actually configure the PSWITCH.
ssh username@192.168.1.103 "ansible-playbook -i inventory playbook.yaml"

echo "Finish configuring PSWITCH"
```

Note: In order to run commands remotely, we need to initialize the passwordless ssh access from Jenkins node to Ansible node in advance. Run the following commands on Jenkins node.

```
$ sudo su -s /bin/bash jenkins

$ ssh-keygen
Enter file in which to save the key (/var/lib/jenkins/.ssh/id_rsa): <just press enter>
Enter passphrase (empty for no passphrase): <just press enter>
Enter same passphrase again: <just press enter>

$ ssh-copy-id username@192.168.1.103
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
username@192.168.1.103's password: <input password>
```

15. Set "Post-build Actions", click on "Add post-build action" and choose "Publish build status to GitLab".

![publish status](images/jenkins-15.png)

16. Click on "Save" to complete the creation.

![complete creation](images/jenkins-16.png)

### For Cluster Administrator

Except for step 14, All steps are exactly the same as "For Network Administrator" section, please use the cluster administrator information to complete those steps.

14. Create shell commands to execute RHOCP IPI deployment.

![deploy rhocp](images/jenkins-17.png)

Command explanation

```
echo "Start IPI"

# Remotely execute ipi-clean.sh to clean up the RHOCP IPI installation environment.
ssh username@192.168.1.105 "bash ipi-clean.sh"

# Copy the install-config.yaml file to provisioning node.
scp /var/lib/jenkins/workspace/ipi-auto/install-config.yaml username@192.168.1.105:~

# Remotely execute ipi.sh to actually start IPI installation for RHOCP deployment.
ssh username@192.168.1.105 "bash ipi.sh"

echo "Finish IPI"
```

ipi-clean.sh

```
#!/bin/bash

# Clean up the existing cluster.
openshift-baremetal-install --dir clusterconfigs --log-level debug destroy cluster

# Shut down the iRMC servers.
ipmitool -I lanplus -U [USERNAME for iRMC] -P [PASSWORD for iRMC USER] -H 192.168.30.11 power off
ipmitool -I lanplus -U [USERNAME for iRMC] -P [PASSWORD for iRMC USER] -H 192.168.30.12 power off
ipmitool -I lanplus -U [USERNAME for iRMC] -P [PASSWORD for iRMC USER] -H 192.168.30.13 power off
ipmitool -I lanplus -U [USERNAME for iRMC] -P [PASSWORD for iRMC USER] -H 192.168.30.14 power off

# Remove the existing configuration files.
rm -rf clusterconfigs
```

ipi.sh

```
#!/bin/bash

# Create configuration file path.
mkdir -p clusterconfigs

# Copy install-config.yaml to the configuration file path.
cp install-config.yaml clusterconfigs

# Create manifests.
openshift-baremetal-install --dir clusterconfigs create manifests

# Start installing the cluster.
openshift-baremetal-install --dir clusterconfigs --log-level debug create cluster
```

Note: In order to run commands remotely, we need to initialize the passwordless ssh access from Jenkins node to Ansible node in advance. Run the following commands on Jenkins node.

```
$ sudo su -s /bin/bash jenkins

$ ssh-keygen
Enter file in which to save the key (/var/lib/jenkins/.ssh/id_rsa): <just press enter>
Enter passphrase (empty for no passphrase): <just press enter>
Enter same passphrase again: <just press enter>

$ ssh-copy-id username@192.168.1.103
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
username@192.168.1.103's password: <input password>
```

## Usage

### For Network Administrator

1. Merge inventory and playbook.yaml into the main branch of your repository, which will trigger a build on Jenkins.

inventory

```
[host]
192.168.1.104

[host:vars]
ansible_ssh_user=****
ansible_ssh_pass=****
ansible_connection=network_cli
ansible_network_os=fos
```

playbook.yaml

```
- hosts: host
  gather_facts: no

  tasks:
  - name: "add port 0/29 to vlan 20"
    fos_config:
      lines:
        - switchport access vlan 20
      parents: interface 0/29

  - name: "add port 0/30 to vlan 30"
    fos_config:
      lines:
        - switchport access vlan 30
      parents: interface 0/30

  - name: "add port 0/31 to vlan 20"
    fos_config:
      lines:
        - switchport access vlan 20
      parents: interface 0/31

  - name: "add port 0/32 to vlan 30"
    fos_config:
      lines:
        - switchport access vlan 30
      parents: interface 0/32

  - name: "add port 0/33 to vlan 20"
    fos_config:
      lines:
        - switchport access vlan 20
      parents: interface 0/33

  - name: "add port 0/34 to vlan 30"
    fos_config:
      lines:
        - switchport access vlan 30
      parents: interface 0/34

  - name: "add port 0/35 to vlan 20"
    fos_config:
      lines:
        - switchport access vlan 20
      parents: interface 0/35

  - name: "add port 0/36 to vlan 30"
    fos_config:
      lines:
        - switchport access vlan 30
      parents: interface 0/36
```

For detailed information about inventory and playbook.yaml please refer to [fujitsu/ansible-collection-for-fos repository](https://github.com/fujitsu/ansible-collection-for-fos#using-this-collection).

2. The build status can be seen on commit page.

    - Pending or in progress

    ![in progress](images/result-1.png)

    - Build succeeded

    ![build ok](images/result-2.png)

    - Build failed

    ![build ng](images/result-3.png)

3. When the build failed, login to Jenkins to check the logs. Go to the build of your project and click on "Console Output" to view error messages.

![check error](images/result-4.png)

In this example. the following error message shows that an authentication failure occurred when connecting to the PSWITCH for configuration.

```
fatal: [192.168.1.104]: FAILED! ==> {"changed": false, "msg": "Failed to authenticate: Authentication failed."}
```

This is usually caused by incorrect login information, such as wrong username or password.

### For Cluster Administrator

1. Merge install-config.yaml into the main branch of your repository, which will trigger a build on Jenkins.

install-config.yaml

```
baseDomain: example.local
metadata:
  name: openshift
networking:
  machineNetwork:
  - cidr: 192.168.30.0/24
compute:
- name: worker
  replicas: 1
controlPlane:
  name: master
  replicas: 3
  platform:
    baremetal: {}
platform:
  baremetal:
    apiVIP: 192.168.30.201
    ingressVIP: 192.168.30.202
    provisioningBridge: provisioning
    provisioningNetworkInterface: eno1
    provisioningNetworkCIDR: 192.168.20.0/24
    hosts:
      - name: master-0
        role: master
        bmc:
          address: irmc://192.168.30.11
          username: ****
          password: ****
        bootMACAddress: 90:1b:0e:e6:7b:c1
      - name: master-1
        role: master
        bmc:
          address: irmc://192.168.30.12
          username: ****
          password: ****
        bootMACAddress: 90:1b:0e:e6:7b:c2
      - name: master-2
        role: master
        bmc:
          address: irmc://192.168.30.13
          username: ****
          password: ****
        bootMACAddress: 90:1b:0e:e6:7b:c3
      - name: worker-0
        role: worker
        bmc:
          address: irmc://192.168.30.14
          username: ****
          password: ****
        bootMACAddress: 90:1b:0e:e6:7b:c4
pullSecret: ****
sshKey: ****
```

For detailed information about install-config.yaml please refer to [RHOCP official documentation](https://docs.openshift.com/container-platform/4.10/installing/installing_bare_metal_ipi/ipi-install-installation-workflow.html#configuring-the-install-config-file_ipi-install-installation-workflow). 

Both step 2 and 3 are exactly the same as "For Network Administrator" section.
