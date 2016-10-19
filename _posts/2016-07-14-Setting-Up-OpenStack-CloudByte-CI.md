---
layout:     post
title:      Setting up OpenStack CloudByte CI.
subtitle:   "[ OPENSTACK CLOUDBYTE CI SETUP ]"
categories: [general, setup, openstack, ci]
tags:       [openstack, setup, ci]
date:       2016-07-14 10:04:00 (UTC +5:30)
contributor_name:     Abhishek Shrivastava
contributor_image:    "abhishek.jpg"
contributor_linkedin: abhishek-shrivastava-03921390
contributor_emailid:  abhishek@cloudbyte.com
---

# What is OpenStack CI ?

OpenStack CI is a Continuous Integration Testing environment used to run the **tempest** test suite on each 
local changes a developer submits to **Gerrit** against any specific OpenStack upstream project. 
This patch code submitted then goes through a lot of reviews and CI tests then gets megered into the main OpenStack project repository.

## Why it is helpful ?

OpenStack CI is helpful in a lot of ways for developers by:

- Checking the stability of code.
- Helps in developing the code better.

## How to Setup a CI ?

For setting up a CI follow the instructions:

### Pre-requisites

#### Registering an Upstream Gerrit Account

- You will need to register a Gerrit account with the upstream OpenStack CI platform. 
- You can read the instructions for doing [that](http://ci.openstack.org/third_party.html#requesting-a-service-account).

#### Ensure Basic Packages on Hosts/VMs

- We will be installing a Jenkins master server and infrastructure on one host or virtual machine and one or more Jenkins slave servers on hosts or VMs.
- On each of these target nodes, you will want the base image to have the `wget`, `openssl`, `ssl-cert` and `ca-certificates` packages installed before running anything in this repository.

#### Set Up Your Data Repository 

- For more info on this refer [CloudByte-CI-Data](https://github.com/CloudByteStorages/CloudByte-CI-Data).


### Installing CI Modules

To know more about which modules which be used refer [CloudByte-CI-Modules](https://github.com/CloudByteStorages/CloudByte-CI-Modules).

#### Setting up the Jenkins Master

- On the machine you will use as your Jenkins master, run the following:

```
wget https://raw.github.com/CloudByteStorages/CloudByte-CI-Modules/master/puppet/install_master.sh
bash install_master.sh
```

- The script will install Puppet, create an SSH key for the Jenkins master, create self-signed certificates for Apache, and then will ask you for the URL of the Git repository you are using as your data repository (see Prerequisites above). 

- Enter the URL of your data repository and hit Enter.

Puppet will proceed to set up the Jenkins master.

#### Manual setup of Jenkins scp 1.9 plugin

- Version 1.8 is publicly available, but does not have all features (e.g. copy console log file, copy files after failure, etc.).
- Follow these steps to manually build and install the scp 1.9 plugin:
  * git clone https://github.com/jenkinsci/scp-plugin
  * cd scp-plugin; mvn package
  * Jenkins Manage Plugins; Advanced; Upload Plugin  (target/scp.hpi from the maven build)

#### Restart Jenkins to get the plugins fully installed

    sudo service jenkins restart

#### Load Jenkins Up with Your Jobs

- Run the following at the command line:
  ```
  sudo jenkins-jobs --flush-cache update /etc/jenkins_jobs/config
  ```
  
#### Configuration
- Start zuul
  ```
  sudo service zuul start
  sudo service zuul-merger start
  ```
  
- After Puppet installs Jenkins and Zuul and Nodepool, you will need to do a couple manual configuration steps in the Jenkins UI.

1. Go to the Jenkins web UI. By default, this will be `http://$IP_OF_MASTER:8080`

2. Click the `Manage Jenkins` link on the left

3. Click the `Configure System` link

4. Scroll down until you see "Gearman Plugin Config". Check the "Enable Gearman" checkbox.

5. Click the "Test Connection" button and verify Jenkins connects to Gearman.
6. Scroll to "ZMQ Event Publisher" and select "Enable on all Jobs". Double-check
 the port matches the URL configured for "zmq-publishers" in `$DATA_REPO/etc/nodepool/nodepool.yaml.erb`

7. Scroll down to the bottom of the page and click `Save`

8. At the command line, do this::

    sudo service zuul restart

#### Setting up Static Jenkins Slaves

- On each machine you will use as a Jenkins slave, run:

```
wget https://raw.github.com/CloudByteStorages/CloudByte-CI-Modules/master/puppet/install_slave.sh
bash install_slave.sh
```

- The script will install Puppet, install a Jenkins slave, and install the Jenkins master's
public SSH key in the `authorized_keys` of the Jenkins slave.

- Once the script completes successfully, you need to add the slave node to
Jenkins master. To do so manually, follow these steps:

1. Go to the Jenkins web UI. By default, this will be `http://$IP_OF_MASTER:8080`

2. Click the `Credentials` link on the left

3. Click the `Global credentials` link

4. Click the `Add credentials` link on the left

5. Select `SSH username with private key` from the dropdown labeled "Kind"

6. Enter "jenkins" in the `Username` textbox

7. Select the "From a file on Jenkins master" radio button and enter `/var/lib/jenkins/.ssh/id_rsa` in the File textbox

8. Click the `OK` button

9. Click the "Jenkins" link in the upper left to go back to home page

10. Click the `Manage Jenkins` link on the left

11. Click the `Manage Nodes` link

12. Click the "New Node" link on the left

13. Enter `devstack_slave1` in the `Node name` textbox

14. Select the `Dumb Slave` radio button

15. Click the `OK` button

16. Enter `2` in the `Executors` textbox

17. Enter `/home/jenkins/workspaces` in the `Remote root directory` textbox

18. Enter `devstack_slave` in the `Labels` textbox

19. Enter the IP Address of your slave host or VM in the `Host` textbox

20. Select `jenkins` from the `Credentials` dropdown

21. Click the `Save` button

22. Click the `Log` link on the left. The log should show the master connecting
    to the slave, and at the end of the log should be: "Slave successfully connected and online"

#### Setting up Log Server

- The Log server is a simple VM with an Apache web server installed that provides http access to all the log files uploaded by the jenkins jobs. 
- It is a separate script because the jenkins-zuul-nodepool 'master' server may/can not be publicly accessible for security reasons.
- In addition, separating out the log server as its own server relaxes the disk space requirements needed by the jenkins master. 
- It's configuration uses the openstack-infra scripts, which provide the friendly log filtering features, hightlighting, the line references, etc.
- For simplicity, it is recommended to use the same jenkins key for authentication.

```
wget https://raw.githubusercontent.com/CloudByteStorages/CloudByte-CI-Modules/master/puppet/install_log_server.sh

# MANUALLY Update the LOG_SERVER_DOMAIN & JENKINS_SSH_PUBLIC_KEY_CONTENTS variables
bash install_log_server.sh
```

Bug:

```
err: /Stage[main]/Logging::Master/Exec[install_os-loganalyze]: Failed to call refresh: python setup.py install returned 1 instead of one of [0] at /home/stack/os-ext-testing/puppet/modules/logging/manifests/master.pp:89
```
Workaround:

```
cd /opt/os-loganalyze
sudo python setup.py install
```

- When completed, the jenkins user will be able to upload files to /srv/static/logs, which Apache will serve via http.


