---
layout:     post
title:      Enabling new floating IP for instances.
subtitle:   "[ OPENSTACK SETUP ]"
categories: [general, setup, openstack]
tags:       [openstack, setup]
date:       2016-05-16 11:10:00 (UTC +5:30)
contributor_name:     Abhishek Shrivastava
contributor_image:    abhishek.jpg
contributor_linkedin: abhishek-shrivastava-03921390
contributor_emailid:  abhishek@cloudbyte.com
---

## Steps to follow :

- Create a IP pool first.

  ```
  nova-manage floating create --pool POOL_NAME --ip_range
  ```

- Now go to openstack horizon and do the following:
	
  - Go to Access & Security -> Floating IPs -> Allocate IP to Project

    ![alt text](https://raw.githubusercontent.com/CloudByteStorages/blog/gh-pages/post_images/floating_ip1.jpg)

  - Now select the pool you created and allocate the IP.

    ![alt text](https://raw.githubusercontent.com/CloudByteStorages/blog/gh-pages/post_images/floating_ip2.jpg)

  - Now go the the IP and associate it with the Instance you wanted.

	![alt text](https://raw.githubusercontent.com/CloudByteStorages/blog/gh-pages/post_images/floating_ip3.jpg)
