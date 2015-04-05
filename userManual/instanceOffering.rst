.. _instance offering:

=================
Instance Offering
=================

.. contents:: `Table of contents`
   :depth: 6

--------
Overview
--------

An instance offering is a specification of VM's memory, CPU, and host allocation algorithm; it defines the volume of computing
resource a VM can have.

.. _instance offering inventory:

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
   * - **cpuNum**
     - VCPU number, see :ref:`CPU capacity <instance offering cpu capacity>`
     -
     -
     - 0.6
   * - **cpuSpeed**
     - VCPU speed, see :ref:`CPU capacity <instance offering cpu capacity>`
     -
     -
     - 0.6
   * - **memorySize**
     - memory size, in bytes
     -
     -
     - 0.6
   * - **type**
     - instance offering type, default is UserVm, see :ref:`type <instance offering type>`
     - true
     - - UserVm
       - VirtualRouter
     - 0.6
   * - **allocatorStrategy**
     - host allocator strategy, see :ref:`allocator strategy <instance offering allocator strategy>`
     -
     - - DefaultHostAllocatorStrategy
       - DesignatedHostAllocatorStrategy
     - 0.6
   * - **state**
     - see :ref:`state <instance offering state>`
     -
     - - Enabled
       - Disabled
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

.. _instance offering cpu capacity:

CPU Capacity
++++++++++++

Instance offerings use cpuNum and cpuSpeed to define a VM's CPU capacity. cpuNum, very straightforward, means the number
of VCPU that a VM has; cpuSpeed is a little special; as a VM's VCPU always has the frequency same to the host's
physical CPU, cpuSpeed here actually means VCPU weight in hypervisors. Depending on hypervisor types, the use and implementation of
cpuSpeed vary.

KVM CPU Speed
-------------

In KVM, ZStack will set the result of 'cpuSpeed * cpuNum' to VM's XML configuration to libvirt::

  <cputune>
    <shares>128</shares>
  </cputune>

  shares = cpuNum * cpuSpeed

.. _instance offering type:

Type
++++

The type of instance offering; currently there are two types:

- **UserVm**: instance offering for creating user VMs.

- **VirtualRouter**: instance offering for creating virtual router VMs; see :ref:`virtual router <virtual router>`.

.. _instance offering allocator strategy:

Allocator Strategy
++++++++++++++++++

Allocator strategy defines the algorithm of selecting destination hosts for creating VMs.

DefaultHostAllocatorStrategy
----------------------------

DefaultHostAllocatorStrategy uses below algorithm:

Input Parameters
****************
.. list-table::
   :widths: 30 70
   :header-rows: 1

   * - Name
     - Description
   * - **image**
     - image used to create the VM
   * - **L3 network**
     - L3 networks the VM will have nics on
   * - **instance offering**
     - instance offering
   * - **tags**
     - tags for host allocation

Algorithm
*********

::

    l2_networks = get_parent_l2_networks(l3_networks)
    host_set1 = find_hosts_in_cluster_that_have_attached_to_l2_networks()
    check_if_backup_storage_having_image_have_attached_to_zone_of_hosts(host_set1)
    host_set2 = remove_hosts_not_having_state_Enabled_and_status_Connected(host_set1)
    host_set3 = remove_hosts_not_having_capacity_required_by_instance_offering(host_set2)
    primary_storage = find_Enabled_Connected_primary_storage_having_enough_capacity_for_root_volume_and_attached_to_clusters_of_hosts(image, host_set3)
    host_set4 = remove_hosts_that_cannot_access_primary_storage(host_set3)
    host_set5 = remove_avoided_hosts(host_set4)
    host_set6 = call_tag_plugin(tags, host_set5)

    return randomly_pick_one_host(host_set6)


.. _DesignatedHostAllocatorStrategy:

DesignatedHostAllocatorStrategy
-------------------------------

DesignatedHostAllocatorStrategy uses algorithm:

Input Parameters
****************
.. list-table::
   :widths: 30 60 10
   :header-rows: 1

   * - Name
     - Description
     - Optional
   * - **image**
     - image used to create the VM
     -
   * - **L3 network**
     - L3 networks the VM will have nics on
     -
   * - **instance offering**
     - instance offering
     -
   * - **tags**
     - tags for host allocation
     -
   * - **zone**
     - the zone the VM wants to run
     - true
   * - **cluster**
     - the cluster the VM wants to run
     - true
   * - **host**
     - the host the VM wants to run
     - true

Algorithm
*********

::

    l2_networks = get_parent_l2_networks(l3_networks)
    host_set1 = find_hosts_in_cluster_that_have_attached_to_l2_networks()
    check_if_backup_storage_having_image_have_attached_to_zone_of_hosts(host_set1)

    if host is not null:
       host_set2 = list(find_host_in_host_set1(host))
    else if cluster is not null:
       host_set2 = find_host_in_cluster_and_host_set1(cluster)
    else if zone is not null:
       host_set2 = find_host_in_zone_and_host_set1(zone)

    host_set3 = remove_hosts_not_having_state_Enabled_and_status_Connected(host_set2)
    host_set4 = remove_hosts_not_having_capacity_required_by_instance_offering(host_set3)
    primary_storage = find_Enabled_Connected_primary_storage_having_enough_capacity_for_root_volume_and_attached_to_clusters_of_hosts(image, host_set4)
    host_set5 = remove_hosts_that_cannot_access_primary_storage(host_set4)
    host_set6 = remove_avoided_hosts(host_set5)
    host_set7 = call_tag_plugin(tags, host_set6)

    return randomly_pick_one_host(host_set7)


.. note:: DesignatedHostAllocatorStrategy is a little special of not being specified in instance offerings; when a zoneUuid or a clusterUuid or
          a hostUuid is specified in :ref:`CreateVmInstance <CreateVmInstance>`, DesignatedHostAllocatorStrategy automatically overrides
          the strategy in instance offering.

.. _instance offering state:

State
+++++

Instance offerings have two states:

- **Enabled**:

  The state that allows VMs to be created from this instance offering

- **Disabled**:

  The state that DOESN'T allows VMs to be created from this instance offering

----------
Operations
----------

.. _CreateInstanceOffering:

Create Instance Offering
========================

Users can use CreateInstanceOffering to create an instance offering. For example::

    CreateInstanceOffering name=small cpuNum=1 cpuSpeed=1000 memorySize=1073741824

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
   * - **cpuNum**
     - VCPU num, see :ref:`CPU capacity <instance offering cpu capacity>`
     -
     -
     - 0.6
   * - **cpuSpeed**
     - VCPU speed, see :ref:`CPU capacity <instance offering cpu capacity>`
     -
     -
     - 0.6
   * - **memorySize**
     - memory size, in bytes
     -
     -
     - 0.6
   * - **type**
     - type, default is UserVm, see :ref:`type <instance offering type>`
     - true
     - - UserVm
       - VirtualRouter
     - 0.6

.. _DeleteInstanceOffering:

Delete Instance Offering
========================

Users can use DeleteInstanceOffering to delete an instance offering. For example::

    DeleteInstanceOffering uuid=1164a094fea34f1e8265c802a8048bae


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
     - instance offering uuid
     -
     -
     - 0.6

Change State
============

Users can use ChangeInstanceOfferingState to change a state of instance offering. For example::

    ChangeInstanceOfferingState uuid=1164a094fea34f1e8265c802a8048bae stateEvent=enable

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
   * - **stateEvent**
     - state trigger event

       - enable: change state to Enabled
       - disable: change state to Disabled
     -
     - - enable
       - disable
     - 0.6
   * - **uuid**
     - instance offering uuid
     -
     -
     - 0.6

Query Instance Offering
=======================

Users can use QueryInstanceOffering to query instance offerings. For example::

    QueryInstanceOffering cpuSpeed=512 cpuNum>2

::

    QueryInstanceOffering vmInstance.state=Stopped


Primitive Fields of Query
+++++++++++++++++++++++++

see :ref:`instance offering inventory <instance offering inventory>`

Nested and Expanded Fields of Query
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
     - VMs that are created from this instance offering
     - 0.6

----
Tags
----

Users can create user tags on an instance offering with resourceType=InstanceOfferingVO. For example::

    CreateUserTag resourceType=InstanceOfferingVO tag=web-server-offering resourceUuid=45f909969ce24865b1bbca4adb66710a

System Tags
===========

Dedicated Primary Storage
+++++++++++++++++++++++++

When creating VMs, users can use a system to specify primary storage on which root volumes will be created.

.. list-table::
   :widths: 20 30 40 10
   :header-rows: 1

   * - Tag
     - Description
     - Example
     - Since
   * - **primaryStorage::allocator::uuid::{uuid}**
     - | if present, the VM's root volume will be allocated on
       | the primary storage whose uuid is *uuid*;
       | an allocation failure will be raised if the specified primary storage
       | doesn't exist or doesn't have enough capacity.
     - primaryStorage::allocator::uuid::b8398e8b7ff24527a3b81dc4bc64d974
     - 0.6
   * - **primaryStorage::allocator::userTag::{tag}::required**
     - | if present, the VM's root volume will be allocated on the
       | primary storage which has user tag *tag*;
       | an allocation failure will be raised if no primary storage has
       | the *tag* or primary storage having the *tag* doesn't
       | have enough capacity
     - primaryStorage::allocator::userTag::SSD::required
     - 0.6
   * - **primaryStorage::allocator::userTag::{tag}**
     - | if present, the VM's root volume will be allocated on the primary storage
       | which has user tag *tag*, if there is any;
       | NO failure will be raised if no primary storage has the *tag* or
       | primary storage having the *tag* doesn't
       | have enough capacity, instead, a random primary storage will be chosen.
     - primaryStorage::allocator::userTag::SSD
     - 0.6

if more than one above system tags present on a disk offering, the precedent order is::

    primaryStorage::allocator::uuid::{uuid} > primaryStorage::allocator::userTag::{tag}::required > primaryStorage::allocator::userTag::{tag}
