Installation of CD Tools
========================

This section of the documentation is broken up in the following pieces:

- :doc:`Prerequisites </installation/prereqs>`: Prerequisites for your CD build host.
- :doc:`Initializing the Repository and Installing on Your Host</installation/repo>`: What's in this repository and what it's going to do for you.
    * add the role to your host
    * setting up the host names in the role file
- :doc:`Manual Install Steps </installation/manual>`: After running chef-client on your build host, you'll need to do a few things manually to glue all the bits together.
- :doc:`The CD Server After Install </installation/server>`: You'll have a number of applications on the CD Server after chef-client runs successfully.
    * postgresql
    * gerrit
    * jenkins
    * port numbers
- :doc:`Troubleshooting </installation/troubleshooting>`: Common errors.
- :doc:`Workstation Set Up </installation/workstation>`: Installing the needed tools on user workstations to make use of the CD server.

Contents:

.. toctree::
   :maxdepth: 2

   installation/prereqs
   installation/repo
   installation/manual
   installation/server
   installation/troubleshooting
   installation/workstation
