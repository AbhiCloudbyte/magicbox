---
layout:     post
title:      OpenStack Issue - No Space left on device
categories: [general, issues, openstack]
tags:       [openstack, issues]
date:       2016-05-13 12:26:00
---

# Log Details

  
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
  .
  .
  .
  .
  2015-11-12 12:47:29.308 85317 TRACE cinder.volume.manager Traceback (most recent call last):
    File "/usr/lib/python2.7/dist-packages/taskflow/engines/action_engine/executor.py", line 35, in _execute_task
            result = task.execute(**arguments)
    File "/usr/lib/python2.7/dist-packages/cinder/volume/flows/manager/create_volume.py", line 638, in execute
            **volume_spec)
    File "/usr/lib/python2.7/dist-packages/cinder/volume/flows/manager/create_volume.py", line 590, in _create_from_image
            image_id, image_location, image_service)
    File "/usr/lib/python2.7/dist-packages/cinder/volume/flows/manager/create_volume.py", line 504, in _copy_image_to_volume
            raise exception.ImageCopyFailure(reason=ex)
  2015-11-12 12:47:29.308 85317 TRACE cinder.volume.manager 
    ImageCopyFailure: Failed to copy image to volume: [Errno 28] No space left on device
  2015-11-12 12:47:29.308 85317 TRACE cinder.volume.manager
  .
  .
  .
  .
  cinder-volume Volume 7a59c80e-740a-4874-94d5-84b5851b66f5: create failed
  Flow 'volume_create_manager' (5ecb731d-b20b-41c4-b9cf-5d746a8be73c) transitioned into state 'REVERTED' from state 'RUNNING'
  Nov 12 12:47:29 node-174 cinder-volume Exception during message handling: 
    Failed to copy image to volume: [Errno 28] No space left on device
  2015-11-12 12:47:29.364 85317 TRACE oslo.messaging.rpc.dispatcher Traceback (most recent call last):
    File "/usr/lib/python2.7/dist-packages/oslo/messaging/rpc/dispatcher.py", line 137, in _dispatch_and_reply
        incoming.message))
    File "/usr/lib/python2.7/dist-packages/oslo/messaging/rpc/dispatcher.py", line 180, in _dispatch
        return self._do_dispatch(endpoint, method, ctxt, args)
    File "/usr/lib/python2.7/dist-packages/oslo/messaging/rpc/dispatcher.py", line 126, in _do_dispatch
        result = getattr(endpoint, method)(ctxt, **new_args)
    File "/usr/lib/python2.7/dist-packages/osprofiler/profiler.py", line 105, in wrapper
        return f(*args, **kwargs)
    File "/usr/lib/python2.7/dist-packages/cinder/volume/manager.py", line 399, in create_volume
        _run_flow()
    File "/usr/lib/python2.7/dist-packages/cinder/volume/manager.py", line 392, in _run_flow
        flow_engine.run()
    File "/usr/lib/python2.7/dist-packages/taskflow/engines/action_engine/engine.py", line 98, in run
        for _state in self.run_iter():
    File "/usr/lib/python2.7/dist-packages/taskflow/engines/action_engine/engine.py", line 155, in run_iter
        misc.Failure.reraise_if_any(failures.values())
    File "/usr/lib/python2.7/dist-packages/taskflow/utils/misc.py", line 670, in reraise_if_any
        failures[0].reraise()
    File "/usr/lib/python2.7/dist-packages/taskflow/utils/misc.py", line 677, in reraise
        six.reraise(*self._exc_info)
    File "/usr/lib/python2.7/dist-packages/taskflow/engines/action_engine/executor.py", line 35, in _execute_task
        result = task.execute(**arguments)
    File "/usr/lib/python2.7/dist-packages/cinder/volume/flows/manager/create_volume.py", line 638, in execute
        **volume_spec)
    File "/usr/lib/python2.7/dist-packages/cinder/volume/flows/manager/create_volume.py", line 590, in _create_from_image
        image_id, image_location, image_service)
    File "/usr/lib/python2.7/dist-packages/cinder/volume/flows/manager/create_volume.py", line 504, in _copy_image_to_volume
        raise exception.ImageCopyFailure(reason=ex)

  ImageCopyFailure: Failed to copy image to volume: [Errno 28] No space left on device
  
  ```

## Analysis:

- We noticed that /dev/mapper/os-root of cinder-volume node was eating up the entire space.
- Hence, it will be advisable to set a location which has enough space e.g. 40 GB to 50 GB 
  (assuming all the images will be within this capacity)
- We can set the permissions of the location to "cinder" user & set it at cinder.conf's "image_conversion_dir" key.


