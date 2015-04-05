.. _volume snapshot:

===============
Volume Snapshot
===============

.. contents:: `Table of contents`
   :depth: 6

--------
Overview
--------

A volume snapshot is a point-in-time capture of a VM's volume; memory and CPU state are not captured. Snapshots can be taken
on root volumes and data volumes, and are arranged in a chain manner that the initial snapshot is usually a full snapshot
containing all contents of a volume, and subsequent snapshots are delta snapshots only containing changes since
the last snapshot. A volume can be restored to its old contents by reverting to a snapshot; images and volumes can
be created from snapshots.

As volume snapshots only capture volumes' states, users need to flush changes in memory to file
system in VMs' operating system before taking snapshots.

.. _snapshot type:

Snapshot Type
=============

There are two ways to create volume snapshots; one is hypervisor based that snapshots are created by hypervisors from VMs' volumes;
another is storage based that snapshots are created by storage systems that store VMs' volumes. In this ZStack version, only hypervisor
based snapshot is supported.

Snapshot Tree
=============

Volume snapshots are normally arranged as a chain like:

.. image:: volumeSnapshot1.png
   :align: center

however, once a volume is reverted to a snapshot and takes a snapshot again, the snapshot chain will grow as a tree where every chain is
a branch:

.. image:: volumeSnapshot2.png
   :align: center

Current volume is always following the last snapshot; when a snapshot chain has too many delta snapshots,
it may hurt the volume's disk IO performance, so ZStack sets max length of a snapshot chain to 16 by default;
a new snapshot chain will be created after taking 16 snapshots.

.. image:: volumeSnapshot3.png
   :align: center

The max length of a snapshot chain can be configured by :ref:`incrementalSnapshot.maxNum`.

.. _delete snapshot:

Delete Snapshot
===============

When deleting a snapshot, if it's not a leaf that is the last one in a snapshot chain, all it's descendants
will be deleted as well. For example:

.. image:: volumeSnapshot4.png
   :align: center


after deleting Snapshot1, the Snapshot2, Snapshot3, Snapshot1.1, and Snapshot1.2 will deleted too, and the snapshot chain turns to be:

.. image:: volumeSnapshot5.png
   :align: center


.. note:: When deleting a volume, all snapshots taken from this volume will be deleted from primary storage.

.. _volume snapshot tree:

------------------------------
Volume Snapshot Tree Inventory
------------------------------

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
   * - **volumeUuid**
     - the uuid of volume the snapshot tree is created
     -
     -
     - 0.6
   * - **current**
     - see :ref:`current <volume snapshot current>`
     -
     - - true
       - false
     - 0.6
   * - **tree**
     - a tree of :ref:`SnapshotLeafInventory <SnapshotLeafInventory>`
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
            "createDate": "Dec 7, 2015 11:45:02 PM",
            "current": true,
            "lastOpDate": "Dec 7, 2015 11:45:02 PM",
            "tree": {
                "children": [
                    {
                        "children": [
                            {
                                "children": [],
                                "inventory": {
                                    "backupStorageRefs": [],
                                    "createDate": "Dec 7, 2015 11:45:16 PM",
                                    "format": "qcow2",
                                    "lastOpDate": "Dec 7, 2015 11:45:16 PM",
                                    "latest": true,
                                    "name": "sp3",
                                    "parentUuid": "3a859e89a39645018772e4d92ca02a09",
                                    "primaryStorageInstallPath": "/opt/zstack/nfsprimarystorage/prim-a82b75ee064a48708960f42b800bd910/rootVolumes/acct-36c27e8ff05c4780bf6d2fa65700f22e/vol-2ad40ef516c540eeb138b7da24105f2e/snapshots/3a859e89a39645018772e4d92ca02a09.qcow2",
                                    "primaryStorageUuid": "a82b75ee064a48708960f42b800bd910",
                                    "size": 197120,
                                    "state": "Enabled",
                                    "status": "Ready",
                                    "treeUuid": "acca6784c70b47fda68de18e2f8380d1",
                                    "type": "Hypervisor",
                                    "uuid": "b4d673e29f724320bb283c6dc4a59225",
                                    "volumeType": "Root",
                                    "volumeUuid": "2ad40ef516c540eeb138b7da24105f2e"
                                },
                                "parentUuid": "3a859e89a39645018772e4d92ca02a09"
                            }
                        ],
                        "inventory": {
                            "backupStorageRefs": [],
                            "createDate": "Dec 7, 2015 11:45:10 PM",
                            "format": "qcow2",
                            "lastOpDate": "Dec 7, 2015 11:45:10 PM",
                            "latest": false,
                            "name": "sp2",
                            "parentUuid": "b885d1e6549c49caab97322243827ca1",
                            "primaryStorageInstallPath": "/opt/zstack/nfsprimarystorage/prim-a82b75ee064a48708960f42b800bd910/rootVolumes/acct-36c27e8ff05c4780bf6d2fa65700f22e/vol-2ad40ef516c540eeb138b7da24105f2e/snapshots/b885d1e6549c49caab97322243827ca1.qcow2",
                            "primaryStorageUuid": "a82b75ee064a48708960f42b800bd910",
                            "size": 197120,
                            "state": "Enabled",
                            "status": "Ready",
                            "treeUuid": "acca6784c70b47fda68de18e2f8380d1",
                            "type": "Hypervisor",
                            "uuid": "3a859e89a39645018772e4d92ca02a09",
                            "volumeType": "Root",
                            "volumeUuid": "2ad40ef516c540eeb138b7da24105f2e"
                        },
                        "parentUuid": "b885d1e6549c49caab97322243827ca1"
                    }
                ],
                "inventory": {
                    "backupStorageRefs": [],
                    "createDate": "Dec 7, 2015 11:45:02 PM",
                    "format": "qcow2",
                    "lastOpDate": "Dec 7, 2015 11:45:02 PM",
                    "latest": false,
                    "name": "sp1",
                    "primaryStorageInstallPath": "/opt/zstack/nfsprimarystorage/prim-a82b75ee064a48708960f42b800bd910/rootVolumes/acct-36c27e8ff05c4780bf6d2fa65700f22e/vol-2ad40ef516c540eeb138b7da24105f2e/2ad40ef516c540eeb138b7da24105f2e.qcow2",
                    "primaryStorageUuid": "a82b75ee064a48708960f42b800bd910",
                    "size": 4718592,
                    "state": "Enabled",
                    "status": "Ready",
                    "treeUuid": "acca6784c70b47fda68de18e2f8380d1",
                    "type": "Hypervisor",
                    "uuid": "b885d1e6549c49caab97322243827ca1",
                    "volumeType": "Root",
                    "volumeUuid": "2ad40ef516c540eeb138b7da24105f2e"
                }
            },
            "uuid": "acca6784c70b47fda68de18e2f8380d1",
            "volumeUuid": "2ad40ef516c540eeb138b7da24105f2e"
        }

.. _volume snapshot current:

Current
+++++++

A current tree is a snapshot tree to which the volume currently links.

.. _SnapshotLeafInventory:

SnapshotLeafInventory
+++++++++++++++++++++

SnapshotLeafInventory is the leaf structure of snapshot tree; a snapshot tree always starts with a root SnapshotLeafInventory.

Properties
----------

.. list-table::
   :widths: 20 40 10 20 10
   :header-rows: 1

   * - Name
     - Description
     - Optional
     - Choices
     - Since
   * - **inventory**
     - the volume snapshot inventory, see :ref:`volume snapshot inventory <volume snapshot inventory>`
     -
     -
     - 0.6
   * - **parentUuid**
     - uuid of :ref:`volume snapshot inventory <volume snapshot inventory>` of parent leaf; if null, this leaf is the root leaf
     - true
     -
     - 0.6
   * - **children**
     - a list of :ref:`SnapshotLeafInventory <SnapshotLeafInventory>` which are child leafs
     -
     -
     - 0.6

.. _volume snapshot inventory:

-------------------------
Volume Snapshot Inventory
-------------------------

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
   * - **type**
     - see :ref:`type <snapshot type>`
     -
     - - Hypervisor
       - Storage
     - 0.6
   * - **volumeUuid**
     - uuid of volume the snapshot is created
     -
     -
     - 0.6
   * - **treeUuid**
     - the uuid of :ref:`tree <volume snapshot tree>` this snapshot belongs
     -
     -
     - 0.6
   * - **parentUuid**
     - uuid of parent snapshot in chain
     -
     -
     - 0.6
   * - **primaryStorageUuid**
     - uuid of primary storage this snapshot locates
     - true
     -
     - 0.6
   * - **primaryStorageInstallPath**
     - the path of this snapshot on primary storage
     - true
     -
     - 0.6
   * - **volumeType**
     - the type of volume this snapshot is created
     -
     - - Root
       - Data
     - 0.6
   * - **size**
     - the snapshot size in bytes
     -
     -
     - 0.6
   * - **state**
     - snapshot state, see :ref:`state <volume snapshot state>`
     -
     - - Enabled
       - Disabled
     - 0.6
   * - **status**
     - snapshot status, see :ref:`status <volume snapshot status>`
     -
     - - Creating
       - Ready
       - Deleting
     - 0.6
   * - **backupStorageRefs**
     - a list of :ref:`VolumeSnapshotBackupStorageRefInventory <VolumeSnapshotBackupStorageRefInventory>`
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
            "backupStorageRefs": [],
            "createDate": "Dec 7, 2015 11:45:02 PM",
            "format": "qcow2",
            "lastOpDate": "Dec 7, 2015 11:45:02 PM",
            "latest": false,
            "name": "sp1",
            "primaryStorageInstallPath": "/opt/zstack/nfsprimarystorage/prim-a82b75ee064a48708960f42b800bd910/rootVolumes/acct-36c27e8ff05c4780bf6d2fa65700f22e/vol-2ad40ef516c540eeb138b7da24105f2e/2ad40ef516c540eeb138b7da24105f2e.qcow2",
            "primaryStorageUuid": "a82b75ee064a48708960f42b800bd910",
            "size": 4718592,
            "state": "Enabled",
            "status": "Ready",
            "treeUuid": "acca6784c70b47fda68de18e2f8380d1",
            "type": "Hypervisor",
            "uuid": "b885d1e6549c49caab97322243827ca1",
            "volumeType": "Root",
            "volumeUuid": "2ad40ef516c540eeb138b7da24105f2e"
        }

.. _volume snapshot state:

State
+++++

Volume snapshots have two states:

- **Enabled**

  The state allows operations to be proceeded

- **Disabled**

  The state that forbids operations; snapshots in this state cannot be used to revert volumes and create templates/volumes; and cannot be backup.

.. _volume snapshot status:

Status
++++++

Volume snapshots have following status:

- **Creating**

  The snapshot is being created from a volume

- **Ready**

  The snapshot is ready for any operations

- **Deleting**

  The snapshot is being deleted


.. _VolumeSnapshotBackupStorageRefInventory:

VolumeSnapshotBackupStorageRefInventory
+++++++++++++++++++++++++++++++++++++++

VolumeSnapshotBackupStorageRefInventory encompasses information about a copy of a snapshot on a backup storage.

Properties
----------

.. list-table::
   :widths: 20 40 10 20 10
   :header-rows: 1

   * - Name
     - Description
     - Optional
     - Choices
     - Since
   * - **volumeSnapshotUuid**
     - snapshot uuid
     -
     -
     - 0.6
   * - **backupStorageUuid**
     - backup storage uuid
     -
     -
     - 0.6
   * - **installPath**
     - the install path of snapshot copy on backup storage
     -
     -
     - 0.6

----------
Operations
----------

Create Snapshot
===============

Users can use CreateVolumeSnapshot to create a volume snapshot. For example::

    CreateVolumeSnapshot name=sp1 volumeUuid=2ad40ef516c540eeb138b7da24105f2e

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
     - volume uuid the snapshot is going to create
     -
     -
     - 0.6

Delete Snapshot
===============

Users can use DeleteVolumeSnapshot to delete a snapshot. For example::

    DeleteVolumeSnapshot uuid=b885d1e6549c49caab97322243827ca1

.. warning:: All descendant snapshots will deleted as well. see :ref:`delete snapshot <delete snapshot>`

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
   * - **deleteMode**
     - see :ref:`delete resource`
     - true
     - - Permissive
       - Enforcing
     - 0.6
   * - **uuid**
     - snapshot uuid
     -
     -
     - 0.6

Revert Volume From Snapshot
===========================

Users can use RevertVolumeFromSnapshot to revert a volume to a snapshot; after reverting, the volume will have contents when the
snapshot was created. For example::

    RevertVolumeFromSnapshot uuid=b885d1e6549c49caab97322243827ca1

the volume is the one where the snapshot is created.

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
     - snapshot uuid
     -
     -
     - 0.6

Backup Snapshot
===============

Users can use BackupVolumeSnapshot to backup a snapshot to a backup storage. For example::

    BackupVolumeSnapshot uuid=b885d1e6549c49caab97322243827ca1 backupStorageUuid=a82b75ee064a48708960f42b800bd910

ancestor snapshots not backup on any backup storage will be backup as well.

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
     - snapshot uuid
     -
     -
     - 0.6
   * - **backupStorageUuid**
     - backup storage uuid; if omitted, ZStack will find a proper one.
     - true
     -
     - 0.6

Delete Snapshot Backup
======================

Users can use DeleteVolumeSnapshotFromBackupStorage to delete a copy of snapshot from backup storage. For example::

    DeleteVolumeSnapshotFromBackupStorage uuid=b885d1e6549c49caab97322243827ca1 backupStorageUuid=a82b75ee064a48708960f42b800bd910,b885d1e6549c49caab97322243827ca1

if the copy is the only copy of this snapshot on backup storage, all copies of descendant snapshots of this snapshot will be deleted as well;

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
   * - **deleteMode**
     - see :ref:`delete resource`
     - true
     - - Permissive
       - Enforcing
     - 0.6
   * - **uuid**
     - snapshot uuid
     -
     -
     - 0.6
   * - **backupStorageUuids**
     - a list of uuid of backup storage from which to delete the snapshot's copy
     -
     -
     - 0.6

Create RootVolumeTemplate From Snapshot
=======================================

see :ref:`Create RootVolumeTemplate From Volume Snapshot <create RootVolumeTemplate from volume snapshot>`.


Create Data Volume From Snapshot
================================

see :ref:`create data volume from volume snapshot <create RootVolumeTemplate from volume snapshot>`.

Query Volume Snapshot
=====================

Users can use QueryVolumeSnapshot to query volume snapshots. For example::

    QueryVolumeSnapshot primaryStorageUuid=6572ce44c3f6422d8063b0fb262cbc62

::

    QueryVolumeSnapshot volume.vmInstance.uuid=bd1652b1e44144e6b9b5b286b82edb69


Primitive Fields
++++++++++++++++

see :ref:`volume snapshot inventory <volume snapshot inventory>`

Nested And Expanded Fields
++++++++++++++++++++++++++

.. list-table::
   :widths: 20 30 40 10
   :header-rows: 1

   * - Field
     - Inventory
     - Description
     - Since
   * - **volume**
     - :ref:`volume inventory <volume inventory>`
     - the volume the volume snapshot is created
     - 0.6
   * - **tree**
     - :ref:`volume snapshot tree inventory <volume snapshot tree>`
     - the parent volume snapshot tree
     - 0.6
   * - **primaryStorage**
     - :ref:`primary storage inventory <primary storage inventory>`
     - primary storage the volume snapshot locates
     - 0.6
   * - **backupStorageRef**
     - :ref:`VolumeSnapshotBackupStorageRefInventory <VolumeSnapshotBackupStorageRefInventory>`
     - the backup storage reference
     - 0.6
   * - **backupStorage**
     - :ref:`backup storage inventory <backup storage inventory>`
     - backup storage that the volume snapshot locates
     - 0.6


---------------------
Global Configurations
---------------------

.. _incrementalSnapshot.maxNum:

incrementalSnapshot.maxNum
==========================

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **incrementalSnapshot.maxNum**
     - volumeSnapshot
     - 16
     - > 0

The max length of a snapshot chain.

.. _volumeSnapshot.delete.parallelismDegree:

delete.parallelismDegree
========================

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **delete.parallelismDegree**
     - volumeSnapshot
     - 1
     - > 0

The number of snapshots that can be deleted in parallel when deleting a snapshot or a snpashot tree.

.. _volumeSnapshot.backup.parallelismDegree:

backup.parallelismDegree
========================

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **backup.parallelismDegree**
     - volumeSnapshot
     - 5
     - > 0

The number of snapshots that can be backup in parallel when backup snapshots.

----
Tags
----

Users can create user tags on a volume snapshot with resourceType=VolumeSnapshotVO. For example::

    CreateUserTag resourceType=VolumeSnapshotVO tag=firstSnapshot resourceUuid=fae9a6f43c8e4017b0e2a251d67d650d

and create user tags on a volume snapshot tree with resourceType=VolumeSnapshotTreeVO. For example::

    CreateUserTag resourceType=VolumeSnapshotVO tag=devops-tree resourceUuid=d6c49e73927d40abbfcf13852dc18367
