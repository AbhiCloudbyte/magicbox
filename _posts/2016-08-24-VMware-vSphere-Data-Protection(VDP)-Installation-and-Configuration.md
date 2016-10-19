---
layout:     post
title:      VMware vSphere Data Protection(VDP) Installation and Configuration
subtitle:   "[ VMWARE SETUP ]"
categories: [general, setup, vmware, vdp]
tags:       [vmware, setup]
date:       2016-08-24 12:30:00 (UTC +5:30)
contributor_name:     Abhishek Shrivastava
contributor_image:    "abhishek.jpg"
contributor_linkedin: abhishek-shrivastava-03921390
contributor_emailid:  abhishek@cloudbyte.com
---

# VMware vSphere Data Protection(VDP)

- VMware vSphere Data Protection is a backup and recovery solution from VMware. 
- Fully integrated with VMware vCenter Server and VMware vSphere Web Client, providing disk-based backup of virtual machines and applications.
- vSphere Data Protection is based on the industry-leading EMC Avamar deduplication backup and recovery software.
 
## Softwares Used

  Following softwares are used for the Setup purpose:
  
| SOFTWARE        | BUILD/VERSION NUMBER           | 
| --------------- |:------------------------------:| 
| vSPHERE ESX     |              6.0               | 
| vSPHERE vCENTER |              6.0               | 
| VDP             |              6.1               |
| ELASTICENTER    |           1.3.0.920            |

## Prerequisites for VMware vSphere Data Protection(VDP)

- Before installing VDP you need to make sure that your VMware vCenter and the ESX hosts **NTP servers** are synced with the same DNS server,
  which is to be used by the VDP.
  
## Installing VMware vSphere Data Protection(VDP)

- Download the OVA template from 

  ```http
  https://my.vmware.com/web/vmware/details?downloadGroup=VDP61&productId=491
  ```

- Deploy the OVA template on a vCenter.

- While deploying select the Datastore (of size atleast 2TB), on top of which you want to deploy the VDP VM.

  ![alt text](https://raw.githubusercontent.com/CloudByteStorages/blog/gh-pages/post_images/vdp_image1.PNG)

  NOTE: In this case the Datastore seelcted is a CloudByte Volume mounted on the ESX host.

- Now after couple of steps, use the following as the initial Network configuration for VDP VM.
  
  ![alt text](https://raw.githubusercontent.com/CloudByteStorages/blog/gh-pages/post_images/vdp_image2.PNG)

- The deployment will take around 20-25 minutes to complete.

- Once the deployement is done, the next steps will involve the configuration of VDP.

## Configuration of VDP - PART I

- After the deployment is done and the VDP VM is powered on, the first step will be configuring the timezone.

  ![alt text](https://raw.githubusercontent.com/CloudByteStorages/blog/gh-pages/post_images/vdp_image3.PNG)

- Once the above is done SSH into the system using **root** as username and **changeme** as the password. 

- We will then need to update **/etc/hosts** to reflect the IP Address/Hostname of our VDP appliance as well as 
  any others you may want VDP to be able to resolve. 

- It should look something like this:
  
  ![alt text](https://raw.githubusercontent.com/CloudByteStorages/blog/gh-pages/post_images/vdp_image4.PNG)

- Now add the following highighted part in **/etc/reslov.conf**:
  
  ![alt text](https://raw.githubusercontent.com/CloudByteStorages/blog/gh-pages/post_images/vdp_image5.PNG)

- Next we need to edit the dnsmasq configuration file located in **/etc/dnsmasq.conf** and add the following 
  entry (replace it with the IP Address you have assigned your VDP appliance):

  ```
  server=20.10.112.131
  ```
- Finally we just need to start the dnsmasq service as well as enable it to automatically startup on boot by 
  running the following two commands:

  ```
  /etc/init.d/dnsmasq start
  chkconfig dnsmasq on
  ```
- To confirm that everything has been configured correctly, we will perform both a forward and reverse lookup of 
  our VDP hostname and IP Address. You can do so by using **nslookup** command or any others such as dig or host. 
  In this example, I will be using nslookup and you will need to run the following two commands (ensure to replace 
  it with your hostname/IP Address):

  ```
  nslookup vdp.vmcert.com
  nslookup 20.10.112.131
  ```
  ![alt text](https://raw.githubusercontent.com/CloudByteStorages/blog/gh-pages/post_images/vdp_image6.PNG)

- Once you have confirmed both forward and reverse lookups are successful, you can then proceed to configuring your 
  VDP appliance by opening a browser to **https://[VDP-ADDRESS]:8543/vdp-configure** and proceed with moving forward 
  with the VDP configuration wizard.
  
  ``` 
  Note: 
  - Ensure that 127.0.0.1 is shown in the VDP configuration UI for the DNS entry. 
  - For hostname, you will need to use the short hostname even though a hostname is officially defined as specifying the FQDN
  ```

## Configuration of VDP - PART II

- **STEP 1: Network Settings**
  - By default your Stactic IP, Netmask, Gateway and Primary DNS will be present in the settings page.
  - Make sure your Primary DNS is **_127.0.0.1_**, if so proceed further.
  - Enter the Hostname and Domain you provided in **/etc/hosts**.

  ![alt text](https://raw.githubusercontent.com/CloudByteStorages/blog/gh-pages/post_images/vdp_image7.PNG)

- **STEP 2: Setting Up Timezone**
  - Select the timezone according to your region and then proceed further.

- **STEP 3: Updating VDP Credentials**
  - Update the VDP password with you credentials.

  ![alt text](https://raw.githubusercontent.com/CloudByteStorages/blog/gh-pages/post_images/vdp_image8.PNG)

- **STEP 4: vCenter Registration**
  - Enter your vCenter Credentials and do the Test Connection

  ![alt text](https://raw.githubusercontent.com/CloudByteStorages/blog/gh-pages/post_images/vdp_image9.PNG)

  - Once the Test Connection is done you will get the following

  ![alt text](https://raw.githubusercontent.com/CloudByteStorages/blog/gh-pages/post_images/vdp_image10.PNG)

- **STEP 5: Storage Creation**
  - Select the first option and then click next.

  ![alt text](https://raw.githubusercontent.com/CloudByteStorages/blog/gh-pages/post_images/vdp_image11.PNG)

- **STEP 6: Device Allocation**
  - Select the options as mentioned below and click next.

  ![alt text](https://raw.githubusercontent.com/CloudByteStorages/blog/gh-pages/post_images/vdp_image12.PNG)

- **STEP 7: CPU and Memory Configuration**
  - You will get a default value, don't change it and proceed to next step.

  ![alt text](https://raw.githubusercontent.com/CloudByteStorages/blog/gh-pages/post_images/vdp_image13.PNG)

- **STEP 8: Ready To Complete**
  - in this step just click next and you will get the following:
  
  ![alt text](https://raw.githubusercontent.com/CloudByteStorages/blog/gh-pages/post_images/vdp_image14.PNG)

  - On clicking **Yes** it will start configuring the setup and you will get something like below.

  ![alt text](https://raw.githubusercontent.com/CloudByteStorages/blog/gh-pages/post_images/vdp_image15.PNG)

- Once the above steps are done the VDP VM will be rebooted.

## Verification 

- To verify the installation is completed successfully, refersh the browser and enter the new password.
- You will get the following after logging:

  ![alt text](https://raw.githubusercontent.com/CloudByteStorages/blog/gh-pages/post_images/vdp_image16.PNG)

## Connecting VDP to VMware vCenter

- **STEP 1 :** Login to vCenter WEB-Console
- **STEP 2 :** Go to **vSphere Data Protection** tab and connect to the VDP VM you created.
  
  ![alt text](https://raw.githubusercontent.com/CloudByteStorages/blog/gh-pages/post_images/vdp_image17.PNG)

- **STEP 3 :** While adding if you get the following error, then follow STEP 4.

  ![alt text](https://raw.githubusercontent.com/CloudByteStorages/blog/gh-pages/post_images/vdp_image18.PNG)

- **STEP 4 :** Go to **/etc/hosts** and check if the below is present or not:

  ![alt text](https://raw.githubusercontent.com/CloudByteStorages/blog/gh-pages/post_images/vdp_image4.PNG)

  If not add the same again to your **/etc/hosts** and then connect the VDP VM.

- After connecting you will be prompted to the final screen where ypu can run the operations provided in VDP.

  ![alt text](https://raw.githubusercontent.com/CloudByteStorages/blog/gh-pages/post_images/vdp_image19.PNG)
