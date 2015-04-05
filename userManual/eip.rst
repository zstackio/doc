.. _eip:

==================
Elastic IP Address
==================

.. contents:: `Table of contents`
   :depth: 6

--------
Overview
--------

An elastic IP(EIP) provides a way that allows outside network to reach a L3 network behind
a source nat. EIP is based on network address translation(NAT) that maps an IP address of one network(usually a public network)
to an IP address of another network(usually a private network); as being called elastic IP address, an EIP can be attached/detached
to/from VMs dynamically.

.. image:: eip1.png
   :align: center

.. _eip inventory:

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
   * - **vmNicUuid**
     - uuid of VM nic the EIP is bound
     - true
     -
     - 0.6
   * - **vipUuid**
     - VIP uuid
     -
     -
     - 0.6
   * - **state**
     - EIP state, not implemented in this version
     -
     - - Enabled
       - Disabled
     - 0.6
   * - **vipIp**
     - VIP IP address
     -
     -
     - 0.6
   * - **guestIp**
     - IP of VM nic
     - true
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
            "createDate": "Nov 28, 2015 6:52:14 PM",
            "guestIp": "10.0.0.170",
            "lastOpDate": "Nov 28, 2015 6:52:14 PM",
            "name": "eip-vlan10",
            "state": "Enabled",
            "uuid": "76b9231c94cd4a3aac497200bb26a643",
            "vipIp": "192.168.0.189",
            "vipUuid": "429106d5a63a4995911c2c5f14299b85",
            "vmNicUuid": "70cac1fd0c2f4940ba32645e09d3e22f"
        }

----------
Operations
----------

Create EIP
==========

Users can use CreateEip to create an EIP. For example::

      CreateEip name=eip1 vipUuid=429106d5a63a4995911c2c5f14299b85 vmNicUuid=70cac1fd0c2f4940ba32645e09d3e22f

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
     - VIP uuid
     -
     -
     - 0.6
   * - **vmNicUuid**
     - VM nic uuid; if omitted, the EIP is created without attaching to any VM nic.
     - true
     -
     - 0.6

Delete EIP
==========

Users can use DeleteEip to delete an EIP. For example::

    DeleteEip uuid=76b9231c94cd4a3aac497200bb26a643

After deleting, the VIP to which this EIP bound is recycled so other network services can reuse it.

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
     - EIP uuid
     -
     -
     - 0.6

Attach EIP
==========

Users can use AttachEip to attach an EIP to a VM nic. For example::

    AttachEip eipUuid=76b9231c94cd4a3aac497200bb26a643 vmNicUuid=70cac1fd0c2f4940ba32645e09d3e22f


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
   * - **eipUuid**
     - EIP uuid
     -
     -
     - 0.6
   * - **vmNicUuid**
     - VM nic uuid
     -
     -
     - 0.6


Detach EIP
==========

Users can use DetachEip to detach an EIP from the VM nic. For example::

    DetachEip uuid=76b9231c94cd4a3aac497200bb26a643


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
     - EIP uuid
     -
     -
     - 0.6

Query EIP
=========

Users can use QueryEip to query EIPs. For example::

    QueryEip vipIp=191.13.10.2

::

    QueryEip vmNic.vmInstance.state=Running


Primitive Fields
++++++++++++++++

see :ref:`EIP inventory <eip inventory>`

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
     - VIP this EIP is bound
     - 0.6
   * - **vmNic**
     - :ref:`VM nic inventory <vm nic inventory>`
     - VM nic is EIP is attached
     - 0.6

---------------------
Global Configurations
---------------------

.. _eip.snatInboundTraffic:

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
     - eip
     - false
     - - true
       - false

Whether to source NAT inbound traffics of an EIP. If true, the traffics reaching eip.guestIp will have a source IP equal to eip.vipIp; this is
useful when a VM has multiple EIP attached; it forces a VM to reply incoming traffic through the EIP where the traffic comes from, rather than replying
through the default route.

----
Tags
----

Users can create user tags on an EIP with resourceType=EipVO. For example::

    CreateUserTag resourceType=EipVO tag=web-public-ip resourceUuid=29fa6c2830c441aaa388d8165b80c24c
