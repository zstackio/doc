.. _backup storage:

==============
Backup Storage
==============

.. contents:: `Table of contents`
   :depth: 6

--------
Overview
--------

A backup Storage is a storage system that stores :ref:`images <image>` for creating volumes. Backup storage can be
filesystem based storage(e.g. NFS) or object store based storage(e.g. OpenStack swift), as long as the storage is network
shared storage. Besides providing templates for creating volumes, backup storage also allow users to backup entities including
volumes and volume snapshots.

A backup storage must be attached to a :ref:`zone <zone>` before the zone's descendant resources can access it.
Admins can take this advantage to share images across multiple zones, for example:

.. image:: backupStorage1.png
   :align: center

In the early stage of a cloud, there may be only one zone(Zone1) with a single backup storage. In pace with business development,
admins may decide to create another zone(Zone2) but still use existing images for VMs; then admins can attach the backup
storage to Zone2, so both Zone1 and Zone2 share the same images.

.. image:: backupStorage2.png
   :align: center

.. note:: In this ZStack version, the only supported backup storage is :ref:`SFTP backup storage <sftp backup storage>`

.. _backup storage inventory:

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
   * - **url**
     - see :ref:`url <backup storage url>`
     -
     -
     - 0.6
   * - **totalCapacity**
     - total disk capacity in bytes, see :ref:`capacity <backup storage capacity>`
     -
     -
     - 0.6
   * - **availableCapacity**
     - available disk capacity in bytes, see :ref:`capacity <backup storage capacity>`
     -
     -
     - 0.6
   * - **type**
     - backup storage type
     -
     - - SftpBackupStorage
     - 0.6
   * - **state**
     - see :ref:`state <backup storage state>`
     -
     - - Enabled
       - Disabled
     - 0.6
   * - **status**
     - see :ref:`status <backup storage status>`
     -
     - - Connecting
       - Connected
       - Disconnected
     - 0.6
   * - **attachedZoneUuids**
     - a list of zone UUID the backup storage has been attached
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
            "attachedZoneUuids": [
                "36de66d82f424639af67215a465418f6"
            ],
            "availableCapacity": 1258407346176,
            "name": "sftp",
            "state": "Enabled",
            "status": "Connected",
            "totalCapacity": 1585341214720,
            "type": "SftpBackupStorage",
            "url": "/export/backupStorage/sftp",
            "uuid": "33a35f75885f45ab96ea2626ce9c05a6",
            "lastOpDate": "Jun 1, 2015 3:42:26 PM",
            "createDate": "Jun 1, 2015 3:42:26 PM"
        }

.. _backup storage url:

URL
+++

URL is a string that contains information needed by backup storage plugins for manipulating storage systems. Although it's named as
URL, the certain format of the string is up to backup storage types and is not necessary to strictly follow the URL convention, to give
flexibilities to plugins to encode information that may not be able to fit in the URL format.

.. _sftp backup storage url:

SFTP Backup Storage URL
-----------------------

For SFTP backup storage, the URL is the absolute path of a directory in the filesystem. For example, /storage/sftp.

.. _backup storage capacity:

Capacity
++++++++

ZStack keeps tracking disk capacities of backup storage in order to select suitable one when allocating space for storing images.
The capacity is calculated by below formulas::

    totalCapacity = backup storage's total capacity
    availableCapacity = totalCapacity - sum(images' real sizes)

.. _backup storage state:

State
+++++

Backup storage have two states:

- **Enabled**:

  The state that allows images to be registered, backup, and downloaded

- **Disabled**:

  The state that DOESN'T allow images to be registered, backup, and downloaded. Especially, if an image is only stored on
  a disabled backup storage, and if that image is not downloaded to image caches of primary storage yet, no VMs can be
  created from that image.

.. _backup storage status:

Status
++++++

Status reflects the status of command channels amid ZStack management nodes and backup storage.

- **Connecting**:

  A ZStack management node is trying to establish the command channel between itself and a backup storage. No operations can be performed to the backup storage.

- **Connected**

  The command channel has been successfully established between a ZStack management node and a backup storage. Operations can be performed to the backup storage.

- **Disconnected**

  The command channel has lost between a ZStack management node and a backup storage. No operations can be performed to the backup storage.

ZStack management nodes will try to setup command channels every time when they boot, and will periodically send
ping commands to backup storage to check the health of command channels. Once a backup storage fails to respond,
or a ping command times out, the command channel is considered as lost and the backup storage will be placed in
the status of Disconnected.

.. warning:::: ZStack will keep sending ping commands when a backup storage is in the status of Disconnected. Once the backup storage recovers and responds to ping commands, ZStack
               will reestablish the command channel and place the backup storage in the status of Connected. So when a backup storage is physically removed from a cloud, please delete
               it from ZStack, otherwise ZStack will keep pinging it.

Here is the transition diagram:

.. image:: backup-storage-status.png
   :align: center

.. _sftp backup storage:

-------------------
SFTP Backup Storage
-------------------

SFTP backup storage is a Linux server that stores images in native filesystem and uses OpenSSH server/client to transfer images.
ZStack uses a python agent (SftpBackupStorageAgent) to manage the Linux server; images are uploaded/downloaded to/from the server
by `SCP <http://en.wikipedia.org/wiki/Secure_copy>`_. Besides properties in :ref:`backup storage inventory <backup storage inventory>`,
SFTP backup storage has an extra property:

.. list-table::
   :widths: 20 40 10 20 10
   :header-rows: 1

   * - Name
     - Description
     - Optional
     - Choices
     - Since
   * - **hostname**
     - the IP address or DNS name of the SFTP backup storage
     -
     -
     - 0.6

Example
=======

::

        {
            "attachedZoneUuids": [
                "36de66d82f424639af67215a465418f6"
            ],
            "availableCapacity": 1258407346176,
            "hostname": "172.16.0.220",
            "name": "sftp",
            "state": "Enabled",
            "status": "Connected",
            "totalCapacity": 1585341214720,
            "type": "SftpBackupStorage",
            "url": "/export/backupStorage/sftp",
            "uuid": "33a35f75885f45ab96ea2626ce9c05a6",
            "lastOpDate": "Jun 1, 2015 3:42:26 PM",
            "createDate": "Jun 1, 2015 3:42:26 PM"
        }

----------
Operations
----------

Add Backup Storage
==================

The commands to add a backup storage vary for different backup storage types.

Add SFTP Backup Storage
+++++++++++++++++++++++

Admins can use AddSftpBackupStorage to add a new backup storage. For example::

    AddSftpBackupStorage name=sftp1 url=/storage/sftp1 hostname=192.168.0.220 username=root password=password

Parameters
----------

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
     - see :ref:`url <backup storage url>`
     -
     -
     - 0.6
   * - **hostname**
     - the IP address or DNS name of the SFTP backup storage
     -
     -
     - 0.6
   * - **username**
     - the user **root**
     -
     - root
     - 0.6
   * - **password**
     - the SSH password for user **root**
     -
     -
     - 0.6

Delete Backup Storage
=====================

Admins can use DeleteBackupStorage to delete a backup storage. For example::

    DeleteBackupStorage uuid=1613b627cb2e4ffcb30e7e59935064be

.. warning:: When deleting, a backup storage will be detached from attached zones. Copies of images and of volume snapshots
             on the backup storage will be deleted; if a copy is the only copy of an image or a volume snapshot, the image
             or the volume snapshot will be deleted as well. There is no way to recover a deleted backup storage.

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
     - backup storage uuid
     -
     -
     - 0.6
   * - **deleteMode**
     - see :ref:`delete resource`
     - true
     - - Permissive
       - Enforcing
     - 0.6


Change State
============

Admins can use ChangeBackupStorageState to change the state of a backup storage. For example::

    ChangeBackupStorageState uuid=33a35f75885f45ab96ea2626ce9c05a6 stateEvent=enable

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
     - backup storage uuid
     -
     -
     - 0.6
   * - **stateEvent**
     - state trigger event

       - enable: change the state to Enabled
       - disable: change the state to Disabled
     -
     - - enable
       - disable
     - 0.6

.. _attach backup storage to zone:

Attach Zone
===========

Admins can use AttachBackupStorageToZone to attach a backup storage to a zone. For example::

    AttachBackupStorageToZone backupStorageUuid=d086c30f33914c98a6078269bab7bc8f zoneUuid=d086c30f33914c98a6078269bab7bc8f

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
   * - **backupStorageUuid**
     - the backup storage uuid
     -
     -
     - 0.6
   * - **zoneUuid**
     - the zone uuid
     -
     -
     - 0.6

.. _detach backup storage from zone:

Detach Zone
===========

Admins can use DetachBackupStorageFromZone to detach a backup storage from a zone. For example::

    DetachBackupStorageFromZone backupStorageUuid=d086c30f33914c98a6078269bab7bc8f zoneUuid=d086c30f33914c98a6078269bab7bc8f

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
   * - **backupStorageUuid**
     - the backup storage uuid
     -
     -
     - 0.6
   * - **zoneUuid**
     - the zone uuid
     -
     -
     - 0.6

Query Backup Storage
====================

Admins can use QueryBackupStorage to query backup storage. For example::

    QueryBackupStorage state=Enabled

::

    QueryBackupStorage image.platform=Linux


Primitive Fields of Query
+++++++++++++++++++++++++

see :ref:`backup storage inventory <backup storage inventory>`


.. _backup storage nested fields:

Nested And Expanded Fields of Query
+++++++++++++++++++++++++++++++++++

.. list-table::
   :widths: 20 30 40 10
   :header-rows: 1

   * - Field
     - Inventory
     - Description
     - Since
   * - **zone**
     - :ref:`zone inventory <zone inventory>`
     - zones this backup storage is attached to
     - 0.6
   * - **image**
     - :ref:`image inventory <image inventory>`
     - images this backup storage contains
     - 0.6
   * - **volumeSnapshot**
     - :ref:`volume snapshot inventory <volume snapshot inventory>`
     - volume snapshots this backup storage contains
     - 0.6

Query SFTP Backup Storage
=========================

Admins can use QuerySftpBackupStorage to query SFTP backup storage::

    QuerySftpBackupStorage name=sftp

Primitive Fields of Query
+++++++++++++++++++++++++

see :ref:`SFTP backup storage inventory <sftp backup storage>`

Nested and Expanded Fields of Query
+++++++++++++++++++++++++++++++++++

see :ref:`backup storage nested and expanded fields <backup storage nested fields>`

---------------------
Global Configurations
---------------------

.. _ping.interval:

ping.interval
=============

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **ping.interval**
     - backupStorage
     - 60
     - > 0

The interval that management nodes send ping commands to backup storage, in seconds.

.. _ping.parallelismDegree:

ping.parallelismDegree
======================

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **ping.parallelismDegree**
     - backupStorage
     - 50
     - > 0

The max number of backup storage that management nodes will ping in parallel.

----
Tags
----

Admins can create user tags on a backup storage with resourceType=BackupStorageVO. For example::

    CreateUserTag tag=lab1 resourceType=BackupStorageVO resourceUuid=2906471068802c501773d3ee55b7766e

