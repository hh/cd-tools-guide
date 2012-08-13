Installation Prerequisites
==========================

Before installing the CD server, please read the following pre-requisites.

Server
------

Hardware
~~~~~~~~

TODO: add hardware recs

Networking
~~~~~~~~~~

TODO: double check all ports that would need to be opened through firewalls

* The CD server should be accessible from all networks that contain user workstations.
* The CD server must be able to access the appropriate Chef server on port 443
* If replicating repositories to github.com or to an internal git or github server, the CD server must have access to that host.

Chef Organization
-----------------

A Chef organization must be available for the CD server to interact with.  Adding the CD server to the Chef organization is documented in the :doc:`manual </installation/manual>` section of this document.

Every Chef organization that wishes to use CD with this workflow must have its own dedicated CD server.
