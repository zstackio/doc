.. _tag:

====
Tags
====

.. contents:: `Table of contents`
   :depth: 6

--------
Overview
--------

ZStack provides two types of tags to help users and plugins organize resources, introduce extra resource properties, and
instruct ZStack to perform specific business logic. For the architecture design of tags see `The Tag System <http://zstack.org/blog/tag.html>`_.

---------
User Tags
---------

Users can create user tags on resources they own, which is particular useful when aggregating a set of similar resources;
for example, users can put a tag 'web' on VMs that work as web servers::

    CreateUserTag resourceType=VmInstanceVO resourceUuid=613af3fe005914c1643a15c36fd578c6 tag=web

::

    CreateUserTag resourceType=VmInstanceVO resourceUuid=5eb55c39db015c1782c7d814900a9609 tag=web

::

    CreateUserTag resourceType=VmInstanceVO resourceUuid=0cd1ef8c9b9e0ba82e0cc9cc17226a26 tag=web

and later on, use :ref:`Query API with tags <query with tags>` to retrieve those VMs::

    QueryVmInstance __userTag__=web


Users can also use user tags cooperating with system tags to change ZStack's business logic; for example, users may want all VMs working as web
servers to create their root volumes on a special primary storage which provides better IO performance by SSD; to do so,
users can create a user tag 'forWebTierVM' on the primary storage::

    CreateUserTag tag=forWebTierVM resourceType=PrimaryStorageVO resourceUuid=6572ce44c3f6422d8063b0fb262cbc62

then create a system tag on an instance offering::

    CreateSystemTag tag=primaryStorage::allocator::userTag::forWebTierVM resourceType=InstanceOfferingVO resourceUuid=8f69ef6c2c444cdf8c019fa0969d56a5

then, when users create a VM with the instance offering[uuid:8f69ef6c2c444cdf8c019fa0969d56a5], ZStack will make sure the VM's root volume
will be created on only the primary storage with user tag 'forWebTierVM', in this case, which is the primary storage with UUID 6572ce44c3f6422d8063b0fb262cbc62.

-----------
System Tags
-----------

System tags have wider usage than user tags; users can use them to instruct ZStack to do some specific business logic, like the example in the section above. Plugins,
which extend ZStack's functionality, can use system tags to introduce additional resource properties, or to record metadata which tightly bind to resources.

for example, to carry out live migration or live snapshot on KVM hosts, ZStack needs to know KVM hosts' libvirt version and QEMU version all of which are treated
as meta data, so ZStack records them as system tags of hosts. For example, admins can view system tags of a KVM host by::

    QuerySystemTag fields=tag resourceUuid=d07066c4de02404a948772e131139eb4

*d07066c4de02404a948772e131139eb4* is the KVM host UUID, the output is like::

    {
      "inventories": [
          {
              "tag": "capability:liveSnapshot"
          },
          {
              "tag": "qemu-img::version::2.0.0"
          },
          {
              "tag": "os::version::14.04"
          },
          {
              "tag": "libvirt::version::1.2.2"
          },
          {
              "tag": "os::release::trusty"
          },
          {
              "tag": "os::distribution::Ubuntu"
          }
      ],
      "success": true
    }

this kind of system tags, which record meta data, are called inherent system tags; inherent system tags can only be created by ZStack's services or plugins, and cannot
be deleted by DeleteTag API.

To add new functionality, a plugin usually needs to add new properties to a resource; though a plugin cannot change a resource's database schema to add a new
column, it can create new properties as system tags of a resource. For example, when creating a VM, users can specify the VM's hostname for the default L3 network::

    CreateVmInstance name=testTag systemTags=hostname::web-server-1 l3NetworkUuids=6572ce44c3f6422d8063b0fb262cbc62 instanceOfferingUuid=04b5419ca3134885be90a48e372d3895 imageUuid=f1205825ec405cd3f2d259730d47d1d8

this hostname is implemented by a system tag; if you look at :ref:`VM inventory in chapter 'Virtual Machine' <vm inventory>`, there is no property called 'hostname'; however, you can find it
from the VM's system tags::

    QuerySystemTag fields=tag,uuid resourceUuid=76e119bf9e16461aaf3d1b47c645c7b7

::

    {
      "inventories": [
          {
              "tag": "hostname::web-server-1",
              "uuid": "596070a6276746edbf0f54ef721f654e"
          }
      ],
      "success": true
    }

this kind of system tags are non-inherent, users can delete them by DeleteTag; for example, if users want to change the hostname of the former VM to
'web-server-nginx', they can do::


    DeleteTag uuid=596070a6276746edbf0f54ef721f654e

::

    CreateSystemTag resourceType=VmInstanceVO tag=hostname::web-server-nginx resourceUuid=76e119bf9e16461aaf3d1b47c645c7b7

after stopping and starting the VM, the guest operating system will receive the new hostname as 'web-server-nginx'.

.. note:: System tags are pre-defined by ZStack's services and plugins; user cannot create a non-existing system tag on a resource.
          You can find resources' system tags in *Tags* section of every resource chapter.

---------------
Name Convention
---------------

Both user tags and system tags can have at most 2048 characters.

For user tags, there is no enforced name convention, but it's recommended to use human readable and meaningful strings.

For system tags, as defined by ZStack's services and plugins, they follow the format that uses *::* as delimiters.

.. _tag resource type:

-------------
Resource Type
-------------

When creating a tag, user must specify the resource type that the tag is associated with. In this version, a list of resource types
is showed as follows:

.. list-table::
   :widths: 100

   * - ZoneVO
   * - ClusterVO
   * - HostVO
   * - PrimaryStorageVO
   * - BackupStorageVO
   * - ImageVO
   * - InstanceOfferingVO
   * - DiskOfferingVO
   * - VolumeVO
   * - L2NetworkVO
   * - L3NetworkVO
   * - IpRangeVO
   * - VipVO
   * - EipVO
   * - VmInstanceVO
   * - VmNicVO
   * - SecurityGroupRuleVO
   * - SecurityGroupVO
   * - PortForwardingRuleVO
   * - VolumeSnapshotTreeVO
   * - VolumeSnapshotVO

Derived resources use their parent types; for example, SftpBackupStorage's resourceType is 'BackupStorageVO'.
In *Tags* section of every resource chapter, we will explain what resource types to use when creating tags.

----------
Operations
----------

.. _create tags:

Create Tags
===========

There are two ways to create tags; for resources that have been created, users can use command CreateUserTag or CreateSystemTag
to create a user tag or a system tag. For example::

    CreateUserTag resourceType=DiskOfferingVO resourceUuid=50fcc61947f7494db69436ebbbefda34 tag=for-large-DB

::

    CreateSystemTag resourceType=HostVO resourceUuid=50fcc61947f7494db69436ebbbefda34 tag=reservedMemory::1G

For a resource that is going to be created, as it's not been created yet, there is no resource UUID that can be referred in the CreateUserTag
and CreateSystemTag commands; in this case, users can use *userTags* and *systemTags* fields, both of which are of a list type that receives a list of tags,
of every *creational API command*; for example::

    CreateVmInstance name=testTag systemTags=hostname::web-server-1
    userTags=in-super-data-center,has-public-IP,hot-fix-applied-2015-5-1
    l3NetworkUuids=6572ce44c3f6422d8063b0fb262cbc62
    instanceOfferingUuid=04b5419ca3134885be90a48e372d3895 imageUuid=f1205825ec405cd3f2d259730d47d1d8

Parameters
++++++++++

CreateUserTag and CreateSystemTag have the same API parameters:

.. list-table::
   :widths: 20 40 20 20
   :header-rows: 1

   * - Name
     - Description
     - Optional
     - Since
   * - **resourceUuid**
     - resource UUID; for example, VM's UUID uuid, instance offering's UUID
     -
     - 0.6
   * - **resourceType**
     - resource type; see :ref:`resource type <tag resource type>`
     -
     - 0.6
   * - **tag**
     - tag string
     -
     - 0.6

Delete Tag
==========

Users can use DeleteTag to delete a user tag or a non-inherent system tag. For example::

    DeleteTag uuid=7813d03bb85840c489789f8df3a5915b

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
     - tag UUID
     -
     -
     - 0.6

Query Tags
==========

Users can use QueryUserTag to query user tags, for example::

    QueryUserTag resourceUuid=0cd1ef8c9b9e0ba82e0cc9cc17226a26 tag~=web-server-%

or QuerySystemTag to query system tags, for example::

    QuerySystemTag resourceUuid=50fcc61947f7494db69436ebbbefda34

.. note:: When querying tags, as the resourceUuid has uniquely identified a resource, you don't need to specify the resource type; for example::

              QueryUserTag resourceUuid=0cd1ef8c9b9e0ba82e0cc9cc17226a26 resourceType=VmInstanceVO

          is redundant because ZStack knows resourceUuid *0cd1ef8c9b9e0ba82e0cc9cc17226a26* maps to type *VmInstanceVO*.

          And don't forget you can use *__userTag__* and *__systemTag__* to query resources with tags, see :ref:`Query API with tags <query with tags>`.
