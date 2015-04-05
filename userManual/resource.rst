.. _resource:

==============
Resource Model
==============

.. contents:: `Table of contents`
   :depth: 6

ZStack is essentially a configuration management system for resources in the cloud. Resources can be physical resources(e.g. hosts, networks, storage)
or virtual resources(e.g. VMs). In this version, a full diagram of ZStack resources is like:

.. image:: resource-overall.png
   :align: center
   :alt: resource overall


.. note:: This diagram aims to give an overall idea that what ZStack resources look like.
          It neither exhibits exact relationship among resources nor shows amount of resources.

---------------------
Resource Relationship
---------------------

Resources have four relationships:

- **Parent - Child**:

  A resource can be the parent or a child of another resource. For example, a cluster is a child resource of a zone, while a zone is the parent resource of a cluster.

- **Ancestor - Descendant**:

  A resource can be the lineal ancestor or a lineal descendant of another resource. For example, a zone is the ancestor resource of VMs; a VM is a descendant resource of a zone.

- **Sibling**:

  Resources sharing the same parent resource are siblings. For example, clusters, hosts, primary storage, L2 networks are sibling resources because all of them are child resources of zones.

- **Friend**:

  Resources which don't have above three relationships but still need to cooperate with each other in some scenarios are friends. For example, primary storage and
  backup storage are friends, because primary storage need to download images from backup storage in order to create VMs.


Resources that don't have any relationship are irrelevant resources; for example, security groups and clusters are irrelevant resources.

.. note:: In this version, ZStack doesn't have a concept of region, so zones and backup storage doesn't have a parent resource; however, they are still considered as
          siblings.

.. _resource properties:

-------------------
Resource Properties
-------------------

There are four properties common to almost all resources:

- **UUID**:

  ZStack uses `UUIDv4 (Universally Unique Identifier) <http://en.wikipedia.org/wiki/Universally_unique_identifier>`_ to uniquely identify a resource. Unlike regular UUIDs which
  has four hyphens in string, the UUIDs ZStack use have hyphens stripped. For example, 80b5ca2c76154da298a1a248b975372a.

- **Name**:

  Names are human readable strings with maximum 255 characters. Names can be duplicated, as ZStack doesn't use them as resource identifiers.
  Names can have any visible ASCII characters(e.g. %, #, ^, space); however, putting symbols
  in a name may make query APIs hard to use. The best practice for naming a resource is only using letters, digits, '-'(hyphen), and '_'(underscore).

  .. note:: Please avoid using ','(comma) in names, though it's legal. In query APIs, ZStack uses comma to split a value into a list when the condition operator is '?='(which means 'in').
            For example, querying VMs by a condition 'name' and an operator '?=' is like::

                QueryVmInstance name?=vm1,vm2,vm3

            what it does is: finding out VMs whose names are vm1 or vm2 or vm3. For people familiar with SQL, it is equal to::

                select * from VmInstance where name in ('vm1', 'vm2', 'vm3')

            if you have a comma in VMs' names, for example, 'v,m1', then the query becomes::

                QueryVmInstance name?=v,m1,vm2,vm3

            which turns out to find VMs whose names are in ['v', 'm1', 'vm2', 'vm3]

- **Description**:

  Descriptions are human readable strings with maximum 2048 characters. Still, ZStack doesn't enforces any limited character set on descriptions.

- **Created Date**:

  A immutable date indicating the time that resources were created.

- **Last Operation Date**:

  A date indicating the last time resources were updated. The date changes every time after a update has been performed on a resource;
  updates can be either from user operations or ZStack's internal operations.

.. note:: Some resources may not have names and descriptions, for example, DNS, security group rules. These resources are not considered as independent resources and must be with their parent
          resources.

Each resource may have its specific properties, for example, VMs have a property 'hostUuid'. As ZStack uses JSON in APIs, properties of resources are encompassed in a JSON map in most
API responses. The JSON map is called *'inventory'* in ZStack's language. In following chapters, when talking about an inventory of a resource, we are referring to the map containing the resource's properties.
Here is an example of VM inventory::

    {
      "inventory": {
        "uuid": "94d991c631674b16be65bfdf28b9e84a",
        "name": "TestVm",
        "description": "Test",
        "zoneUuid": "acadddc85a604db4b1b7358605cd6015",
        "clusterUuid": "f6cd5db05a0d49d8b12721e0bf721b4c",
        "imageUuid": "061141410a0449b6919b50e90d68b7cd",
        "hostUuid": "908131845d284d7f821a74362fff3d19",
        "lastHostUuid": "908131845d284d7f821a74362fff3d19",
        "instanceOfferingUuid": "91cb47f1416748afa7e0d34f4d0731ef",
        "rootVolumeUuid": "19aa7ec504a247d89b511b322ffa483c",
        "type": "UserVm",
        "hypervisorType": "KVM",
        "createDate": "Jun 1, 2015 6:11:47 PM",
        "lastOpDate": "Jun 1, 2015 6:11:47 PM",
        "state": "Running",
        "vmNics": [
          {
            "uuid": "6b58e6b2ba174ef4bce8a549de9560e8",
            "vmInstanceUuid": "94d991c631674b16be65bfdf28b9e84a",
            "usedIpUuid": "4ecc80a2d1d93d48b32680827542ddbb",
            "l3NetworkUuid": "55f85b8fa9a647f1be251787c66550ee",
            "ip": "10.12.140.148",
            "mac": "fa:f0:08:8c:20:00",
            "netmask": "255.0.0.0",
            "gateway": "10.10.2.1",
            "deviceId": 0,
            "createDate": "Jun 1, 2015 6:11:47 PM",
            "lastOpDate": "Jun 1, 2015 6:11:47 PM"
          },
          {
            "uuid": "889cfcab8c08409296c649611a4df50c",
            "vmInstanceUuid": "94d991c631674b16be65bfdf28b9e84a",
            "usedIpUuid": "8877537e11783ee0bfe8af0fcf7a6388",
            "l3NetworkUuid": "c6134efd3af94db7b2928ddc5deba540",
            "ip": "10.4.224.72",
            "mac": "fa:e3:87:b1:71:01",
            "netmask": "255.0.0.0",
            "gateway": "10.0.0.1",
            "deviceId": 1,
            "createDate": "Jun 1, 2015 6:11:47 PM",
            "lastOpDate": "Jun 1, 2015 6:11:47 PM"
          },
          {
            "uuid": "cba0da7a12d44b2e878dd5803d078337",
            "vmInstanceUuid": "94d991c631674b16be65bfdf28b9e84a",
            "usedIpUuid": "f90d01a098303956823ced02438ae3ab",
            "l3NetworkUuid": "c7e9e14f2af742c29c3e25d58f16a45f",
            "ip": "10.29.42.155",
            "mac": "fa:2d:31:08:da:02",
            "netmask": "255.0.0.0",
            "gateway": "10.20.3.1",
            "deviceId": 2,
            "createDate": "Jun 1, 2015 6:11:47 PM",
            "lastOpDate": "Jun 1, 2015 6:11:47 PM"
          }
        ],
        "allVolumes": [
          {
            "uuid": "19aa7ec504a247d89b511b322ffa483c",
            "name": "ROOT-for-TestVm",
            "description": "Root volume for VM[uuid:94d991c631674b16be65bfdf28b9e84a]",
            "primaryStorageUuid": "24931b95b45e41fb8e41a640302d4c00",
            "vmInstanceUuid": "94d991c631674b16be65bfdf28b9e84a",
            "rootImageUuid": "061141410a0449b6919b50e90d68b7cd",
            "installUrl": "/opt/zstack/nfsprimarystorage/prim-24931b95b45e41fb8e41a640302d4c00/rootVolumes/acct-36c27e8ff05c4780bf6d2fa65700f22e/vol-19aa7ec504a247d89b511b322ffa483c/19aa7ec504a247d89b511b322ffa483c.qcow2",
            "type": "Root",
            "format": "qcow2",
            "size": 3.221225472E10,
            "deviceId": 0,
            "state": "Enabled",
            "status": "Ready",
            "createDate": "Jun 1, 2015 6:11:47 PM",
            "lastOpDate": "Jun 1, 2015 6:11:47 PM"
          }
        ]
      }
    }

-------------------
Resource Operations
-------------------

Resources support full or partial CRUD(Create, Read, Update, Delete) operations.


.. _create resource:

Create Resources
================

Every resource has own creational APIs. There is one parameter 'resourceUuid' common to all creational APIs.
When 'resourceUuid' is not null, ZStack will use its value as the UUID for the resource being created; otherwise ZStack will automatically generate a UUID.

.. warning:: When using 'resourceUuid', please make sure the UUID you provide is a UUIDv4 with hyphens striped. Otherwise, ZStack will return an invalid
             argument error if it's not a valid UUIDv4 with hyphens stripped, or an internal error if there has been a resource of the same type with the same UUID in
             the database.

Here is an example of creating a cluster::

    CreateCluster name=cluster1 description='awesome cluster' hypervisorType=KVM zoneUuid=061141410a0449b6919b50e90d68b7cd

or::

    CreateCluster resourceUuid=f31e38309e2047beac588e111fa2051f name=cluster1 description='awesome cluster' hypervisorType=KVM zoneUuid=061141410a0449b6919b50e90d68b7cd


Read Resources
==============

Every resource has own query API that returns a list of inventories for read.
For details, see :ref:`Query <query>`. Here is an example of querying VMs::

    QueryVmInstance allVolumes.type=Data allVolumes.size>1099511627776

The example does: finding out all VMs which have one or more data volumes with size greater than 1099511627776 bytes(1T)


Update Resources
================

A resource can be updated by various APIs. Updating a resource is actually performing an action to the resource. For example,
starting a VM, stopping a VM. Please refer to corresponding chapters for actions for resources. Here is an example
of starting a VM::

    StartVmInstance uuid=94d991c631674b16be65bfdf28b9e84a

Most update APIs will return a resource inventory.


.. _delete resource:

Delete Resources
================

A resource can be deleted. ZStack's philosophy for deleting is: every resource should be deletable; and deleting a resource should always be success
unless user allows an expected failure; for example, a plugin may allow user to set a 'none-deletable' tag on a VM, and throw an error when the VM is being
deleted.

Deleting a resource is not always easy in IaaS, especially for a resource that has many descendants; some software hard code to delete all descendant resources;
some software simply throws an error when a resource being deleted still has descendant resources alive.

ZStack handles deleting in an elegant way. When a resource is being deleted, a so-called `Cascade Framework <http://zstack.org/blog/cascade.html>`_ will calculate relationships among this resource and
rest resources in the cloud, and propagate proper actions to related resources if necessary. For example, when deleting a zone, a deleting
action will be spread to all descendants of the zone, which means all descendant resources like VMs, hosts, clusters, L2 Networks in this zone will be deleted before the zone
deleted; and backup storage attached to the zone will be detached. With the cascade framework, deleting resources in ZStack is easy and reliable.

Every resource has own deleting API. A parameter *deleteMode* which has options **Permissive** and **Enforcing** is common to all deleting APIs.
When *deleteMode* is set to Permissive, ZStack will stop the deleting if an error happens, or the deleting is not permitted; in this
case, an error code with detailed reason will be returned. When *deleteMode* is set to Enforcing, ZStack will ignore any errors and permissions but delete resources directly; in this case,
a deleting will always be success.

Here is an example of deleting a VM::

    DestroyVmInstance uuid=94d991c631674b16be65bfdf28b9e84a


