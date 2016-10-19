---
layout:     post
title:      HA issue in cinder.
subtitle:   "[ OPENSTACK ISSUE ]"
categories: [general, issues, openstack]
tags:       [openstack, issues]
date:       2016-05-15 15:10:00 (UTC +5:30)
contributor_name:     Abhishek Shrivastava
contributor_image:    abhishek.jpg
contributor_linkedin: abhishek-shrivastava-03921390
contributor_emailid:  abhishek@cloudbyte.com
---

## Observation :

- Check for host property in /etc/cinder/cinder.conf

  ![alt text](https://raw.githubusercontent.com/CloudByteStorages/blog/gh-pages/post_images/cinder_ha1.jpg)

- Check the host column in volumes table in cinder database

  ![alt text](https://raw.githubusercontent.com/CloudByteStorages/blog/gh-pages/post_images/cinder_ha2.jpg)

## Solution :

- Above screenshot typically imply, host property was set to FQDN of the cinder node. This has got issues in the current HA implementation in cinder.
- Uncomment the **host=cinder** in **/etc/cinder/cinder.conf**.
- Update the volumes records host column to cinder irrespective of the host in which the volume was created.
