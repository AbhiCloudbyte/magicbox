---
layout:     post
title:      Volume attachment failed in VMs.
subtitle:   "[ OPENSTACK ISSUE ]"
categories: [general, issues, openstack]
tags:       [openstack, issues]
date:       2016-05-25 17:15:00 (UTC +5:30)
contributor_name:     Abhishek Shrivastava
contributor_image:    abhishek.jpg
contributor_linkedin: abhishek-shrivastava-03921390
contributor_emailid:  abhishek@cloudbyte.com
---

## Observation :

- Info gathered from vinder volume logs.
  
  ```
  2015-04-28 12:47:47.672 690 ERROR cinder.volume.manager [req-7ad310ec-8ae1-48d4-8c2b-0ae174695b6c 3f3ecfa581fd42b397b06fe9f9aa2578 bd5fdc9a39fd4586bf2ed6182a4c7585 - - -] Unable to fetch connection information from backend: Command requested root, but did not specify a root helper.

  2015-04-28 12:47:47.673 690 ERROR oslo.messaging.rpc.dispatcher [req-7ad310ec-8ae1-48d4-8c2b-0ae174695b6c 3f3ecfa581fd42b397b06fe9f9aa2578 bd5fdc9a39fd4586bf2ed6182a4c7585 - - -] Exception during message handling: Bad or unexpected response from the storage volume backend API: Unable to fetch connection information from backend: Command requested root, but did not specify a root helper.

  2015-04-28 12:47:47.673 690 TRACE oslo.messaging.rpc.dispatcher Traceback (most recent call last):

  2015-04-28 12:47:47.673 690 TRACE oslo.messaging.rpc.dispatcher   File "/usr/lib/python2.7/site-packages/oslo/messaging/rpc/dispatcher.py", line 134, in _dispatch_and_reply

  2015-04-28 12:47:47.673 690 TRACE oslo.messaging.rpc.dispatcher   incoming.message))

  2015-04-28 12:47:47.673 690 TRACE oslo.messaging.rpc.dispatcher   File "/usr/lib/python2.7/site-packages/oslo/messaging/rpc/dispatcher.py", line 177, in _dispatch

  2015-04-28 12:47:47.673 690 TRACE oslo.messaging.rpc.dispatcher   return self._do_dispatch(endpoint, method, ctxt, args)

  2015-04-28 12:47:47.673 690 TRACE oslo.messaging.rpc.dispatcher   File "/usr/lib/python2.7/site-packages/oslo/messaging/rpc/dispatcher.py", line 123, in _do_dispatch

  2015-04-28 12:47:47.673 690 TRACE oslo.messaging.rpc.dispatcher   result = getattr(endpoint, method)(ctxt, **new_args)

  2015-04-28 12:47:47.673 690 TRACE oslo.messaging.rpc.dispatcher   File "/usr/lib/python2.7/site-packages/osprofiler/profiler.py", line 105, in wrapper

  2015-04-28 12:47:47.673 690 TRACE oslo.messaging.rpc.dispatcher   return f(*args, **kwargs)

  2015-04-28 12:47:47.673 690 TRACE oslo.messaging.rpc.dispatcher   File "/usr/lib/python2.7/site-packages/osprofiler/profiler.py", line 105, in wrapper

  2015-04-28 12:47:47.673 690 TRACE oslo.messaging.rpc.dispatcher   return f(*args, **kwargs)

  2015-04-28 12:47:47.673 690 TRACE oslo.messaging.rpc.dispatcher   File "/usr/lib/python2.7/site-packages/cinder/volume/manager.py", line 902, in initialize_connection

  2015-04-28 12:47:47.673 690 TRACE oslo.messaging.rpc.dispatcher   raise exception.VolumeBackendAPIException(data=err_msg)

  2015-04-28 12:47:47.673 690 TRACE oslo.messaging.rpc.dispatcher VolumeBackendAPIException: Bad or unexpected response from the storage volume backend API: Unable to fetch connection information from backend: Command requested root, but did not specify a root helper.
  ```

## Solution :

- Start the **tgt** service and retry.
