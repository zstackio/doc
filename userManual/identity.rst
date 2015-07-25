.. _identity:

========
Identity
========

.. contents:: `Table of contents`
   :depth: 6


--------
Overview
--------

ZStack's identity service provides access control to ZStack resources for users. The system consists of concepts of
account, user, group, policy, and quota. A global picture of the identity system is like:

.. image:: identity.png
   :align: center

.. _account:

Account
=======

To manipulate resources, people need to create accounts that are the root identity to own all their resources. There
are two types of accounts: admin and normal. Admin accounts, which have unlimited permissions, are owned by administrators.
Normal accounts, which have only permissions to VM, instance offerings, disk offerings, L3 networks, images and so on, are created
by admin accounts to allow people to manipulate those resources.

APIs are categorized in to admin-only APIs and non-admin APIs. A list of admin-only APIs can be found at :ref:`admin-only APIs <admin-only APIs>`,
and a list of non-admin APIs can be found at :ref:`non-admin APIs <non-admin APIs>`.

Account Inventory
-----------------

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
     - 0.8
   * - **name**
     - account name. see :ref:`resource properties`
     -
     -
     - 0.8
   * - **description**
     - see :ref:`resource properties`
     - true
     -
     - 0.8
   * - **createDate**
     - see :ref:`resource properties`
     -
     -
     - 0.8
   * - **lastOpDate**
     - see :ref:`resource properties`
     -
     -
     - 0.8

.. note:: The password will not be shown in the API returns for security reason.

Example
-------

::

  {
    "inventory": {
        "createDate": "Jul 22, 2015 10:18:34 AM",
        "lastOpDate": "Jul 22, 2015 10:18:34 AM",
        "name": "frank",
        "uuid": "3153a08ab21f46ca9e8b40ecfeec4255"
    }
  }


Users
=====

As an non-admin account has unlimited permissions to all resources it owns, people may create users to have finely-grained
permission control. Users can only perform APIs assigned by :ref:`policies <policy>`.

User Inventory
--------------

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
     - 0.8
   * - **name**
     - user name, see :ref:`resource properties`
     -
     -
     - 0.8
   * - **description**
     - see :ref:`resource properties`
     - true
     -
     - 0.8
   * - **accountUuid**
     - uuid of the owner account
     -
     -
     - 0.8
   * - **createDate**
     - see :ref:`resource properties`
     -
     -
     - 0.8
   * - **lastOpDate**
     - see :ref:`resource properties`
     -
     -
     - 0.8

.. note:: The password will not be shown in the API returns for security reason.

Example
-------

::

  {
    "inventory": {
        "accountUuid": "36c27e8ff05c4780bf6d2fa65700f22e",
        "createDate": "Jul 22, 2015 10:21:50 AM",
        "lastOpDate": "Jul 22, 2015 10:21:50 AM",
        "name": "user1",
        "uuid": "68ebcf6260c94adab9dcce9e059e0025"
    }
  }

Groups
======

Accounts can create groups to aggregate users. By assigning policies to groups, accounts can grant the same permissions
to a group of users.

Group Inventory
---------------

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
     - 0.8
   * - **name**
     - group name, see :ref:`resource properties`
     -
     -
     - 0.8
   * - **description**
     - see :ref:`resource properties`
     - true
     -
     - 0.8
   * - **accountUuid**
     - uuid of the owner account
     -
     -
     - 0.8
   * - **createDate**
     - see :ref:`resource properties`
     -
     -
     - 0.8
   * - **lastOpDate**
     - see :ref:`resource properties`
     -
     -
     - 0.8

Example
-------

::

  {
    "inventory": {
        "accountUuid": "36c27e8ff05c4780bf6d2fa65700f22e",
        "createDate": "Jul 22, 2015 10:23:02 AM",
        "name": "group1",
        "uuid": "0939fc6f772d44d6a8f9d45c89c2a716"
    }
  }

Policies
========

Polices are permissions that define what APIs users can perform. A policy consists of an array of :ref:`statements <statements>` each of which
defines permissions to APIs.

Policy Inventory
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
     - 0.8
   * - **name**
     - policy name, see :ref:`resource properties`
     -
     -
     - 0.8
   * - **description**
     - see :ref:`resource properties`
     - true
     -
     - 0.8
   * - **accountUuid**
     - uuid of the owner account, see :ref:`account <account>`
     -
     -
     - 0.8
   * - **statements**
     - a list of :ref:`statements <statements>` defining API permissions
     -
     -
     - 0.8
   * - **createDate**
     - see :ref:`resource properties`
     -
     -
     - 0.8
   * - **lastOpDate**
     - see :ref:`resource properties`
     -
     -
     - 0.8

Example
-------

::

  {
    "inventories": [
        {
            "accountUuid": "3153a08ab21f46ca9e8b40ecfeec4255",
            "name": "DEFAULT-READ-3153a08ab21f46ca9e8b40ecfeec4255",
            "statements": [
                {
                    "actions": [
                        ".*:read"
                    ],
                    "effect": "Allow",
                    "name": "read-permission-for-account-3153a08ab21f46ca9e8b40ecfeec4255"
                }
            ],
            "uuid": "b5169828533b47988a0d09f262b5769c"
        }
    ]
  }

.. _statements:

Statements:
-----------

A statement is a JSON text, containing a list of string matching API identities and an effect: *Allow* or *Deny*. A statement looks like:

::

  {
    "actions": [
        ".*:read",
        "instance:APICreateVmInstanceMsg"
    ],
    "effect": "Allow",
    "name": "read-permission-for-account-3153a08ab21f46ca9e8b40ecfeec4255"
  }

**actions** is a list of action strings that match one or more API identities. An API identity is a string in format of ***api_category:api_name*** that uniquely
identifies an API. An action string can be a full identity like *instance:APICreateVmInstanceMsg* that only matches one API,
or a regular expression that matches multiple APIs, for example, ***instance:.**** will match all APIs under the category ***instance***.
Most of APIs have only one identity that is ***api_category:api_name***; some APIs have more identities so people can use regular expressions
to match a group of APIs.

**effect** tells the decision when a action string matches an API call, allow or deny.

.. note:: In this version, all ***read*** APIs have an extra identity in format of ***api_category:read***, for example, instance:read.
          The read APIs are those not performing operations but getting information from ZStack. For example, all query
          APIs are read APIs; for example, the API APIQueryVmInstanceMsg has a default identity *instance:APIQueryVmInstanceMsg*
          and an extra identity *instance:read*.

          A category may have many read APIs. For example, the VM category('instance') has APIQueryVmInstanceMsg, APIQueryVmNicMsg,
          APIGetVmAttachableDataVolumeMsg and so forth. They all have an API identity ***instance:read***. So a statement containing
          such action string can grant all read APIs in VM category to users and groups.

A list of API identities can be found at :ref:`API identities <API identities>`.

Statement Inventory
+++++++++++++++++++

.. list-table::
   :widths: 20 40 10 20 10
   :header-rows: 1

   * - Name
     - Description
     - Optional
     - Choices
     - Since
   * - **name**
     - statement name
     -
     -
     - 0.8
   * - **effect**
     - permission decision
     -
     - - Allow
       - Deny
     - 0.8
   * - **actions**
     - a list of strings to match API identities
     -
     -
     - 0.8


Quota
=====

Admin accounts can use quotas to limit how many resources non-admin accounts can create. When creating a non-admin account,
ZStack automatically assigns default quotas to it, and admins can change them by the API :ref:`UpdateQuota <update quota>`. A list of default quotas
can be found at :ref:`default quotas <default quotas>`.


------------------
Permission Control
------------------

The most exciting thing of identity system is that you can control API permissions, deciding what people can call what
APIs. When people login into ZStack, depending on the way they login, they can get different permissions regarding APIs.

**Administrators**: When login as an admin account, the people can call any APIs.

**Non-admin Account**: When login as a non-admin account, the people can perform any non-admin APIs.

**User**: When login as a user under an account, the people can only perform APIs granted by polices attached to the user or
groups the user is in.

Using users and groups
======================

The best way to limit people's permission in ZStack is only allowing they to login as users. Let's say you are a manager
in a team that needs to apply some VMs in your company's IT infrastructure managed by ZStack. The first
thing is to ask ZStack administrators in your company to create a non-admin account for you; once you get the account,
you can create multiple users and groups with proper polices attached; then you can give those users to your team members
, who can manipulate ZStack resources under the permissions you granted by polices.

An example helps to understand all those stuff, say you want to create below organization for your team:

.. image:: identity2.png
   :align: center

In this organization, you have an infrastructure group responsible for managing VMs; the group has three members:
David, Tony, Frank; you have another operation group for operating the VMs, which also has three users: Lucy, Arhbi, Jeff. The
infrastructure group has permissions to manage VMs' lifecycle while the operation group can only use VMs by accessing their consoles.
In addition, you as the manager have all API permissions of your team's account(ops-team). To create such an organization:

**Create the account ops-team**::

    >>>CreateAccount name=ops-team password=password

.. note:: make sure you login as the admin account to create the account

**Login using the account ops-team**::

    >>>LogInByAccount accountName=ops-team password=password

**Create users**::

    >>>CreateUser name=david password=password

repeat the step to create all users (tony, frank, lucy, arhbi, jeff, mgr)

**Create groups**::

    >>>CreateUserGroup name=infra

repeat the step to create another group(ops)

**Add users to groups**::

    >>>AddUserToGroup userUuid=d7646ae8af2140c0a3ccef2ad8da816d groupUuid=92c523a43651442489f8d2d598c7c3da

.. note:: The userUuid and groupUuid are printed on the screen when you create users and groups

repeat the step to add users into proper groups. infra group(david, tony, frank), ops group(lucy, arhbi, jeff).

**Create polices**

create the first policy allowing to call all VM related APIs::

    >>>CreatePolicy name=vm-management statements='[{"actions":["instance:.*"], "effect":"Allow"}]'

create the second policy only allowing to access VM's console::

    >>>CreatePolicy name=vm-console statements='[{"actions":["instance:APIRequestConsoleAccessMsg"], "effect":"Allow"}]'

create the third policy allowing all APIs::

    >>>CreatePolicy name=all statements='[{"actions":[".*"], "effect":"Allow"}]'

.. warning:: Please note the *statements* field is a JSON string encompassed by **single quotes**, and its contents
             are using **double quotes**. Please follow this convention otherwise the JSON string may not be able to be
             correctly parsed.

**Attach policies to groups**

attach the policy *vm-management* to the infrastructure group::

    >>>AttachPolicyToUserGroup groupUuid=92c523a43651442489f8d2d598c7c3da policyUuid=afb3bfbb911a42e0a662286728e49891

attach the policy *vm-console* to the operation group::

    >>>AttachPolicyToUserGroup groupUuid=0939fc6f772d44d6a8f9d45c89c2a716 policyUuid=3bddf41e2ba6469881a65287879e5d58

.. note:: The policyUuid and groupUuid are printed on the screen when you create groups and policies

**Attach policies to user manager(mgr)**

attach the policy *all* to the manager(user: mgr)::

    >>>AttachPolicyToUser userUuid=d55c5fba4d1b4533961db9952dc15b00 policyUuid=36c27e8ff05c4780bf6d2fa65700f22e

.. note:: The policyUuid and userUuid are printed on the screen when you create the policy and the user

Now your organization is created successfully, your team members can use user credentials to login.

Permission Evaluation
=====================

A policy consists of a list of statements each of which defines permissions(Allow or Deny) to APIs; users can have
multiple polices attached either to themselves or to groups they are in. When users call APIs, it always evaluates
from their polices then to group polices until a decision is made(Allow or Deny). If there is no policy matching an
API, the API will be denied by default.

.. image:: identity3.png
   :align: center


Default Read Policy
===================

When creating a user, a default read policy **(action: .*:read, effect: Allow)** is attached to the new user so the user
can query resources(e.g. VMs, L3 networks).

-------------
Admin Account
-------------

After installing ZStack, an admin account(account name: admin, password: password) is created by default. Administrators can
use this account to create admin users which will have unlimited permissions just like the admin account, in order
to allow different administrators to use own credentials to login. The password of the admin account can be changed
by the API *ResetAccountPassword*.

----------------
Shared Resources
----------------

An account can share resources to other accounts. This is particularly useful in public clouds that the admin account
can pre-defined some templates (e.g. images, instance offerings, disk offerings, l3 networks) so non-admin accounts(usually
registered by customers) can use those templates to create VMs. See API :ref:`ShareResource <share resources>`.

Resources can be shared to specified accounts or all accounts. When the API :ref:`ShareResource <share resources>` is called
with the parameter *toPublic* set to true, the resources specified in *resourceUuids* are shared to all accounts, otherwise
they are shared to accounts specified in *accountUuids*. When you revoke the shared resources by the API :ref:`RevokeSharing <revoke sharing>`,
you can specify *accountUuids* to revoke resources from certain accounts, or can set *toPublic* to true to revoke resources that have
been shared to all accounts.

.. note:: In this version as the concept *role* has not been supported, other accounts can only read shared resources.
          That is to say, other accounts can query shared resources and use them (e.g. use images to create VMs) but cannot
          perform operations on them, for example, other accounts cannot delete a shared image.


----------
Operations
----------

.. _create account:

Create Account
==============

After login, the admin account can use CreateAccount create non-admin accounts. For example::

    CreateAccount name=frank password=123456

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
     - 0.8
   * - **resourceUuid**
     - resource uuid, see :ref:`create resource`
     - true
     -
     - 0.8
   * - **description**
     - resource description, see :ref:`resource properties`
     - true
     -
     - 0.8
   * - **name**
     - account name
     -
     -
     - 0.8
   * - **password**
     - account password
     -
     -
     - 0.8

.. _reset account password:

Create Users
============

An account can user CreateUser to create a user. For example::

    >>>CreateUser name=david password=123456

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
     - 0.8
   * - **resourceUuid**
     - resource uuid, see :ref:`create resource`
     - true
     -
     - 0.8
   * - **description**
     - resource description, see :ref:`resource properties`
     - true
     -
     - 0.8
   * - **name**
     - user name
     -
     -
     - 0.8
   * - **password**
     - user password
     -
     -
     - 0.8

.. _create group:

Create Groups
=============

An account can use CreateUserGroup to create a group. For example::

    >>>CreateUserGroup name=group

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
     - 0.8
   * - **resourceUuid**
     - resource uuid, see :ref:`create resource`
     - true
     -
     - 0.8
   * - **description**
     - resource description, see :ref:`resource properties`
     - true
     -
     - 0.8
   * - **name**
     - group name
     -
     -
     - 0.8

.. _create policy:

Create Polices
==============

An account can use CreatePolicy to create a policy. For example::

    >>>CreatePolicy name=all statements='[{"actions":[".*"], "effect":"Allow"}]'

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
     - 0.8
   * - **resourceUuid**
     - resource uuid, see :ref:`create resource`
     - true
     -
     - 0.8
   * - **name**
     - policy name
     -
     -
     - 0.8
   * - **statements**
     - a JSON string representing :ref:`statements <statements>`
     -
     -
     - 0.8

Add Users into Groups
=====================

An account can use AddUserToGroup to add a user into a group. For example::

    >>>AddUserToGroup userUuid=d7646ae8af2140c0a3ccef2ad8da816d groupUuid=92c523a43651442489f8d2d598c7c3da

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
   * - **userUuid**
     - user uuid
     -
     -
     - 0.8
   * - **groupUuid**
     - group uuid
     -
     -
     - 0.8

Attach Polices to Groups
========================

An account can use AttachPolicyToUserGroup to attach a policy to a group. For example::

    >>>AttachPolicyToUserGroup groupUuid=92c523a43651442489f8d2d598c7c3da policyUuid=afb3bfbb911a42e0a662286728e49891

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
   * - **groupUuid**
     - group uuid
     -
     -
     - 0.8
   * - **policyUuid**
     - policy uuid
     -
     -
     - 0.8

Attach Polices to Users
=======================

An account can use AttachPolicyToUser to attach a policy to a user. For example::

    >>>AttachPolicyToUser userUuid=d55c5fba4d1b4533961db9952dc15b00 policyUuid=36c27e8ff05c4780bf6d2fa65700f22e

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
   * - **userUuid**
     - user uuid
     -
     -
     - 0.8
   * - **policyUuid**
     - policy uuid
     -
     -
     - 0.8

Detach Polices from Groups
==========================

An account can use DetachPolicyFromUserGroup to detach a policy from a group. For example::

    >>>DetachPolicyFromUserGroup groupUuid=f1a092c6914840c9895c564abbc55375 policyUuid=afb3bfbb911a42e0a662286728e49891

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
   * - **groupUuid**
     - group uuid
     -
     -
     - 0.8
   * - **policyUuid**
     - policy uuid
     -
     -
     - 0.8

Detach Polices from Users
=========================

An account can use DetachPolicyFromUser to detach a policy from a user. For example::

    >>>DetachPolicyFromUser policyUuid=36c27e8ff05c4780bf6d2fa65700f22e userUuid=d7646ae8af2140c0a3ccef2ad8da816d

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
   * - **policyUuid**
     - policy uuid
     -
     -
     - 0.8
   * - **userUuid**
     - user uuid
     -
     -
     - 0.8

Reset Account Password
======================

An account can use ResetAccountPassword to reset its password. For example::

    >>>ResetAccountPassword password=password

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
   * - **password**
     - the new password
     -
     -
     - 0.8
   * - **uuid**
     - the uuid of account to reset the password. It's mainly used by the admin account to reset passwords of other
       accounts. For non-admin accounts, this field is ignored as ZStack can figure out the account uuid by the
       current session.
     - true
     -
     - 0.8

Reset User Password
===================

An account or a user can use ResetUserPassword to reset the password. For example::

    >>>ResetUserPassword password=password

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
   * - **password**
     - the new password
     -
     -
     - 0.8
   * - **uuid**
     - the user uuid. It's mainly used by the account to change passwords of users. For user changing own
       password, this field is ignored as ZStack can figure out the user uuid by the current session.
     - true
     -
     - 0.8

Delete Groups
=============

An account can use DeleteUserGroup to delete a group. For example::

    >>>DeleteUserGroup uuid=bb0e50fe0cfa4ec1af1835f9c210ae8e

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
     - 0.8
   * - **uuid**
     - group uuid
     -
     -
     - 0.8

Delete Users
============

An account can use DeleteUser to delete a user. For example::

    >>>DeleteUser uuid=fa4ec1af1835f9c210ae8e

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
     - 0.8
   * - **uuid**
     - user uuid
     -
     -
     - 0.8

Delete Policies
===============

An account can use DeletePolicy to delete a policy. For example::

    >>>DeletePolicy uuid=bb0e50fe0cfa4ec1af1835f9c210ae8e

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
     - 0.8
   * - **uuid**
     - policy uuid
     -
     -
     - 0.8

Delete Accounts
===============

The admin account can use DeleteAccount to delete an non-admin account. For example::

    >>>DeleteAccount uuid=bb0e50fe0cfa4ec1af1835f9c210ae8e

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
     - 0.8
   * - **uuid**
     - account uuid
     -
     -
     - 0.8

.. warning:: After deleting, all resources owned by the account will be deleted as well

.. _update quota:

Update Account Quota
====================

The admin account can use UpdateQuota to update an account's quotas. For example::

    >>>UpdateQuota identityUuid=bb0e50fe0cfa4ec1af1835f9c210ae8e name=vm.num value=100

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
   * - **identityUuid**
     - the account uuid
     -
     -
     - 0.8
   * - **name**
     - quota name
     -
     - - vip.num
       - securityGroup.num
       - l3.num
       - portForwarding.num
       - vm.num
       - vm.cpuNum
       - vm.memorySize
       - volume.data.num
       - volume.capacity
       - eip.num
     - 0.8

.. _share resources:

Share Resources
===============

An account can use ShareResource to share resources to other accounts. For example::

    ShareResource accountUuids=bb0e50fe0cfa4ec1af1835f9c210ae8e,bb0e50fe0cfa4ec1af1835f9c210ae8e resourceUuids=b0662d80cc4945f8abaf6d1096da9eb5,d55c5fba4d1b4533961db9952dc15b00

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
   * - **accountUuids**
     - a list of account uuids to which the resources are shared. If omitted, the *toPublic* must be set to true
     - true
     -
     - 0.8
   * - **resourceUuids**
     - a list of resource uuids
     -
     -
     - 0.8
   * - **toPublic**
     - if set to true, resources are shared to all accounts
     - true
     - - true
       - false
     - 0.8

.. _revoke sharing:

Revoke Shared Resources
=======================

An account can use RevokeResourceSharing to revoke shared resources from accounts. For example::

    RevokeResourceSharing accountUuids=bb0e50fe0cfa4ec1af1835f9c210ae8e resourceUuids=b0662d80cc4945f8abaf6d1096da9eb5,d55c5fba4d1b4533961db9952dc15b00

::

    RevokeResourceSharing all=true accountUuids=bb0e50fe0cfa4ec1af1835f9c210ae8e

::

    RevokeResourceSharing resourceUuids=b0662d80cc4945f8abaf6d1096da9eb5 toPublic=true

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
   * - **accountUuids**
     - the accounts from which the shared resources are revoked. When field *all* is set, this field is ignored,
       as the resources will be revoked from all accounts to which the resources have been shared.
     - true
     -
     - 0.6
   * - **resourceUuids**
     - resources to be revoked from accounts
     -
     -
     - 0.6
   * - **all**
     - if set, the resources will be revoked from all accounts to which the resources have been shared.
     - true
     - - true
       - false
     - 0.6
   * - **toPublic**
     - if the resources are shared with 'toPublic = true' when calling ShareResource, this field must be also set
       to true when revoking.
     - true
     - - true
       - false
     - 0.6

Query Accounts
==============

An account can use QueryAccount query its own, or the admin account can query all accounts. For example::

    >>>QueryAccount name=test

::

    >>>QueryAccount group.name=group1

Primitive Fields of Query
+++++++++++++++++++++++++

see :ref:`account inventory <account inventory>`

Nested And Expanded Fields of Query
+++++++++++++++++++++++++++++++++++

.. list-table::
   :widths: 20 30 40 10
   :header-rows: 1

   * - Field
     - Inventory
     - Description
     - Since
   * - **group**
     - :ref:`group inventory <group inventory>`
     - child group
     - 0.6
   * - **user**
     - :ref:`user inventory <user inventory>`
     - child user
     - 0.6
   * - **policy**
     - :ref:`policy inventory <policy inventory>`
     - child policy
     - 0.6
   * - **quota**
     -
     - child quota
     - 0.6

Query Users
===========

An account can use QueryUser to query users. For example::

    >>>QueryUser name=frank

::

    >>>QueryUser name=frank policy.name=allow

Primitive Fields of Query
+++++++++++++++++++++++++

see :ref:`user inventory <user inventory>`

Nested And Expanded Fields of Query
+++++++++++++++++++++++++++++++++++

.. list-table::
   :widths: 20 30 40 10
   :header-rows: 1

   * - Field
     - Inventory
     - Description
     - Since
   * - **account**
     - see :ref:`account inventory <account inventory>`
     - the parent account
     - 0.6
   * - **group**
     - see :ref:`group inventory <group inventory>`
     - the group the user is in
     - 0.6
   * - **policy**
     - see :ref:`policy inventory <policy inventory>`
     - the policy attached to the user
     - 0.6

Query Policy
============

An account can use QueryPolicy to query policies. For example::

    >>>QueryPolicy name=vm-management

::

    >>>QueryPolicy user.name=frank

Primitive Fields of Query
+++++++++++++++++++++++++

see :ref:`policy inventory <policy inventory>`

Nested And Expanded Fields of Query
+++++++++++++++++++++++++++++++++++

.. list-table::
   :widths: 20 30 40 10
   :header-rows: 1

   * - Field
     - Inventory
     - Description
     - Since
   * - **account**
     - see :ref:`account inventory <account inventory>`
     - the parent account
     - 0.6
   * - **group**
     - see :ref:`group inventory <group inventory>`
     - groups the policy attached
     - 0.6
   * - **user**
     - see :ref:`user inventory <user inventory>`
     - users the policy attached
     - 0.6

Query Groups
============

An account can use QueryUserGroup to query groups. For example::

    >>>QueryUserGroup name=group1

::

    >>>QueryUserGroup user.name=frank

Primitive Fields of Query
+++++++++++++++++++++++++

see :ref:`group inventory <group inventory>`

Nested And Expanded Fields of Query
+++++++++++++++++++++++++++++++++++

.. list-table::
   :widths: 20 30 40 10
   :header-rows: 1

   * - Field
     - Inventory
     - Description
     - Since
   * - **account**
     - see :ref:`account inventory <account inventory>`
     - the parent account
     - 0.6
   * - **user**
     - see :ref:`user inventory <user inventory>`
     - users in the group
     - 0.6
   * - **policy**
     - see :ref:`policy inventory <policy inventory>`
     - the policy attached to the group
     - 0.6

---------
Reference
---------

.. _admin-only APIs:

Admin-only APIs
===============

::

    QueryGlobalConfig
    GetGlobalConfig
    UpdateGlobalConfig
    GetHostAllocatorStrategies
    GetCpuMemoryCapacity
    ChangeInstanceOffering
    IsReadyToGo
    GetPrimaryStorageTypes
    AttachPrimaryStorageToCluster
    GetPrimaryStorageCapacity
    UpdatePrimaryStorage
    QueryPrimaryStorage
    ChangePrimaryStorageState
    SyncPrimaryStorageCapacity
    DeletePrimaryStorage
    ReconnectPrimaryStorage
    DetachPrimaryStorageFromCluster
    GetPrimaryStorageAllocatorStrategies
    GetVolumeSnapshotTree
    QueryBackupStorage
    AttachBackupStorageToZone
    GetBackupStorageTypes
    ChangeBackupStorageState
    GetBackupStorageCapacity
    DetachBackupStorageFromZone
    UpdateBackupStorage
    DeleteBackupStorage
    AddNetworkServiceProvider
    AttachNetworkServiceProviderToL2Network
    DetachNetworkServiceProviderFromL2Network
    AttachL2NetworkToCluster
    QueryL2VlanNetwork
    CreateL2VlanNetwork
    DetachL2NetworkFromCluster
    DeleteL2Network
    CreateL2NoVlanNetwork
    UpdateL2Network
    GetL2NetworkTypes
    DeleteSearchIndex
    SearchGenerateSqlTrigger
    CreateSearchIndex
    QueryManagementNode
    CreateMessage
    QueryCluster
    DeleteCluster
    UpdateCluster
    CreateCluster
    ChangeClusterState
    CreateAccount
    LogInByUser
    SessionMessage
    UpdateQuota
    QueryAccount
    LogInByAccount
    ValidateSession
    LogOut
    UpdateZone
    DeleteZone
    CreateZone
    QueryZone
    ChangeZoneState
    ChangeHostState
    ReconnectHost
    UpdateHost
    DeleteHost
    GetHypervisorTypes
    QueryHost
    QueryApplianceVm
    AddIscsiFileSystemBackendPrimaryStorage
    QueryIscsiFileSystemBackendPrimaryStorage
    UpdateIscsiFileSystemBackendPrimaryStorage
    AddLocalPrimaryStorage
    UpdateKVMHost
    AddKVMHost
    AddNfsPrimaryStorage
    QuerySftpBackupStorage
    ReconnectSftpBackupStorage
    UpdateSftpBackupStorage
    AddSftpBackupStorage

.. _non-admin APIs:

Non-admin APIs
==============

::

    UpdateVmInstance
    GetVmAttachableL3Network
    MigrateVm
    StopVmInstance
    GetVmAttachableDataVolume
    QueryVmNic
    AttachL3NetworkToVm
    DestroyVmInstance
    GetVmMigrationCandidateHosts
    QueryVmInstance
    DetachL3NetworkFromVm
    RebootVmInstance
    CreateVmInstance
    StartVmInstance
    ChangeImageState
    UpdateImage
    DeleteImage
    CreateDataVolumeTemplateFromVolume
    CreateRootVolumeTemplateFromRootVolume
    QueryImage
    CreateRootVolumeTemplateFromVolumeSnapshot
    AddImage
    RequestConsoleAccess
    BackupDataVolume
    AttachDataVolumeToVm
    UpdateVolume
    QueryVolume
    CreateDataVolumeFromVolumeSnapshot
    CreateDataVolumeFromVolumeTemplate
    DetachDataVolumeFromVm
    CreateDataVolume
    GetDataVolumeAttachableVm
    GetVolumeFormat
    DeleteDataVolume
    CreateVolumeSnapshot
    ChangeVolumeState
    DeleteDiskOffering
    QueryInstanceOffering
    UpdateInstanceOffering
    CreateInstanceOffering
    CreateDiskOffering
    DeleteInstanceOffering
    ChangeInstanceOfferingState
    QueryDiskOffering
    UpdateDiskOffering
    ChangeDiskOfferingState
    QueryVolumeSnapshotTree
    DeleteVolumeSnapshot
    UpdateVolumeSnapshot
    DeleteVolumeSnapshotFromBackupStorage
    QueryVolumeSnapshot
    RevertVolumeFromSnapshot
    BackupVolumeSnapshot
    AddDnsToL3Network
    CreateL3Network
    GetFreeIp
    UpdateL3Network
    DeleteIpRange
    ChangeL3NetworkState
    AddIpRange
    GetL3NetworkTypes
    AddIpRangeByNetworkCidr
    QueryIpRange
    RemoveDnsFromL3Network
    GetIpAddressCapacity
    DeleteL3Network
    UpdateIpRange
    QueryL3Network
    AttachNetworkServiceToL3Network
    QueryNetworkServiceL3NetworkRef
    QueryNetworkServiceProvider
    GetNetworkServiceTypes
    QueryL2Network
    QueryUserTag
    QuerySystemTag
    DeleteTag
    CreateUserTag
    CreateSystemTag
    QueryTag
    AttachPolicyToUserGroup
    RemoveUserFromGroup
    AttachPolicyToUser
    ResetUserPassword
    AddUserToGroup
    QueryQuota
    ShareResource
    DeleteAccount
    CreateUserGroup
    CreateUser
    DetachPolicyFromUserGroup
    QueryPolicy
    QueryUser
    DeletePolicy
    RevokeResourceSharing
    ResetAccountPassword
    DeleteUser
    DeleteUserGroup
    CreatePolicy
    DetachPolicyFromUser
    QueryUserGroup
    ReconnectVirtualRouter
    QueryVirtualRouterOffering
    CreateVirtualRouterOffering
    QueryVirtualRouterVm
    AttachPortForwardingRule
    DetachPortForwardingRule
    GetPortForwardingAttachableVmNics
    ChangePortForwardingRuleState
    UpdatePortForwardingRule
    CreatePortForwardingRule
    QueryPortForwardingRule
    DeletePortForwardingRule
    DetachEip
    GetEipAttachableVmNics
    UpdateEip
    QueryEip
    ChangeEipState
    DeleteEip
    CreateEip
    AttachEip
    ChangeSecurityGroupState
    DetachSecurityGroupFromL3Network
    DeleteSecurityGroupRule
    CreateSecurityGroup
    QueryVmNicInSecurityGroup
    QuerySecurityGroup
    AddSecurityGroupRule
    QuerySecurityGroupRule
    DeleteSecurityGroup
    UpdateSecurityGroup
    DeleteVmNicFromSecurityGroup
    GetCandidateVmNicForSecurityGroup
    AttachSecurityGroupToL3Network
    AddVmNicToSecurityGroup
    DeleteVip
    UpdateVip
    ChangeVipState
    CreateVip
    QueryVip

.. _api identities:

API Identities
==============

::

    ReconnectVirtualRouter: virtualRouter:APIReconnectVirtualRouterMsg

    GetNetworkServiceProvider: l2Network:read, l2Network:APIGetNetworkServiceProviderMsg

    AddDnsToL3Network: l3Network:APIAddDnsToL3NetworkMsg

    DeleteSecurityGroup: securityGroup:APIDeleteSecurityGroupMsg

    AddImage: image:APIAddImageMsg

    QueryUser: identity:read, identity:APIQueryUserMsg

    GetL3NetworkTypes: l3Network:read, l3Network:APIGetL3NetworkTypesMsg

    ShareResource: identity:APIShareResourceMsg

    QueryVirtualRouterOffering: virtualRouter:read, virtualRouter:APIQueryVirtualRouterOfferingMsg

    QueryIpRange: l3Network:read, l3Network:APIQueryIpRangeMsg

    AttachDataVolumeToVm: volume:APIAttachDataVolumeToVmMsg

    QueryUserGroup: identity:read, identity:APIQueryUserGroupMsg

    QueryVmNicInSecurityGroup: securityGroup:read, securityGroup:APIQueryVmNicInSecurityGroupMsg

    CreateSystemTag: tag:APICreateSystemTagMsg

    CreateVip: vip:APICreateVipMsg

    DeleteDiskOffering: configuration:APIDeleteDiskOfferingMsg

    StartVmInstance: instance:APIStartVmInstanceMsg

    GetVmAttachableL3Network: instance:read, instance:APIGetVmAttachableL3NetworkMsg

    DeleteVip: vip:APIDeleteVipMsg

    GetDataVolumeAttachableVm: volume:read, volume:APIGetDataVolumeAttachableVmMsg

    QuerySystemTag: tag:read, tag:APIQuerySystemTagMsg

    AttachL3NetworkToVm: instance:APIAttachL3NetworkToVmMsg

    CreateUserTag: tag:APICreateUserTagMsg

    CreateVmInstance: instance:APICreateVmInstanceMsg

    CreateSecurityGroup: securityGroup:APICreateSecurityGroupMsg

    UpdateVolumeSnapshot: volumeSnapshot:APIUpdateVolumeSnapshotMsg

    QueryDiskOffering: configuration:read, configuration:APIQueryDiskOfferingMsg

    StopVmInstance: instance:APIStopVmInstanceMsg

    CreateEip: eip:APICreateEipMsg

    ChangePortForwardingRuleState: portForwarding:APIChangePortForwardingRuleStateMsg

    UpdateL3Network: l3Network:APIUpdateL3NetworkMsg

    ChangeDiskOfferingState: configuration:APIChangeDiskOfferingStateMsg

    MigrateVm: instance:APIMigrateVmMsg

    ChangeVipState: vip:APIChangeVipStateMsg

    AddIpRange: l3Network:APIAddIpRangeMsg

    CreateDataVolume: volume:APICreateDataVolumeMsg

    CreateDataVolumeFromVolumeSnapshot: volume:APICreateDataVolumeFromVolumeSnapshotMsg

    UpdateImage: image:APIUpdateImageMsg

    QueryVmNic: instance:read, instance:APIQueryVmNicMsg

    QueryTag: tag:read, tag:APIQueryTagMsg

    GetPortForwardingAttachableVmNics: portForwarding:APIGetPortForwardingAttachableVmNicsMsg

    DeleteInstanceOffering: configuration:APIDeleteInstanceOfferingMsg

    AttachPortForwardingRule: portForwarding:APIAttachPortForwardingRuleMsg

    DeletePortForwardingRule: portForwarding:APIDeletePortForwardingRuleMsg

    CreatePortForwardingRule: portForwarding:APICreatePortForwardingRuleMsg

    UpdateIpRange: l3Network:APIUpdateIpRangeMsg

    GetFreeIp: l3Network:read, l3Network:APIGetFreeIpMsg

    ChangeL3NetworkState: l3Network:APIChangeL3NetworkStateMsg

    QueryVip: vip:read, vip:APIQueryVipMsg

    UpdateEip: eip:APIUpdateEipMsg

    QueryVolumeSnapshotTree: volumeSnapshot:read, volumeSnapshot:APIQueryVolumeSnapshotTreeMsg

    DetachDataVolumeFromVm: volume:APIDetachDataVolumeFromVmMsg

    RebootVmInstance: instance:APIRebootVmInstanceMsg

    UpdateInstanceOffering: configuration:APIUpdateInstanceOfferingMsg

    DestroyVmInstance: instance:APIDestroyVmInstanceMsg

    ResetUserPassword: identity:APIResetUserPasswordMsg

    QueryNetworkServiceL3NetworkRef: l3Network:read, l3Network:APIQueryNetworkServiceL3NetworkRefMsg

    CreateL3Network: l3Network:APICreateL3NetworkMsg

    GetNetworkServiceTypes: l3Network:read, l3Network:APIGetNetworkServiceTypesMsg

    GetVmAttachableDataVolume: instance:read, instance:APIGetVmAttachableDataVolumeMsg

    QueryL3Network: l3Network:read, l3Network:APIQueryL3NetworkMsg

    CreateDataVolumeTemplateFromVolume: image:APICreateDataVolumeTemplateFromVolumeMsg

    DeleteSecurityGroupRule: securityGroup:APIDeleteSecurityGroupRuleMsg

    QueryUserTag: tag:read, tag:APIQueryUserTagMsg

    DeleteVolumeSnapshotFromBackupStorage: volumeSnapshot:APIDeleteVolumeSnapshotFromBackupStorageMsg

    CreateDiskOffering: configuration:APICreateDiskOfferingMsg

    QuerySecurityGroup: securityGroup:read, securityGroup:APIQuerySecurityGroupMsg

    QueryVolumeSnapshot: volumeSnapshot:read, volumeSnapshot:APIQueryVolumeSnapshotMsg

    QueryPortForwardingRule: portForwarding:read, portForwarding:APIQueryPortForwardingRuleMsg

    UpdateDiskOffering: configuration:APIUpdateDiskOfferingMsg

    GetCandidateVmNicForSecurityGroup: securityGroup:read, securityGroup:APIGetCandidateVmNicForSecurityGroupMsg

    QueryPolicy: identity:read, identity:APIQueryPolicyMsg

    GetEipAttachableVmNics: eip:APIGetEipAttachableVmNicsMsg

    CreateInstanceOffering: configuration:APICreateInstanceOfferingMsg

    AddIpRangeByNetworkCidr: l3Network:APIAddIpRangeByNetworkCidrMsg

    UpdateVmInstance: instance:APIUpdateVmInstanceMsg

    QueryVirtualRouterVm: virtualRouter:read, virtualRouter:APIQueryVirtualRouterVmMsg

    RequestConsoleAccess: console:APIRequestConsoleAccessMsg

    ChangeEipState: eip:APIChangeEipStateMsg

    QuerySecurityGroupRule: securityGroup:read, securityGroup:APIQuerySecurityGroupRuleMsg

    DetachSecurityGroupFromL3Network: securityGroup:APIDetachSecurityGroupFromL3NetworkMsg

    CreateDataVolumeFromVolumeTemplate: volume:APICreateDataVolumeFromVolumeTemplateMsg

    DeleteDataVolume: volume:APIDeleteDataVolumeMsg

    AddVmNicToSecurityGroup: securityGroup:APIAddVmNicToSecurityGroupMsg

    DeleteVolumeSnapshot: volumeSnapshot:APIDeleteVolumeSnapshotMsg

    DetachEip: eip:APIDetachEipMsg

    DetachPortForwardingRule: portForwarding:APIDetachPortForwardingRuleMsg

    CreateVirtualRouterOffering: virtualRouter:APICreateVirtualRouterOfferingMsg

    RevertVolumeFromSnapshot: volumeSnapshot:APIRevertVolumeFromSnapshotMsg

    DeleteIpRange: l3Network:APIDeleteIpRangeMsg

    UpdateVip: vip:APIUpdateVipMsg

    AttachNetworkServiceToL3Network: l3Network:APIAttachNetworkServiceToL3NetworkMsg

    DeleteTag: tag:APIDeleteTagMsg

    RemoveDnsFromL3Network: l3Network:APIRemoveDnsFromL3NetworkMsg

    DeleteL3Network: l3Network:APIDeleteL3NetworkMsg

    UpdatePortForwardingRule: portForwarding:APIUpdatePortForwardingRuleMsg

    ChangeVolumeState: volume:APIChangeVolumeStateMsg

    QueryVmInstance: instance:read, instance:APIQueryVmInstanceMsg

    GetVmMigrationCandidateHosts: instance:read, instance:APIGetVmMigrationCandidateHostsMsg

    UpdateVolume: volume:APIUpdateVolumeMsg

    QueryL2Network: l2Network:read, l2Network:APIQueryL2NetworkMsg

    BackupVolumeSnapshot: volumeSnapshot:APIBackupVolumeSnapshotMsg

    QueryQuota: identity:read, identity:APIQueryQuotaMsg

    QueryImage: image:read, image:APIQueryImageMsg

    RevokeResourceSharing: identity:APIRevokeResourceSharingMsg

    UpdateSecurityGroup: securityGroup:APIUpdateSecurityGroupMsg

    ChangeImageState: image:APIChangeImageStateMsg

    AddSecurityGroupRule: securityGroup:APIAddSecurityGroupRuleMsg

    QueryVolume: volume:read, volume:APIQueryVolumeMsg

    AttachSecurityGroupToL3Network: securityGroup:APIAttachSecurityGroupToL3NetworkMsg

    DeleteEip: eip:APIDeleteEipMsg

    QueryEip: eip:read, eip:APIQueryEipMsg

    DeleteImage: image:APIDeleteImageMsg

    GetIpAddressCapacity: l3Network:read, l3Network:APIGetIpAddressCapacityMsg

    ChangeInstanceOfferingState: configuration:APIChangeInstanceOfferingStateMsg

    DeleteVmNicFromSecurityGroup: securityGroup:APIDeleteVmNicFromSecurityGroupMsg

    CreateVolumeSnapshot: volumeSnapshot:APICreateVolumeSnapshotMsg

    CreateRootVolumeTemplateFromRootVolume: image:APICreateRootVolumeTemplateFromRootVolumeMsg

    GetVolumeFormat: volume:read, volume:APIGetVolumeFormatMsg

    BackupDataVolume: volume:APIBackupDataVolumeMsg

    CreateRootVolumeTemplateFromVolumeSnapshot: image:APICreateRootVolumeTemplateFromVolumeSnapshotMsg

    QueryInstanceOffering: configuration:read, configuration:APIQueryInstanceOfferingMsg

    ChangeSecurityGroupState: securityGroup:APIChangeSecurityGroupStateMsg

    QueryNetworkServiceProvider: l3Network:read, l3Network:APIQueryNetworkServiceProviderMsg

    AttachEip: eip:APIAttachEipMsg

    DetachL3NetworkFromVm: instance:APIDetachL3NetworkFromVmMsg


.. _default quotas:

Default Quotas
==============

.. list-table::
   :widths: 20 40 20 20
   :header-rows: 1

   * - Name
     - Description
     - Value
     - Since
   * - **vip.num**
     - max number of VIPs
     - 20
     - 0.8
   * - **securityGroup.num**
     - max number of security groups
     - 20
     - 0.8
   * - **l3.num**
     - max number of L3 networks
     - 20
     - 0.8
   * - **portForwarding.num**
     - max number of port forwarding rules
     - 20
     - 0.8
   * - **vm.num**
     - max number of VMs
     - 20
     - 0.8
   * - **vm.cpuNum**
     - max number of VCPU cores
     - 80
     - 0.8
   * - **vm.memorySize**
     - total size of memory
     - 85899345920 bytes (80G)
     - 0.8
   * - **volume.data.num**
     - max number of data volumes
     - 40
     - 0.8
   * - **volume.capacity**
     - total volume capacity of both data volumes and root volumes
     - 10995116277760 bytes (10T)
     - 0.8
   * - **eip.num**
     - max number of EIPs
     - 20
     - 0.8
