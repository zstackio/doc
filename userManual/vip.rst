.. _vip:

==================
Virtual IP Address
==================

.. contents:: `Table of contents`
   :depth: 6

--------
Overview
--------

When bridging communication between two networks, many network services such as Port Forwarding, EIP, VPN, Load Balancing need
virtual Ip addresses (VIP); incoming packets are sent to VIPs and are routed to private network IPs.

.. image:: vip1.png
   :align: center

In real world cases, VIPs are usually public IPs that can be reached by the internet, routing traffics to behind private IPs
which are often on a private network not visible to the internet.

In this ZStack version, a VIP must be allocated before creating a port forwarding rule or an EIP. For this time being,
as the virtual router provider is the only network service provider, a VIP should be
created from a virtual router VM's public network(see :ref:`virtual router offering <virtual router offering>`) in order to route traffics
to the guest network.

.. _vip inventory:

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
   * - **ipRangeUuid**
     - uuid of IP range the VIP is allocated
     -
     -
     - 0.6
   * - **l3NetworkUuid**
     - uuid of L3 network the VIP is allocated
     -
     -
     - 0.6
   * - **ip**
     - IP address
     -
     -
     - 0.6
   * - **state**
     - VIP state, not implemented in this version
     -
     - - Enabled
       - Disabled
     - 0.6
   * - **gateway**
     - gateway
     -
     -
     - 0.6
   * - **netmask**
     - netmask
     -
     -
     - 0.6
   * - **serviceProvider**
     - name of service provider that uses this VIP
     - true
     -
     - 0.6
   * - **peerL3NetworkUuid**
     - uuid of L3 network to which this VIP routes traffic
     -
     -
     - 0.6
   * - **useFor**
     - the service name which uses the VIP
     - true
     - - EIP
       - PortForwarding
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
        "createDate": "Nov 28, 2015 6:52:01 PM",
        "gateway": "192.168.0.1",
        "ip": "192.168.0.189",
        "l3NetworkUuid": "95dede673ddf41119cbd04bcb5d73660",
        "lastOpDate": "Nov 28, 2015 6:52:01 PM",
        "name": "vip-905d8a5c191c6e30173037e9d4c0ec56",
        "netmask": "255.255.255.0",
        "peerL3NetworkUuid": "6572ce44c3f6422d8063b0fb262cbc62",
        "serviceProvider": "VirtualRouter",
        "state": "Enabled",
        "useFor": "Eip",
        "uuid": "429106d5a63a4995911c2c5f14299b85"
    }


----------
Operations
----------

Create VIP
==========

Users can use CreateVip to create a VIP. For example::

    CreateVip name=vip1 l3NetworkUuid=95dede673ddf41119cbd04bcb5d73660

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
   * - **l3NetworkUuid**
     - uuid of the L3 network that the VIP will be allocated
     -
     -
     - 0.6
   * - **requiredIp**
     - the IP address you want to acquire, see :ref:`requiredIp <requiredIp>`
     -
     -
     - 0.6
   * - **allocatorStrategy**
     - the algorithm of allocating a VIP
     -
     - - RandomIpAllocatorStrategy
     - 0.6

.. _requiredIp:

RequiredIp
----------

Users can instruct ZStack to allocate a specific VIP by specifying 'requiredIp', as long as the IP is still available on the target L3
network.

Delete VIP
==========

Users can use DeleteVip to delete a VIP. For example::

    DeleteVip uuid=429106d5a63a4995911c2c5f14299b85


.. warning:: If there is a network service bound to the VIP, for example, an EIP; the network service entity(an EIP or a port forwarding rule)
             will be deleted automatically as well.

Query VIP
=========

Users can use QueryVip to query a VIP. For example::

    QueryVip ip=17.16.89.2 serviceProvider!=null

::

    QueryVip eip.guestIp=10.256.99.2


Primitive Fields
++++++++++++++++

see :ref:`VIP inventory <vip inventory>`

Nested And Expanded Fields
++++++++++++++++++++++++++

.. list-table::
   :widths: 20 30 40 10
   :header-rows: 1

   * - Field
     - Inventory
     - Description
     - Since
   * - **eip**
     - :ref:`EIP inventory <eip inventory>`
     - the EIP that the VIP is bound to
     - 0.6
   * - **portForwarding**
     - :ref:`port forwarding rule inventory <port forwarding inventory>`
     - the port forwarding rule that the VIP is bound to
     - 0.6

----
Tags
----

Users can create user tags on a VIP with resourceType=VipVO. For example::

    CreateUserTag tag=web-tier-vip resourceType=VipVO resourceUuid=c3206d0e29074e21984c584074c63920
