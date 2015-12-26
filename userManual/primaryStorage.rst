.. _primary storage:

===============
Primary Storage
===============

.. contents:: `Table of contents`
   :depth: 6

--------
Overview
--------

A primary storage is the storage system in datacenter that stores disk volumes for VMs. Primary storage can be local disks(e.g. hard
drives of hosts) or network shared (e.g. NAS, SAN ) storage.

.. image:: primary-storage.png
   :align: center

A primary storage stores volumes for VMs running in clusters that have been attached to this primary storage.

A primary storage can be attached to only sibling clusters.

.. note:: In this ZStack version, NFS is the only supported primary storage

.. _primary storage inventory:

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
   * - **zoneUuid**
     - uuid of parent zone, see :ref:`zone <zone>`
     -
     -
     - 0.6
   * - **totalCapacity**
     - total disk capacity, in bytes, see :ref:`capacity <primary storage capacity>`
     -
     -
     - 0.6
   * - **availableCapacity**
     - available disk capacity, in bytes, see :ref:`capacity <primary storage capacity>`
     -
     -
     - 0.6
   * - **url**
     - see :ref:`url <primary storage url>`
     -
     -
     - 0.6
   * - **type**
     - primary storage type
     -
     - - NFS
     - 0.6
   * - **state**
     - see :ref:`state <primary storage state>`
     -
     - - Enabled
       - Disabled
     - 0.6
   * - **status**
     - see :ref:`status <primary storage status>`
     -
     - - Connecting
       - Connected
       - Disconnected
     - 0.6
   * - **attachedClusterUuids**
     - a list of cluster uuid to which the primary storage has been attached, see :ref:`attach cluster <primary storage attached cluster>`
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
=======

::

    {
      "inventory": {
        "uuid": "f4ac0a3119c94c6fae844c2298615d27",
        "zoneUuid": "f04caf351c014aa890126fc78193d063",
        "name": "nfs",
        "url": "192.168.0.220:/storage/nfs",
        "description": "Test Primary Storage",
        "totalCapacity": 10995116277768819,
        "availableCapacity": 10995162768,
        "type": "NFS",
        "state": "Enabled",
        "mountPath": "/opt/zstack/f4ac0a3119c94c6fae844c2298615d27",
        "createDate": "Jun 1, 2015 2:42:51 PM",
        "lastOpDate": "Jun 1, 2015 2:42:51 PM",
        "attachedClusterUuids": [
          "f23e402bc53b4b5abae87273b6004016",
          "4a1789235a86409a9a6db83f97bc582f",
          "fe755538d4e845d5b82073e4f80cb90b",
          "1f45d6d6c02b43bfb6196dcacb5b8a25"
        ]
      }
    }

.. _primary storage capacity:

Capacity
++++++++

ZStack keeps tracking disk capacities of primary storage in order to select suitable one to create volumes. The capacities reported by
different primary storage plugins may be different; for example, for those supporting over-provisioning, the capacity reported may be larger
than real; for those not supporting over-provisioning, the capacity reported may be equal to or smaller than real.

NFS Capacity
------------

NFS doesn't support over-provisioning, so the capacity is counted by volumes' virtual sizes using below formulas::

    totalCapacity = NFS's total capacity
    availableCapacity = totalCapacity - sum(volumes' virtual sizes)

Volumes' virtual sizes will be discussed in chapter :ref:`volume <volume>`; for those impatient, a volume's virtual size is the size when a volume is
fully filled; for example, when you created a volume with 1G capacity, before it's fully filled, its real size may be 10M because of
thin-provisioning technology.

.. _primary storage url:

URL
+++

A URL is a string that contains information needed by primary storage plugins for manipulating storage systems. Although it's named as URL,
the certain format of the string is up to primary storage types and is not necessary to strictly follow the URL convention, to give
flexibilities to plugins to encode information that may not be able to fit in the URL format.

NFS URL
-------

For NFS primary storage, the URL is encoded as::

    ip-or-dns-name-of-nfs-server:/absolute-path-to-directory

For example::

    192.168.0.220:/storage/nfs/


.. _primary storage state:

State
=====

Primary storage has two states:

- **Enabled**:

  the state that allows volumes to be created

- **Disabled**:

  the state that DOESN'T allow volumes to be created

.. _primary storage status:

Status
======

Like :ref:`host status <host status>`, primary storage status reflect the status of command channels amid ZStack management nodes
and primary storage. Command channels are the ways ZStack management nodes communicate with storage systems that primary storage represent;
depending on primary storage types, for example, it can be HTTP connections among ZStack management nodes and primary storage agents or communication
methods provided by storage SDKs.

There are three status:

- **Connecting**:

  A ZStack management node is trying to establish the command channel between itself and the primary storage. No operations can be performed to the primary storage.

- **Connected**

  The command channel has been successfully established between a ZStack management node and the primary storage. Operations can be performed to the primary storage.

- **Disconnected**

  The command channel has lost between a ZStack management node and the primary storage. No operations can be performed to the primary storage.

ZStack management nodes will try to establish command channels when booting and will periodically send
ping commands to primary storage to check health of command channels during running; once a primary storage fails to respond,
or a ping command times out, the command channel is considered as lost and the primary storage will be placed in Disconnected.

.. note:: ZStack will keep sending ping commands when a primary storage is in status of Disconnected. Once the primary storage recovers and responds to ping commands, ZStack
          will reestablish the command channel and place the primary storage in status of Connected. So when a primary storage is physically removed from the cloud, please delete
          it from ZStack, otherwise ZStack will keep pinging it.

Here is the transition diagram:

.. image:: primary-storage-status.png
   :align: center

State and Status
================

There is no direct relations between states and status. States represent admin's decisions to primary storage,
while status represent communication conditions of primary storage.

.. _primary storage attached cluster:

Attaching Cluster
=================

Attaching clusters is to associate primary storage to sibling clusters, which provides a flexible way that manifests relations between hosts and storage systems in a real datacenter.
Let's see a concreted example; assuming you have a cluster (cluster A) attached to a NFS primary storage (NFS1), like below diagram:

.. image:: primary-storage-cluster1.png
   :align: center

Some time later, the cluster A is running out of memory but the primary storage still have plenty of disk spaces,
so you decide to add another cluster (cluster B) which will also use NFS1; then you can create cluster B and attach NFS1 to it.

.. image:: primary-storage-cluster2.png
   :align: center

After running a while, the hardware of cluster A is getting outdated and you decide to retire them; you add a new powerful cluster (cluster C) attached to NFS1
and place all hosts in cluster A into maintenance mode, so all VMs running in cluster A are migrated to cluster B or cluster C; lastly, you detach NFS1 from
cluster A and delete it. Now the datacenter looks like:

.. image:: primary-storage-cluster3.png
   :align: center

Finally, NFS1 starts running out of capacity, you add one more primary storage (NFS2), and attach it to both cluster B and cluster C.

.. image:: primary-storage-cluster4.png
   :align: center

----------
Operations
----------

Add Primary Storage
===================

The commands adding a primary storage varies for different types of primary storage.

Add NFS Primary Storage
+++++++++++++++++++++++

Admins can use AddNfsPrimaryStorage to add a NFS primary storage. For example::

    AddNfsPrimaryStorage name=nfs1 zoneUuid=1b830f5bd1cb469b821b4b77babfdd6f url=192.168.0.220:/storage/nfs

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
   * - **zoneUuid**
     - uuid of parent zone, see :ref:`zone <zone>`
     -
     -
     - 0.6
   * - **url**
     - see :ref:`url <primary storage url>`
     -
     -
     - 0.6

Delete Primary Storage
======================

Admins can use DeletePrimaryStorage to delete a primary storage. For example::

    DeletePrimaryStorage uuid=2c830f5bd1cb469b821b4b77babfdd6f

.. danger:: Deleting a primary storage will delete all volumes and volume snapshots it contains. VMs will be deleted as results of
            deleting root volumes. There is no way to recover a deleted primary storage. Clusters attached will be detached.

Properties
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
     - primary storage uuid
     -
     -
     - 0.6

Change Primary Storage State
============================

Admins can use ChangePrimaryStorageState to change the state of a primary storage. For example::

    ChangePrimaryStorageState stateEvent=enable uuid=2c830f5bd1cb469b821b4b77babfdd6f

Properties
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
     - primary storage uuid
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

Attach Cluster
==============

See :ref:`attach primary storage to cluster`.


Detach Cluster
==============

See :ref:`detach primary storage from cluster`.

Query Primary Storage
=====================

Admins can use QueryPrimaryStorage to query primary storage. For example::

    QueryPrimaryStorage totalCapacity<100000000000

::

    QueryPrimaryStorage volumeSnapshot.uuid?=13238c8e0591444e9160df4d3636be82,33107835aee84c449ac04c9622892dec

Primitive Fields of Query
+++++++++++++++++++++++++

see :ref:`primary storage inventory <primary storage inventory>`

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
     - parent zone
     - 0.6
   * - **volume**
     - :ref:`volume inventory <volume inventory>`
     - volumes on this primary storage
     - 0.6
   * - **volumeSnapshot**
     - :ref:`volume snapshot inventory <volume snapshot inventory>`
     - volume snapshots on this primary storage
     - 0.6
   * - **cluster**
     - :ref:`cluster inventory <cluster inventory>`
     - clusters the primary storage is attached to
     - 0.6

---------------------
Global Configurations
---------------------

.. _mount.base:

mount.base
==========

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **mount.base**
     - nfsPrimaryStorage
     - /opt/zstack/nfsprimarystorage
     - absolute path that starts with '/'

The mount point that NFS primary storage is mounted on the KVM hosts.

.. note:: Changing this value only affect new NFS primary storage

----
Tags
----

Users can create user tags on a primary storage with resourceType=PrimaryStorageVO. For example::

    CreateUserTag resourceType=PrimaryStorage tag=SSD resourceUuid=e084dc809fec4092ab0eff797d9529d5

System Tags
===========

Storage Volume Snapshot
+++++++++++++++++++++++

.. list-table::
   :widths: 20 30 40 10
   :header-rows: 1

   * - Tag
     - Description
     - Example
     - Since
   * - **capability:snapshot**
     - if present, the primary storage supports storage volume snapshot
     - capability:snapshot
     - 0.6


