.. _cluster:

=======
Cluster
=======

.. contents:: `Table of contents`
   :depth: 6

--------
Overview
--------

A cluster is a logic group of analogy hosts. Hosts in the same cluster must be installed with the same operating systems(hypervisor), have
the same layer2 network connectivity, and can access the same primary storage. In real datacenters, a cluster usually maps to a rack.

A typical cluster and its relationship to primary storage, L2 networks is shown in below diagram.

.. image:: cluster.png
   :align: center

A cluster can have one or more primary storage attached, as long as hosts in the cluster can all access these primary storage. Also, a
primary storage can be detached from a cluster; this is particularly useful when network typology changes in datacenters, which causes
the primary storage no longer accessible to hosts in the cluster.

A cluster can have one or more L2 networks attached, as long as hosts in the cluster are all in the physical layer2 broadcast domains those
L2 networks represent. Also, a L2 network can be detached from a cluster, if network typology changes in the datacenter cause
hosts in the cluster no longer in the layer2 broadcast domain of the L2 network.

The size of a cluster, which is the maximum hosts the cluster can contain, is not enforced.


.. _cluster inventory:

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
   * - **hypervisorType**
     - see `cluster hypervisor type`_
     -
     - - KVM
     - 0.6
   * - **state**
     - see `cluster state`_
     -
     - - Enabled
       - Disabled
     - 0.6
   * - **zoneUuid**
     - uuid of zone containing the cluster. See :ref:`zone <zone>`.
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
   * - **type**
     - reserved field
     -
     -
     - 0.6
   * - **userTags**
     - user tags, see :ref:`create tags`
     - true
     -
     - 0.6
   * - **systemTags**
     - system tags, see :ref:`create tags`
     - true
     -
     - 0.6

Example
=======

::

    {
      "inventory": {
        "uuid": "c1bd173d5cd84f0e9e7c47195ae27ec6",
        "name": "cluster1",
        "description": "test",
        "state": "Enabled",
        "zoneUuid": "1b830f5bd1cb469b821b4b77babfdd6f"
        "hypervisorType": "KVM",
        "lastOpDate": "Jun 1, 2015 5:54:09 PM",
        "createDate": "Jun 1, 2015 5:54:09 PM",
        "type": "zstack",
      }
    }

.. _cluster hypervisor type:

Hypervisor Type
===============

Hypervisor type indicates what hypervisor(operating system) installed on hosts in the cluster. In this ZStack version, the only supported hypervisor is KVM.

.. _cluster state:

State
=====

Cluster has two states: Enabled and Disabled, just like :ref:`zone <zone>`. When changing the state of a cluster, the operation will be spread to all hosts of the cluster;
For example, disabling a cluster will disable all hosts in the cluster as well.

.. note:: Admins can selectively enable hosts in a disabled cluster or disable them in an enabled cluster, in order to have fine-grained state control.

----------
Operations
----------

Create Cluster
==============

Admins can use CreateCluster command to create a cluster. For example::

    CreateCluster name=cluster1 hypervisorType=KVM zoneUuid=1b830f5bd1cb469b821b4b77babfdd6f

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
   * - **zoneUuid**
     - uuid of parent zone
     -
     -
     - 0.6
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
   * - **hypervisorType**
     - see `cluster hypervisor type`_
     -
     -
     - 0.6
   * - **type**
     - reserved field, don't evaluate it
     - true
     -
     - 0.6

Delete Cluster
==============

Admins can use DeleteCluster to delete a cluster. For example::

    DeleteCluster uuid=c1bd173d5cd84f0e9e7c47195ae27ec6

.. danger:: Deleting a cluster will delete hosts in the cluster; VMs will be migrated to other clusters or be stopped if no available clusters to migrate;
            primary storage and L2 networks attached to the cluster will be detached. There is no way to recover a deleted cluster.

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
     - cluster uuid
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

Admins can use ChangeClusterState to change the state of a cluster. For example::

    ChangeClusterState uuid=c1bd173d5cd84f0e9e7c47195ae27ec6 stateEvent=disable

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
     - cluster uuid
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

.. _attach primary storage to cluster:

Attach Primary Storage
======================

Admins can use AttachPrimaryStorageToCluster command to attach a primary storage to a cluster. For example::

    AttachPrimaryStorageToCluster clusterUuid=c1bd173d5cd84f0e9e7c47195ae27ec6 primaryStorageUuid=1b830f5bd1cb469b821b4b77babfdd6f

.. note:: Only sibling primary storage can be attached to a cluster. In other words, primary storage and clusters must be in the
          same zone.

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
   * - **clusterUuid**
     - cluster uuid
     -
     -
     - 0.6
   * - **primaryStorageUuid**
     - primary storage uuid
     -
     -
     - 0.6

.. _detach primary storage from cluster:

Detach Primary Storage
======================

Admin cans use DetachPrimaryStorageFromCluster to detach a primary storage from a cluster. For example::

    DetachPrimaryStorageFromCluster clusterUuid=c1bd173d5cd84f0e9e7c47195ae27ec6 primaryStorageUuid=1b830f5bd1cb469b821b4b77babfdd6f

.. note:: During detaching, VMs that have root volumes on the primary storage and that run in the cluster will be stopped. Users can
          start those VMs again if the primary storage is still attached to some other clusters, or start them after the primary storage
          is attached to a new cluster.

Detaching primary storage is useful when admin wants to make a primary storage on longer accessible to a cluster. For example, in order to move VMs
from a cluster equipped with aged hosts to a cluster with new, powerful hosts, admins can detach the primary storage on which root volumes of VMs locate
from the old cluster and attach it to the new cluster, then start those stopped VMs; because the old cluster cannot access the primary storage anymore,
ZStack will choose the new cluster to start VMs.

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
   * - **clusterUuid**
     - cluster uuid
     -
     -
     - 0.6
   * - **primaryStorageUuid**
     - primary storage uuid
     -
     -
     - 0.6

.. _cluster attach L2 Network:

Attach L2 Network
=================

Admin can use AttachL2NetworkToCluster command to attach a L2 network to a cluster. For example::

    AttachL2NetworkToCluster clusterUuid=c1bd173d5cd84f0e9e7c47195ae27ec6 l2NetworkUuid=1b830f5bd1cb469b821b4b77babfdd6f

.. note:: Only sibling L2 networks can be attached to a cluster. In other words, L2 networks and clusters must be in the
          same zone.

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
   * - **clusterUuid**
     - cluster uuid
     -
     -
     - 0.6
   * - **l2NetworkUuid**
     - L2 network uuid
     -
     -
     - 0.6

.. _cluster detach L2 network:

Detach L2 Network
=================

Admins can use DetachL2NetworkFromCluster command to detach a L2 network from a cluster. For example::

    DetachL2NetworkFromCluster clusterUuid=c1bd173d5cd84f0e9e7c47195ae27ec6 l2NetworkUuid=1b830f5bd1cb469b821b4b77babfdd6f

.. note:: During detaching, VMs which run in the clusters and have nics on the L2 networks(through L3 networks) will be stopped. Users can
          start those VMs again if the L2 networks are still attached to other clusters, or start them after the L2 networks
          are attached to new clusters.

Detaching L2 networks can be useful when admins want to make network typology changes in datacenters. After hosts in a cluster no longer connect to a physical layer2 network,
admin can detach the L2 network representing the physical layer2 network from the cluster.

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
   * - **clusterUuid**
     - cluster uuid
     -
     -
     - 0.6
   * - **l2NetworkUuid**
     - L2 network uuid
     -
     -
     - 0.6

Query Cluster
=============

Admins can use QueryCluster to query clusters. For example::

    QueryCluster hypervisorType=KVM

::

    QueryCluster primaryStorage.availableCapacity>100000000

Primitive Fields of Query
+++++++++++++++++++++++++

see :ref:`cluster inventory <cluster inventory>`

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
     - see :ref:`zone inventory <zone inventory>`
     - parent zone
     - 0.6
   * - **host**
     - see :ref:`host inventory <host inventory>`
     - hosts belonging to this cluster
     - 0.6
   * - **vmInstance**
     - see :ref:`vm inventory <vm inventory>`
     - VMs belonging to this cluster
     - 0.6
   * - **l2Network**
     - see :ref:`L2 network inventory <l2Network inventory>`
     - L2 networks attached to this cluster
     - 0.6
   * - **primaryStorage**
     - see :ref:`primary storage inventory <primary storage inventory>`
     - primary storage attached to this cluster
     - 0.6

----
Tags
----

Admins can create user tags on a cluster with resourceType=ClusterVO. For example::

    CreateUserTag resourceType=ClusterVO resourceUuid=80a979b9e0234564a22a4cca8c1dff43 tag=secureCluster

System Tags
===========

.. _cluster.host.reservedMemory:

Reserved Capacity
+++++++++++++++++

.. list-table::
   :widths: 20 30 40 10
   :header-rows: 1

   * - Tag
     - Description
     - Example
     - Since
   * - **host::reservedMemory::{capacity}**
     - see :ref:`host capacity reservation`
     - host::reservedMemory::1G
     - 0.6

.. _hypervisor: http://en.wikipedia.org/wiki/Hypervisor

