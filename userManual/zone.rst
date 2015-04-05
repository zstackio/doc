.. _zone:

====
Zone
====

.. contents:: `Table of contents`
   :depth: 6

--------
Overview
--------

A zone is a logic group of resources such as primary storage, clusters, L2 networks; it defines a visibility boundary that resources
in the same zone can see each other and establish relationships, while resources in different zones cannot. For example, a primary storage in zone A
can be attached to a cluster also in zone A, but cannot be attached to a cluster in zone B.

Zones' child resources, including clusters, L2 Networks and primary Storage, are organized as follows:

.. image:: zone.png
   :align: center


Descendant resources of zones are not listed in above diagram. For instance, a host in a cluster is a descendant resource of the parent zone of the cluster.

As a logic resource, zones maps facilities in datacenters to logic groups. Though there is no enforcement on how facilities must be mapped,
some advices are given to make things simple and clear:

- Hosts in the same physical layer 2 broadcast domain should be in the same zone, grouped as one or more clusters.
- Physical layer2 broadcast domains should not span multiple zones, and should be mapped as L2 networks in a single zone.
- Physical storage that provide disk spaces for VM volumes, known as primary storage, should not span multiple zones, and should be mapped as primary storage
  in a single zone.
- A datacenter can have multiple zones.

A zone can has one or more :ref:`backup storage` attached. Resources in a zone, for example primary storage, can only access backup storage attached
to the zone. Also, a backup storage can be detached from a zone; after detaching, resources in the zone will not see the backup storage any more. Detaching backup storage
is particularly useful when network typology changes in a datacenter, if the changes cause backup storage no longer accessible to resources of a zone.

.. _zone inventory:

---------
Inventory
---------

Properties:
===========

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
     - see `zone state`_
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
   * - **type**
     - reserved field
     -
     -
     - 0.6

Example
=======

::

    {
      "uuid": "b729da71b1c7412781d5de22229d5e17",
      "name": "TestZone",
      "description": "Test",
      "state": "Enabled",
      "type": "zstack",
      "createDate": "Jun 1, 2015 6:04:52 PM",
      "lastOpDate": "Jun 1, 2015 6:04:52 PM"
    }


.. _`zone state`:

State
=====

Zones have two states: Enabled and Disabled. When changing a zone's state, the operation will be cascaded to all clusters and hosts all of which belong to the zone.
For example, disabling a zone will change states of all clusters and hosts in this zone to Disabled. Because no VM can be created or started on a disabled host,
putting a zone into Disabled state can prevent any VM from being created or started in this zone.

.. note:: Admins can selectively enable hosts or clusters in a disabled zone or disable them in an enabled zone, in order to
          have fine-grained state control.


----------
Operations
----------

Create Zone
===========

Admins can use CreateZone command to create a new zone. For example::

    CreateZone name='San Jose Zone' description='this is a zone in San Jose datacenter'

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
   * - **type**
     - reserved field, don't evaluate it
     - true
     -
     - 0.6
   * - **userTags**
     - user tags, see :ref:`create tags`; resource type is ZoneVO
     - true
     -
     - 0.6
   * - **systemTags**
     - system tags, see :ref:`create tags`; resource type is ZoneVO
     - true
     -
     - 0.6

Delete Zone
===========

Admins can use DeleteZone command to delete a zone. For example::

    DeleteZone uuid=28e94936284b45f99842ababfc3f976d

.. danger:: There is no way to recover a deleted zone.

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
     - zone uuid
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

Admins can use ChangeZoneState command to change the state of a zone. For example::

    ChangeZoneState stateEvent=enable uuid=737896724f2645de9372f11b13a48223

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
     - zone uuid
     -
     -
     - 0.6
   * - **stateEvent**
     - state trigger event.

       - enable: change state to Enabled
       - disable: change state to Disabled
     -
     - - enable
       - disable
     - 0.6

Attach Backup Storage
=====================

see :ref:`attach backup storage to zone <attach backup storage to zone>`.

Detach Backup Storage
=====================

see :ref:`detach backup storage from zone <detach backup storage from zone>`.

Query Zone
==========

Admins can use QueryZone to query zones. For example::

    QueryZone name=zone1

::

    QueryZone vmInstance.uuid=13238c8e0591444e9160df4d3636be82

Primitive Fields of Query
+++++++++++++++++++++++++

see :ref:`zone inventory <zone inventory>`

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
     - :ref:`vm inventory <vm inventory>`
     - VMs belonging to this zone
     - 0.6
   * - **cluster**
     - :ref:`cluster inventory <cluster inventory>`
     - clusters belonging to this zone
     - 0.6
   * - **host**
     - :ref:`host inventory <host inventory>`
     - hosts belonging to this zone
     - 0.6
   * - **primaryStorage**
     - :ref:`primary storage inventory <primary storage inventory>`
     - primary storage belonging to this zone
     - 0.6
   * - **l2Network**
     - :ref:`L2 network inventory <l2Network inventory>`
     - L2 networks belonging to this zone
     - 0.6
   * - **l3Network**
     - :ref:`L3 network inventory <l3Network inventory>`
     - L3 networks belonging to this zone
     - 0.6
   * - **backupStorage**
     - :ref:`backup storage inventory <backup storage inventory>`
     - backup storage belonging to this zone
     - 0.6


----
Tags
----

Admins can create user tags on a zone with resourceType=ZoneVO. For example::

    CreateUserTag resourceType=ZoneVO resourceUuid=0cd1ef8c9b9e0ba82e0cc9cc17226a26 tag=privateZone

System Tags
===========

.. _zone.host.reservedMemory:

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
