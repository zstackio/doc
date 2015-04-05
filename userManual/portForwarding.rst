.. _port forwarding:

=======================
Elastic Port Forwarding
=======================

.. contents:: `Table of contents`
   :depth: 6

--------
Overview
--------

When user VMs are on a :ref:`private network or isolated network <l3Network typology>` with SNAT service enabled, they can
reach outside network but cannot be reached by outside network, which is the nature of SNAT. Users can create port forwarding
rules to allow outside network to reach specific ports of user VMs behind SNAT. ZStack supports elastic port forwarding rules,
which means rules can be attached/detached to/from VMs on demand.

As the virtual router provider is the only network service provider in this ZStack version, a port forwarding rule is actually
created between a virtual router VM's public network and guest network.

.. image:: portforwarding1.png
   :align: center

A VIP can be used for multiple port forwarding rules, as long as rules' port ranges don't overlap; for example:

.. image:: portforwarding2.png
   :align: center

.. _port forwarding rule inventory:

------------------------------
Port Forwarding Rule Inventory
------------------------------

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
   * - **vipIp**
     - IP address of VIP
     -
     -
     - 0.6
   * - **guestIp**
     - IP address of VM nic
     - true
     -
     - 0.6
   * - **vipUuid**
     - uuid of VIP
     -
     -
     - 0.6
   * - **vipPortStart**
     - the start port of VIP
     -
     - 1 ~ 65535
     - 0.6
   * - **vipPortEnd**
     - the end port of VIP
     -
     - 1 ~ 65535
     - 0.6
   * - **privatePortStart**
     - the start port of guest IP
     -
     - 1 ~ 65535
     - 0.6
   * - **privatePortEnd**
     - the end port of guest IP
     -
     - 1 ~ 65535
     - 0.6
   * - **vmNicUuid**
     - uuid of guest VM nic
     - true
     -
     - 0.6
   * - **protocolType**
     - protocol type of network traffic
     -
     - - TCP
       - UDP
     - 0.6
   * - **state**
     - rule state, not implemented in this version
     -
     - - Enabled
       - Disabled
     - 0.6
   * - **allowedCidr**
     - source CIDR; the port forwarding rule only applies to traffics with this source CIDR
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
            "allowedCidr": "0.0.0.0/0",
            "createDate": "Dec 6, 2015 3:04:34 PM",
            "guestIp": "10.0.0.244",
            "lastOpDate": "Dec 6, 2015 3:04:34 PM",
            "name": "pf-9uf4",
            "privatePortEnd": 33,
            "privatePortStart": 33,
            "protocolType": "TCP",
            "state": "Enabled",
            "uuid": "310a6cd618144ca683d78d74307f16a4",
            "vipIp": "192.168.0.187",
            "vipPortEnd": 33,
            "vipPortStart": 33,
            "vipUuid": "433769b59a7c42199d762af01e08ec16",
            "vmNicUuid": "4b9c27321b794679a9ba8c18239bbb0d"
        }

----------
Operations
----------

Create Port Forwarding Rule
===========================

Users can use CreatePortForwardingRule to create a port forwarding rule, with or without attaching to a VM nic. For example::

    CreatePortForwardingRule name=pf1 vipPortStart=22 vipUuid=433769b59a7c42199d762af01e08ec16 protocolType=TCP vmNicUuid=4b9c27321b794679a9ba8c18239bbb0d

A unattached rule can be attached to a VM nic later.

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
   * - **vipUuid**
     - VIP UUID
     -
     -
     - 0.6
   * - **vipPortStart**
     - the start port of VIP
     -
     - 1 - 65535
     - 0.6
   * - **vipPortEnd**
     - the end port of VIP; if omitted, it's set to vipPortStart.
     - true
     - 1 - 65535
     - 0.6
   * - **privatePortStart**
     - the start port of guest IP (VM nic's IP); if omitted, it's set to vipPortStart
     - true
     - 1 - 65535
     - 0.6
   * - **privatePortEnd**
     - the end port for guest IP (VM nic's IP); if omitted, it's set to vipPortEnd
     - true
     - 1 - 65535
     - 0.6
   * - **protocolType**
     - network traffic protocol type
     -
     - - TCP
       - UDP
     - 0.6
   * - **vmNicUuid**
     - uuid of VM nic this port forwarding rule will be attached to
     - true
     -
     - 0.6
   * - **allowedCidr**
     - source CIDR; the port forwarding rule only applies to traffics having this source CIDR; if omitted, it's set to 0.0.0.0/0
     - true
     -
     - 0.6

Delete Port Forwarding Rule
===========================

Users can use DeletePortForwardingRule to delete a port forwarding rule. For example::

    DeletePortForwardingRule uuid=310a6cd618144ca683d78d74307f16a4

The VIP is recycled for other network services to use, if no more port forwarding rules bound to it.

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
     - rule uuid
     -
     -
     - 0.6

Attach Port Forwarding Rule
===========================

Users can use AttachPortForwardingRule to attach a rule to a VM nic. For example::

    AttachPortForwardingRule ruleUuid=310a6cd618144ca683d78d74307f16a4 vmNicUuid=4b9c27321b794679a9ba8c18239bbb0d

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
   * - **ruleUuid**
     - rule uuid
     -
     -
     - 0.6
   * - **vmNicUuid**
     - VM nic uuid
     -
     -
     - 0.6

Detach Port Forwarding Rule
===========================

Users can use DetachPortForwardingRule to detach a rule from a VM nic. For example::

    DetachPortForwardingRule uuid=310a6cd618144ca683d78d74307f16a4

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
     - rule uuid
     -
     -
     - 0.6

Query Port Forwarding Rule
==========================

Users can use QueryPortForwardingRule to query rules. For example::

    QueryPortForwardingRule vipPortStart=22 vipIp=17.200.20.6

::

    QueryPortForwardingRule vmNic.l3Network.name=database-tier


Primitive Fields
++++++++++++++++

see :ref:`port forwarding rule inventory <port forwarding rule inventory>`

Nested And Expanded Fields
++++++++++++++++++++++++++

.. list-table::
   :widths: 20 30 40 10
   :header-rows: 1

   * - Field
     - Inventory
     - Description
     - Since
   * - **vip**
     - :ref:`VIP inventory <vip inventory>`
     - VIP this rule is bound
     - 0.6
   * - **vmNic**
     - :ref:`VM nic inventory <vm nic inventory>`
     - VM nic this rule is attached
     - 0.6

---------------------
Global Configurations
---------------------

.. _portForwarding.snatInboundTraffic:

snatInboundTraffic
==================

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **snatInboundTraffic**
     - portForwarding
     - false
     - - true
       - false

Whether to source NAT inbound traffic of a port forwarding rule. If true, the traffics reaching portForwardingRule.guestIp will have a source IP equal to portForwardingRule.vipIp; this is
useful when a VM has multiple port forwarding rules attached; it forces a VM to reply incoming traffics through VIPs where traffics come from, rather than replying
through the default route.

----
Tags
----

Users can create user tags on a port forwarding rule with resourceType=PortForwardingRuleVO. For example::

    CreateUserTag resourceType=PortForwardingRuleVO tag=ssh-rule resourceType=e960a93b7f974690bb779808f3c12a33
