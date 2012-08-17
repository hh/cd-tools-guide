Installation Prerequisites
==========================

Before installing the CD server, please read the following pre-requisites.

Server
------

The current version of these recipes are configured to work on RHAT 6. Make sure you aren't running ``selinux``; these recipes are *NOT* selinux-safe.  See below for notes on ``iptables`` firewalling.

Hardware
~~~~~~~~

TODO: add hardware recs

Networking
~~~~~~~~~~

TODO: double check all ports that would need to be opened through firewalls

* The CD server should be accessible from all networks that contain user workstations.
* The CD server must be able to access the appropriate Chef server on port 443
* If replicating repositories to github.com or to an internal git or github server, the CD server must have access to that host.

IPTables
~~~~~~~~

The Gerrit and Jenkins run on port 80.

The Gerrit ssh server runs on 29418.  You will want this port to be open to the Jenkins server and to all user workstations.

Other apps on the server are accessed over localhost.

Chef Organization
-----------------

A Chef organization must be available for the CD server to interact with.  Adding the CD server to the Chef organization is documented in the :doc:`manual </installation/manual>` section of this document.

Every Chef organization that wishes to use CD with this workflow must have its own dedicated CD server.
