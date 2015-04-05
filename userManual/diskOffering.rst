.. _disk offering:

=============
Disk Offering
=============

.. contents:: `Table of contents`
   :depth: 6

--------
Overview
--------

A disk offering is a specification of a volume, which defines a volume's size and how it will be created. Disk offerings can
be used to create both root volumes and data volumes.

.. note:: There is no API to create a root volume; but if you provision a VM with an ISO image, you need to specify a disk
          offering that defines size and allocator strategy for the VM's root volume, which is the only way that creates a root volume
          from a disk offering.

.. _disk offering inventory:

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
   * - **diskSize**
     - the size of volume in bytes, see :ref:`disk size <disk offering size>`
     -
     -
     - 0.6
   * - **state**
     - see :ref:`state <disk offering state>`
     -
     - - Enabled
       - Disabled
     - 0.6
   * - **type**
     - reserved field
     -
     - - zstack
     - 0.6
   * - **allocatorStrategy**
     - see :ref:`allocator strategy <disk offering allocator strategy>`
     -
     - - DefaultPrimaryStorageAllocationStrategy
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

.. _disk offering size:

Disk Size
+++++++++

DiskSize defines a volume's virtual size. As mentioned in :ref:`volume <volume>`, virtual size is the max size a volume can occupy in
storage system after it is fully filled. Putting in a straight way, it's the size you want for the volume.

.. _disk offering state:

State
+++++

Disk offerings have two states:

- **Enabled**:

  The state that allows volumes to be created from this disk offering

- **Disabled**:

  The state that DOESN'T allow volumes to be created from this disk offering

.. _disk offering allocator strategy:

Allocator Strategy
++++++++++++++++++

Allocator strategy defines how ZStack selects a primary storage when creating a new volume. Currently the only supported strategy is
DefaultPrimaryStorageAllocationStrategy that finds a primary storage satisfying conditions::

    1. state is Enabled
    2. status is Connected
    3. availableCapacity is greater than disk offering's diskSize
    4. has been attached to the cluster that runs the VM to which the volume will be attached

.. note:: A volume created from a disk offering is only instantiated on primary storage when it's being attached to a VM. See :ref:`volume status NotInstantiated <volume status>`.

----------
Operations
----------

Create Disk Offering
====================

Users can use CreateDiskOffering create a disk offering. For example::

    CreateDiskOffering name=small diskSize=1073741824

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
   * - **diskSize**
     - disk size in bytes, see :ref:`size <disk offering size>`
     -
     -
     - 0.6
   * - **allocationStrategy**
     - see :ref:`allocator strategy <disk offering allocator strategy>`
     - true
     - - DefaultPrimaryStorageAllocationStrategy
     - 0.6
   * - **type**
     - reserved filed, leave it alone
     - true
     -
     - 0.6

Change State
============

Users can use ChangeDiskOfferingState to change the state of a disk offering. For example::

    ChangeDiskOfferingState uuid=178c662bfcdd4145920682c58ebcbed4 stateEvent=enable

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
     - disk offering uuid
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

Delete Disk Offering
====================

Users can use DeleteDiskOffering to delete a disk offering. For example::

    DeleteDiskOffering uuid=178c662bfcdd4145920682c58ebcbed4

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
     - disk offering uuid
     -
     -
     - 0.6

Query Disk Offering
===================

Users can use QueryDiskOffering to query disk offerings. For example::

    QueryDiskOffering diskSize>=10000000

::

    QueryDiskOffering volume.name=data1


Primitive Fields of Query
+++++++++++++++++++++++++

see :ref:`disk offering inventory <disk offering inventory>`

Nested And Expanded Fields of Query
+++++++++++++++++++++++++++++++++++

.. list-table::
   :widths: 20 30 40 10
   :header-rows: 1

   * - Field
     - Inventory
     - Description
     - Since
   * - **volume**
     - :ref:`volume inventory <volume inventory>`
     - volumes that are created from this disk offering
     - 0.6

----
Tags
----

Users can create user tags on a disk offering with resourceType=DiskOfferingVO. For example::

    CreateUserTag tag=smallDisk resourceType=DiskOfferingVO resourceUuid=d6c49e73927d40abbfcf13852dc18367

System Tags
===========

Dedicated Primary Storage
+++++++++++++++++++++++++

When creating volumes from disk offerings, users can use a system tag to specify primary storage
on which the volumes will be created.

.. list-table::
   :widths: 20 30 40 10
   :header-rows: 1

   * - Tag
     - Description
     - Example
     - Since
   * - **primaryStorage::allocator::uuid::{uuid}**
     - | if present, volumes created from this disk offering will be
       | allocated on the primary storage of *uuid*;
       | an allocation failure will be raised if the specified primary storage
       | doesn't exist or doesn't have enough capacity.
     - primaryStorage::allocator::uuid::b8398e8b7ff24527a3b81dc4bc64d974
     - 0.6
   * - **primaryStorage::allocator::userTag::{tag}::required**
     - | if present, volumes created from this disk offering will be
       | allocated on the primary storage having user tag *tag*;
       | an allocation failure will be raised if no primary storage
       | has the *tag* or primary storage having the *tag* doesn't
       | have enough capacity.
     - primaryStorage::allocator::userTag::SSD::required
     - 0.6
   * - **primaryStorage::allocator::userTag::{tag}**
     - | if present, volumes created from this disk offering will
       | be primarily allocated on the primary storage having user tag *tag*,
       | if there is any; no failure will be raised if no primary storage
       | has the *tag* or primary storage having the *tag* doesn't
       | have enough capacity, instead, a random primary storage will be chosen
       | for the volume.
     - primaryStorage::allocator::userTag::SSD
     - 0.6

if more than one above system tags present on a disk offering, the precedent order is::

    primaryStorage::allocator::uuid::{uuid} > primaryStorage::allocator::userTag::{tag}::required > primaryStorage::allocator::userTag::{tag}
