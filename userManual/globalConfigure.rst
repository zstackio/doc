.. _global configure:

=====================
Global Configurations
=====================

.. contents:: `Table of contents`
   :depth: 6

--------
Overview
--------

Admins can use global configurations to configure a variety of features; all global configurations come with a default value; updating
a global configuration doesn't require to restart the management node.

We arrange resource related global configurations in each chapter, for those configurations that don't specifically categorise
in any resource we list them in this chapter.

---------
Inventory
---------

.. list-table::
   :widths: 20 40 10 20 10
   :header-rows: 1

   * - Name
     - Description
     - Optional
     - Choices
     - Since
   * - **category**
     - configuration category
     -
     -
     - 0.6
   * - **description**
     - configuration description
     -
     -
     - 0.6
   * - **name**
     - configuration name
     -
     -
     - 0.6
   * - **defaultValue**
     - default value
     -
     -
     - 0.6
   * - **value**
     - current value
     -
     -
     - 0.6

Example
=======

::

        {
            "category": "identity",
            "defaultValue": "500",
            "description": "Max number of sessions management server accepts. When this limit met, new session will be rejected",
            "name": "session.maxConcurrent",
            "value": "500"
        }


----------
Operations
----------

Update Global Configurations
============================

Admins can use UpdateGlobalConfig to update a global configuration. For example::

    UpdateGlobalConfig category=host name=connection.autoReconnectOnError value=true


--------------------
Other Configurations
--------------------

For configurations that don't categorise in individual chapter.


.. _cloudBus.statistics.on:

statistics.on
=============

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **statistics.on**
     - cloudBus
     - false
     - - true
       - false

Whether enables statistics that count time consuming of each message through JMX.


.. _node.heartbeatInterval:

node.heartbeatInterval
======================

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **node.heartbeatInterval**
     - managementServer
     - 5
     - > 0

The interval that each management node writes heartbeat to database, in seconds.


.. _node.joinDelay:

node.joinDelay
==============

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **node.joinDelay**
     - managementServer
     - 0
     - >= 0

If non zero, each management node will delay random seconds from 0 to 'node.joinDelay' before publishing join event on the message bus. This
avoid storm of join event when a large number of management nodes start at the same time.


.. _configuration.key.public:

key.public
==========

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **key.public**
     - configuration
     - see your database
     -

ZStack will inject this public SSH key to Linux servers that need to deploy agents; in this version, the Linux servers include KVM host, virtual router VMs,
SFTP backup storage. After injecting, ZStack will use :ref:`key.private <configuration.key.private>` when needing SSH login.


.. _configuration.key.private:

key.private
===========

.. list-table::
   :widths: 20 30 20 30
   :header-rows: 1

   * - Name
     - Category
     - Default Value
     - Choices
   * - **key.private**
     - configuration
     - see your database
     -

The private SSH key ZStack uses to SSH login remote Linux servers; see :ref:`key.public <configuration.key.public>`.
