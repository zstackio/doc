.. _image:

=====
Image
=====

.. contents:: `Table of contents`
   :depth: 6

--------
Overview
--------

Images provide templates for virtual machine file systems. Images can be RootVolumeTemplate that provide templates for VMs' root volumes
where VMs' operating systems install; or DataVolumeTemplate that provide templates for VMs' data volumes that usually contain non
operating system data; or ISO that can be used to install operating systems to blank root volumes.

Images are stored on :ref:`backup storage <backup storage>`. Prior to starting a VM, if the image to create VM root volume is not
in :ref:`primary storage <primary storage>`'s image cache, it will be downloaded to the cache first. So when creating a VM with an image
at the first time, it takes longer than normal because the downloading process.

ZStack uses `thin provisioning <http://en.wikipedia.org/wiki/Thin_provisioning>`_ to create root volumes. Root volumes from the same image share
the same base in primary storage's image cache, and any changes made to the root volumes do not affect the base image.

.. _image inventory:

---------
Inventory
---------

Properties
==========

.. list-table::
   :widths: 20 40 10 20 10
   :header-rows: 1

   * - Name
     - Description
     - Optional
     - Choices
     - Since
   * - **uuid**
     - see :ref:`resource properties`
     -
     -
     - 0.6
   * - **name**
     - see :ref:`resource properties`
     -
     -
     - 0.6
   * - **description**
     - see :ref:`resource properties`
     - true
     -
     - 0.6
   * - **state**
     - see :ref:`state <image state>`
     -
     - - Enabled
       - Disabled
     - 0.6
   * - **status**
     - see :ref:`status <image status>`
     -
     - - Creating
       - Downloading
       - Ready
     - 0.6
   * - **size**
     - image size, in bytes
     -
     -
     - 0.6
   * - **url**
     - url the image registered from, see :ref:`url <image url>`
     -
     -
     - 0.6
   * - **mediaType**
     - image's media type, see :ref:`media type <image media type>`
     -
     - - RootVolumeTemplate
       - DataVolumeTemplate
       - ISO
     - 0.6
   * - **guestOsType**
     - a string for user records VM's operating system type
     - true
     -
     - 0.6
   * - **platform**
     - indicates platform of VM's operating system, see :ref:`platform <image platform>`
     -
     - - Linux
       - Windows
       - Paravirtualization
       - Other
     - 0.6
   * - **system**
     - see :ref:`system image <system image>`
     -
     -
     - 0.6
   * - **format**
     - see :ref:`format <image format>`
     -
     - - qcow2
       - raw
     - 0.6
   * - **md5Sum**
     - image's md5sum

       .. note:: MD5 sum is not calculated at this ZStack version
     -
     -
     - 0.6
   * - **type**
     -  reserved field
     -
     - - zstack
     - 0.6
   * - **backupStorageRefs**
     - a list of :ref:`backup storage reference <image backup storage reference>`
     -
     -
     - 0.6

Example
=======

::

        {
            "backupStorageRefs": [
                {
                    "backupStorageUuid": "8b99641a4d644820932e0ec5ada78eed",
                    "createDate": "Jun 1, 2015 6:17:48 PM",
                    "imageUuid": "b395386bdb4a4ff1b1850a457c949c5e",
                    "installPath": "/export/backupStorage/sftp/templates/acct-36c27e8ff05c4780bf6d2fa65700f22e/b395386bdb4a4ff1b1850a457c949c5e/centos_400m_140925.template",
                    "lastOpDate": "Jun 1, 2015 6:17:48 PM"
                }
            ],
            "createDate": "Jun 1, 2015 6:17:40 PM",
            "description": "Test Image Template for network test",
            "format": "qcow2",
            "guestOsType": "unknown",
            "lastOpDate": "Jun 1, 2015 6:17:40 PM",
            "md5Sum": "not calculated",
            "mediaType": "RootVolumeTemplate",
            "name": "image_for_sg_test",
            "platform": "Linux",
            "size": 419430400,
            "state": "Enabled",
            "status": "Ready",
            "system": false,
            "type": "zstack",
            "url": "http://172.16.0.220/templates/centos_400m_140925.img",
            "uuid": "b395386bdb4a4ff1b1850a457c949c5e"
        },

.. _image state:

State
=====

Images have two states:

- **Enabled**:

  The state that allows VMs to be created from this image

- **Disabled**:

  The state that DOESN'T allow VMs to be created from this image

.. _image status:

Status
======

Status indicates images' lifecycle:

- **Creating**:

  The image is in process of creating from a volume or a volume snapshot; not ready to use.

- **Downloading**:

  The image is in process of downloading from a url; not ready to use.

- **Ready**:

  The image is on backup storage and ready to use.

.. _image url:

URL
===

Depending on how an image was created on a backup storage, the url has different meanings; when an image was downloaded from a web server,
the url is the HTTP/HTTPS link; when an image was created from a volume or a volume snapshot, the url is a string encoding UUID of the volume or the volume snapshot, like::

    volume://b395386bdb4a4ff1b1850a457c949c5e
    volumeSnapshot://b395386bdb4a4ff1b1850a457c949c5e

.. note:: In this ZStack version, the only way to register an image to backup storage is providing a URL that is a HTTP/HTTPS
          link and calling AddImage.


.. _image media type:

Media Type
==========

A media type indicates the image's usage.

- **RootVolumeTemplate**:

  The image is used to create root volumes.

- **DataVolumeTemplate**:

  The image is used to create data volumes.

- **ISO**:

  The image is used to install operating systems to blank root volumes.

.. _image platform:

Platform
========

Platform gives ZStack a hint that whether to use `paravirtualization <http://en.wikipedia.org/wiki/Paravirtualization>`_
for VMs created from this image.

.. list-table::
   :widths: 50 50

   * - Use paravirtualization
     - - Linux
       - Paravirtualization
   * - Not to use paravirtualization
     - - Windows
       - Other

.. _system image:

System Image
============

System images are images used only for appliance VMs but not for user VMs. This is normally used for :ref:`virtual router <virtual router>` image in
this ZStack version.


.. _image format:

Format
======

Format exhibits relationships between hypervisors and images. For example, images of format qcow2 can only be used for VMs of KVM.
In this ZStack version,  as KVM is the only supported hypervisor, the relationship table is like:


.. list-table::
   :widths: 50 50
   :header-rows: 1

   * - Hypervisor Type
     - Format
   * - KVM
     - - qcow2
       - raw

Volumes will inherit formats of images from which they are created; for example, root volumes created from images of format qcow2 will have format qcow2 too.
Format 'raw' is an exception, volumes created from 'raw' images will have the format qcow2 because ZStack will thin-clone it using qcow2 format.

.. _image backup storage reference:

Backup Storage Reference
========================

An image can be stored on more than one backup storage. For every backup storage, the image has a backup storage reference
encompassing backup storage UUID and image's installation path.


.. list-table::
   :widths: 20 40 10 20 10
   :header-rows: 1

   * - Name
     - Description
     - Optional
     - Choices
     - Since
   * - **imageUuid**
     - image uuid
     -
     -
     - 0.6
   * - **backupStorageUuid**
     - backup storage uuid, see :ref:`backup storage <backup storage>`
     -
     -
     - 0.6
   * - **installPath**
     - installation path on backup storage
     -
     -
     - 0.6
   * - **createDate**
     - see :ref:`resource properties`
     -
     -
     - 0.6
   * - **lastOpDate**
     - see :ref:`resource properties`
     -
     -
     - 0.6

Example
+++++++

::

     {
         "backupStorageUuid": "8b99641a4d644820932e0ec5ada78eed",
         "imageUuid": "b395386bdb4a4ff1b1850a457c949c5e",
         "installPath": "/export/backupStorage/sftp/templates/acct-36c27e8ff05c4780bf6d2fa65700f22e/b395386bdb4a4ff1b1850a457c949c5e/centos_400m_140925.template",
         "createDate": "Jun 1, 2015 6:17:48 PM",
         "lastOpDate": "Jun 1, 2015 6:17:48 PM"
     }


----------
Operations
----------

.. _add image:

Add Image
=========

Admins can use AddImage to add an image. For example::

    AddImage name=CentOS7 format=qcow2 backupStorageUuids=8b99641a4d644820932e0ec5ada78eed url=http://172.16.0.220/templates/centos7_400m_140925.img mediaType=RootVolumeTemplate platform=Linux

Parameters
++++++++++

.. list-table::
   :widths: 20 40 10 20 10
   :header-rows: 1

   * - Name
     - Description
     - Optional
     - Choices
     - Since
   * - **name**
     - resource name, see :ref:`resource properties`
     -
     -
     - 0.6
   * - **resourceUuid**
     - resource uuid, see :ref:`create resource`
     - true
     -
     - 0.6
   * - **description**
     - resource description, see :ref:`resource properties`
     - true
     -
     - 0.6
   * - **url**
     - HTTP/HTTPS url, see :ref:`url <image url>`
     -
     -
     - 0.6
   * - **mediaType**
     - image media type, see :ref:`media type <image media type>`. Default is RootVolumeTemplate
     - true
     - - RootVolumeTemplate
       - DataVolumeTemplate
       - ISO
     - 0.6
   * - **guestOsType**
     - a string that indicates VM's operating system type, for example, CentOS7
     - true
     -
     - 0.6
   * - **system**
     - indicates whether this is a system image, see :ref:`system image <system image>`. Default is false
     - true
     - - true
       - false
     - 0.6
   * - **format**
     - image format, see :ref:`format <image format>`
     -
     - - qcow2
       - raw
     - 0.6
   * - **platform**
     - image platform, see :ref:`platform <image platform>`. Default is Linux
     - true
     - - Linux
       - Windows
       - Other
       - Paravirtualization
     - 0.6
   * - **backupStorageUuids**
     - a list of backup storage uuid to which the image is going to add
     -
     -
     - 0.6
   * - **type**
     - reserved field, leave it alone
     - true
     - - zstack
     - 0.6

An image can be added to multiple backup storage by providing a list of backup storage UUID in 'backupStorageUuids';
The AddImage command succeeds as long as the image is successfully added to one backup storage, and fails if the image
fails on all backup storage. Backup storage that successfully added the image can be retrieved from
:ref:`image backup storage reference <image backup storage reference>` of the image inventory in the API response.

Delete Image
============

Admins can use DeleteImage to delete an image from specified backup storage or all backup storage. For example::

    DeleteImage uuid=b395386bdb4a4ff1b1850a457c949c5e backupStorageUuids=c310386bdb4a4ff1b1850a457c949c5e,f295386bdb4a4ff1b1850a457c949c5e

Parameters
++++++++++

.. list-table::
   :widths: 20 40 10 20 10
   :header-rows: 1

   * - Name
     - Description
     - Optional
     - Choices
     - Since
   * - **uuid**
     - image uuid
     -
     -
     - 0.6
   * - **deleteMode**
     - see :ref:`delete resource`
     - true
     - - Permissive
       - Enforcing
     - 0.6
   * - **backupStorageUuids**
     - a list of backup storage storing the image; if omitted, the image will be deleted from all backup storage it's on.
     -
     -
     - 0.6

An image is considered as deleted only if it is deleted from all backup storage; otherwise, its copy get deleted on
some specific backup storage.

.. danger:: There is no way to recover an image if it has been deleted from all backup storage.

Change State
============

Admins can use ChangeImageState to change the state of an image. For example::

    ChangeImageState stateEvent=enable uuid=b395386bdb4a4ff1b1850a457c949c5e

Parameters
++++++++++

.. list-table::
   :widths: 20 40 10 20 10
   :header-rows: 1

   * - Name
     - Description
     - Optional
     - Choices
     - Since
   * - **uuid**
     - image uuid
     -
     -
     - 0.6
   * - **stateEvent**
     - state trigger event

       - enable: change state to Enabled
       - disable: change state to Disabled
     -
     - - enable
       - disable
     - 0.6

Create RootVolumeTemplate From Root Volume
==========================================

Users can create an RootVolumeTemplate image from a root volume. For example::

    CreateRootVolumeTemplateFromRootVolume name=CentOS7 rootVolumeUuid=1ab2386bdb4a4ff1b1850a457c949c5e backupStorageUuids=backupStorageUuids,f295386bdb4a4ff1b1850a457c949c5e

Parameters
++++++++++

.. list-table::
   :widths: 20 40 10 20 10
   :header-rows: 1

   * - Name
     - Description
     - Optional
     - Choices
     - Since
   * - **name**
     - resource name, see :ref:`resource properties`
     -
     -
     - 0.6
   * - **resourceUuid**
     - resource uuid, see :ref:`create resource`
     - true
     -
     - 0.6
   * - **description**
     - resource description, see :ref:`resource properties`
     - true
     -
     - 0.6
   * - **backupStorageUuids**
     - a list of backup storage uuid on which the image is going to created, see :ref:`backup storage uuids <backup storage uuids1>`
     - true
     -
     - 0.6
   * - **rootVolumeUuid**
     - uuid of root volume from which the image is going to create
     -
     -
     - 0.6
   * - **platform**
     - image platform, see :ref:`platform <image platform>`; default to Linux
     - true
     - - Linux
       - Windows
       - Other
       - Paravirtualization
     - 0.6
   * - **guestOsType**
     - a string that indicates VM's operating system type, for example, CentOS7
     - true
     -
     - 0.6
   * - **system**
     - indicates whether this is system image, see :ref:`system image <system image>`; default to false
     - true
     - - true
       - false
     - 0.6

.. _backup storage uuids1:

Backup Storage UUIDs
++++++++++++++++++++

When calling CreateRootVolumeTemplateFromRootVolume, users can provide a list of backup storage UUIDs to specify where
the image is going to create; if this field is omitted, a random backup storage will be chosen.

.. _create RootVolumeTemplate from volume snapshot:

Create RootVolumeTemplate From Volume Snapshot
==============================================

Users can use CreateRootVolumeTemplateFromVolumeSnapshot to create a RootVolumeTemplate from a volume snapshot. For example::

    CreateRootVolumeTemplateFromVolumeSnapshot name=CentOS7 snapshotUuid=1ab2386bdb4a4ff1b1850a457c949c5e

Parameters
++++++++++

.. list-table::
   :widths: 20 40 10 20 10
   :header-rows: 1

   * - Name
     - Description
     - Optional
     - Choices
     - Since
   * - **name**
     - resource name, see :ref:`resource properties`
     -
     -
     - 0.6
   * - **resourceUuid**
     - resource uuid, see :ref:`create resource`
     - true
     -
     - 0.6
   * - **description**
     - resource description, see :ref:`resource properties`
     - true
     -
     - 0.6
   * - **snapshotUuid**
     - uuid of volume snapshot, see :ref:`volume snapshot <volume snapshot>`
     -
     -
     - 0.6
   * - **backupStorageUuids**
     - a list of backup storage uuid on which the image is going to created, see :ref:`backup storage uuids <backup storage uuids2>`
     - true
     -
     - 0.6
   * - **platform**
     - image platform, see :ref:`platform <image platform>`. Default to Linux
     - true
     - - Linux
       - Windows
       - Other
       - Paravirtualization
     - 0.6
   * - **guestOsType**
     - a string that indicates VM's operating system type, for example, CentOS7
     - true
     -
     - 0.6
   * - **system**
     - indicates whether this is system image, see :ref:`system image <system image>`. Default is false
     - true
     - - true
       - false
     - 0.6

.. _backup storage uuids2:

Backup Storage Uuids
++++++++++++++++++++

When calling CreateRootVolumeTemplateFromVolumeSnapshot, users can provide a list of backup storage UUIDs to specify where
the image is going to create; if this field is omitted, a random backup storage will be chosen.

Create DataVolumeTemplate From Volume
=====================================

Users can use CreateDataVolumeTemplateFromVolume to create a DataVolumeTemplate from a volume. For example::

    CreateDataVolumeTemplateFromVolume name=data volumeUuid=1ab2386bdb4a4ff1b1850a457c949c5e

The volume can be either root volume or data volume. This provides a way to create a data volume from a root volume.
Users can firstly create a DataVolumeTemplate from a root volume, then create a data volume from the DataVolumeTemplate.

Parameters
++++++++++

.. list-table::
   :widths: 20 40 10 20 10
   :header-rows: 1

   * - Name
     - Description
     - Optional
     - Choices
     - Since
   * - **name**
     - resource name, see :ref:`resource properties`
     -
     -
     - 0.6
   * - **resourceUuid**
     - resource uuid, see :ref:`create resource`
     - true
     -
     - 0.6
   * - **description**
     - resource description, see :ref:`resource properties`
     - true
     -
     - 0.6
   * - **volumeUuid**
     - uuid of volume, see :ref:`volume <volume>`
     -
     -
     - 0.6
   * - **backupStorageUuids**
     - a list of backup storage uuid on which the image is going to created, see :ref:`backup storage uuids <backup storage uuids3>`
     - true
     -
     - 0.6

.. _backup storage uuids3:

Backup Storage Uuids
++++++++++++++++++++

When calling CreateDataVolumeTemplateFromVolume, users can provide a list of backup storage UUIDs to specify where
the image is going to create; if this field is omitted, a random backup storage will be chosen.

Query Image
===========

Users can use QueryImage to query images. For example::

    QueryImage status=Ready system=true

::

    QueryImage volume.vmInstanceUuid=85ab231e392d4dfb86510191278e9fc3


Primitive Fields of Query
+++++++++++++++++++++++++

see :ref:`image inventory <image inventory>`

Nested And Expanded Fields of Query
+++++++++++++++++++++++++++++++++++

.. list-table::
   :widths: 20 30 40 10
   :header-rows: 1

   * - Field
     - Inventory
     - Description
     - Since
   * - **backupStorage**
     - :ref:`backup storage inventory <backup storage inventory>`
     - backup storage this image is on
     - 0.6
   * - **volume**
     - :ref:`volume inventory <volume inventory>`
     - volumes created from this image
     - 0.6
   * - **backupStorageRef**
     - :ref:`backup storage reference <image backup storage reference>`
     - reference used to query by backup storage install path
     - 0.6

----
Tags
----

Users can create user tags on an image with resourceType=ImageVO. For example::

    CreateUserTag resourceType=ImageVO tag=golden-image resourceUuid=ff7c04c4e2874a21a3e795501f1bc516

