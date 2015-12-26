.. _host:

====
Host
====

.. contents:: `Table of contents`
   :depth: 6

--------
Overview
--------

A host is a physical server installed with an operating system(hypervisor).

.. image:: host.png
   :align: center

In ZStack, a host is the smallest unit providing computing resources that run VMs. Zones and clusters, which usually contain grouped
hosts, are bigger units. Unlike its parent and ancestor both of which are logical resources, a host is a physical resource; many
operations, which are seemingly applied to zones or clusters, are actually delegated to hosts. For example, when attaching
a primary storage to a cluster, the real action performed might be mounting the primary storage on every host in
the cluster.

.. note:: In this ZStack version, KVM is the only supported host

.. _host inventory:

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
     - uuid of ancestor zone. see :ref:`zone <zone>`
     -
     -
     - 0.6
   * - **clusterUuid**
     - uuid of parent cluster. see :ref:`cluster <cluster>`
     -
     -
     - 0.6
   * - **managementIp**
     - see :ref:`management ip <host management ip>`
     -
     -
     - 0.6
   * - **hypervisorType**
     - see :ref:`cluster hypervisor type <cluster hypervisor type>`
     -
     -
     - 0.6
   * - **state**
     - see :ref:`state <host state>`
     -
     - - Enabled
       - Disabled
       - PreMaintenance
       - Maintenance
     - 0.6
   * - **status**
     - see :ref:`status <host status>`
     -
     - - Connecting
       - Connected
       - Disconnected
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
        "zoneUuid": "2893ce85c43d4a3a8d78f414da39966e",
        "name": "host1-192.168.0.203",
        "uuid": "43673938584447b2a29ab3d53f9d88d3",
        "clusterUuid": "8524072a4274403892bcc5b1972c2576",
        "description": "Test",
        "managementIp": "192.168.0.203",
        "hypervisorType": "KVM",
        "state": "Enabled",
        "status": "Connected",
        "createDate": "Jun 1, 2015 6:49:24 PM",
        "lastOpDate": "Jun 1, 2015 6:49:24 PM"
      }
    }

.. _host management ip:

Management IP
=============

The management IP is used by ZStack management nodes to reach the operating systems(hypervisor) of hosts; depending on hypervisor types,
it's necessary or not. For example, in VMWare, the official way to reach an ESXi host is through the VCenter Server, then the
management IP is not necessary; however, in KVM, ZStack will deploy an agent to the Linux operating system, then the management IP is necessary.

.. note:: A management IP can be either an IP address or a DNS name, as long as the DNS name can be resolved by the operating systems on which ZStack management
          nodes run.

.. note:: In this ZStack version, as KVM is the only supported host, the management ip is a mandatory field.

Management Network
++++++++++++++++++

Though it's not enforced, it is recommended to have one or more dedicated subnets used as management networks. The Linux servers that run ZStack
management nodes must be able to reach management networks, because management nodes need to send commands to hosts and other appliances on the management
networks. In future chapters, we will see management network again when talking about appliance VMs, which are specific to :ref:`virtual router <virtual router>`
in this ZStack version.

.. warning:: Specific to KVM, it's recommended to make all management IPs of hosts in the same zone be inter-reachable. In this ZStack version, there are
             no dedicated networks for VM migration; ZStack essentially uses management IPs to transfer data amid hosts during VM migrations.
             If hosts can not reach each other by management IPs, even they can be reached by ZStack management nodes, VM migrations among them
             will still fail.

.. _host state:

State
=====

Hosts have four states:

- **Enabled**:

  the state that allows VMs to be created, started, or migrated to

- **Disabled**:

  the state that DOESN'T allow VMs to be created, started, or migrated to

- **PreMaintenance**:

  the intermediate state indicating host is entering Maintenance state. See :ref:`maintenance mode <host maintenance mode>`.

- **Maintenance**:

  the state indicating host has been in maintenance mode.

A state transition diagram is like:

.. image:: host-state.png
   :align: center

.. _host maintenance mode:

Maintenance Mode
++++++++++++++++

A host can be placed in maintenance mode when admins need to carry out maintenance work, for example, to install more memory.
When a host is in the maintenance mode, neither API operations nor ZStack internal tasks can be performed to it. That is to say, tasks
like starting VMs(API), stopping VMs(API), mounting primary storage(internal) cannot be performed.
ZStack defines maintenance mode in two states: PreMaintenance and Maintenance. The sequence a host enters maintenance mode is shown as follows:

1. Changing the host's state to PreMaintenance. At this phase, ZStack will try to migrate all VMs running on the host to other appropriate hosts.
   If migrations fail, ZStack will stop those VMs.


2. After VMs are properly migrated or stopped, ZStack will change the host's state to Maintenance. Since now, admins can do
   maintenance work to the host.

Admins can take a host out of maintenance mode by placing it in Enabled or Disabled state, after maintenance work is done.

.. note:: When a host is in maintenance mode, admins can still attach primary storage or L2 networks to its parent cluster. Once the host quits
          maintenance mode, ZStack will send a reconnect message which will instruct the host to catch up work missed during it was in the maintenance mode; for
          example, mounting a NFS primary storage.

.. _host status:

Status
======

A host's status reflects the status of command channel between the host and a ZStack management node. Command channels are the ways that
ZStack management nodes communicate with hosts to perform operations. For example, in KVM, command channels are the HTTP connections
between ZStack management nodes and Python agents running on hosts; in VMWare, command channels are connections between the VCenter Server
and ESXi hosts.

Hosts have three status:

- **Connecting**:

  A ZStack management node is trying to establish the command channel between itself and the host. No operations can be performed to the host.

- **Connected**

  The Command channel has been successfully established between a ZStack management node and the host. Operations can be performed to the host.
  This is the only status that a host can start or create VMs.

- **Disconnected**

  The Command channel has lost between a ZStack management node and the host. No operations can be performed to the host.

When booting, a ZStack management node will start the process of establishing the command channel to hosts it manages; in this stage, hosts's status are
Connecting; after command channels are established, hosts' status change to Connected; if the management node fails to setup a command channel,
or the command channel is detected as lost later on, the status of the host to which the command channel connect changes to Disconnected.

ZStack management nodes will periodically send ping commands to hosts to check health of command channels; once a host fails to respond, or a ping
command times out, the host's status changes to Disconnected.

.. note:: ZStack will keep sending ping commands to a disconnected host. Once the host recovers and responds to the ping command, ZStack will reestablish
          the command channel and alter the host to Connected. So when a host is physically removed from a cloud, please remember to delete it
          from ZStack, otherwise ZStack management nodes will keep pinging it.

.. note:: No ping command will be sent if a host is in maintenance mode.

A status transition diagram is like:

.. image:: host-status.png
   :align: center

State and Status
================

There are no direct relations between states and status. States represent admin's decisions to a host, while status represents communication condition of a host.

----------
Operations
----------

Add Host
========

The commands adding a host varies for different hypervisors.

Add KVM Host
++++++++++++

Admins can use AddKVMHost to add a KVM host. For example::

    AddKVMHost clusterUuid=8524072a4274403892bcc5b1972c2576 managementIp=192.168.10.10 name=kvm1 username=root password=password

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
   * - **clusterUuid**
     - uuid of parent cluster, see :ref:`cluster <cluster>`
     -
     -
     - 0.6
   * - **managementIp**
     - see :ref:`management ip <host management ip>`
     -
     -
     - 0.6
   * - **username**
     - see :ref:`kvm credentials <kvm credentials>`
     -
     -
     - 0.6
   * - **password**
     - see :ref:`kvm credentials <kvm credentials>`
     -
     -
     - 0.6


.. _kvm credentials:

KVM Credentials
---------------

ZStack uses a Python agent called kvmagent to manage KVM hosts. To make things full automation,
ZStack utilizes `Ansible <http://www.ansible.com/home>`_ to configure target Linux operating systems and deploy kvmagents; and to bootstrap Ansible on
target Linux operating systems, ZStack needs SSH username/password of **root** user to inject SSH public keys in KVM hosts in order
to make Ansible work without prompting username/password. The **root** privilege is required as both Ansible and kvmagent need full
control of the system.

Delete Host
===========

Admins can use DeleteHost command to delete a host. For example::

    DeleteHost uuid=2893ce85c43d4a3a8d78f414da39966e

.. danger:: Deleting hosts will stop all VMs on the host. There is no way to recover a deleted host.

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
     - host uuid
     -
     -
     - 0.6

Change Host State
=================

Admins can use ChangeHostState command to change a host's state. For example::

    ChangeHostState stateEvent=preMaintain uuid=2893ce85c43d4a3a8d78f414da39966e

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
     - host uuid
     -
     -
     - 0.6
   * - **stateEvent**
     - state trigger event. See :ref:`state <host state>`

       .. note:: The state trigger event 'maintain' shown in :ref:`state <host state>` section is
                 used internally and is not available in the API.

     -
     - - enable
       - disable
       - preMaintain
     - 0.6

Reconnect Host
==============

Admins can use ReconnectHost to re-establish the command channel between a ZStack management node and a host. For example::

    ReconnectHost uuid=2893ce85c43d4a3a8d78f414da39966e

See :ref:`status <host status>` for details.

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
     - host uuid
     -
     -
     - 0.6

Query Host
==========

Admins can use QueryHost to query hosts. For example::

    QueryHost managementIp=192.168.0.100

::

    QueryHost vmInstance.vmNics.ip=10.21.100.2


Primitive Fields of Query
+++++++++++++++++++++++++

see :ref:`host inventory <host inventory>`


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
     - :ref:`zone inventory <zone inventory>`
     - ancestor zone
     - 0.6
   * - **cluster**
     - :ref:`cluster inventory <cluster inventory>`
     - parent cluster
     - 0.6
   * - **vmInstance**
     - :ref:`VM inventory <vm inventory>`
     - VMs running on this host
     - 0.6


---------------------
Global Configurations
---------------------

.. _load.all:

load.all
========

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **load.all**
     - host
     - true
     - - true
       - false

Whether to connect all hosts when management nodes boot. If set to true, management nodes will connect to all hosts simultaneously
during booting time, which may exhaust resources of the machines running management nodes if there are a large number of hosts
in the cloud; if set to false, accompanying with :ref:`load.parallelismDegree <load.parallelismDegree>`, management nodes will
connect a portion of hosts each time and repeat until all hosts are connected.

.. _load.parallelismDegree:

load.parallelismDegree
======================

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **load.parallelismDegree**
     - host
     - 100
     - > 0

When :ref:`load.all <load.all>` is set to false, this configuration defines the number of hosts that management nodes will
connect simultaneously during booting time.

.. _.host.ping.interval:

ping.timeout
============

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **ping.interval**
     - host
     - 60
     - > 0

The interval that management nodes periodically send ping commands to hosts in order to check connection status, in seconds.

.. _host.ping.parallelismDegree:

ping.parallelismDegree
======================

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **ping.parallelismDegree**
     - host
     - 100
     - > 0

The parallel degree that management nodes send ping commands. If the amount of hosts are larger than this value, management nodes
will repeat until all hosts are pinged. For example, ping first 100 hosts, then ping second 100 hosts ...

.. _connection.autoReconnectOnError:

connection.autoReconnectOnError
===============================

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **connection.autoReconnectOnError**
     - host
     - true
     - - true
       - false

Whether to reconnect hosts when their status change from Connected to Disconnected. If set to true, management nodes will reconnect
hosts whose status change from Connected to Disconnected by ping commands, in order to catch up with operations missed during hosts in
disconnected; if set to false, management nodes will not automatically reconnect them, admins may need to manually do it if necessary.

.. _maintenanceMode.ignoreError:

maintenanceMode.ignoreError
===========================

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **maintenanceMode.ignoreError**
     - host
     - false
     - - true
       - false

Whether to ignore errors happening during hosts enter maintenance mode. If set to true, errors are ignored and hosts always
successfully enter maintenance mode; if set to false, hosts will fail to enter maintenance mode if any error happens, for example,
failing to migrate a VM.

.. _reservedCapacity.zoneLevel:

reservedCapacity.zoneLevel
==========================

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **reservedCapacity.zoneLevel**
     - hostAllocator
     - true
     - - true
       - false

Whether to enable host capacity reservation at zone level; see :ref:`host capacity reservation <host capacity reservation>`.

.. _reservedCapacity.clusterLevel:

reservedCapacity.clusterLevel
=============================

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **reservedCapacity.clusterLevel**
     - hostAllocator
     - true
     - - true
       - false

Whether to enable host capacity reservation at cluster level; see :ref:`host capacity reservation <host capacity reservation>`.

.. _reservedCapacity.hostLevel:

reservedCapacity.hostLevel
==========================

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **reservedCapacity.hostLevel**
     - hostAllocator
     - true
     - - true
       - false

Whether to enable host capacity reservation at host level; see :ref:`host capacity reservation <host capacity reservation>`.

.. _vm.migrationQuantity:

vm.migrationQuantity
====================

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **vm.migrationQuantity**
     - kvm
     - 2
     - > 0

The number that how many VMs can be migrated in parallel when KVM hosts enter maintenance mode.

.. _kvm.reservedMemory:

reservedMemory
==============

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **reservedMemory**
     - kvm
     - 512M
     - >= 0

A string that memory capacity reserved on KVM hosts if :ref:`reservedCapacity.hostLevel <reservedCapacity.hostLevel>` is set to true.
The value is a number followed by a unit character that can be one of B/K/M/G/T; if no unit character followed, the number is
treated as bytes.

.. _dataVolume.maxNum:

dataVolume.maxNum
=================

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **dataVolume.maxNum**
     - kvm
     - 24
     - 0 - 24

The max number of data volumes that can be attached to VMs of hypervisor type -- KVM.

.. _host.syncLevel:

host.syncLevel
==============

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **host.syncLevel**
     - kvm
     - 10
     - > 2

The max number of concurrent commands that can be simultaneously executed on KVM hosts.

----
Tags
----

Admins can create user tags on a host with resourceType=HostVO. For example::

    CreateUserTag tag=largeMemoryHost resourceUuid=0a9f95a659444848846b5118e15bff32 resourceType=HostVO

System Tags
===========

.. _host capacity reservation:

Host Capacity Reservation
+++++++++++++++++++++++++

Admins can use system tags to reserve a portion of memory on hosts for system software. ZStack provides various
system tags and global configurations for fine-grained memory reservation policies:

- **Hypervisor Global Level**:

  The global configuration :ref:`kvm.reservedMemory` applies to all KVM hosts if not overridden by settings of other levels.

- **Zone Level**:

  See :ref:`zone host::reservedMemory <zone.host.reservedMemory>`; the value of this system tag applies to all hosts in the zone if not
  overridden by settings of other levels. This overrides global level.

- **Cluster Level**:

  See :ref:`cluster host::reservedMemory <cluster.host.reservedMemory>`; the value of this system tag applies to all hosts in the cluster
  if not overridden by the setting of host level. This overrides zone level and global level.

- **Host Level**:

  .. list-table::
     :widths: 20 30 40 10
     :header-rows: 1

     * - Tag
       - Description
       - Example
       - Since
     * - **reservedMemory::{capacity}**
       - reserved memory on this host.
       - reservedMemory::1G
       - 0.6

  this overrides all above levels.

For example, assuming you have 3 KVM hosts in zone1->cluster1->{host1, host2, host3}; by default the memory reservation is controlled by the global configuration
:ref:`kvm.reservedMemory` that defaults to 512M; then you create a system tag *host::reservedMemory::1G* on zone1, so memory reservation on all
3 hosts is 1G now; then you create a system tag *host::reservedMemory::2G* on cluster1, memory reservation of 3 hosts changes to 2G; finally, you create a
system tag *reservedMemory::3G* on host1, then memory reservation is 3G on host1 but still 2G on host2 and host3.

.. _host metadata information:

Host Meta Data Information
++++++++++++++++++++++++++

.. list-table::
   :widths: 20 30 40 10
   :header-rows: 1

   * - Tag
     - Description
     - Example
     - Since
   * - **capability:liveSnapshot**
     - if present, the host's hypervisor supports live volume snapshot
     - capability:liveSnapshot
     - 0.6
   * - **os::distribution::{distribution}**
     - OS distribution of the host
     - os::distribution::Ubuntu
     - 0.6
   * - **os::release::{release}**
     - OS release of the host
     - os::release::trusty
     - 0.6
   * - **os::version::{version}**
     - OS version the host
     - os::version::14.04
     - 0.6

KVM Host Meta Data Information
++++++++++++++++++++++++++++++

.. list-table::
   :widths: 20 30 40 10
   :header-rows: 1

   * - Tag
     - Description
     - Example
     - Since
   * - **qemu-img::version::{version}**
     - qemu-img version
     - qemu-img::version::2.0.0
     - 0.6
   * - **libvirt::version::{version}**
     - libvirt version
     - libvirt::version::1.2.2
     - 0.6
   * - **hvm::{flag}**
     - host hardware virtualization flag; vmx means Intel CPU; svm means AMD CPU
     - hvm::vmx
     - 0.6

