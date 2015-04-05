.. _query:

=====
Query
=====

.. contents:: `Table of contents`
   :depth: 6

--------
Overview
--------

A main challenge for users operating large clouds is to find wanted resources accurately and quickly. For example, to find a
VM which has an EIP (17.12.53.8) out of 100,000 VMs. ZStack provides comprehensive APIs that can query every field of every
resource. See `The Query API <http://zstack.org/blog/query.html>`_ for the architecture design.

------------
Architecture
------------

Every ZStack resource groups its properties as an inventory in JSON format; for example, a zone inventory::

    {
      "uuid": "b729da71b1c7412781d5de22229d5e17",
      "name": "TestZone",
      "description": "Test",
      "state": "Enabled",
      "type": "zstack",
      "createDate": "Jun 1, 2015 6:04:52 PM",
      "lastOpDate": "Jun 1, 2015 6:04:52 PM"
    }

a resource inventory can include inventories of other resources; for example, a L3 network inventory contains IP range inventories::

        {
            "createDate": "Nov 10, 2015 7:52:57 PM",
            "dns": [
                "8.8.8.8"
            ],
            "ipRanges": [
                {
                    "createDate": "Nov 10, 2015 7:52:58 PM",
                    "endIp": "192.168.0.190",
                    "gateway": "192.168.0.1",
                    "l3NetworkUuid": "95dede673ddf41119cbd04bcb5d73660",
                    "lastOpDate": "Nov 10, 2015 7:52:58 PM",
                    "name": "ipr-mmbj",
                    "netmask": "255.255.255.0",
                    "startIp": "192.168.0.180",
                    "uuid": "13238c8e0591444e9160df4d3636be82"
                }
            ],
            "l2NetworkUuid": "33107835aee84c449ac04c9622892dec",
            "lastOpDate": "Nov 10, 2015 7:52:57 PM",
            "name": "L3-SYSTEM-PUBLIC",
            "networkServices": [],
            "state": "Enabled",
            "system": true,
            "type": "L3BasicNetwork",
            "uuid": "95dede673ddf41119cbd04bcb5d73660",
            "zoneUuid": "3a3ed8916c5c4d93ae46f8363f080284"
        }

there are two types of inventory fields: primitive field and nested field; a field is of primitive types of number, string, boolean and date;
in above example, uuid, name, system are primitive fields; a nested field is of composite types which usually represent inventories of other resources;
in above example, ipRanges is a nested fields.

.. note:: A nested field can only be queried by its sub-fields; for example, for the field *ipRanges*, you cannot do::

              QueryL3Network ipRanges='[{"name":"ipr-mmbj""}]'

          instead, you need to query its sub-field::

              QueryL3Network ipRanges.name=ipr-mmbj


Every field of every inventory is queryable unless it's explicitly stated as unqueryable;
for an inventory, there is a corresponding query API, for example, QueryZone, QueryHost, QueryVmInstance; the responses of
query APIs always carry a list of inventories, or an empty list if no matching result is found. A query response is like::

    {
      "inventories": [
          {
              "availableCpuCapacity": 13504,
              "availableMemoryCapacity": 16824565760,
              "clusterUuid": "b429625fe2704a3e94d698ccc0fae4fb",
              "createDate": "Nov 10, 2015 6:32:43 PM",
              "hypervisorType": "KVM",
              "lastOpDate": "Nov 10, 2015 6:32:43 PM",
              "managementIp": "192.168.0.212",
              "name": "U1404-192.168.0.212",
              "state": "Enabled",
              "status": "Connected",
              "totalCpuCapacity": 14400,
              "totalMemoryCapacity": 16828235776,
              "uuid": "d07066c4de02404a948772e131139eb4",
              "zoneUuid": "3a3ed8916c5c4d93ae46f8363f080284"
          }
      ],
      "success": true
    }

A query API consists of a list of query conditions and several helper parameters:

Query API Parameters
====================

.. list-table::
   :widths: 20 40 10 20 10
   :header-rows: 1

   * - Name
     - Description
     - Optional
     - Choices
     - Since
   * - **conditions**
     - a list of :ref:`QueryCondition <QueryCondition>`
     -
     -
     - 0.6
   * - **limit**
     - the maximum number of inventories returned by the query API; default to 1000
     - true
     -
     - 0.6
   * - **start**
     - the first inventory to return; default to 0
     - true
     -
     - 0.6
   * - **count**
     - if true, the query response will return only count of inventories; default to false
     -
     - - true
       - false
     - 0.6
   * - **replyWithCount**
     - if true, the query response will return both inventories and count; default to false
     -
     - - true
       - false
     - 0.6
   * - **sortBy**
     - the field by which the result inventories will be sorted. The field must be a primitive field
     - true
     -
     - 0.6
   * - **sortDirection**
     - if 'sortBy' is not null, this field specifies the sorting direction; default to 'asc'
     -
     - - asc
       - desc
     - 0.6
   * - **fields**
     - a list of primitive fields; when specified, the result inventory will contain only those fields.
     - true
     -
     - 0.6

.. _QueryCondition:

Query Condition
===============

Query APIs receive a list of query conditions which have properties as following:

.. list-table::
   :widths: 20 40 10 20 10
   :header-rows: 1

   * - Name
     - Description
     - Optional
     - Choices
     - Since
   * - **name**
     - field name
     -
     -
     - 0.6
   * - **op**
     - comparison operator
     -
     - - =
       - !=
       - >
       - >=
       - <
       - <=
       - in
       - not in
       - is null
       - is not null
       - like
       - not like
     - 0.6
   * - **value**
     - query value
     -
     -
     - 0.6

a field name can be of a primitive field, or of a sub-field of a nested field, or of a sub-field of an expanded field(see :ref:`Join <query join>`);
'op' are comparison operators which are from SQL language.

.. note:: for CLI tool, some operators have different forms from SQL, listed in column 'CLI Form'

.. list-table::
   :widths: 10 10 80
   :header-rows: 1

   * - Op
     - CLI Form
     - Description
   * - =
     - =
     - equal operator; case insensitive for string comparison
   * - !=
     - !=
     - not equal operator; case insensitive for string comparison
   * - >
     - >
     - greater than operator; check MySQL specification for string comparison
   * - >=
     - >=
     - greater than or equal operator; check MySQL specification for string comparison
   * - <
     - <
     - less than; check MySQL specification for string comparison
   * - <=
     - <=
     - less than or equal operator; check MySQL specification for string comparison
   * - in
     - ?=
     - check whether a value is within a set of values
   * - not in
     - !?=
     - check whether a value is NOT within a set of values
   * - is null
     - =null
     - NULL value test
   * - is not null
     - !=null
     - NOT NULL value test
   * - like
     - ~=
     - simple pattern matching. Use % to match any number of characters, even zero characters; use _ to matches exactly one character
   * - not like
     - !~=
     - negation of simple pattern matching. Use % to match any number of characters, even zero characters; use _ to matches exactly one character

The relation among conditions is logical AND, it's the only relation supported in this ZStack version. For example::

    QueryL3Network ipRanges.name=range1 name=L3Network1

is to find L3 networks whose names are 'L3Network1' AND which have one or more IP ranges with names 'range1'.

CLI Query Conditions
====================

There are two ways to write conditions in CLI, one is the original form of query API::

    QueryHost conditions='[{"name":"name", "op":"=", "value":"KVM1"}]'

another is CLI form::

    QueryHost name=KVM1

I am sure you will prefer the CLI form as it's more intuitive and human readable. The CLI form always expresses query conditions in formula of::

    condition_name(no_space)CLI_comparison_operator(no_space)condition_value

.. warning:: please note there is no space between condition_name and CLI_comparison_operator and condition_value::

                name=KVM1

             is valid but::

                name = KVM1

             is INVALID. See :ref:`CLI <cli>` for more details.

When typing in CLI, you can use *Tab* key for auto-completion and reminding you about queryable fields including primitive fields,
nested fields, and expanded fields:

.. image:: query1.png
   :align: center

.. _query join:

Join(Expanded Query)
====================

Join is called expanded query in ZStack; it allows users to query a resource by fields that are neither primitive nor nested but
other resources' fields that have relation to this resource; those fields are called expanded fields in ZStack's terms.

For example, to find the parent L3 network of a VM nic having an EIP with VIP 17.16.0.53::

    QueryL3Network vmNic.eip.vipIp=17.16.0.53

here L3 network inventory has no field called 'vmNic.eip.vipIp'; however, it has a relation to VM nic inventory that has a relation to EIP inventory; so we can
construct an expanded query that spans to three inventories: L3 network inventory, VM nic inventory, and EIP inventory. Thanks for this nuclear weapon, ZStack
has around four millions query conditions and countless combinations of conditions. Let's see a more complex and artificial example::

    QueryVolumeSnapshot volume.vmInstance.vmNics.l3Network.l2Network.attachedClusterUuids?=13238c8e0591444e9160df4d3636be82

This complex query is to find volume snapshots created from volumes of VMs that have nics on L3 networks whose parent L2 networks are
attached to a cluster of uuid equal to 13238c8e0591444e9160df4d3636be82. Though users will barely do such a query, it shows the power of the query APIs.

.. note:: Check query operations in each chapter for expanded queries a resource can make, or use CLI auto-completion as a reminder.

Query List
==========

When a field is a list, it can contain primitive types such as int, long, string or nested inventories. Querying list has nothing special; we have this section
to remind you that don't incorrectly think you can only use 'in'(?=) and 'not in'(!?=) when querying a list field; in fact, you can use all comparison operators;
for example::

    QueryL3Network dns~=72.72.72.%

is to find all L3 networks that have DNS like 72.72.72.*::

    QueryL3Network ipRanges.startIp=192.168.0.10

is to find all L3 networks whose IP ranges starting with IP 192.168.0.10.

.. _query with tags:

Query Tags
==========

In section :ref:`tags <tag>` you will see every resource can have user tags and system tags both of which can be a part of query conditions.
ZStack uses two special fields: *__userTag__* and *__systemTag__* for query; for example::

    QueryVmInstance __userTag__?=web-tier-VMs

::

    QueryHost __systemTag__?=os::distribution::Ubuntu managementIp=192.168.0.212

operators >, >=, <, <= only return resources that have tags matching specified conditions; 'is not null' returns resources that have tags;
'is null' returns resources that have no tags; !=, 'not in', 'not like' return resources that have tags not matching conditions as well as resources that have no tags.

.. note:: If you want to make negative comparison operators(!=, 'not in', 'not like') not to return resources that have no tags, you can use them with 'not null'.
          For example::

              QueryVmInstance __userTag__!=database  __userTag__!=null

          is to find VMs that have user tags not equal to 'database'.

Avoid Loop Query
================

Most ZStack resources have bi-direction expanded queries, for example,  hosts have an expanded query to clusters and clusters also have an expanded
query to hosts. This makes it's possible to query a resource from any directions, which may also lead to loop queries. For example::

    QueryHost vmInstance.vmNics.eip.vmNic.vmInstance.uuid=d40e459b97db5a63dedaffcd05cfe3c2

is a loop query, it does the thing equal to::

    QueryHost vmInstance.uuid=d40e459b97db5a63dedaffcd05cfe3c2

Behaviors of loop queries is undefined; you may or may not get the correct results. Please avoid loop query in your practice.

Use Query Efficiently
=====================

Query APIs are powerful that you can do the same thing by different routes. For example, to find VMs that are running on the
host of UUID e497e90ab1e64db099eea93f998d525b, you can either do::

    QueryVmInstance hostUuid=e497e90ab1e64db099eea93f998d525b

or::

    QueryVmInstance host.uuid=e497e90ab1e64db099eea93f998d525b

The first one is more efficient, because it queries a primitive field which only involves the VM table; the later one is an expanded
query which joins both VM table and host table. When your query condition is a UUID, it's always suggested querying the primitive field instead of the sub-field
of an expanded field.


--------
Examples
--------

Normal Query
============

::

    QueryL3Network name=L3-SYSTEM-PUBLIC

Query Count
===========

::

    QueryL3Network name=L3-SYSTEM-PUBLIC count=true


Normal Query With Count
=======================

::

    QueryL3Network name=L3-SYSTEM-PUBLIC replyWithCount=true


Set Limit
=========

::

    QueryL3Network l2NetworkUuid=33107835aee84c449ac04c9622892dec limit=10

Set Start
=========

::

    QueryL3Network l2NetworkUuid=33107835aee84c449ac04c9622892dec start=10 limit=100


.. note:: Using start and limit, UI can implement pagination.


Select Fields
=============

::

    QueryL3Network fields=name,uuid l2NetworkUuid=33107835aee84c449ac04c9622892dec


.. note:: Only primitive fields can be selected.

Sort
====

::

    QueryL3Network l2NetworkUuid=33107835aee84c449ac04c9622892dec sortBy=createDate sortDirection=desc

.. note:: Only primitive field can be used as sorted field.




