.. _l3Network:

==========
L3 Network
==========

.. contents:: `Table of contents`
   :depth: 6

--------
Overview
--------

A L3 network is a logic network that contains a subnet and a set of network services, and that is built up on a :ref:`L2 network <l2Network>` that is responsible
for providing isolation method. Network services, which are provided by network service providers associated with the underlying L2 network, are usually software that
implement protocols spanning from OSI layer 3 to OSI layer 7.

.. _l3Network subnet:

Subnet
======

In a L3 network, the subnet can have a single consecutive IP range or multiple split IP ranges. The split IP ranges are typically useful when a portion of IP addresses
needs to be reserved from the subnet. For example, let's say you are going to create a L3 network as a management network that has a subnet 192.168.0.0/24; however, IP addresses of
192.168.0.50 ~ 192.168.0.100 have been occupied by some network devices and you don't want ZStack to use them, then you can create two split IP ranges::

    IP Range1

    start IP: 192.168.0.2
    end IP: 192.168.0.49
    gateway: 192.168.0.1
    netmask: 255.255.255.0

    IP Range2

    start IP: 192.168.0.101
    end IP: 192.168.0.254
    gateway: 192.168.0.1
    netmask: 255.255.255.0

You can create split IP ranges as many as you want, as long as they all belong to the same `CIDR <http://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing>`_.


.. _l3Network network services:

Network Services
================

Network services implementing OSI layer 3 ~ layer 7 protocols aim to serve VMs on a L3 network. Network services are provided by network services providers that
are associated to the parent L2 network of a L3 network. A type of network service can have multiple providers, and a provider can provide several types of different services.
After a L3 network is created, users can attach network services to it and choose network services providers. In this ZStack version,
a table of supported services/providers is shown as follows:

.. list-table::
   :widths: 30 30 30 10
   :header-rows: 1

   * - Network Service
     - Provider
     - Attachable L2 Network
     - Since
   * - DHCP
     - Virtual Router
     - - L2NoVlanNetwork
       - L2VlanNetwork
     - 0.6
   * - DNS
     - Virtual Router
     - - L2NoVlanNetwork
       - L2VlanNetwork
     - 0.6
   * - Source NAT (SNAT)
     - Virtual Router
     - - L2NoVlanNetwork
       - L2VlanNetwork
     - 0.6
   * - Port Forwarding
     - Virtual Router
     - - L2NoVlanNetwork
       - L2VlanNetwork
     - 0.6
   * - Elastic IP (EIP)
     - Virtual Router
     - - L2NoVlanNetwork
       - L2VlanNetwork
     - 0.6
   * - Security Group
     - Security Group
     - - L2NoVlanNetwork
       - L2VlanNetwork
     - 0.6

In the table, the column 'Attachable L2 Network' indicates what L2 networks providers can attach. If a provider cannot attach to a L2 network,
it cannot provide services to child L3 networks of the L2 network.

.. _l3Network inventory:

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
     - uuid of ancestor zone, see :ref:`zone <zone>`
     -
     -
     - 0.6
   * - **l2NetworkUuid**
     - uuid of parent L2 network, see :ref:`L2 network <l2Network>`
     -
     -
     - 0.6
   * - **state**
     - see :ref:`state <l3Network state>`
     -
     - - Enabled
       - Disabled
     - 0.6
   * - **dnsDomain**
     - see :ref:`domain <l3Network dnsDomain>`
     - true
     -
     - 0.6
   * - **ipRanges**
     - a list of :ref:`IP ranges <l3Network IP range>`
     -
     -
     - 0.6
   * - **dns**
     - a list of :ref:`DNS <l3Network DNS>`
     -
     -
     - 0.6
   * - **networkServices**
     - a list of :ref:`network services references <l3Network network service reference>`
     -
     -
     - 0.6
   * - **type**
     - L3 network type
     -
     - - L3BasicNetwork
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
        "uuid": "f73926eb4f234f8195c61c33d8db419d",
        "name": "GuestNetwork",
        "description": "Test",
        "type": "L3BasicNetwork",
        "zoneUuid": "732fbb4383b24b019f60d862995976bf",
        "l2NetworkUuid": "f1a092c6914840c9895c564abbc55375",
        "state": "Enabled",
        "createDate": "Jun 1, 2015 11:07:24 PM",
        "lastOpDate": "Jun 1, 2015 11:07:24 PM",
        "dns": [],
        "ipRanges": [
          {
            "uuid": "78b43f4b0a9745fab49c967e1c35beb1",
            "l3NetworkUuid": "f73926eb4f234f8195c61c33d8db419d",
            "name": "TestIpRange",
            "description": "Test",
            "startIp": "10.10.2.100",
            "endIp": "10.20.2.200",
            "netmask": "255.0.0.0",
            "gateway": "10.10.2.1",
            "createDate": "Jun 1, 2015 11:07:24 PM",
            "lastOpDate": "Jun 1, 2015 11:07:24 PM"
          }
        ],
        "networkServices": [
          {
            "l3NetworkUuid": "f73926eb4f234f8195c61c33d8db419d",
            "networkServiceProviderUuid": "bbb525dc4cc8451295d379797e092dba",
            "networkServiceType": "DHCP"
          }
        ]
      }
    }

.. _l3Network state:

State
=====

L3 networks have two states:

- **Enabled**

  The state that allows new VMs to be created

- **Disabled**

  The state that DOESN'T allow new VMs to be created

  .. note:: Existing VMs on disabled L3 networks can still be stopped, started, rebooted, and deleted.

.. _l3Network dnsDomain:

DNS Domain
==========

The DNS domain is used to expand hostnames of VMs on the L3 network to FQDNs(Full Qualified Domain Name);
for example, if the hostname of a VM is 'vm1' and the DNS domain of the L3 network is
'zstack.org', the final hostname will be expanded to 'vm1.zstack.org'.


.. _l3Network IP range:

IP Range
========

In this ZStack version, only IPv4 IP range is supported.

.. _l3Network IP range inventory:

Inventory
+++++++++

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
   * - **startIp**
     - the first IP in range
     -
     -
     - 0.6
   * - **endIp**
     - the last IP in range
     -
     -
     - 0.6
   * - **netmask**
     - netmask of subnet
     -
     -
     - 0.6
   * - **gateway**
     - gateway of subnet
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
-------
::

    {
      "inventory": {
        "uuid": "b1cfcdeca4024d13ac82edbe8d959720",
        "l3NetworkUuid": "50e637dc68b7480291ba87cbb81d94ad",
        "name": "TestIpRange",
        "description": "Test",
        "startIp": "10.0.0.100",
        "endIp": "10.10.1.200",
        "netmask": "255.0.0.0",
        "gateway": "10.0.0.1",
        "createDate": "Jun 1, 2015 4:30:23 PM",
        "lastOpDate": "Jun 1, 2015 4:30:23 PM"
      }
    }


.. _l3Network DNS:

DNS
===

A L3 network can have one or more DNS that take effect when the DNS network service is enabled.

.. note:: In this ZStack version, only IPv4 DNS is supported

L2 Networks and L3 Networks
===========================

As a layer2 broadcast domain can contain multiple subnets, nothing will stop you from creating multiple L3 networks on the same
L2 network; however, those L3 networks are not isolated and network snooping can happen; please use on your own risks.

.. _l3Network network service reference:

Network Service References
==========================

Network service references exhibit network services enabled on the L3 network and their providers.

Inventory
+++++++++

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
   * - **l3NetworkUuid**
     - L3 network Uuid
     -
     -
     - 0.6
   * - **networkServiceProviderUuid**
     - network service provider UUID
     -
     -
     - 0.6
   * - **networkServiceType**
     - network service type
     -
     - - DHCP
       - DNS
       - SNAT
       - PortForwarding
       - EIP
       - SecurityGroup
     - 0.6

Example
-------

::

    {
      "l3NetworkUuid": "f73926eb4f234f8195c61c33d8db419d",
      "networkServiceProviderUuid": "bbb525dc4cc8451295d379797e092dba",
      "networkServiceType": "PortForwarding"
    }

.. _l3Network typology:

----------------
Network Typology
----------------

The most common network typologies in IaaS software managed clouds are:

- **Flat Network or Shared Network**:

  In this typology, all tenants share a single subnet; IaaS software only provides DHCP, DNS services; the router of datacenter is responsible for routing

  .. image:: l3Network1.png
     :align: center

- **Private Network or Isolated Network**:

  In this typology, each tenant has own subnet; IaaS software is responsible for providing routers for all subnets, which usually have DHCP, DNS, and NAT services.

  .. image:: l3Network2.png
     :align: center

- **Virtual Private Network (VPC)**:

  In this typology, each tenant can have multiple subnets; IaaS software is responsible for providing a router coordinating all subnets; tenants can configure the routing
  table of the router to control connectivity amid subnets.

  .. image:: l3Network3.png
     :align: center


Besides, typical typologies can be combined to new typologies; for example, a flat network and a private network can be put together, as:

.. image:: l3Network4.png
   :align: center

In ZStack, all those typologies can be implemented by assembling L2 networks, L3 networks and network services. For example, to create a flat network,
users can create a L3 network with only DHCP, DNS enabled; to create a private network, users can create a L3 network on a L2VlanNetwork with
DHCP, DNS, SNAT enabled.

.. note:: In this ZStack version, VPC is not supported yet.

----------
Operations
----------

.. _create L3 network:

Create L3 Network
=================

Users can use CreateL3Network to create a L3 network. For example::

    CreateL3Network l2NetworkUuid=f1a092c6914840c9895c564abbc55375 name=GuestNetwork

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
   * - **l2NetworkUuid**
     - uuid of parent L2 network, see :ref:`L2 network <l2Network>`
     -
     -
     - 0.6
   * - **dnsDomain**
     - a DNS domain, see :ref:`domain <l3Network dnsDomain>`
     - true
     -
     - 0.6
   * - **type**
     - L3 network type, see :ref:`type <l3Network type>`
     - true
     - - L3BasicNetwork
     - 0.6
   * - **system**
     - indicates whether this is a system L3 network, see :ref:`system l3Network`
     - true
     - - true
       - false
     - 0.6

.. _l3Network type:

Type
====

In this ZStack version, the only L3 network type is L3BasicNetwork. Users can leave field 'type' alone when calling CreateL3Network.

.. _system l3Network:

System L3 Network
=================

A system L3 network is reserved for ZStack and cannot be used to create user VMs. System L3 networks are typically used for public networks and
management networks. Usually, user VMs in a cloud should not have nics on a public network and a management network, but appliance VMs (e.g router
VM) do need have nics on those networks; then the management network and the public network can be created as system L3 networks.

.. note:: Management networks and public networks can also be created as non-system L3 networks, which allows user VMs to use them.
          This is normally seen in private clouds; for example, creating a user VM with a public IP directly.

.. _delete l3Network:

Delete L3 Network
=================

Users can use DeleteL3Network to delete a L3 network. For example::

    DeleteL3Network uuid=f73926eb4f234f8195c61c33d8db419d

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
     - L3 network uuid
     -
     -
     - 0.6
   * - **deleteMode**
     - see :ref:`delete resource`
     - true
     - - Permissive
       - Enforcing
     - 0.6

.. danger:: Deleting a L3 network will stop all VMs that have nics on it and will delete the nics from VMs; if the nic on the L3 network
            is the only nic of a VM, the VM will be deleted as well. There is no way to recover a deleted L3 network.

Add IP Ranges
=============

Add Split Ranges
++++++++++++++++

Users can use AddIpRange to add an IP range to a L3 network; this is useful for adding split IP ranges. For example::

    AddIpRange name=ipr1 startIp=192.168.0.2 endIp=192.168.0.100 netmask=255.255.255.0 gateway=192.168.0.1 resourceUuid=50e637dc68b7480291ba87cbb81d94ad

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
   * - **l3NetworkUuid**
     - uuid of parent L3 network
     -
     -
     - 0.6
   * - **startIp**
     -  the first IP in range
     -
     -
     - 0.6
   * - **endIp**
     - the last IP in range
     -
     -
     - 0.6
   * - **netmask**
     - netmask of subnet
     -
     -
     - 0.6
   * - **gateway**
     - gateway of subnet
     -
     -
     - 0.6

Add CIDR
++++++++

Users can also use AddIpRangeByNetworkCidr to add an IP range. For example::

    AddIpRangeByNetworkCidr name=ipr1 l3NetworkUuid=50e637dc68b7480291ba87cbb81d94ad networkCidr=10.0.1.0/24

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
   * - **l3NetworkUuid**
     - uuid of parent L3 network
     -
     -
     - 0.6
   * - **networkCidr**
     - network CIDR; it must be in format of::

            network-number/prefix-length
     -
     -
     - 0.6

Delete IP Range
===============

Users can use DeleteIpRange to delete an IP range. For example::

    DeleteIpRange uuid=b1cfcdeca4024d13ac82edbe8d959720

.. warning:: Deleting a IP range will stop all VMs that have IP addresses in the range.
             There is no way to recover a deleted IP range.

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
     - IP range uuid
     -
     -
     - 0.6
   * - **deleteMode**
     - see :ref:`delete resource`
     - true
     - - Permissive
       - Enforcing
     - 0.6

Add DNS
=======

Users can use AddDnsToL3Network to add a DNS to a L3 network. For example::

    AddDnsToL3Network l3NetworkUuid=50e637dc68b7480291ba87cbb81d94ad dns=8.8.8.8

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
   * - **l3NetworkUuid**
     - uuid of parent L3 network
     -
     -
     - 0.6
   * - **dns**
     - dns IPv4 address
     -
     -
     - 0.6

.. _l3Network attach service:

Attach Network Service
======================

After creating a L3 network and before creating any VMs on it, users can use AttachNetworkServiceToL3Network to attach network
services to the L3 network. If a network service is attached to a L3 network that already has VMs running, the existing VMs can
not use the network service until they are rebooted.

.. note:: In this ZStack version, detaching a network service from a L3 network is not supported.

For example::

    AttachNetworkServiceToL3Network l3NetworkUuid=50e637dc68b7480291ba87cbb81d94ad networkServices='{"1d1d5ff248b24906a39f96aa3c6411dd": ["DHCP", "DNS", "SNAT", "EIP"]}'

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
   * - **l3NetworkUuid**
     - L3 network uuid
     -
     -
     - 0.6
   * - **networkServices**
     - A map whose key is network service provider UUID and value is a list of network service types
     -
     -
     - 0.6

.. note:: You can use QueryNetworkServiceProvider to get the UUID of a network service provider, for example::

              QueryNetworkServiceProvider fields=uuid name=VirtualRouter

          If you want to view network services a provider provides, omit the parameter 'field', for example::

              QueryNetworkServiceProvider name=VirtualRouter

Query L3 Network
================

Users can use QueryL3Network to query L3 networks. For example::

    QueryL3Network dnsDomain=zstack.org

::

    QueryL3Network vmNic.ip=192.168.10.2


Primitive Fields of Query
+++++++++++++++++++++++++

see :ref:`L3 network inventory <l3Network inventory>`

Nested And Expanded Fields of Query
+++++++++++++++++++++++++++++++++++

.. list-table::
   :widths: 20 30 40 10
   :header-rows: 1

   * - Field
     - Inventory
     - Description
     - Since
   * - **ipRanges**
     - :ref:`IP range inventory <l3Network IP range inventory>`
     - IP ranges this L3 network contains
     - 0.6
   * - **networkServices**
     - :ref:`l3Network network service reference <l3Network network service reference>`
     - network services attached to this L3 network
     - 0.6
   * - **l2Network**
     - :ref:`L2 network <l2Network>`
     - parent L2 network
     - 0.6
   * - **vmNic**
     - :ref:`VM nic inventory <vm nic inventory>`
     - VM nics on this L3 network
     - 0.6
   * - **serviceProvider**
     - :ref:`network service provider inventory <network service provider inventory>`
     - network service providers that provides network services attached to this L3 network
     - 0.6
   * - **zone**
     - :ref:`zone inventory <zone inventory>`
     - ancestor zone
     - 0.6

---------------
L3 Network Tags
---------------

Users can create user tags on a L3 network with resourceType=L3NetworkVO. For example::

    CreateUserTag resourceType=L3NetworkVO tag=web-tier-l3 resourceUuid=f6be73fa384a419986fc6d1b92f95be9

-------------
IP Range Tags
-------------

Users can create user tags on an IP range with resourceType=IpRangeVO. For example::

    CreateUserTag resourceType=IpRangeVO tag=web-tier-IP resourceUuid=8191d946954940428b7d003166fa641e

