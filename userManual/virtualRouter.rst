.. _virtual router:

===================================
Network Services And Virtual Router
===================================

.. contents:: `Table of contents`
   :depth: 6

--------
Overview
--------

ZStack supports a couple of OSI layer 4 ~ 7 network services: DHCP, DNS, SNAT, EIP, and PortForwarding.
A L3 network can enable network services supplied by providers attached to its parent
L2 network. Check :ref:`l3Network network services` for a list of supported network services.

ZStack comes with a builtin network service provider -- Virtual Router Provider, which uses customized Linux VMs to implement network services.
When creating a new VM on a L3 network that has network services attached from the virtual router provider, a virtual router VM known as appliance
VM will be created if there isn't one yet.

.. image:: virtualrouter1.png
   :align: center

Computing capacity(CPU, Memory) of a virtual router VM is defined by a special instance offering called virtual router offering. Besides
CPU and memory, several extra parameters like image, management L3 network, public L3 network can be defined in a virtual router offering;
details can be found in :ref:`virtual router offering inventory <virtual router offering inventory>`.

Though the virtual router provider is the only network service provider(except security group provider) in current ZStack version,
the network services framework is highly pluggable that vendors can easily add their implementation by implementing small plugins.

----------------
Network typology
----------------

A virtual router VM typically has three L3 networks:

- **Management Network**:

  The network that ZStack management nodes communicate to virtual router agents; eth0 in is the nic on
  the management network.

- **Public Network**:

  The network that provides internet access, and provides public IPs for user VMs that use EIP, port forwarding, and source NAT;
  eth1 is the nic on the public network.

  .. note:: A RFC 1918 private subnet can be used as a public network as long as it can reach internet.

- **Guest Network**

  The network where user VMs connect. eth2 is the nic on the guest network.

In a normal setup, all three networks should be separate L3 networks; however, two or even three networks can be combined to one network, depending on
what network typology you want.

For a :ref:`flat network <l3Network typology>`, a virtual router VM provides only DHCP and DNS services, the network typologies can be:

- **Combined public network and guest network; a separate management network**

  .. image:: virtualrouter2.png
     :align: center

- **Combined all of public network, guest network, and management network**

  .. image:: virtualrouter3.png
     :align: center

For a :ref:`private network or isolated network <l3Network typology>`, a virtual router VM provides DHCP, DNS, SNAT; and may provide EIP and Port Forwarding too, depending on users' choices; the network
typologies can be:

- **Combined public network and management network; a separate guest network**

  .. image:: virtualrouter4.png
     :align: center

- **Separate public network, management network, and guest network**

  .. image:: virtualrouter5.png
     :align: center

.. note:: Because SSH port 22 is open on the management network, combining management network with other networks may lead to security issues.
          It's highly recommended to use a separate management network.

.. note:: VPC is not supported in this ZStack version.

-------------------------------
Virtual Router Network Services
-------------------------------

In this ZStack version, the virtual router provider provides five network services: DHCP, DNS, SNAT, EIP, and PortForwarding; we will talk about EIP and
Port Forwarding in dedicated chapters because they have own APIs.

- **DHCP**

  The virtual router VM acts as a DHCP server on the guest L3 network; the virtual router DHCP server
  uses static IP-MAC mapping so user VMs always get the same IP address.

- **DNS**

  The virtual router VM, no matter the DNS service is enabled or not, is always the DNS server of the guest L3 network.
  If the DNS service is enabled, DNS of the guest L3 network will be set as upstream DNS servers of the virtual router VM.
  See :ref:`L3 network <l3Network>` for how to add DNS to a L3 network.

- **SNAT**

  The virtual router VM acts as a router and provides source NAT to user VMs.

.. _appliance vm inventory:

---------
Inventory
---------

Besides properties included in the :ref:`VM instance inventory <vm inventory>`, the virtual router VM has some extra properties.

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
   * - **applianceVmType**
     - appliance VM type
     -
     - - VirtualRouter
     - 0.6
   * - **managementNetworkUuid**
     - the management L3 network uuid
     -
     -
     - 0.6
   * - **defaultRouteL3NetworkUuid**
     - the uuid of L3 network which provides default routing in the virtual router VM
     -
     -
     - 0.6
   * - **publicNetworkUuid**
     - the public L3 network uuid
     -
     -
     - 0.6
   * - **status**
     - virtual router agent status
     -
     - - Connecting
       - Connected
       - Disconnected
     - 0.6

Example
=======

::

        {
            "allVolumes": [
                {
                    "createDate": "August 2, 2015 5:54:12 PM",
                    "description": "Root volume for VM[uuid:f1e76cb2ef0c4dfa87f3b807eb4d7437]",
                    "deviceId": 0,
                    "format": "qcow2",
                    "installPath": "/opt/zstack/nfsprimarystorage/prim-a82b75ee064a48708960f42b800bd910/rootVolumes/acct-36c27e8ff05c4780bf6d2fa65700f22e/vol-2acccd875e364b53824def6248c94a51/2acccd875e364b53824def6248c94a51.qcow2",
                    "lastOpDate": "Dec 2, 2015 5:54:12 PM",
                    "name": "ROOT-for-virtualRouter.l3.8db7eb2ccdab4c4eb4784e46895bb016",
                    "primaryStorageUuid": "a82b75ee064a48708960f42b800bd910",
                    "rootImageUuid": "b4fe2ebbc4522e199d36985012254d7d",
                    "size": 462945280,
                    "state": "Enabled",
                    "status": "Ready",
                    "type": "Root",
                    "uuid": "2acccd875e364b53824def6248c94a51",
                    "vmInstanceUuid": "f1e76cb2ef0c4dfa87f3b807eb4d7437"
                }
            ],
            "applianceVmType": "VirtualRouter",
            "clusterUuid": "b429625fe2704a3e94d698ccc0fae4fb",
            "createDate": "Dec 2, 2015 5:54:12 PM",
            "defaultRouteL3NetworkUuid": "95dede673ddf41119cbd04bcb5d73660",
            "hostUuid": "d07066c4de02404a948772e131139eb4",
            "hypervisorType": "KVM",
            "imageUuid": "b4fe2ebbc4522e199d36985012254d7d",
            "instanceOfferingUuid": "f50a232a1448401cb8d049aad9c3860b",
            "lastHostUuid": "d07066c4de02404a948772e131139eb4",
            "lastOpDate": "Dec 2, 2015 5:54:12 PM",
            "managementNetworkUuid": "95dede673ddf41119cbd04bcb5d73660",
            "name": "virtualRouter.l3.8db7eb2ccdab4c4eb4784e46895bb016",
            "rootVolumeUuid": "2acccd875e364b53824def6248c94a51",
            "publicNetworkUuid": "95dede673ddf41119cbd04bcb5d73660",
            "state": "Running",
            "status": "Connected",
            "type": "ApplianceVm",
            "uuid": "f1e76cb2ef0c4dfa87f3b807eb4d7437",
            "vmNics": [
                {
                    "createDate": "Dec 2, 2015 5:54:12 PM",
                    "deviceId": 1,
                    "gateway": "10.1.1.1",
                    "ip": "10.1.1.155",
                    "l3NetworkUuid": "8db7eb2ccdab4c4eb4784e46895bb016",
                    "lastOpDate": "Dec 2, 2015 5:54:12 PM",
                    "mac": "fa:99:e7:31:98:01",
                    "metaData": "4",
                    "netmask": "255.255.255.0",
                    "uuid": "30bd463b926e4299a1326293ee75ae13",
                    "vmInstanceUuid": "f1e76cb2ef0c4dfa87f3b807eb4d7437"
                },
                {
                    "createDate": "Dec 2, 2015 5:54:12 PM",
                    "deviceId": 0,
                    "gateway": "192.168.0.1",
                    "ip": "192.168.0.188",
                    "l3NetworkUuid": "95dede673ddf41119cbd04bcb5d73660",
                    "lastOpDate": "Dec 2, 2015 5:54:12 PM",
                    "mac": "fa:74:3f:40:cb:00",
                    "metaData": "3",
                    "netmask": "255.255.255.0",
                    "uuid": "dc02fee25e9244ad8cbac151657a7b34",
                    "vmInstanceUuid": "f1e76cb2ef0c4dfa87f3b807eb4d7437"
                }
            ],
            "zoneUuid": "3a3ed8916c5c4d93ae46f8363f080284"
        }

.. _virtual router offering:

-----------------------
Virtual Router Offering
-----------------------

A virtual router offering is an :ref:`instance offering <instance offering>` with some extra properties.

.. _virtual router offering inventory:

Inventory
=========

Besides properties in :ref:`instance offering inventory <instance offering inventory>`, the virtual router offering has below additional properties:

Properties
++++++++++

.. list-table::
   :widths: 20 40 10 20 10
   :header-rows: 1

   * - **managementNetworkUuid**
     - management L3 network uuid
     -
     -
     - 0.6
   * - **publicNetworkUuid**
     - public L3 network uuid
     -
     -
     - 0.6
   * - **zoneUuid**
     - uuid of ancestor zone. A virtual router VM will only be created from a virtual router offering in the same zone.
     -
     -
     - 0.6
   * - **isDefault**
     - see ::ref:`default offering <default offering>`
     -
     -
     - 0.6
   * - **imageUuid**
     - virtual router image uuid, see :ref:`image <virtual router image>`
     -
     -
     - 0.6

Example
+++++++

::

        {
            "allocatorStrategy": "DefaultHostAllocatorStrategy",
            "cpuNum": 1,
            "cpuSpeed": 128,
            "createDate": "Nov 30, 2015 3:31:43 PM",
            "imageUuid": "b4fe2ebbc4522e199d36985012254d7d",
            "isDefault": true,
            "lastOpDate": "Nov 30, 2015 3:31:43 PM",
            "managementNetworkUuid": "95dede673ddf41119cbd04bcb5d73660",
            "memorySize": 536870912,
            "name": "VROFFERING5",
            "publicNetworkUuid": "95dede673ddf41119cbd04bcb5d73660",
            "sortKey": 0,
            "state": "Enabled",
            "type": "VirtualRouter",
            "uuid": "f50a232a1448401cb8d049aad9c3860b",
            "zoneUuid": "3a3ed8916c5c4d93ae46f8363f080284"
        }

.. _default offering:

Default Offering
----------------

When creating a virtual router VM on a L3 network, ZStack needs to decide what virtual router offering to use; the strategy is:

1. use a virtual router offering if it has a system tag :ref:`guestL3Network <vr tag guestL3Network>` that includes the L3 network's uuid.
2. use the default virtual router offering if nothing found in step 1.

for every zone, there must be a default virtual router offering.

.. _virtual router image:

Image
-----

A virtual router VM uses a customized Linux image that can be download from http://download.zstack.org/templates/zstack-virtualrouter-0.6.qcow2.
The root credential of the Linux operating system is::

    username: root
    password: password

users who have console access to the virtual router VM can use this credential to login.

Before creating a virtual router offering, users need to add the image to a backup storage using command
:ref:`add image <add image>`; to prevent creating user VMs from this image, users can set parameter
'system' to true.

.. note:: In future ZStack version, there will be a feature that generates random passwords for the root account, which makes the virtual router VM more secure.

Management Network and Public Network
-------------------------------------

Before creating a virtual router offering, users must create those L3 networks using command :ref:`create L3 network <create L3 network>`.
To prevent creating user VMs on those networks, users can set parameter 'system' to true.

----------
Operations
----------

Create Virtual Router Offering
==============================

Users can use CreateVirtualRouterOffering to create a virtual router offering. For example::

    CreateVirtualRouterOffering name=small cpuNum=1 cpuSpeed=1000 memorySize=1073741824 isDefault=true
    managementNetworkUuid=95dede673ddf41119cbd04bcb5d73660 publicNetworkUuid=8db7eb2ccdab4c4eb4784e46895bb016 zoneUuid=3a3ed8916c5c4d93ae46f8363f080284
    imageUuid=95dede673ddf41119cbd04bcb5d73660

Besides parameters that :ref:`CreateInstanceOffering <CreateInstanceOffering>` has, there are additional parameters:

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
   * - **managementNetworkUuid**
     - uuid of management L3 network
     -
     -
     - 0.6
   * - **publicNetworkUuid**
     - uuid of public L3 network; default to managementNetworkUuid.
     - true
     -
     - 0.6
   * - **zoneUuid**
     - uuid of ancestor zone
     -
     -
     - 0.6
   * - **imageUuid**
     - image uuid
     -
     -
     - 0.6

Delete Virtual Router Offering
==============================

see :ref:`DeleteInstanceOffering <DeleteInstanceOffering>`


.. _ReconnectVirtualRouter:

Reconnect Virtual Router Agent
==============================

As mentioned before, there is a Python virtual router agent inside the virtual router VM.
Users can use ReconnectVirtualRouter to reinitialize a connection process from
a ZStack management node to a virtual router VM, which will:

1. Upgrade the virtual router agent if the md5sum of the agent binary doesn't match the md5sum of the one in the management node's agent repository.
2. Restart the agent
3. Reapply all network services configurations including DHCP, DNS, SNAT, EIP, and PortForwarding to the virtual router VM.


A command example is like::

    ReconnectVirtualRouter vmInstanceUuid=bd1652b1e44144e6b9b5b286b82edb69

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
   * - **vmInstanceUuid**
     - virtual router VM uuid
     -
     -
     - 0.6

Start Virtual Router VM
=======================

see :ref:`StartVmInstance <StartVmInstance>`. While starting,
the virtual router VM will perform agent connection process described in :ref:`ReconnectVirtualRouter <ReconnectVirtualRouter>`.

Reboot Virtual Router VM
========================

see :ref:`RebootVmInstance <RebootVmInstance>`. While rebooting,
the virtual router VM will perform agent connection process described in :ref:`ReconnectVirtualRouter <ReconnectVirtualRouter>`.

Stop Virtual Router VM
======================

see :ref:`StopVmInstance <StopVmInstance>`.

.. warning:: After the virtual router VM stops, user VMs on the guest L3 network served by the virtual router VM may lose their network functions.

Destroy Virtual Router VM
=========================

see :ref:`DestroyVmInstance <DestroyVmInstance>`.

.. warning:: After the virtual router VM is destroyed, user VMs on the guest L3 network served by the virtual router VM may lose their network functions.

Migrate Virtual Router VM
=========================

see :ref:`MigrateVm <MigrateVm>`.

Create Virtual Router VM
========================

Though there is no ready API to create a virtual router VM manually, users can trigger an automatic creation by creating or staring a user VM on
the guest L3 network. If the L3 network doesn't have a virtual router VM running, creating, or stopping/starting
a user VM will trigger the creation of a virtual router VM.


Query Virtual Router VM
=======================

Users can use QueryVirtualRouterVm to query virtual router VMs. For example::

    QueryVirtualRouterVm defaultRouteL3NetworkUuid=95dede673ddf41119cbd04bcb5d73660

::

    QueryVirtualRouterVm vmNics.mac=fa:d9:af:a1:38:01


Primitive Fields
++++++++++++++++

see :ref:`appliance vm inventory <appliance vm inventory>`.

Nested And Expanded Fields
++++++++++++++++++++++++++

.. list-table::
   :widths: 20 30 40 10
   :header-rows: 1

   * - Field
     - Inventory
     - Description
     - Since
   * - **vmNics**
     - :ref:`VM nic inventory <vm nic inventory>`
     - VM nics of the virtual router VM
     - 0.6
   * - **allVolumes**
     - :ref:`volume inventory <volume inventory>`
     - volumes of the virtual router VM
     - 0.6
   * - **host**
     - :ref:`host inventory <host inventory>`
     - host the virtual router VM is running
     - 0.6
   * - **cluster**
     - :ref:`cluster inventory <cluster inventory>`
     - cluster the virtual router VM belongs
     - 0.6
   * - **image**
     - :ref:`image inventory <image inventory>`
     - image from which the virtual router VM is created
     - 0.6
   * - **zone**
     - :ref:`zone inventory <zone inventory>`
     - zone the virtual router VM belongs
     - 0.6
   * - **rootVolume**
     - :ref:`volume inventory <volume inventory>`
     - root volume of the virtual router VM
     - 0.6
   * - **virtualRouterOffering**
     - :ref:`virtual router offering inventory <virtual router offering inventory>`
     -
     - 0.6
   * - **eip**
     - :ref:`EIP inventory <eip inventory>`
     - EIP that the virtual router VM serves
     - 0.6
   * - **vip**
     - :ref:`VIP inventory <vip inventory>`
     - VIP that the virtual router VM serves
     - 0.6
   * - **portForwarding**
     - :ref:`port forwarding rule inventory <port forwarding rule inventory>`
     - port forwarding rule that the virtual router VM serves
     - 0.6

Query Virtual Router Offering
=============================

Users can use QueryVirtualRouterOffering to query virtual router offerings. For example::

    QueryVirtualRouterOffering managementNetworkUuid=a82b75ee064a48708960f42b800bd910 imageUuid=6572ce44c3f6422d8063b0fb262cbc62

::

    QueryVirtualRouterOffering managementL3Network.name=systemL3Network image.name=newVirtualRouterImage

Primitive Fields
++++++++++++++++

see :ref:`virtual router offering inventory <virtual router offering inventory>`.

Nested And Expanded Fields
++++++++++++++++++++++++++

.. list-table::
   :widths: 20 30 40 10
   :header-rows: 1

   * - Field
     - Inventory
     - Description
     - Since
   * - **image**
     - :ref:`image inventory <image inventory>`
     - image the offering contains
     - 0.6
   * - **managementL3Network**
     - :ref:`L3 network inventory <l3Network inventory>`
     - management L3 network the offering contains
     - 0.6
   * - **publicL3Network**
     - :ref:`L3 network inventory <l3Network inventory>`
     - public L3 network the offering contains
     - 0.6
   * - **zone**
     - :ref:`zone inventory <zone inventory>`
     - zone the offering belongs to
     - 0.6

---------------------
Global Configurations
---------------------

.. _agent.deployOnStart:

agent.deployOnStart
===================

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **agent.deployOnStart**
     - virtualRouter
     - false
     - - true
       - false

Whether to deploy a virtual router agent when a virtual router VM starts/stops/reboots;
as the virtual router agent is builtin in the virtual router VM, this value should only be set to true
when users want to upgrade the agent.

.. _command.parallelismDegree:

command.parallelismDegree
=========================

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **command.parallelismDegree**
     - virtualRouter
     - 100
     - > 0

The max number of concurrent commands that can be executed by the virtual router agent.

.. _applianceVm.connect.timeout:

connect.timeout
===============

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **connect.timeout**
     - applianceVm
     - 300
     - > 0

The connecting timeout of SSH connection when management nodes connect virtual router agents, in seconds. If a management node
cannot establish a SSH connection to a virtual router VM within the given timeout, an error will be raised.

.. _applianceVm.agent.deployOnStart:

agent.deployOnStart
===================

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **agent.deployOnStart**
     - applianceVm
     - false
     - - true
       - false

Whether to deploy an appliance VM agent when an appliance VM starts/stops/reboots; as the agent is builtin in the appliance VM,
this value should only be set to true when users need to upgrade the agent.

.. note:: There are actually two agents in virtual router VM, one is virtual router agent and another is appliance VM agent.
          They work for different purposes, users normally don't need to care about them.

----
Tags
----

Users can create user tags on a virtual router offering or a virtual router VM using the same way mentioned in chapter of instance offering
and chapter of virtual macine.

System Tags
===========

Parallel Command Level
++++++++++++++++++++++

Admins can limit max number of commands that can be executed in parallel in a virtual router VM.

.. list-table::
   :widths: 20 30 40 10
   :header-rows: 1

   * - Tag
     - Description
     - Example
     - Since
   * - **commandsParallelismDegree::{parallelismDegree}**
     - the max number of commands that can be executed in parallel in a virtual router VM
     - commandsParallelismDegree::100
     - 0.6


This tag can be created on a virtual router offering or a virtual router VM; if it's on a virtual router offering, virtual router
VMs created form the offering will inherit the tag. Please use resourceType=InstanceOfferingVO for virtual router offerings,
resourceType=VmInstanceVO for virtual router VMs.

.. _vr tag guestL3Network:

Guest L3 Network
++++++++++++++++

Admins can bind a virtual router offering to a guest L3 network, in order to specify which virtual router offering to use when creating
a virtual router VM on the guest L3 network.

.. list-table::
   :widths: 20 30 40 10
   :header-rows: 1

   * - Tag
     - Description
     - Example
     - Since
   * - **guestL3Network::{guestL3NetworkUuid}**
     - the uuid of guest L3 network
     - guestL3Network::dd56c5c209a74b669b3fe6115a611d57
     - 0.6
