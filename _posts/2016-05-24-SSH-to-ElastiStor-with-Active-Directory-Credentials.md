---
layout:     post
title:      SSH to ElastiStor with Active Directory Credentials
subtitle:   "[ AUTHENTICATION ]"
categories: [general, setup, authentication]
tags:       [authentication, setup]
date:       2016-05-24 15:13:00
---

# SSH to ElastiStor with Active Directory Credentials

To join ElastiStor to an Active directory and performing SSH, we need to follow some guidelines:

- Create a kerberos conf **/etc/krb5.conf** file in ElastiStor.

  ```
  SAMPLE :

  [libdefaults]
  default_realm = <ACTIVE_DIRECTORY_DOMIAN_NAME>.COM
  ticket_lifetime = 24h
  forwardable = yes

  [realms]
  <ACTIVE_DIRECTORY_DOMIAN_NAME>.COM = {
  kdc = <ACTIVE_DIRECTORY_SERVER_IP>
  default_domain = <ACTIVE_DIRECTORY_DOMIAN_NAME>.COM
  admin_server = <ACTIVE_DIRECTORY_SERVER_IP>
  }

  [appdefaults]
  pam = {
  debug = false
  ticket_lifetime = 36000
  renew_lifetime = 36000
  forwardable = true
  krb4_convert = false
  }
  
  EXAMPLE :
  
  [libdefaults]
  default_realm = CBAD.COM
  ticket_lifetime = 24h
  forwardable = yes

  [realms]
  CBAD.COM = {
  kdc = 172.16.51.90
  default_domain = CBAD.COM
  admin_server = 172.16.51.90
  }

  [appdefaults]
  pam = {
  debug = false
  ticket_lifetime = 36000
  renew_lifetime = 36000
  forwardable = true
  krb4_convert = false
  }

  ```

- Edit **/etc/hosts** and add the following :

  ```
  SAMPLE :
  
  #Management IP
  #<NODE_IP>   <NODE_NAME>   <NODE_NAME>.xyz.com
  <NODE_IP>   <NODE_NAME>
  #Management IP
  #AD
  <ACTIVE_DIRECTORY_SERVER_IP>  <ACTIVE_DIRECTORY_DOMIAN_NAME>  <ACTIVE_DIRECTORY_DOMIAN_NAME>.COM
  #AD
  
  EXAMPLE :

  #Management IP
  #172.16.134.3   NODE2   NODE2.cb.com
  172.16.134.3    NODE2
  #Management IP
  #AD
  172.16.51.90    CBAD    CBAD.COM
  #AD  
  
  ```

- In order to connect to Active Directory server we need to install **samba**

- Go to **/cbdir/packages/** in order to install Samba and its dependencies:

  - Installing **openldap-client**

    ```
    pkg_add opnldap-client-X.X.XX.tbz

    ```

  - Installing **samba**

    ```
    pkg_add sambaXX-X.X.XX.tbz

    ```

- Now we need to edit **/usr/local/etc/smb.conf**.

- It contains the sample information, just delete everything and put the following:

  ```
  SAMPLE :
  
  [global]
  workgroup = <ACTIVE_DIRECTORY_DOMIAN_NAME>
  realm = <ACTIVE_DIRECTORY_DOMIAN_NAME>.COM
  security = ads
  template shell = /bin/sh
  winbind use default domain = true
  winbind offline logon = false
  load printers = no
  dns proxy = no
  winbind enum groups = yes
  winbind enum users = yes
  winbind expand groups = 5
  password server = <ACTIVE_DIRECTORY_SERVER_IP>
  idmap uid = 10000-39999
  idmap gid = 10000-39999

  EXAMPLE :
   
  [global]
  workgroup = CBAD
  realm = CBAD.COM
  security = ads
  template shell = /bin/sh
  winbind use default domain = true
  winbind offline logon = false
  load printers = no
  dns proxy = no
  winbind enum groups = yes
  winbind enum users = yes
  winbind expand groups = 5
  password server = 172.16.51.90
  idmap uid = 10000-39999
  idmap gid = 10000-39999 
  
  ```

- Edit **/etc/resolv.conf** with following:

  ```
  SAMPLE :
  
  #Dns info
  #search cb.com
  #nameserver 8.8.8.8
  #Dns info
  nameserver <ACTIVE_DIRECTORY_SERVER_IP>
  domain <ACTIVE_DIRECTORY_DOMAIN_NAME>.com
  
  EXAMPLE :

  #Dns info
  #search cb.com
  #nameserver 8.8.8.8
  #Dns info
  nameserver 172.16.51.90
  domain cbad.com  
  
  ```

- Modify **/etc/nsswitch.conf** with following :

  ```
  OLD_CONFIG :

  group: compat
  passwd: compat

  NEW_CONFIG :

  group: files winbind
  passwd: files winbind

  ```

- Modify **/etc/pam.d/sshd** with following :

  ```
  REPLACE -
    auth            required        pam_unix.so             no_warn try_first_pass
  WITH -
    auth            sufficient      pam_unix.so             no_warn try_first_pass
    auth            required        /usr/local/lib/pam_winbind.so use_first_pass

  ADD -
    session         required        /usr/local/lib/pam_winbind.so mkhomedir

  REPLACE -
    password        required        pam_unix.so             no_warn try_first_pass
  WITH -
    password        sufficient      pam_unix.so             no_warn try_first_pass
    password        required        /usr/local/lib/pam_winbind.so use_first_pass

  ```

- Now start samba

  ```
  service samba onestart

  ```

- Now join the Active Directory server using :

  ```
  SAMPLE :
  
  net ads join -U <ACTIVE_DIRECTORY_SERVER_UNAME>%<AD_SERVER_PASSWORD>

  EXAMPLE :
  
  net ads join -U Administrator%Test@123

  SAMPLE OUTPUT :
    Using short domain name -- CBAD
    Joined 'NODE2' to dns domain 'cbad.com'

  ```

- Now do **wbinfo -u** to get the users list from Active Directory server.

- Go to the Unix client and SSH to ElastiStor using the users in Active Directory.
