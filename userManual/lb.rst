.. _lb:

=====================
Elastic Load Balancer
=====================

.. contents:: `Table of contents`
   :depth: 6

--------
Overview
--------

Elastic Load Balancing automatically distributes your incoming application traffic across multiple VM instances.
It detects unhealthy instances and reroutes traffic to healthy instances until the unhealthy instances have been restored.

.. image:: lb1.png
   :align: center

-------------
Load Balancer
-------------

A load balancer consists of a :ref:`VIP <vip>` to which incoming traffics visit, a set of listeners
that defines a variety of properties such as load balancer port, instance port, health-check configurations, and a group
of VM nics where the incoming traffics will be routed.

.. _load balancer inventory:

Inventory
=========

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
     - 0.9
   * - **name**
     - see :ref:`resource properties`
     -
     -
     - 0.9
   * - **description**
     - see :ref:`resource properties`
     - true
     -
     - 0.9
   * - **state**
     - reserved in 0.9 version, always Enabled
     -
     - - Enabled
       - Disabled
     - 0.9
   * - **vipUuid**
     - uuid of :ref:`VIP <vip>`
     -
     -
     - 0.9
   * - **listeners**
     - a list of :ref:`listener <load balancer listener>`
     -
     -
     - 0.9
   * - **createDate**
     - see :ref:`resource properties`
     -
     -
     - 0.9
   * - **lastOpDate**
     - see :ref:`resource properties`
     -
     -
     - 0.9

Example
=======

::

        {
            "listeners": [
                {
                    "createDate": "Aug 20, 2015 2:54:14 PM",
                    "instancePort": 80,
                    "lastOpDate": "Aug 20, 2015 2:54:14 PM",
                    "loadBalancerPort": 80,
                    "loadBalancerUuid": "0188cec6635845e0b2526a8e7e090e2a",
                    "name": "80",
                    "protocol": "http",
                    "uuid": "ba5f192472ab4fc4b36e5af873f0fec5",
                    "vmNicRefs": [
                        {
                            "createDate": "Aug 20, 2015 2:55:49 PM",
                            "id": 18,
                            "lastOpDate": "Aug 20, 2015 2:55:49 PM",
                            "listenerUuid": "ba5f192472ab4fc4b36e5af873f0fec5",
                            "status": "Active",
                            "vmNicUuid": "35b8aadef2f847d9836bdf06121e1c29"
                        },
                        {
                            "createDate": "Aug 20, 2015 2:55:49 PM",
                            "id": 19,
                            "lastOpDate": "Aug 20, 2015 2:55:49 PM",
                            "listenerUuid": "ba5f192472ab4fc4b36e5af873f0fec5",
                            "status": "Active",
                            "vmNicUuid": "df7d40a47cb640a9b40001f2f318989a"
                        }
                    ]
                },
                {
                    "createDate": "Aug 20, 2015 5:29:39 AM",
                    "instancePort": 22,
                    "lastOpDate": "Aug 20, 2015 5:29:39 AM",
                    "loadBalancerPort": 22,
                    "loadBalancerUuid": "0188cec6635845e0b2526a8e7e090e2a",
                    "name": "ssh",
                    "protocol": "tcp",
                    "uuid": "2901fd13765c492b9a3d004e806a0beb",
                    "vmNicRefs": [
                        {
                            "createDate": "Aug 20, 2015 5:30:07 AM",
                            "id": 15,
                            "lastOpDate": "Aug 20, 2015 5:30:07 AM",
                            "listenerUuid": "2901fd13765c492b9a3d004e806a0beb",
                            "status": "Active",
                            "vmNicUuid": "35b8aadef2f847d9836bdf06121e1c29"
                        },
                        {
                            "createDate": "Aug 20, 2015 5:30:07 AM",
                            "id": 16,
                            "lastOpDate": "Aug 20, 2015 5:30:07 AM",
                            "listenerUuid": "2901fd13765c492b9a3d004e806a0beb",
                            "status": "Active",
                            "vmNicUuid": "df7d40a47cb640a9b40001f2f318989a"
                        }
                    ]
                }
            ],
            "name": "lb",
            "state": "Enabled",
            "uuid": "0188cec6635845e0b2526a8e7e090e2a",
            "vipUuid": "df6a73601f1741fd847cf5456b0d42ac"
        }

.. _load balancer listener:

--------
Listener
--------

A listener defines how the load balancer routes incoming traffics from a VIP port(called loadBalancer port) to a
backend port(called instancePort) of VM instances, and a set of properties that how the load balancer should handle
stuff like connection timeout, health-check threshold.

From users' perspective, they create a listener whenever they want to load balance traffics from a frontend
port(loadBalancerPort) on the load balancer to a backend port(instancePort) of VM instances running on a private network.

A load balancer can have many listeners each of which defines a mapping between a load balancer port and an instance port.

A variety of properties used to control behaviors of listeners are defined as system tags, including idle connection timeout,
max connections, healthy threshold, unhealthy threshold and so on. Details can be found in :ref:`load balancer system tags <load balancer system tags>`.

.. _load balancer listener inventory:

Inventory
=========

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
     - 0.9
   * - **name**
     - see :ref:`resource properties`
     -
     -
     - 0.9
   * - **description**
     - see :ref:`resource properties`
     - true
     -
     - 0.9
   * - **loadBalancerUuid**
     - load balancer uuid
     -
     -
     - 0.9
   * - **loadBalancerPort**
     - the frontend port where the incoming traffics visit; it's bond to
       the VIP of the load balancer
     -
     - 1 ~ 65536
     - 0.9
   * - **instancePort**
     - the backend port where the incoming traffics are routed; it's bound to
       VM nics on the private network
     -
     - 1 ~ 65336
     - 0.9
   * - **protocol**
     - see :ref:`protocol <load balancer protocol>`
     -
     - - http
       - tcp
     - 0.9
   * - **vmNicRefs**
     - see :ref:`nic reference <listener nic reference>`
     -
     -
     - 0.9
   * - **createDate**
     - see :ref:`resource properties`
     -
     -
     - 0.9
   * - **lastOpDate**
     - see :ref:`resource properties`
     -
     -
     - 0.9

.. _load balancer protocol:

Protocol
========

The protocol defines how the load balancer should route incoming traffic. There are two modes: tcp(layer 4) and http(layer 7). When the protocol
is *tcp* which is the default mode, the load balancer will work in pure TCP mode; a full-duplex connection will be established between clients and servers.
When the protocol is *http*, connections from clients to the load balancer and from the load balancer to your back-end instance are established respectively,

Example
=======

::

    {
        "createDate": "Aug 20, 2015 2:54:14 PM",
        "instancePort": 80,
        "lastOpDate": "Aug 20, 2015 2:54:14 PM",
        "loadBalancerPort": 80,
        "loadBalancerUuid": "0188cec6635845e0b2526a8e7e090e2a",
        "name": "80",
        "protocol": "http",
        "uuid": "ba5f192472ab4fc4b36e5af873f0fec5",
        "vmNicRefs": [
            {
                "createDate": "Aug 20, 2015 2:55:49 PM",
                "id": 18,
                "lastOpDate": "Aug 20, 2015 2:55:49 PM",
                "listenerUuid": "ba5f192472ab4fc4b36e5af873f0fec5",
                "status": "Active",
                "vmNicUuid": "35b8aadef2f847d9836bdf06121e1c29"
            },
            {
                "createDate": "Aug 20, 2015 2:55:49 PM",
                "id": 19,
                "lastOpDate": "Aug 20, 2015 2:55:49 PM",
                "listenerUuid": "ba5f192472ab4fc4b36e5af873f0fec5",
                "status": "Active",
                "vmNicUuid": "df7d40a47cb640a9b40001f2f318989a"
            }
        ]
    },


---------------
Backend VM Nics
---------------

Users can add a VM instance to a load balancer by joining its nic to the load balancer's listeners. Once the nic joined, the load balancer
routes incoming traffics from the *loadBalancerPort* of the VIP to the *instancePort* of the nic according listeners' balancing
algorithm. A nic can join different listeners of different load balancers; it's applications' responsibilities to handle traffics
from various load balancers.

The load balancer listener encompasses information of joined VM nics into an inventory called *nic reference*, which has properties
as following:

.. _listener nic reference:

Nic Reference Inventory
=======================

.. list-table::
   :widths: 20 40 10 20 10
   :header-rows: 1

   * - Name
     - Description
     - Optional
     - Choices
     - Since
   * - **id**
     - id of the reference
     -
     -
     - 0.9
   * - **listenerUuid**
     - listener uuid
     -
     -
     - 0.9
   * - **vmNicUuid**
     - VM nic uuid
     -
     -
     - 0.9
   * - **status**
     - when the nic's owner VM is running, the status is active; otherwise it's inactive
     -
     - - Active
       - Inactive
     - 0.9


After a VM nic joins a load balancer listener, stopping the VM will change the nic status to *Inactive*; starting the
VM will change the nic status to *Active*; Destroying the VM will remove the nic from the listener.

--------------
A Full Example
--------------

Let's say you are about to create a load balancer which routes incoming traffics from port 80 and 22 on the public VIP to two
backend VMs.

.. image:: lb2.png
   :align: center


.. list-table::
   :widths: 50 50
   :header-rows: 1

   * - **Public L3 Network UUID**
     - see :ref:`resource properties`
   * - **VM1 nic UUId**
     - 35b8aadef2f847d9836bdf06121e1c29
   * - **VM2 nic UUID**
     - df7d40a47cb640a9b40001f2f318989a


**Create a VIP**

::
    >>>CreateVip l3NetworkUuid=db6379182e524c06bc8d3ec900ab78d4

**Create LB**

::
    >>>CreateLoadBalancer name=lb vipUuid=df6a73601f1741fd847cf5456b0d42ac

**Create listeners**

::

    CreateLoadBalancerListener loadBalancerUuid=0188cec6635845e0b2526a8e7e090e2a loadBalancerPort=22 instancePort=22 name=ssh protocol=tcp

::

    CreateLoadBalancerListener loadBalancerUuid=0188cec6635845e0b2526a8e7e090e2a loadBalancerPort=80 instancePort=80 name=80 protocol=http

**Add nics to listeners**

::

    >>>AddVmNicToLoadBalancer listenerUuid=2901fd13765c492b9a3d004e806a0beb vmNicUuids=35b8aadef2f847d9836bdf06121e1c29,df7d40a47cb640a9b40001f2f318989a

::

    >>>AddVmNicToLoadBalancer listenerUuid=4be2244667d948e286722a4a32e02e65 vmNicUuids=35b8aadef2f847d9836bdf06121e1c29,df7d40a47cb640a9b40001f2f318989a


----------
Operations
----------

Create Load Balancer
====================

Users can use CreateLoadBalancer to create a load balancer. For example::

    >>>CreateLoadBalancer name=lb vipUuid=df6a73601f1741fd847cf5456b0d42ac

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
     - 0.9
   * - **resourceUuid**
     - resource uuid, see :ref:`create resource`
     - true
     -
     - 0.9
   * - **description**
     - resource description, see :ref:`resource properties`
     - true
     -
     - 0.9
   * - **vipUuid**
     - VIP uuid
     -
     -
     - 0.9
   * - **userTags**
     - user tags, see :ref:`create tags`; resource type is
     - true
     -
     - 0.9
   * - **systemTags**
     - system tags, see :ref:`create tags`; resource type is
     - true
     -
     - 0.9


Delete Load Balancer
====================

Users can use DeleteLoadBalancer to delete a load balancer. For example::

    >>>DeleteLoadBalancer uuid=4be2244667d948e286722a4a32e02e65


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
   * - **deleteMode**
     - see :ref:`delete resource`
     - true
     - - Permissive
       - Enforcing
     - 0.9
   * - **uuid**
     - load balancer uuid
     -
     -
     - 0.9

Create Listener
===============

Users can use CreateLoadBalancerListener to create a load balancer listener. For example::

    CreateLoadBalancerListener loadBalancerUuid=0188cec6635845e0b2526a8e7e090e2a loadBalancerPort=22 instancePort=22 name=ssh protocol=tcp


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
     - 0.9
   * - **resourceUuid**
     - resource uuid, see :ref:`create resource`
     - true
     -
     - 0.9
   * - **description**
     - resource description, see :ref:`resource properties`
     - true
     -
     - 0.9
   * - **loadBalancerUuid**
     - load balancer uuid
     -
     -
     - 0.9
   * - **loadBalancerPort**
     - frontend load balancer port
     -
     -
     - 0.9
   * - **instancePort**
     - backend instance port. If omitted, use loadBalancerPort as instancePort
     - true
     -
     - 0.9
   * - **protocol**
     - see :ref:`load balancer protocol <load balancer protocol>`
     -
     - - tcp
       - http
     - 0.9
   * - **userTags**
     - user tags, see :ref:`create tags`; resource type is
     - true
     -
     - 0.9
   * - **systemTags**
     - system tags, see :ref:`create tags`; resource type is
     - true
     -
     - 0.9

Delete Listener
===============

Users can use DeleteLoadBalancerListener to delete a listener. For example::

    >>DeleteLoadBalancerListener uuid=0188cec6635845e0b2526a8e7e090e2a

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
   * - **deleteMode**
     - see :ref:`delete resource`
     - true
     - - Permissive
       - Enforcing
     - 0.9
   * - **uuid**
     - listener uuid
     -
     -
     - 0.9

Add VM Nic to Load Balancer
===========================

Users can use AddVmNicToLoadBalancer to add VM nics to a load balancer. For example::

     >>>AddVmNicToLoadBalancer listenerUuid=2901fd13765c492b9a3d004e806a0beb vmNicUuids=35b8aadef2f847d9836bdf06121e1c29,df7d40a47cb640a9b40001f2f318989a

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
   * - **listenerUuid**
     - listener uuid
     -
     -
     - 0.9
   * - **vmNicUuids**
     - a list of VM nic uuid
     -
     -
     - 0.9

Remove VM Nic from Load Balancer
================================

Users can use RemoveVmNicFromLoadBalancer to remove VM nics from a load balancer. For example::

     >>>RemoveVmNicFromLoadBalancer listenerUuid=2901fd13765c492b9a3d004e806a0beb vmNicUuids=35b8aadef2f847d9836bdf06121e1c29,df7d40a47cb640a9b40001f2f318989a

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
   * - **listenerUuid**
     - listener uuid
     -
     -
     - 0.9
   * - **vmNicUuids**
     - a list of VM nic uuid
     -
     -
     - 0.9

Query Load Balancer
===================

Users can use QueryLoadBalancer to query load balancers. For example::

    >>>QueryLoadBalancer name=lb

::

    >>>QueryLoadBalancer listeners.vmNic.vmInstance.name=web

Primitive Fields of Query
-------------------------

see :ref:`load balancer inventory <load balancer inventory>`

Nested And Expanded Fields of Query
-----------------------------------

.. list-table::
   :widths: 20 30 40 10
   :header-rows: 1

   * - Field
     - Inventory
     - Description
     - Since
   * - **listeners**
     - see :ref:`load balancer listener inventory <load balancer listener inventory>`
     - child listeners
     - 0.9
   * - **vip**
     - see :ref:`vip inventory <vip inventory>`
     - bound VIP
     - 0.9

Query Listener
==============

Users can use QueryLoadBalancerListener to query load balancer listeners. For example::

    >>>QueryLoadBalancerListener loadBalancerPort=80

::

    >>>QueryLoadBalancerListener loadBalancer.vip.ip=192.168.0.10

Primitive Fields of Query
-------------------------

see :ref:`load balancer listener inventory <load balancer listener inventory>`

Nested And Expanded Fields of Query
-----------------------------------

.. list-table::
   :widths: 20 30 40 10
   :header-rows: 1

   * - Field
     - Inventory
     - Description
     - Since
   * - **loadBalancer**
     - see :ref:`load balancer inventory <load balancer inventory>`
     - parent load balancer
     - 0.9
   * - **vmNic**
     - see :ref:`vm nic inventory <vm nic inventory>`
     - joined VM nics
     - 0.9

----
Tags
----

Users can create user tags on a load balancer with resourceType=LoadBalancerVO. For example::

    CreateUserTag tag=web-lb resourceUuid=0a9f95a659444848846b5118e15bff32 resourceType=LoadBalancerVO

Users can create user tags on a load balancer listener with resourceType=LoadBalancerListenerVO. For example::

    CreateUserTag tag=web-lb-80 resourceUuid=0a9f95a659444848846b5118e15bff32 resourceType=LoadBalancerListenerVO

.. _load balancer system tags:

System Tags
===========

Separate Virtual Router
-----------------------

In this version(0.9), the load balancer service is provided by the virtual router provider. Normally users may need only
one virtual router VM providing services like SNAT, EIP, port forwarding and load balancer. However, users can use a system
tag to instruct ZStack to spawn an individual virtual router VM for a load balancer. That is to say, creating a virtual router
VM dedicated to a load balancer.

.. list-table::
   :widths: 20 60 20
   :header-rows: 1

   * - Tag
     - Example
     - Since
   * - **separateVirtualRouterVm**
     - separateVirtualRouterVm
     - 0.9


::

    >>>CreateLoadBalancer name=lb vipUuid=df6a73601f1741fd847cf5456b0d42ac systemTags=separateVirtualRouterVm


Listener Configurations
-----------------------

A set of system tags can be used to configure a load balancer listener, controlling various listener behaviors such as
max connections, idle connection timeout, balancing algorithm and so on. Users can specify those system tags when creating
a listener, or ignore them to let ZStack choose default values.

.. _healthyThreshold:

Healthy Threshold
+++++++++++++++++

The number of consecutive health checks successes required before moving the VM nic to the healthy state.

.. list-table::
   :widths: 20 60 20
   :header-rows: 1

   * - Tag
     - Example
     - Since
   * - **healthyThreshold::{healthyThreshold}**
     - healthyThreshold::2
     - 0.9

.. _healthCheckInterval:

Health Check Interval
+++++++++++++++++++++

The approximate interval, in seconds, between health checks of an individual VM nic

.. list-table::
   :widths: 20 60 20
   :header-rows: 1

   * - Tag
     - Example
     - Since
   * - **healthCheckInterval::{healthCheckInterval}**
     - healthCheckInterval::5
     - 0.9

.. _unhealthyThreshold:

Unhealthy Threshold
+++++++++++++++++++

The number of consecutive health check failures required before moving the instance to the unhealthy state.

.. list-table::
   :widths: 20 60 20
   :header-rows: 1

   * - Tag
     - Example
     - Since
   * - **unhealthyThreshold::{unhealthyThreshold}**
     - unhealthyThreshold::2
     - 0.9

.. _connectionIdleTimeout:

Connection Idle Timeout
+++++++++++++++++++++++

The amount of time, in seconds, during the load balancer closes idle connections on both server and client side.

.. list-table::
   :widths: 20 60 20
   :header-rows: 1

   * - Tag
     - Example
     - Since
   * - **connectionIdleTimeout::{connectionIdleTimeout}**
     - 60
     - 0.9

.. _maxConnection:

Max Connections
+++++++++++++++

The max concurrent connections

.. list-table::
   :widths: 20 60 20
   :header-rows: 1

   * - Tag
     - Example
     - Since
   * - **maxConnection::{maxConnection}**
     - maxConnection::5000
     - 0.9

.. _balancerAlgorithm:

Balancing Algorithm
+++++++++++++++++++

The algorithm the load balancer routes incoming traffic; valid choices are: roundrobin, leastconn, source

.. list-table::
   :widths: 20 60 20
   :header-rows: 1

   * - Tag
     - Example
     - Since
   * - **balancerAlgorithm::{balancerAlgorithm}**
     - balancerAlgorithm::leastconn
     - 0.9

::

    CreateLoadBalancerListener loadBalancerUuid=0188cec6635845e0b2526a8e7e090e2a loadBalancerPort=22 instancePort=22 name=ssh protocol=tcp
    systemTags=maxConnection::10000,balancerAlgorithm::source,healthyThreshold::5


---------------------
Global Configurations
---------------------

Connection Idle Timeout
=======================

The default value of system tag :ref:`Connection Idle Timeout <connectionIdleTimeout>`.

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **connectionIdleTimeout**
     - loadBalancer
     - 60
     -

Healthy Threshold
=================

The default value of system tag :ref:`Healthy Threshold <healthyThreshold>`.

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **healthyThreshold**
     - loadBalancer
     - 2
     -

Unhealthy Threshold
===================

The default value of system tag :ref:`Unhealthy Threshold <unhealthyThreshold>`.

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **unhealthyThreshold**
     - loadBalancer
     - 2
     -

Health Check Interval
=====================

The default value of system tag :ref:`Health Check Interval <healthCheckInterval>`.

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **healthCheckInterval**
     - loadBalancer
     - 5
     -

Max Connection
==============

The default value of system tag :ref:`Max Connection <maxConnection>`.

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **maxConnection**
     - loadBalancer
     - 5000
     -

Balancing Algorithm
===================

The default value of system tag :ref:`Balancing Algorithm <balancerAlgorithm>`.

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **balancerAlgorithm**
     - loadBalancer
     - roundrobin
     - - roundrobin
       - leastconn
       - source

