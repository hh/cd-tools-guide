CD Tools Chef Repository
========================

Included Cookbooks
------------------

There are about 20 cookbooks in the ``cd-tools`` repo.  Many of them are included to satisfy dependencies in the various primary cookbooks.

Jenkins
~~~~~~~

The ``jenkins`` cookbook installs and configures the Jenkins server.  It requires a webserver of some sort; we have chosen ``nginx`` in this version of the tools, but ``apache`` can also be used.  Jenkins also requires ``java``, and ``runit`` for application daemon management, though currently only Debian and Ubuntu package ``runit`` in the default repositories. The ``jenkins::jnlp`` recipe uses ``runit``, but using Jenkins with a regular webserver does not.

Gerrit
~~~~~~

Since Gerrit interacts with a number of subsystems in the CD toolchain, it has a couple of additional dependencies.

* The ``gerrit`` cookbook installs and configures the ``gerrit`` software.  
* Gerrit needs a webserver to run, this version of the toolset uses ``nginx``.  
* Gerrit requires a relational database for its datastore.  Several options are available, but we have chosen ``postgresql`` in this version.  You can see the Gerrit documentation for discussion of the other databases that can be used and how to configure them for Gerrit.
* Gerrit keeps all of your source code in its own internal ``git`` repository, which it can then replicate to a centralized ``git`` server.  The ``git`` cookbook is required for this essential functionality.

build-essential
~~~~~~~~~~~~~~~

This repository includes the ability to build ``nginx`` from source.  If you choose to change the configuration to Apache, this would not be necessary. Also, if your servers are able to connect to external repositories on the Internet, you could change this set up to download packages for ``nginx`` rather than building from source. 

Foodcritic also requires ``build-essential``, so you won't remove it from the runlist if you change the webserver configuration.

foodcritic
~~~~~~~~~~

Foodcritic is a lint tool for Chef recipes.  When using Chef and this contiuous deployment workflow, ``foodcritic`` is a key tool for checking syntax and best practice in your Chef recipes.

See http://acrmp.github.com/foodcritic/ for documentation on Foodcritic.

cd-tools
~~~~~~~~

Finally, ``cd-tools`` configures Jenkins to work with the pipelines. The cookbook will contain the databags for your Jenkins pipelines that are documented in the :doc:`pipeline </pipeline>` section of this document.

The "jenkins" Role
------------------

The included ``jenkins.json`` and ``jenkins.rb`` files in the ``roles`` directory.  Choose one of these formats and be consistent about any changes made; you might want to delete the file format you aren't using in order to avoid confusion.

There are several hostname-related settings in the role's ``default_attributes``.  Jenkins and Gerrit are configured as named virtual hosts in the webserver, so you can access each of them as "jenkins.*" and "review.*".  Adding CNAMES to your actual server is fine; you'll also want to add the hostnames to ``/etc/hosts`` on the server itself.

default_attributes
~~~~~~~~~~~~~~~~~~

build-essential

* ``['compiletime']`` : this attribute is set to ``true`` so that later recipes in the runlist have access to build tools during the convergence part of the ``chef-client`` run.  The execution steps for installing build tool packages will take place in the earlier stages of the ``chef-client`` run.

jenkins

* ``['http_proxy']['host_name']`` : Set the appropriate ``host_name`` for your server. This will be the named virtual host in the webserver configuration.
* ``['http_proxy']['variant']`` : Change this setting if you choose not to use ``nginx``.
* ``['server']['plugins']`` : The list of plugins to be installed in your Jenkins server. If your server does not have access to the internet, make sure you download the hpi files for each plugin to your internal repository and make them available for download.

gerrit

* ``['http_proxy']['host_name']`` : This will be the named virtual host for the webserver configuration.
* ``['canonical_url']`` : the full url to access the Gerrit server, including "http://"

cd-tools

TODO: ask if this hostname and the above host_name can be spelled the same?? or the ['gerrit'] attrs read into cd-tools?
* ``['gerrit']['hostname']`` : The Gerrit named virtual host name. 
* ``['gerrit']['front_end_url']`` : as above
  

Making Changes Appropriate for Your Environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Pipeline Data Bags
------------------
