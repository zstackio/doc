.. _introduction:

============
Introduction
============

.. contents:: `Table of contents`
   :depth: 6

--------
Overview
--------

Depending on the scale of a cloud, a ZStack setup can be as simple as a single Linux machine running one ZStack management node,
or a cluster of Linux servers running multiple ZStack management nodes.

The Setup of A Single Management Node
=====================================

.. image:: single-node-deployment.png
   :alt: a single management node deployment
   :align: center
   :height: 500px
   :width: 600px

In the simplest setup, all ZStack software and third party dependencies are installed on a single Linux server.
A typical setup includes five parts:

- `RabbitMQ Message Server <http://www.rabbitmq.com/>`_: The central message bus ZStack services use for communication.
- `MySQL Database <http://www.mysql.com/>`_: The database ZStack stores metadata for resources in the cloud.
- `Ansible <http://www.ansible.com/home>`_: The configuration management tool ZStack uses to remotely deploy and upgrade agents.
- ZStack Management Node: The main process encompassing all ZStack orchestration services.
- ZStack UI Server: A web server providing user interface for end users.

Besides, several python agents, which need deploying to local or remote machines at runtime, are packaged in the WAR file of
ZStack management node and are deployed using Ansible.

Because of ZStack's asynchronous architecture, a single management node is normally enough to manage a big cloud that may have tens of thousands
of physical servers, hundreds of thousands of virtual machines(virtual machine is referred as VM in future chapters), and tens of thousands of concurrent API requests.
However, in case of high availability and scaling out for a super large cloud, a setup of multiple management nodes is still valuable.

A Deployment of Multiple Management Nodes
=========================================

.. image:: multiple-nodes-deployment.png
   :alt: a multiple management nodes deployment
   :align: center
   :height: 1000px

In this multiple nodes setup, the RabbitMQ server and MySQL database server are moved out to dedicated Linux machines; ZStack management nodes
and Ansible are installed on every Linux server; multiple management nodes share the same RabbitMQ message server and MySQL database. ZStack UI servers,
which also send API requests to management nodes through RabbitMQ, are deployed behind a load balancer which dispatches requests from users.

In terms of clustering RabbitMQ and MySQL, admin can setup two RabbitMQ servers and an additional slave MySQL database server.

ZStack's World View of A Cloud
==============================

IaaS software usually use some terms such as 'zone', 'cluster' to describe how facilities in a data center make up a cloud, so does ZStack.
To reduce the learning curve and to eliminate misunderstandings caused by self-created terms, ZStack tries to use terminologies that have been well known in existing IaaS software
and datacenters as much as possible.

Below is a diagram that how ZStack maps facilities of datacenters into its own language.

.. image:: word-view1.png
   :alt: word view1
   :align: center
   :height: 1000px

A datacenter, in ZStack's terms, is organized as follows:

- **Zone**:

  A zone is a logic group of resources, such as clusters, L2 networks, primary storage. ZStacks uses zones to define visibility boundary between resources.
  For example, a primary storage in zone A is not visible to a cluster in zone B. In practice, zones can also be used as isolated domains for fault tolerance, just as
  Amazon EC2 availability zones.

- **Cluster**:

  A cluster is a logic group of hosts. Hosts in the same cluster must have the same operating systems(hypervisor) and network configurations. Clusters are also known
  as host aggregations or host pools in other IaaS software.

- **Host**:

  A host is a physical server installed with an operating system(hypervisor) to run VMs.

- **L2 Network**:

  A L2 network is an abstraction of a layer 2 broadcast domain. Any technology, as long as providing a layer 2 broadcast domain,
  can be a type of L2 Network in ZStack. For example, VLAN, VxLan, or SDN technologies that create layer 2 overlay on layer 3 network.

- **Primary Storage**:

  A primary storage provides disk spaces to store VMs' volumes which will be accessed by VMs' operating system during running. Primary Storage can be either filesystem
  based like NFS or block storage based like ISCSI.

- **Backup Storage**:

  A backup Storage provides disk spaces to store images and volume snapshots both of which can be used to create volumes. Files on backup storage are not directly accessible
  to VMs; before being used, they need to be downloaded to primary storage. Backup Storage can either be filesystem based or object storage based.

ZStack uses a so-called 'attaching strategy' to describe relationships between resources, for example, a cluster can be attached with multiple primary storage and L2 networks, vice versa.
See related chapters(e.g. primary storage, L2 network) for details.

A data center can have one or more zones. A diagram of multiple zones looks like:

.. image:: world-view2.png
   :alt: world view2
   :align: center
   :height: 1000px


.. note:: For simplicity, the diagram omits some facilities like aggregation switches, core switches, routers, load balancer, firewalls and so on.

Besides above terms describing datacenter facilities, there are some other terms such as VM, instance offering, disk offering, which describe
virtual resources; check details in relevant chapters.
