.. _vm:

===============
Virtual Machine
===============

.. contents:: `Table of contents`
   :depth: 6

--------
Overview
--------

A virtual machine(VM) consumes datacenter resources of computing, storage, and network.

.. _vm inventory:

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
     - uuid of ancestor zone, see :ref:`zone` and :ref:`location <vm location>`
     - true
     -
     - 0.6
   * - **clusterUuid**
     - uuid of ancestor cluster, see :ref:`cluster` and :ref:`location <vm location>`
     - true
     -
     - 0.6
   * - **hostUuid**
     - uuid of parent host the VM is currently running, see :ref:`host` and :ref:`location <vm location>`
     - true
     -
     - 0.6
   * - **lastHostUuid**
     - uuid of parent host the VM was running last time, see :ref:`host` and :ref:`location <vm location>`
     - true
     -
     - 0.6
   * - **imageUuid**
     - uuid of image from which the VM's root volume is created, see :ref:`image`
     -
     -
     - 0.6
   * - **instanceOfferingUuid**
     - uuid of instance offering, see :ref:`instance offering`
     -
     -
     - 0.6
   * - **rootVolumeUuid**
     - uuid of VM's root volume, see :ref:`volume`
     -
     -
     - 0.6
   * - **defaultL3NetworkUuid**
     - uuid of VM's default L3 network, see :ref:`L3 network<l3Network>` and :ref:`networks <vm networks>`
     -
     -
     - 0.6
   * - **type**
     - VM type

       - UserVm: created by users
       - ApplianceVm: created by ZStack to help manage the cloud
     -
     - - UserVm
       - ApplianceVm
     - 0.6
   * - **hypervisorType**
     - VM's hypervisor type, see :ref:`host` and :ref:`hypervisor type <vm hypervisor type>`
     -
     - - KVM
     - 0.6
   * - **state**
     - VM's state, see :ref:`state <vm state>`
     - - Created
       - Starting
       - Running
       - Stopping
       - Stopped
       - Rebooting
       - Destroying
       - Destroyed
       - Migrating
       - Unknown
     -
     - 0.6
   * - **vmNics**
     - a list of :ref:`nic inventory <vm nic inventory>`, see :ref:`networks <vm networks>`
     -
     -
     - 0.6
   * - **allVolumes**
     - a list of :ref:`volume inventory <volume inventory>`, see :ref:`volumes <vm volumes>`
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
+++++++

::

        {
            "allVolumes": [
                {
                    "createDate": "Dec 2, 2015 5:53:42 PM",
                    "description": "Root volume for VM[uuid:d92a03ed745a0d32fe63dc30051d3862]",
                    "deviceId": 0,
                    "format": "qcow2",
                    "installPath": "/opt/zstack/nfsprimarystorage/prim-a82b75ee064a48708960f42b800bd910/rootVolumes/acct-36c27e8ff05c4780bf6d2fa65700f22e/vol-e9555324042542288ec20a67797d476c/e9555324042542288ec20a67797d476c.qcow2",
                    "lastOpDate": "Dec 2, 2015 5:53:42 PM",
                    "name": "ROOT-for-vm-4-vlan10",
                    "primaryStorageUuid": "a82b75ee064a48708960f42b800bd910",
                    "rootImageUuid": "f1205825ec405cd3f2d259730d47d1d8",
                    "size": 419430400,
                    "state": "Enabled",
                    "status": "Ready",
                    "type": "Root",
                    "uuid": "e9555324042542288ec20a67797d476c",
                    "vmInstanceUuid": "d92a03ed745a0d32fe63dc30051d3862"
                }
            ],
            "clusterUuid": "b429625fe2704a3e94d698ccc0fae4fb",
            "createDate": "Dec 2, 2015 5:53:42 PM",
            "defaultL3NetworkUuid": "6572ce44c3f6422d8063b0fb262cbc62",
            "hostUuid": "d07066c4de02404a948772e131139eb4",
            "hypervisorType": "KVM",
            "imageUuid": "f1205825ec405cd3f2d259730d47d1d8",
            "instanceOfferingUuid": "04b5419ca3134885be90a48e372d3895",
            "lastHostUuid": "d07066c4de02404a948772e131139eb4",
            "lastOpDate": "Dec 2, 2015 5:53:42 PM",
            "name": "vm-4-vlan10",
            "rootVolumeUuid": "e9555324042542288ec20a67797d476c",
            "state": "Running",
            "type": "UserVm",
            "uuid": "d92a03ed745a0d32fe63dc30051d3862",
            "vmNics": [
                {
                    "createDate": "Dec 2, 2015 5:53:42 PM",
                    "deviceId": 0,
                    "gateway": "10.0.0.1",
                    "ip": "10.0.0.218",
                    "l3NetworkUuid": "6572ce44c3f6422d8063b0fb262cbc62",
                    "lastOpDate": "Dec 2, 2015 5:53:42 PM",
                    "mac": "fa:ef:34:5c:6c:00",
                    "netmask": "255.255.255.0",
                    "uuid": "fb8404455cf84111958239a9ec19ca28",
                    "vmInstanceUuid": "d92a03ed745a0d32fe63dc30051d3862"
                }
            ],
            "zoneUuid": "3a3ed8916c5c4d93ae46f8363f080284"
        }

.. _vm location:

Location
++++++++

As ZStack arranges computing resources by zones, clusters, and hosts, a VM's location can be identified by zoneUuid, clusterUuid, and hostUuid.
After a VM is running, those UUIDs will be set to values that represent the VM's current location; after stopped,
the hostUuid is set to NULL, zoneUuid and clusterUuid are unchanged. The lastHostUuid is special, as it represents the host the VM
run last time; for a new created VM, the lastHostUuid is NULL; once the VM is stopped, it's set to the previous value of the hostUuid.

The algorithms of selecting hosts for new created VMs are elaborated in :ref:`host allocator strategy <instance offering allocator strategy>`.
In later of this chapter, strategies for starting VMs and migrating VMs will be demonstrated.

.. _vm networks:

Networks
++++++++

VMs can be on one or more :ref:`L3 networks <l3Network>`; :ref:`vm nics <vm nic inventory>` encompass information
like IP address, netmask, MAC of every L3 network. If a VM has more than one L3 networks, a default L3 network
to provide default routing, DNS, and hostname must be specified; if a VM has only one L3 network,
the one becomes the default L3 network automatically.

An example may help understand what is the default L3 network. Assuming you have a user vm like below picture:

.. image:: vm-networks1.png
   :align: center

The VM is on three L3 networks all providing SNAT service, and the default L3 network is 10.10.1.0/24:

::

    CIDR: 10.10.1.0/24
    Gateway: 10.10.1.1
    DNS domain: web.tier.mycompany.com

then the VM's routing table is like:

::

    default via 10.10.1.1 dev eth0
    10.10.1.0/24 dev eth0  proto kernel  scope link  src 10.10.1.99
    192.168.0.0/24 dev eth1  proto kernel  scope link  src 192.168.0.10
    172.16.0.0/24 dev eth0  proto kernel  scope link  src 172.16.0.55

see the default routing is pointing to **10.10.1.1** that is the gateway of the default L3 network; and the VM's /etc/resolv.conf is like:

::

    search web.tier.mycompany.com
    nameserver 10.10.1.1

the DNS domain is from the default L3 network too; and the DNS name server is also the gateway **10.10.1.1** because the default L3 network
provides the DNS server; at last, the FQDN(Full Qualified Domain Name) of the VM is like:

::

    vm2.web.tier.mycompany.com

which is expanded by the DNS domain.

.. _vm nic inventory:

VM Nic Inventory
----------------

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
   * - **vmInstanceUuid**
     - uuid of parent VM
     -
     -
     - 0.6
   * - **l3NetworkUuid**
     - uuid of :ref:`l3 network <l3Network>` the nic is on
     -
     -
     - 0.6
   * - **ip**
     - IP address
     -
     -
     - 0.6
   * - **mac**
     - MAC address
     -
     -
     - 0.6
   * - **netmask**
     - netmask
     -
     -
     - 0.6
   * - **gateway**
     - gateway
     -
     -
     - 0.6
   * - **metaData**
     - reserved field for internal use
     - true
     -
     - 0.6
   * - **deviceId**
     - an integer that identifies nic's order in guest operating system's ethernet device list. For example, 0 usually means eth0, 1 usually means eth1.
     -
     -
     - 0.6

In this ZStack version, once an IP is assigned to a VM nic, it will be with the nic through the entire life of the VM until
the VM is destroyed.

Example
*******

::

      {
          "createDate": "Dec 2, 2015 5:53:42 PM",
          "deviceId": 0,
          "gateway": "10.0.0.1",
          "ip": "10.0.0.218",
          "l3NetworkUuid": "6572ce44c3f6422d8063b0fb262cbc62",
          "lastOpDate": "Dec 2, 2015 5:53:42 PM",
          "mac": "fa:ef:34:5c:6c:00",
          "netmask": "255.255.255.0",
          "uuid": "fb8404455cf84111958239a9ec19ca28",
          "vmInstanceUuid": "d92a03ed745a0d32fe63dc30051d3862"
      }

.. _vm volumes:

Volumes
+++++++

Field `allVolumes` is a list of :ref:`volume inventory <volume inventory>` that contains the root volume and data volumes. To find out the root volume, users can
iterate the list, either by checking if a volume's type is Root or using the field 'rootVolumeUuid' to match volumes' UUIDs. A root volume will
be with the VM through its entire life until it's destroyed.

.. _vm hypervisor type:

Hypervisor Type
+++++++++++++++

VM's hypervisor type is inherited from image's hypervisor type or host's hypervisor type, depending on how the VM is created.

- **from a RootVolumeTemplate**:

  as the image already has operating system installed, the VM will be created on a host of the
  same hypervisor type to the image, so the VM's hypervisor type is inherited from the image.

- **from an ISO**:
  as the ISO will be used to install the VM's blank root volume, the VM can be created on hosts of any hypervisor
  types, then the VM's hypervisor type is inherited from the host it's created.

.. _vm state:

State
+++++

VMs have 10 states representing life cycles.

- **Created**

  The VM is just created as a record in database, but has not been started on any host. The state only exists when creating a new VM.

- **Starting**

  The VM is starting on a host

- **Running**

  The VM is running on a host

- **Stopping**

  The VM is stopping on a host

- **Stopped**

  The VM is stopped and not running on any host

- **Rebooting**

  The VM is rebooting on the host it's running previously

- **Destroying**

  The VM is being destroyed

- **Migrating**

  The VM is being migrated to another host

- **Unknown**

  For some reason, for example, losing connection to the host, ZStack fails to detect the VM's state


.. image:: vm-state.png
   :align: center

ZStack uses a VmTracer to periodically track VMs' states; the default interval is 60s. A VM's state may be changed outside ZStack,
for example, a host power outage will make all VMs stop on the host; once the VmTracer detects a mismatch between the real state of a VM
and the record in database, it will update database to catch up the real state. If the VmTracer fails to detect a VM's state,
for example, because of losing connection between a ZStack management node and a host, it will place the VM into state Unknown;
once the VmTracer successfully detects the VM's state again, for example, after connection recovers between the ZStack management node and the host,
it will update the VM to the real state.

----------
Operations
----------

.. _CreateVmInstance:

Create VM
=========

Users can use CreateVmInstance to create a new VM. For example::

    CreateVmInstance name=vm imageUuid=d720ff0c60ee48d3a2e6263dd3e12c33 instanceOfferingUuid=76789b62aeb542a5b4b8b8488fbaced2 l3NetworkUuids=37d3c4a1e2f14a1c8316a23531e62988,05266285f96245f096f3b7dce671991d defaultL3NetworkUuid=05266285f96245f096f3b7dce671991d

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
   * - **instanceOfferingUuid**
     - uuid of :ref:`instance offering <instance offering>`
     -
     -
     - 0.6
   * - **imageUuid**
     - uuid of :ref:`image <image>`. Image can only be type of RootVolumeTemplate or ISO
     -
     -
     - 0.6
   * - **l3NetworkUuids**
     - a list of :ref:`L3 network <l3Network>` uuid
     -
     -
     - 0.6
   * - **type**
     - reserved field, default is UserVm
     -
     - - UserVm
       - ApplianceVm
     - 0.6
   * - **rootDiskOfferingUuid**
     - uuid of :ref:`disk offering <disk offering>` for root volume, see :ref:`rootDiskOfferingUuid`
     - true
     -
     - 0.6
   * - **dataDiskOfferingUuids**
     - a list of :ref:`disk offering <disk offering>` uuid, see :ref:`dataDiskOfferingUuids`
     - true
     -
     - 0.6
   * - **zoneUuid**
     - if not null, the VM will be created in the specified zone; this field can be overridden by clusterUuid or hostUuid
     - true
     -
     - 0.6
   * - **clusterUuid**
     - if not null, the VM will be created in the specified cluster; this field can be overridden by hostUuid
     - true
     -
     - 0.6
   * - **hostUuid**
     - if not null, the VM will be created on the specified host
     - true
     -
     - 0.6
   * - **defaultL3NetworkUuid**
     - if l3NetworkUuids includes more than one L3 network UUIDs, this field indicates which L3 network is the default L3 network.
       leave it alone if l3NetworkUuids has only one L3 network uuid.
     - true
     -
     - 0.6q

.. _rootDiskOfferingUuid:

rootDiskOfferingUuid
--------------------

If a VM is created from an ISO image, users must specify a :ref:`disk offering <disk offering>` by rootDiskOfferingUuid
so ZStack knows the disk size of the root volume; if the VM is created from an RootVolumeTemplate image, this field is ignored.

.. _dataDiskOfferingUuids:

dataDiskOfferingUuids
---------------------

By providing a list of disk offering UUIDs in dataDiskOfferingUuids, users can create a VM with multiple data volumes attached.
If a data volume failed to be created, the whole VM creation fails.

.. _StopVmInstance:

Stop VM
=======

Users can use StopVmInstance to stop a running VM. For example::

    StopVmInstance uuid=76789b62aeb542a5b4b8b8488fbaced2

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
     - VM uuid
     -
     -
     - 0.6

.. _StartVmInstance:

Start VM
========

Users can use StartVmInstance to start a stopped VM. For example::

    StartVmInstance uuid=76789b62aeb542a5b4b8b8488fbaced2

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
     - VM uuid
     -
     -
     - 0.6

When starting a VM, ZStack uses LastHostPreferredAllocatorStrategy algorithm that will start the VM on the host it previously run if possible;
otherwise, start the VM on a new host using the algorithm of :ref:`DesignatedHostAllocatorStrategy`.

.. _RebootVmInstance:

Reboot VM
=========

Users can use RebootVmInstance to reboot a running VM. For example::

    RebootVmInstance uuid=76789b62aeb542a5b4b8b8488fbaced2

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
     - VM uuid
     -
     -
     - 0.6

.. _DestroyVmInstance:

Destroy VM
==========

Users can use DestroyVmInstance to destroy a VM. For example::

    DestroyVmInstance uuid=76789b62aeb542a5b4b8b8488fbaced2

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
     - VM uuid
     -
     -
     - 0.6

.. warning:: There is no way to recover a destroyed VM; once a VM is destroyed, its root volume will be deleted; if global
             configuration :ref:`dataVolume.deleteOnVmDestroy` is true, attached data volumes will be deleted as well; otherwise,
             data volumes will be detached.

.. _MigrateVm:

Migrate VM
==========

Admins can use MigrateVm to live migrate a running VM from the current host to another host. For example::

    MigrateVm vmInstanceUuid=76789b62aeb542a5b4b8b8488fbaced2 hostUuid=37d3c4a1e2f14a1c8316a23531e62988

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
     - VM uuid
     -
     -
     - 0.6
   * - **hostUuid**
     - target host uuid; if omitted, ZStack will try to find a proper host automatically
     - true
     -
     - 0.6

A VM can migrate between two hosts only if their OS versions are exactly matching. For KVM, OS versions are determined by three system
tags: :ref:`os::distribution <host metadata information>`, :ref:`os::release <host metadata information>`, and :ref:`os::version <host metadata information>`.

When migrating, OS versions are checked by *MigrateVmAllocatorStrategy* which uses a similar algorithm of :ref:`DesignatedHostAllocatorStrategy` to
choose target migration host.

.. warning:: For KVM, if you use customized libvirt and qemu rather than those builtin ones, migration may fail even OS versions match on
             two hosts. Please make sure OS version, libvirt version, and qemu version are all the same on two hosts for migration.


Attach Data Volume
==================

See :ref:`attach volume to vm <AttachDataVolumeToVm>`.

Detach Data volume
==================

See :ref:`detach volume from vm <DetachDataVolumeFromVm>`.

Query VM
========

Users can use QueryVmInstance to query VMs. For example::

    QueryVmInstance state=Running hostUuid=33107835aee84c449ac04c9622892dec

::

    QueryVmInstance vmNics.eip.guestIp=10.23.109.23


Primitive Fields of Query
+++++++++++++++++++++++++

see :ref:`VM inventory <vm inventory>`

Nested And Expanded Fields of Query
+++++++++++++++++++++++++++++++++++

.. list-table::
   :widths: 20 30 40 10
   :header-rows: 1

   * - Field
     - Inventory
     - Description
     - Since
   * - **vmNics**
     - :ref:`VM nic inventory <vm nic inventory>`
     - VM nics belonging to this VM
     - 0.6
   * - **allVolumes**
     - :ref:`volume inventory <volume inventory>`
     - volumes belonging to this VM
     - 0.6
   * - **zone**
     - :ref:`zone inventory <zone inventory>`
     - ancestor zone
     - 0.6
   * - **cluster**
     - :ref:`cluster inventory <cluster inventory>`
     - ancestor cluster
     - 0.6
   * - **host**
     - :ref:`host inventory <host inventory>`
     - parent host
     - 0.6
   * - **image**
     - :ref:`image inventory <image inventory>`
     - image this VM is created from
     - 0.6
   * - **instanceOffering**
     - :ref:`instance offering inventory <instance offering inventory>`
     - instance offering this VM is created from
     - 0.6
   * - **rootVolume**
     - :ref:`volume inventory <volume inventory>`
     - root volume belonging to this VM
     - 0.6

Query VM Nic
============

Users can use QueryVmNic to query VM nics. For example::

    QueryVmNic gateway=10.1.1.1

::

    QueryVmNic eip.guestIp=11.168.2.13


Primitive Fields of Query Nic
+++++++++++++++++++++++++++++

see :ref:`VM nic inventory <vm nic inventory>`

Nested And Expanded Fields of Query Nic
+++++++++++++++++++++++++++++++++++++++

.. list-table::
   :widths: 20 60 20
   :header-rows: 1

   * - Field
     - Inventory
     - Since
   * - **vmInstance**
     - :ref:`VM inventory <vm inventory>`
     - 0.6
   * - **l3Network**
     - :ref:`L3 network inventory <l3Network inventory>`
     - 0.6
   * - **eip**
     - :ref:`EIP inventory <eip inventory>`
     - 0.6
   * - **portForwarding**
     - :ref:`port forwarding inventory <port forwarding rule inventory>`
     - 0.6
   * - **securityGroup**
     - :ref:`security group inventory <security group inventory>`
     - 0.6

---------------------
Global Configurations
---------------------

.. _dataVolume.deleteOnVmDestroy:

dataVolume.deleteOnVmDestroy
============================

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **dataVolume.deleteOnVmDestroy**
     - vm
     - false
     - - true
       - false

If true, data volumes attached to the VM will be deleted as well when the VM is being deleted;
otherwise, the data volumes will be detached.

----
Tags
----

Users can create user tags on a VM with resourceType=VmInstanceVO. For example::

    CreateUserTag tag=web-server-vm resourceType=VmInstanceVO resourceUuid=a12b3cc9ee4440dfb00d41c1d2f72d08

System Tags
===========

HostName
++++++++

Users can specify a hostname for a VM's default L3 network. This tag is usually specified in *systemTags* parameter
when calling CreateVmInstance; if the default L3 network has a DNS domain, the hostname that VM's operating system receives
will be automatically expanded with the DNS domain. For example, assuming the hostname is 'web-server'
and DNS domain of the default L3 network is 'zstack.org', the final hostname will be 'web-server.zstack.org'.

.. list-table::
   :widths: 20 30 40 10
   :header-rows: 1

   * - Tag
     - Description
     - Example
     - Since
   * - **hostname::{hostname}**
     - hostname for VM's default L3 network
     - hostname::web-server
     - 0.6

For example::

    CreateVmInstance name=vm systemTags=hostname::vm1 imageUuid=d720ff0c60ee48d3a2e6263dd3e12c33 instanceOfferingUuid=76789b62aeb542a5b4b8b8488fbaced2 l3NetworkUuids=37d3c4a1e2f14a1c8316a23531e62988,05266285f96245f096f3b7dce671991d defaultL3NetworkUuid=05266285f96245f096f3b7dce671991d
