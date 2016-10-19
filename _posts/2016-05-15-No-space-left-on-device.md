---
layout:     post
title:      No space left on device.
subtitle:   "[ OPENSTACK ISSUE ]"
categories: [general, issues, openstack]
tags:       [openstack, issues]
date:       2016-05-15 16:35:00 (UTC +5:30)
contributor_name:     Abhishek Shrivastava
contributor_image:    abhishek.jpg
contributor_linkedin: abhishek-shrivastava-03921390
contributor_emailid:  abhishek@cloudbyte.com
---

## Observation :

- Cinder volume logs

  ```
  cinder-volume Volume 7a59c80e-740a-4874-94d5-84b5851b66f5: being created as image with specification: 
    {'status': u'creating',
    'image_location': (u'file:///nfsmount/283c991d-89df-4a70-89c6-1b4474cc067f', None),
    'volume_size': 40, 
    'volume_name': u'volume-7a59c80e-740a-4874-94d5-84b5851b66f5',
    'image_id': u'283c991d-89df-4a70-89c6-1b4474cc067f', 
    'image_service': <cinder.image.glance.GlanceImageService object at 0x7fa348ab3dd0>, 
    'image_meta': {'status': u'active', 'name': u'kvm-W2K8ent', 'deleted': None, 'container_format': u'bare', 
        'created_at': datetime.datetime(2015, 9, 22, 13, 15, 58, tzinfo=<iso8601.iso8601.Utc object at 0x7fa348dd3350>), 
        'disk_format': u'qcow2', 
        'updated_at': datetime.datetime(2015, 9, 22, 13, 29, 14, tzinfo=<iso8601.iso8601.Utc object at 0x7fa348dd3350>), 
        'id': u'283c991d-89df-4a70-89c6-1b4474cc067f', 
        'owner': u'c4a555c49ce84120933bce825464a256', 
        'min_ram': 0, 
        'checksum': u'7884543f3b6c63d40da21dc2270645f5', 
        'min_disk': 0, 
        'is_public': None, 
        'deleted_at': None, 
        'properties': {}, 
        'size': 21474836480}
    }

  cinder-volume Failed to copy image 283c991d-89df-4a70-89c6-1b4474cc067f to 
      volume: 7a59c80e-740a-4874-94d5-84b5851b66f5, 
      error: [Errno 28] No space left on device
  ```

## Solution :

- We noticed that **/dev/mapper/os-root** of cinder-volume node was eating up the entire space.
- Hence, it will be advisable to set a location which has enough space e.g. 40 GB to 50 GB 
  (assuming all the images will be within this capacity)
- We can set the permissions of the location to "cinder" user & set it at cinder.conf's **image_conversion_dir** key.
