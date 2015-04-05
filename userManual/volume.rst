.. _volume:

======
Volume
======

.. contents:: `Table of contents`
   :depth: 6

--------
Overview
--------

Volumes provide storage to guest VMs. A volume can be of type of root or data, depending on the role it plays. A
root volume is a disk where the VM operating system is installed, for example, C: or sda; a data volume which provides
additional storage is like an extra hard drive, for example: D: or sdb.

Volumes are hypervisor specific; that is to say, a volume created for one hypervisor may not be able to get attached to
VMs of other hypervisor types; for example, a volume for KVM VMs cannot be attached to VMWare VMs. The hypervisor attribute
of volumes is implied by the field :ref:`format <volume format>`, which is similar to :ref:`image format <image format>` except
the image format has an extra value 'ISO' that volumes don't have.

Because of `thin provisioning <http://en.wikipedia.org/wiki/Thin_provisioning>`_, a volume can has two sizes: real size and virtual
size. The real size is the size that a volume actually occupies in storage system; the virtual size is the size a volume claims for, which
is the max size a volume can have when it is fully filled. The virtual size is always greater than or equal to the real size.

Volumes on :ref:`primary storage <primary storage>` can be directly accessed by VMs. A volume can only be attached to one VM
at any given time. A root volume is always attached to its owner VM and cannot be detached; a data volume, on the contrary,
can be attached/detached to/from different VMs of the same hypervisor type, as long as the VMs can access the primary storage
on which the data volume locates.

.. _volume inventory:

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
   * - **primaryStorageUuid**
     - the uuid of primary storage the volume locates on, see :ref:`primary storage <primary storage>`
     -
     -
     - 0.6
   * - **vmInstanceUuid**
     - uuid of the VM the volume is attached, or NULL if not attached; see :ref:`attach VM <volume attach VM>`
     - true
     -
     - 0.6
   * - **diskOfferingUuid**
     - the uuid of :ref:`disk offering <disk offering>`, if the volume is created from a disk offering
     - true
     -
     - 0.6
   * - **rootImageUuid**
     - the uuid of :ref:`image <image>`, if the volume is created from an image
     - true
     -
     - 0.6
   * - **installPath**
     - the path where the volume is installed on the primary storage
     -
     -
     - 0.6
   * - **type**
     - volume type
     -
     - - Root
       - Data
     - 0.6
   * - **format**
     - see :ref:`format <volume format>`
     -
     - - qcow2
     - 0.6
   * - **size**
     - the volume's virtual size, in bytes
     -
     -
     - 0.6
   * - **deviceId**
     - see :ref:`device id <volume device id>`
     - true
     -
     - 0.6
   * - **state**
     - see :ref:`state <volume state>`
     -
     - - Enabled
       - Disabled
     - 0.6
   * - **status**
     - see :ref:`status <volume status>`
     -
     - - Creating
       - Ready
       - NotInstantiated
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
            "description": "Root volume for VM[uuid:1a2b197060eb4593bf5bbf2a83b3d625]",
            "deviceId": 0,
            "format": "qcow2",
            "installPath": "/opt/zstack/nfsprimarystorage/prim-302055ec45794423af7f5d3c5081bc87/rootVolumes/acct-36c27e8ff05c4780bf6d2fa65700f22e/vol-f7bbb3ae1c674ecda3b0f4c025e333f9/f7bbb3ae1c674ecda3b0f4c025e333f9.qcow2",
            "createDate": "Jun 1, 2015 3:45:44 PM",
            "lastOpDate": "Jun 1, 2015 3:45:44 PM",
            "name": "ROOT-for-virtualRouter.l3.1b7f47f5350c488c99e8f54142ddffbd",
            "primaryStorageUuid": "302055ec45794423af7f5d3c5081bc87",
            "rootImageUuid": "178c662bfcdd4145920682c58ebcbed4",
            "size": 1364197376,
            "state": "Enabled",
            "status": "Ready",
            "type": "Root",
            "uuid": "f7bbb3ae1c674ecda3b0f4c025e333f9",
            "vmInstanceUuid": "1a2b197060eb4593bf5bbf2a83b3d625"
        }

.. _volume attach VM:

Attached VM
+++++++++++

A data volume can be attached to a Running or Stopped VM, but can only be attached to one VM at any given time; after being attached, the VM's
UUID is shown up in the field 'vmInstanceUuid'. A data volume can also be detached from one VM and be re-attached to another VM, as long as
VMs are of the same hypervisor type. A root volume is always attached to its owner VM and can never be detached.

.. _volume format:

Format
++++++

Format reveals relationship between a volume and a hypervisor type, indicating what VMs of which hypervisor type a volume can be attached.
Volume format is similar to :ref:`image format <image format>`. In this ZStack version, as KVM is the only supported hypervisor type, the only
volume format is 'qcow2'.

.. _volume device id:

Device ID
+++++++++

Device ID shows the order that volumes are attached to a VM. Because the root volume is always the first volume attached, it has a fixed device ID
0; data volumes may have device IDs 1, 2, 3 ... N, depending on the sequence they are attached to the VM. The device ID can be used to identify the disk
letter of the volume in guest operating system; for example, in Linux, 0 usually means /dev/xvda, 1 usually means /dev/xvdb and so fourth.

.. _volume state:

State
+++++

Volumes have two states:

- **Enabled**:

  The state that allows volumes to be attached to VMs.

- **Disabled**:

  The state that DOESN't allow volumes to be attached to VMs; however, an attached data volume can always be detached even if in state of Disabled.

.. note:: Root volumes always have the state of Enabled as they cannot be detached.

.. _volume status:

Status
++++++

Status shows lifecycle of volumes:

- **NotInstantiated**:

  A specific status for only data volumes. Data volumes of this status are only allocated in database and have not been instantiated
  on any primary storage yet; that is to say, they are just database records. Data volumes in status of NotInstantiated can be attached
  to VMs of any hypervisor types; and will be instantiated to concrete binaries on primary storage, with hypervisor types of VMs they are
  being attached. After being attached, data volumes' hypervisorType fields will be evaluated to hypervisor types of VMs, status will
  be changed to Ready; and since then they can only be re-attached to VMs of the same hypervisor types.

- **Ready**:

  Volumes are already instantiated on primary storage and are ready for operations.

- **Creating**:

  Volumes are in process of being created from images or volume snapshots; not ready for operations.


The status transition diagram is like:

.. image:: volume-status.png
   :align: center

.. note:: Root volume is always in status of Ready.


----------
Operations
----------

Create a Data Volume
====================

.. note:: Root volumes are created automatically when creating VMs; there is no API to create root volumes.

From a Disk Offering
++++++++++++++++++++

Users can use CreateDataVolume to create a data volume from a :ref:`disk offering <disk offering>`. For example::

    CreateDataVolume name=data1 diskOfferingUuid=fea135f1d1de40b4915a19aa155983b3

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
   * - **diskOfferingUuid**
     - disk offering uuid, see :ref:`disk offering <disk offering>`
     -
     -
     - 0.6

From an Image
+++++++++++++

Users can use CreateDataVolumeFromVolumeTemplate to create a data volume from an image. For example::

    CreateDataVolumeFromVolumeTemplate name=data1 imageUuid=ee6fa27ade8c42a2bdda8f9b1eee8c93 primaryStorageUuid=302055ec45794423af7f5d3c5081bc87

The image can be of media type of RootVolumeTemplate or DataVolumeTemplate.

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
   * - **imageUuid**
     - image uuid, see :ref:`image <image>`
     -
     -
     - 0.6
   * - **primaryStorageUuid**
     - | uuid of primary storage where the data volume is going to be created; the primary storage must be accessible to VMs
       | that the data volume is planned to be attached; otherwise you may create a dangling data volume that cannot be attached
       | to VMs you want.
       | see :ref:`primary storage <primary storage>`.
     -
     -
     - 0.6

.. _create data volume from volume snapshot:

From a Volume Snapshot
++++++++++++++++++++++

Users can use CreateDataVolumeFromVolumeSnapshot to create a data volume from a :ref:`volume snapshot <volume snapshot>`. For example::

    CreateDataVolumeFromVolumeSnapshot name=data1 primaryStorageUuid=302055ec45794423af7f5d3c5081bc87 volumeSnapshotUuid=178c662bfcdd4145920682c58ebcbed4

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
   * - **volumeSnapshotUuid**
     - volume snapshot uuid, see :ref:`volume snapshot <volume snapshot>`
     -
     -
     - 0.6
   * - **primaryStorageUuid**
     - | uuid of primary storage where the data volume is going to be created; the primary storage must be accessible to VMs
       | that the data volume is planned to be attached; otherwise you may create a dangling data volume that cannot be attached
       | to VMs you want.
       | see :ref:`primary storage <primary storage>`.
     -
     -
     - 0.6

Delete Data Volume
==================

Users can use DeleteDataVolume to delete a data volume. For example::

    DeleteDataVolume uuid=178c662bfcdd4145920682c58ebcbed4

.. note:: Root volumes, which are deleted when deleting VMs, cannot be deleted by APIs.

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
     - volume uuid
     -
     -
     - 0.6

.. danger:: There is no way to recover a deleted data volume.

Change State
============

Users can use ChangeVolumeState to change the state of a data volume. For example::

    ChangeVolumeState uuid=be19ce415bbe44539b0bd276633470e0 stateEvent=enable

.. note:: States of root volumes are unchangeable.

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
     - volume uuid
     -
     -
     - 0.6
   * - **stateEvent**
     - state trigger event

       - enable: change the state to Enabled
       - disable: change the state ot Disabled
     -
     - - enable
       - disable
     - 0.6

.. _AttachDataVolumeToVm:

Attach VM
=========

Users can use AttachDataVolumeToVm to attach a data volume to a VM. For example::

    AttachDataVolumeToVm volumeUuid=178c662bfcdd4145920682c58ebcbed4 vmInstanceUuid=c5b443a20341418b9120c7e3b3cd34f5

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
   * - **volumeUuid**
     - volume uuid
     -
     -
     - 0.6
   * - **vmInstanceUuid**
     - VM uuid, see :ref:`VM <vm>`
     -
     -
     - 0.6

.. _DetachDataVolumeFromVm:

Detach VM
=========

Users can use DetachDataVolumeFromVm to detach a data volume from a VM. For example::

    DetachDataVolumeFromVm uuid=178c662bfcdd4145920682c58ebcbed4

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
     - volume uuid
     -
     -
     - 0.6

.. warning:: Please flush all changes in VM operating system to disk before detaching a data volume and make sure
             no applications are accessing it; otherwise data in the data volume may crash. Imagine the process of detaching
             a data volume as hot unplugging a hard drive from a computer.

Query Volume
============

Users can use QueryVolume to query volumes. For example::

      QueryVolume type=Data vmInstanceUuid=71f5376ef53a46a9abddd59c942cf45f

::

      QueryVolume diskOffering.name=small primaryStorage.uuid=8db7eb2ccdab4c4eb4784e46895bb016


Primitive Fields of Query
+++++++++++++++++++++++++

see :ref:`volume inventory <volume inventory>`

Nested And Expanded Fields of Query
+++++++++++++++++++++++++++++++++++

.. list-table::
   :widths: 20 30 40 10
   :header-rows: 1

   * - Field
     - Inventory
     - Description
     - Since
   * - **vmInstance**
     - :ref:`VM inventory <vm inventory>`
     - the VM the volume is attached to
     - 0.6
   * - **snapshot**
     - :ref:`volume snapshot inventory <volume snapshot inventory>`
     - volume snapshots that are created from this volume
     - 0.6
   * - **diskOffering**
     - :ref:`disk offering inventory <disk offering inventory>`
     - disk offering that the volume is created from
     - 0.6
   * - **primaryStorage**
     - :ref:`primary storage inventory <primary storage inventory>`
     - primary storage that the volume is on
     - 0.6
   * - **image**
     - :ref:`image inventory <image inventory>`
     - image that the volume is create from
     - 0.6

----
Tags
----

Users can create user tags on a volume with resourceType=VolumeVO. For example::

    CreateUserTag resourceType=VolumeVO tag=goldenVolume resourceUuid=f97b8cb9bccc4872a723c8b7785d9a12

