.. _security group:

==============
Security Group
==============

.. contents:: `Table of contents`
   :depth: 6

--------
Overview
--------

.. _ICMP type and code: http://www.nthelp.com/icmp.html

A security group acts as a virtual firewall that controls networking traffics of VMs. Depending on the isolation method a L2 network
takes, users can use security group as firewalls or as a layer 3 isolation method. For example, if multiple tenants share a L3 network,
every tenant can create a security group to protect their VMs from being accessed by other tenants. Tenants can also use security group
along with :ref:`EIP <eip>` to control ports open to the public.

A security group consists of a set of rules that control ports' accessibility. A security group can be attached to one or more L3 networks;
VM nics on attached L3 networks can join those security groups. A VM nic can join multiple security groups, rules applied to the nic
will be merged.

.. image:: security-group1.png
   :align: center

.. image:: security-group2.png
   :align: center

The implementation of security group is hypervisor specific; not all hypervisors will support security group. In this ZStack version, security group
is supported in KVM hypervisor by using IPTables.

.. note:: A large number of security group rules may hurt network performance because the hypervisor needs to check all rules against every network packet.
          ZStack will try to condense security group rules as much as possible; for example, if you specify two rules for two consecutive ports,
          they will be merged into one IPTable(for KVM) rule using a port range match.

To use security group, a L3 network must enable security group service using :ref:`AttachNetworkServiceToL3Network <l3Network attach service>`, for example::

    AttachNetworkServiceToL3Network l3NetworkUuid=50e637dc68b7480291ba87cbb81d94ad networkServices='{"1d1d5ff248b24906a39f96aa3c6411dd": ["SecurityGroup"]}'

For VMs having multiple nics, all nics can join security groups.

The security group is essentially a distributed firewall; every rule change or nic join/leave event may lead firewall rules to be refreshed on multiple hosts.
Given this fact, some security group APIs are implemented in an asynchronous manner, which may return before rules take effect on hosts. If there are more than
one rules for a specific port, the most permissive rule takes effect. For example, if a rule1 allows traffic from 12.12.12.12 to access port 22 but a rule2 allows
everyone to access port 22, the rule2 takes precedence.

.. _security group inventory:

------------------------
Security Group Inventory
------------------------

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
   * - **state**
     - security group state; not implemented in this ZStack version
     -
     - - Enabled
       - Disabled
     - 0.6
   * - **rules**
     - a list of :ref:`security group rule inventory <security group rule inventory>`
     -
     -
     - 0.6
   * - **attachedL3NetworkUuids**
     - a list of uuid of :ref:`L3 networks <l3Network>` that this security group has been attached
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

For an empty security group, there are default polices for ingress traffics and egress traffics; for ingress traffics, the default
policy is to deny, which means all inbound traffics to the nics in this empty security group are blocked; for egress traffics, the default
policy is to allow, which means all outbound traffics from the nics in this empty security group are allowed. To change default policies,
admin can change global configuration :ref:`ingress.defaultPolicy` and :ref:`egress.defaultPolicy`.

Example
=======

::

    {
        "attachedL3NetworkUuids": [
            "0b48770e593e400c8f54e71fd4e7f514"
        ],
        "createDate": "Nov 16, 2015 1:02:22 AM",
        "lastOpDate": "Nov 16, 2015 1:02:22 AM",
        "name": "sg-in",
        "rules": [
            {
                "allowedCidr": "0.0.0.0/0",
                "createDate": "April 29, 2015 9:57:10 PM",
                "state": "Enabled",
                "endPort": 22,
                "lastOpDate": "Nov 29, 2015 9:57:10 PM",
                "protocol": "TCP",
                "securityGroupUuid": "9e0a72fe64814900baa22f78a1b9d235",
                "startPort": 22,
                "type": "Ingress",
                "uuid": "a338d11be18d4e288223597682964dc8"
            }
        ],
        "state": "Enabled",
        "uuid": "9e0a72fe64814900baa22f78a1b9d235"
    }

.. _security group rule inventory:

-----------------------------
Security Group Rule Inventory
-----------------------------

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
   * - **securityGroupUuid**
     - uuid of parent security group
     -
     -
     - 0.6
   * - **type**
     - see :ref:`traffic type <security group traffic type>`
     -
     - - Ingress
       - Egress
     - 0.6
   * - **protocol**
     - traffic protocol type
     -
     - - TCP
       - UDP
       - ICMP
     - 0.6
   * - **startPort**
     - when protocol is TCP/UDP, it's the start of port range; when protocol is ICMP, it's ICMP type
     -
     - - for TCP/UDP: 0 - 65535
       - for ICMP: see `ICMP type and code`_ , use '-1' to represent all types.
     - 0.6
   * - **endPort**
     - when protocol is TCP/UDP, it's the end of port range; when protocol is ICMP, it's ICMP code
     -
     - - for TCP/UDP: 0 - 65535
       - for ICMP: see `ICMP type and code`_, use '-1' to represent all types.
     - 0.6
   * - **allowedCidr**
     - see :ref:`allowedCidr <allowed cidr>`
     -
     -
     - 0.6
   * - **state**
     - rule state, not implemented in this version
     -
     - - Enabled
       - Disabled
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

.. _security group traffic type:

Traffic Type
++++++++++++

There are two types of traffics:

- **Ingress**

  Inbound traffics that access a VM nic

- **Egress**

  Outbound traffics that leave from a VM nic

.. _allowed cidr:

Allowed CIDR
++++++++++++

Depending on traffic types, allowed CIDR has different meanings; its format is::

    ipv4_address/network_prefix

    for example: 12.12.12.12/24

if the traffic type is Ingress, allowed CIDR is a source CIDR that's allowed to reach VM nics; for example, a rule::

    startPort: 22
    endPort: 22
    protocol: TCP
    type: Ingress
    allowedCidr: 12.12.12.12/32

means only TCP traffic **from** IP(12.12.12.12) is allowed to access port 22.

if the traffic type is Egress, allowed CIDR is a destination CIDR that's allowed to leave VM nics; for example, a rule::

    startPort: 22
    endPort: 22
    protocol: TCP
    type: Egress
    allowedCidr: 12.12.12.12/32

means only TCP traffic **to** port 22 of IP 12.12.12.12 is allowed to leave.

The special CIDR 0.0.0.0/0 means all IP addresses.

.. note:: Allowed CIDR only controls IPs outside a security group. Rules are automatically applied to
          IPs of VM nics that are on the same L3 network and in the same security group. For example,
          if two nics: nic1(10.10.1.5) and nic2(10.10.1.6) are in the same security group which has a
          rule::

                  startPort: 22
                  endPort: 22
                  protocol: TCP
                  type: Ingress
                  allowedCidr: 12.12.12.12/32

          nic1 and nic2 can reach port 22 of each other in spite of allowedCidr is set to 12.12.12.12/32.

Example
=======

::

           {
                "allowedCidr": "0.0.0.0/0",
                "state": "Enabled",
                "startPort": 22,
                "endPort": 22,
                "protocol": "TCP",
                "type": "Ingress",
                "createDate": "Nov 29, 2015 9:57:10 PM",
                "lastOpDate": "Nov 29, 2015 9:57:10 PM",
                "uuid": "a338d11be18d4e288223597682964dc8"
                "securityGroupUuid": "9e0a72fe64814900baa22f78a1b9d235",
           }

-----------------------------
Security Group And L3 Network
-----------------------------

As having said, a security group can be attached to multiple L3 networks. The design consideration is that a security group is
a set of firewall rules and can be applied to any L3 networks. For example, two different L3 networks may have the same set of firewall
rules which make much sense to be put into the same security group.

VM nics from different L3 networks in the same security group are irrelevant. As mentioned in :ref:`Allowed CIDR <allowed cidr>`,
VM nics of the same L3 network in a security group are not affected by rules' allowedCIDR, they can always reach ports opened
of each other. However, if two nics in a security group are from different L3 networks, then the allowedCIDR will take
effect when they try to reach each other.

.. image:: security-group3.png
   :align: center

If you find it's confusing to have a security group attached to multiple L3 networks, you can always create a security group per
each L3 network.


----------
Operations
----------

Create Security Group
=====================

Users can use CreateSecurityGroup to create a security group. For example::

    CreateSecurityGroup name=web

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


Add Rules To Security Group
===========================

Users can use AddSecurityGroupRule to add rules to a security group. For example::

   AddSecurityGroupRule securityGroupUuid=29a0f801f77b4b4f866fb4c9503d0fe9 rules="[{'type':'Ingress', 'protocol':'TCP', 'startPort':'22', 'endPort':'22', 'allowedCidr':'0.0.0.0/0'}]"

This command executes asynchronously, it may return before rules are applied to all VM nics.

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
   * - **securityGroupUuid**
     - uuid of security group
     -
     -
     - 0.6
   * - **rules**
     - a list of :ref:`SecurityGroupRuleAO <SecurityGroupRuleAO>`
     -
     -
     - 0.6

.. _SecurityGroupRuleAO:

SecurityGroupRuleAO
-------------------
.. list-table::
   :widths: 20 40 10 20 10
   :header-rows: 1

   * - Name
     - Description
     - Optional
     - Choices
     - Since
   * - **type**
     - traffic type, see :ref:`traffic type <security group traffic type>`
     -
     - - Ingress
       - Egress
     - 0.6
   * - **startPort**
     - start port or ICMP type
     -
     - - port: 0 - 65535
       - ICMP type: see `ICMP type and code`_
     - 0.6
   * - **endPort**
     - end port or ICMP code
     -
     - - port: 0 - 65535
       - ICMP code: see `ICMP type and code`_
     - 0.6
   * - **protocol**
     - protocol type
     -
     - - TCP
       - UDP
       - ICMP
     - 0.6
   * - **allowedCidr**
     - see :ref:`allowed CIDR <allowed cidr>`; default to 0.0.0.0/0
     - true
     -
     - 0.6


Delete Rules From Security Group
================================

User can uses DeleteSecurityGroupRule to delete rules from a security group. For example::

    DeleteSecurityGroupRule ruleUuids=a338d11be18d4e288223597682964dc8,9e0a72fe64814900baa22f78a1b9d235

This command executes asynchronously, it may return before rules are refreshed on all hosts.

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
   * - **ruleUuids**
     - a list of uuid of :ref:`rule inventory <security group rule inventory>`
     -
     -
     - 0.6

Add VM Nics Into Security Group
===============================

Users can use AddVmNicToSecurityGroup to add VM nics to a security group. For example::

    AddVmNicToSecurityGroup securityGroupUuid=0b48770e593e400c8f54e71fd4e7f514 vmNicUuids=b429625fe2704a3e94d698ccc0fae4fb,6572ce44c3f6422d8063b0fb262cbc62,d07066c4de02404a948772e131139eb4

This command executes asynchronously, it may return before rules are applied on those nics.

.. note:: VM nics can only join security groups that have been attached to their L3 networks.

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
   * - **securityGroupUuid**
     - security group uuid
     -
     -
     - 0.6
   * - **vmNicUuids**
     - a list of uuid of :ref:`vm nic inventory <vm nic inventory>`
     -
     -
     - 0.6


Remove VM Nics from Security Group
==================================

Users can use DeleteVmNicFromSecurityGroup to delete VM nics from a security group. For example::

    DeleteVmNicFromSecurityGroup securityGroupUuid=0b48770e593e400c8f54e71fd4e7f514 vmNicUuids=b429625fe2704a3e94d698ccc0fae4fb,6572ce44c3f6422d8063b0fb262cbc62,d07066c4de02404a948772e131139eb4

This command executes asynchronously, it may return before rules are refreshed on rest nics in the security group.

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
   * - **securityGroupUuid**
     - security group uuid
     -
     -
     - 0.6
   * - **vmNicUuids**
     - a list of uuid of :ref:`vm nic inventory <vm nic inventory>`
     -
     -
     - 0.6

Attach Security Group To L3 Network
===================================

Users can use AttachSecurityGroupToL3Network to attach a security group to a L3 network. For example::

    AttachSecurityGroupToL3Network securityGroupUuid=0b48770e593e400c8f54e71fd4e7f514 l3NetworkUuid=95dede673ddf41119cbd04bcb5d73660

.. note::  A security group can only be attached to L3 networks that have security group network service enabled

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
   * - **securityGroupUuid**
     - security group uuid
     -
     -
     - 0.6
   * - **l3NetworkUuid**
     - L3 network uuid
     -
     -
     - 0.6

Detach Security Group From L3 Network
=====================================

Users can use DetachSecurityGroupFromL3Network to detach a security group from a L3 network::

    DetachSecurityGroupFromL3Network securityGroupUuid=0b48770e593e400c8f54e71fd4e7f514 l3NetworkUuid=95dede673ddf41119cbd04bcb5d73660

After detaching, all rules will be removed from VM nics of the L3 network and in this security group. This
command executes asynchronously, it may return before rules are refreshed on those nics.

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
   * - **securityGroupUuid**
     - security group uuid
     -
     -
     - 0.6
   * - **l3NetworkUuid**
     - L3 network uuid
     -
     -
     - 0.6

Delete Security Group
=====================

Users can use DeleteSecurityGroup to delete a security group. For example::

    DeleteSecurityGroup uuid=0b48770e593e400c8f54e71fd4e7f514

After deleting, all rules will be removed from VM nics in this security group.
This command executes asynchronously, it may return before rules are refreshed on those VM nics.

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
     - security group uuid
     -
     -
     - 0.6

Query Security Group
====================

Users can use QuerySecurityGroup to query security groups. For example::

    QuerySecurityGroup rules.startPort=22 rules.type=Ingress rules.protocol=TCP

::

    QuerySecurityGroup vmNic.ip=192.168.0.205


Primitive Fields
++++++++++++++++

see :ref:`security group inventory <security group inventory>`.

Nested And Expanded Fields
++++++++++++++++++++++++++

.. list-table::
   :widths: 20 30 40 10
   :header-rows: 1

   * - Field
     - Inventory
     - Description
     - Since
   * - **rules**
     - :ref:`security group rule inventory <security group rule inventory>`
     - rules the security group has
     - 0.6
   * - **vmNic**
     - :ref:`VM nic inventory <vm nic inventory>`
     - VM nics that have joined this security group
     - 0.6
   * - **l3Network**
     - :ref:`L3 network inventory <l3Network inventory>`
     - L3 networks this security group is attached
     - 0.6


---------------------
Global Configurations
---------------------

.. _ingress.defaultPolicy:

ingress.defaultPolicy
=====================

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **ingress.defaultPolicy**
     - securityGroup
     - deny
     - - deny
       - accept

The default ingress policy for empty security groups.

.. _egress.defaultPolicy:

egress.defaultPolicy
====================

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **egress.defaultPolicy**
     - securityGroup
     - accept
     - - deny
       - accept

The default egress policy for empty security groups.

----
Tags
----

Users can create user tags on a security group with resourceType=SecurityGroupVO. For example::

    CreateUserTag tag=web-tier-security-group resourceType=SecurityGroupVO resourceUuid=f25a28fdb21147f8b183296550a98799
