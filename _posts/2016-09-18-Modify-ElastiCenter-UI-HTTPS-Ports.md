---
layout:     post
title:      How to modify the http and https ports for ElastiCenter UI?
subtitle:   "[ CONFIGURATION ]"
categories: [general, setup, configuration, howto]
tags:       [elasticenter, setup]
date:       2016-09-18 19:30:00 (UTC +5:30)
contributor_name:     Abhishek Shrivastava
contributor_image:    "abhishek.jpg"
contributor_linkedin: abhishek-shrivastava-03921390
contributor_emailid:  abhishek@cloudbyte.com
---

# Introduction

  ElastiCenter UI is enabled using apache-tomcat and it is configured by default to listen on http(80), which is redirected to https(443). 
  
  You can change these default settings using the below steps.
  
  ElastiCenter UI ports configuration is in the file: /usr/local/apache-tomcat-6.0.33/conf/server.xml
  
### Configuration Steps

**STEP 1:**
  ssh into ElastiCenter
  Take a backup of the /usr/local/apache-tomcat-6.0.33/conf/server.xml before making any changes.
  
**STEP 2:**  
  Edit the http(80) and https(443) in the server.xml to the desired port numbers. Also, change the redirect port (443) to the new https port. 
  
  You will required to change the following lines:
  
```
    <Connector port="80" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="443" />
  
  ...<snipped>
  
        <Connector executor="tomcatThreadPool-internal" address="127.0.0.1"
               port="7080" protocol="org.apache.coyote.http11.Http11NioProtocol"
               connectionTimeout="20000" disableUploadTimeout="true"
               acceptCount="150" enableLookups="false" maxThreads="150"
               maxHttpHeaderSize="8192" redirectPort="443" />  

...<snipped>

    <Connector port="443" protocol="HTTP/1.1" SSLEnabled="true"
               maxThreads="150" scheme="https" secure="true"
               clientAuth="false" sslProtocol="TLS"
               keystoreType="JKS"
               keystoreFile="conf/cloud.keystore"
               keystorePass="vmops.com"
               compression="on"
           compressionMinSize="1024"
           noCompressionUserAgents="gozilla, traviata"
           compressableMimeType="text/html,text/xml,text/json,text/javascript,text/css,text/plain,
           application/javascript,application/xml,application/xml+xhtm"
           sslEnabledProtocols="TLSv1,TLSv1.1,TLSv1.2"
               ciphers="TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_DHE_RSA_WITH_AES_128_GCM_SHA256,
                        TLS_DHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,
                        TLS_DHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,
                        TLS_DHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,
                        TLS_DHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,
                        TLS_DHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,
                        SSL_RSA_WITH_3DES_EDE_CBC_SHA"
               />

...<snipped>

    <Connector port="20400" protocol="AJP/1.3" redirectPort="443" />

```

**STEP 3:**
   Restart ElastiCenter UI
   
```
   root@EC:~ # pgrep java
   2188
   root@EC:~ # kill -9 2188
   root@EC:~ # /usr/local/apache-tomcat-6.0.33/bin/startup.sh
   root@EC:~ #
```

