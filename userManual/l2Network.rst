.. _l2Network:

==========
L2 Network
==========

.. contents:: `Table of contents`
   :depth: 6

--------
Overview
--------

A L2 network reflects a `layer2 broadcast domain <http://en.wikipedia.org/wiki/Broadcast_domain>`_ in a datacenter. That means,
in addition to the traditional OSI data link layer, all technologies that provide layer 2 isolation can be L2 networks
in ZStack. For example, VLAN, VxLAN, or SDNs that create layer 2 overlay networks. In ZStack, a L2 network is responsible
for providing the layer 2 isolation method to child L3 networks.

.. image:: l2Network.png
   :align: center

A L2 network can be attached to sibling clusters.

.. _l2Network inventory:

---------
Inventory
---------

.. _l2Network properties:

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
   * - **physicalInterface**
     - see :ref:`physical interface <l2Network physical interface>`
     -
     -
     - 0.6
   * - **type**
     - L2 network type
     -
     - - L2NoVlanNetwork
       - L2VlanNetwork
     - 0.6
   * - **attachedClusterUuids**
     - a list of cluster uuid to which the L2 network has attached, see :ref:`attach cluster <l2Network attach cluster>`
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

.. _l2Network physical interface:

Physical Interface
==================

The physical interface is a string that contains information needed by a L2 network plugin for manipulating network system in a datacenter.
The information encoded in physical interface is specific to L2 network types and hypervisor types of clusters that L2 networks may
attach. This sounds a little complex. The complexity is originated from hypervisors using their own notations to describe L2 networks, and
a L2 network can be attached to multiple clusters of different hypervisor types. A real world example may help to understand this.

Let's say your datacenter has a L2 network (l2Network A) which spans to two clusters, one is a KVM cluster, another is a VMWare cluster. In KVM,
the L2 network is realized by ethernet device in Linux operating system; in this example, let's assume each eth0 of KVM hosts
connects to the L2 network. In the VMWare cluster, the L2 network is realized by vswitch; in this example, let's assume vswitch0 in the VMWare cluster
connects to the L2 network; then the typology is like:

.. image:: l2Network-physical-interface.png
   :align: center

As mentioned in section :ref:`host <host>`, lots of operations seemingly applied to clusters are actually delegated to hosts;
Here, when attaching the L2 network A to the KVM cluster and the VMWare cluster, ZStack must understand what are notations describing the L2
network in those hypervisors of clusters; in this case, ZStack must know that on KVM hosts, eth0 is the representation of the L2 network, but on VMWare
hosts, vswitch0 is the representation. Physical interface is the field that encodes those hypervisor specific information.

.. note:: As this ZStack version supports only KVM, we won't discuss VMWare details for L2 networks. Above example largely aims to help understand
          the design of the physical interface.

.. _l2Network attach cluster:

Attaching Cluster
=================

Attaching cluster is to associate L2 networks to sibling clusters, which provides a flexible way that manifests relations between hosts and
layer 2 networks in a real datacenter. Let's see a concrete example.

.. image:: l2Network-cluster1.png
   :align: center

Let's assume the network typology in your datacenter is as above diagram. Eth0 of hosts in all clusters are on the same layer 2 network called L2
Network1; eth1 of cluster1 and cluster3 are on another layer 2 network called L2 network2. To describe this typology in ZStack, you can attach L2 network1
to all three clusters but attach L2 network2 to only cluster1 and cluster3.

A couple months later, the network typology needs changing because of business requirements, you unplug cables of eth1 of hosts in cluster3 from the rack switch,
so cluster3 is not with L2 network2 anymore; you can detach the L2 network2 from cluster3 to notify ZStack about the network typology change.


.. image:: l2Network-cluster2.png
   :align: center

L2NoVlanNetwork
===============

L2NoVlanNetwork, whose properties are listed in :ref:`properties <l2Network properties>` is the base type of L2 Networks.
The 'NoVlan' in the name DOESN'T mean the network cannot use VLAN technology, it only denotes that ZStack itself will not use VLAN
to create a layer 2 broadcast domain in an active manner. To make it clear, take a look at below two diagrams:

.. image:: l2NoVlanNetwork1.png
   :align: center
   :width: 500px
   :height: 400px

In this setup, two switch ports 5 and 12 are untagged with VLAN 10(access port with VLAN 10 in Cisco term), and connect to eth0 on host1 and host2 respectively. This
is a very valid setup matching to a L2NoVlanNetwork. Admin cans create a L2NoVlanNetwork with 'physicalInterface' = 'eth0' and attach it to the cluster.

.. image:: l2NoVlanNetwork2.png
   :align: center
   :width: 500px
   :height: 400px

In this setup, two switch ports 5 and 12 are tagged with VLAN 10(trunk port with VLAN 10 in Cisco term), and respectively connect to eth0.10 that is a pre-created VLAN device on host1
and host2. This is also a very valid setup matching to a L2NoVlanNetwork. Admins can create a L2NoVlanNetwork with 'physicalInterface' =
'eth0.10' and attach it to the cluster.

Now it should be understood that a L2NoVlanNetwork maps to a pre-created layer 2 broadcast domain; ZStack won't create any new broadcast domain for L2NoVlanNetwork.

L2NoVlanNetwork KVM Specific
++++++++++++++++++++++++++++

When attaching a L2NoVlanNetwork to a KVM cluster, the :ref:`physicalInterface <l2Network physical interface>` should be the ethernet device name in the Linux operating system; for example,
eth0, eth0.10, em1. ZStack will use 'physicalInterface' as device name when creating a bridge using brctl. The pseudo operations are like::

    Assuming physicalInterface = eth0

    brctl create br_eth0
    brctl addif br_eth0 eth0

.. note:: If you have multiple clusters of hosts whose ethernet devices connect to the same L2 network, and you want to attach that L2 network to those clusters,
          please make sure names of all ethernet devices are the same among all Linux operating systems on hosts. For example, all ethernet devices are named as eth0.
          The best practice is installing the same Linux system on hosts of those clusters, or using udev to make all device names same.

L2NoVlanNetwork Inventory Example
+++++++++++++++++++++++++++++++++

::

    {
      "inventory": {
        "uuid": "f685ff94513542bbb8e814027f8deb13",
        "name": "l2-basic",
        "description": "Basic L2 Test",
        "zoneUuid": "45a2864b6ddf4d2fb9b4c3736a923dcb",
        "physicalInterface": "eth0",
        "type": "L2NoVlanNetwork",
        "createDate": "Jun 1, 2015 12:58:35 PM",
        "lastOpDate": "Jun 1, 2015 12:58:35 PM",
        "attachedClusterUuids": []
      }
    }

L2VlanNetwork
=============

A L2VlanNetwork is a L2 network that ZStack will actively use a VLAN to create a layer 2 broadcast domain. The ways that ZStack create layer 2 broadcast domains depend
on hypervisor types of clusters, to which L2 networks are going to attach. In addition to :ref:`properties <l2Network properties>`, a L2VlanNetwork has one more property:

.. list-table::
   :widths: 20 40 10 20 10
   :header-rows: 1

   * - Name
     - Description
     - Optional
     - Choices
     - Since
   * - **vlan**
     - VLAN id used to create layer 2 broadcast domain
     -
     - [0, 4095]
     - 0.6

When attaching a L2VlanNetwork to a cluster, ZStack uses 'vlan' collaborating with 'physicalInterface' to create vlan devices on hosts in the cluster; in order to make this work,
the switch ports to which ethernet devices identified by 'physicalInterface' connect must be tagged with 'vlan'. For example:

.. image:: l2VlanNetwork1.png
   :align: center
   :width: 500px
   :height: 400px

In this setup, switch ports 5 and 12 have been tagged with VLAN 10, then admins can create a L2VlanNetwork with 'physicalInterface' = 'eth0' and 'vlan' = 10 and
attach it to the cluster.

L2VlanNetwork KVM Specific
++++++++++++++++++++++++++

When attaching a L2VlanNetwork to a KVM cluster, ZStack will create VLAN devices on all hosts in the cluster then create bridges. The pseudo operations are like::

    Assuming physicalInterface = eth0, vlan = 10

    vconfig add eth0 10
    brctl create br_eth0_10
    brctl addif br_eth0_10 eth0.10

.. note:: Like L2NoVlanNetwork, please make sure ethernet device names of all hosts in clusters to which a L2VlanNetwork is about to attach are the same.

L2VlanNetwork Inventory Example
+++++++++++++++++++++++++++++++

::

    {
        "inventory": {
          "vlan": 10,
          "uuid": "14a01b0978684b2ea6e5a355c7c7fd73",
          "name": "TestL2VlanNetwork",
          "description": "Test",
          "zoneUuid": "c74f8ff8a4c5456b852713b82c034074",
          "physicalInterface": "eth0",
          "type": "L2VlanNetwork",
          "createDate": "Jun 1, 2015 4:31:47 PM",
          "lastOpDate": "Jun 1, 2015 4:31:47 PM",
          "attachedClusterUuids": []
        }
    }

----------
Operations
----------

Create L2 Network
=================

The commands creating L2 networks vary for different L2 network types.


Create L2NoVlanNetwork
======================

Admins can use CreateL2NoVlanNetwork to create a L2NoVlanNetwork. For example::

    CreateL2NoVlanNetwork name=management-network physicalInterface=eth0 zoneUuid=9a94e647a9f64bb392afcdc5396cc1e4

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
   * - **zoneUuid**
     - uuid of parent zone, see :ref:`zone <zone>`
     -
     -
     - 0.6
   * - **physicalInterface**
     - see :ref:`physical interface <l2Network physical interface>`
     -
     -
     - 0.6

Create L2VlanNetwork
======================

Admins can use CreateL2VlanNetwork to create a L2VlanNetwork. For example::

    CreateL2VlanNetwork name=APPLICATION-L2 physicalInterface=eth0 vlan=100 zoneUuid=69b5be02a15742a08c1b7518e32f442a

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
   * - **vlan**
     - VLAN id used to create layer 2 broadcast domain
     -
     - [0, 4095]
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
   * - **physicalInterface**
     - see :ref:`physical interface <l2Network physical interface>`
     -
     -
     - 0.6

Delete L2 Network
=================

Admins can use DeleteL2Network to delete a L2 network. For example::

    DeleteL2Network uuid=a5535531eb7346ce89cfd7e643ad1ef8

.. danger:: Deleting a L2 network will cause its child L3 network to be deleted. For consequences of deleting L3 networks,
            see :ref:`delete l3Network`. There is no way to recover a deleted L2 network.

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
     - L2 network uuid
     -
     -
     - 0.6

Attach Cluster
==============

See :ref:`cluster attach L2 network`.

Detach Cluster
==============

See :ref:`cluster detach L2 network`.

Query L2 Network
================

Admins can use QueryL2Network to query L2 networks. For example::

    QueryL2Network physicalInterface=eth0

::

    QueryL2Network l3Network.ipRanges.startIp=192.168.0.2


Primitive Fields of Query
+++++++++++++++++++++++++

see :ref:`L2 network inventory <l2Network inventory>`.

Nested and Expanded Fields of Query
+++++++++++++++++++++++++++++++++++

.. list-table::
   :widths: 20 30 40 10
   :header-rows: 1

   * - Field
     - Inventory
     - Description
     - Since
   * - **l3Network**
     - :ref:`L3 network inventory <l3Network inventory>`
     - L3 networks belonging to this L2 network
     - 0.6
   * - **cluster**
     - :ref:`cluster inventory <cluster inventory>`
     - clusters this L2 network is attached to
     - 0.6
   * - **zone**
     - :ref:`zone inventory <zone inventory>`
     - parent zone
     - 0.6

----
Tags
----

Admins can create user tags on a L2 network with resourceType=L2NetworkVO. For example::

    CreateUserTag resourceType=L2NetworkVO tag=publicL2 resourceUuid=cff4be8694174b0fb831a9fe53b1d62b

