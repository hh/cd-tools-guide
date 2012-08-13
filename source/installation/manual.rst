Manual Installation Steps
=========================


Once you have your CD server up and running, there are a few things that you'll need to do manually to get all the pieces of the workflow knitted together.  

Eventually these may end up being automated, of course!

Adding the First User to Gerrit
-------------------------------
The Gerrit server needs an initial administrative user to get started.  

In a web browser, go to the Gerrit server using the hostname included in your ``jenkins`` role as the ``['cd-tools']['gerrit']['front_end_url']``. Click "register" in the upper right corner. The current version of the toolset integrates with OpenID for authentication, so you can use an OpenID linked account to establish your user on the Gerrit server.

The first user becomes the administrative user for the Gerrit system. This user is able to create additional users and delegate administrative privileges to them. 

Add an ssh key to the administrative user's account on Gerrit. You can create a new key or use an existing key.

.. image:: ../images/add_user_to_gerrit.png 
TODO: screencap

Set the Default Permissions for "All-Projects" in Gerrit
--------------------------------------------------------

Part of the initial set up of Gerrit requires that the default permissions be set for "All-Projects".  The following image shows how those permissions should be set for your Gerrit installation:

.. image:: ../images/gerrit_all_projects_perms.png
  :alt: Initial Gerrit permissions for All-Projects

Other permissions combinations are available. Gerrit has an extensive authorization system for projects it manages.  For more discussion of Gerrit's settings, see the Gerrit website.

http://code.google.com/projects/gerrit TODO: CHECK THIS URL

Create the Non-Interactive Users Group
--------------------------------------

Create a new authorization group in Gerrit.  This group will be for non-interactive users like the Jenkins server to access the Gerrit repos.

TODO:  Add instructions for group add

Add git Keys for Git Repo Replication
-------------------------------------

Adding github keys to your Gerrit server will allow the it to replicate its repositories to a central github server or a public server like github.com. Developers will be able to clone and pull from the github repository, but only Gerrit will push to these repositories.  

TODO: screen cap this set up


Make sure you add the ssh host key for git@github.com to Gerrit's ssh known_hosts, and have restarted Gerrit


TODO:  verify file location for this config gerrit's .gitconfig?

Configure replication


.. code-block:: ruby

    [remote "github"]
      url = git@github.com:#{githubuser}/#{repo-name}
      push = +refs/heads/*:refs/heads/*
      push = +refs/tags/*:refs/tags/*


Creating a client.pem for the Jenkins Server
--------------------------------------------

In order for your Continuous Deployment server to talk to your Chef server, it needs to have a client key within the organization it will be working with.  This will be a separate client id from the server's own chef-client configuration. This piece is actually managed by Jenkins, and not Gerrit, so we create a client in the organization called "jenkins" with knife:

.. code-block:: ruby
  
  knife node create jenkins

This command will open an editor buffer.  You don't need to set anything in the node, just save it.  When your buffer session ends, the Chef server will return a public key.  Save this key into the file :file:`/var/lib/jenkins/.chef`. **Note that this location is different from the system's chef-client configuration**.

Then on the Chef server, add ``jenkins`` to the Admin group of the org the server will be working in.

TODO: Screen cap this

Initialize the Jenkins User for Gerrit
--------------------------------------

Gerrit and Jenkins will communicate using Gerrit's ssh daemon, which runs on port 29418.  To create the jenkins user's Gerrit account, run the following command.  Substitute the administrative user you created above for the ``USER@review.local`` and change the hostname if you have altered what is set in the ``jenkins`` role.

.. code-block:: ruby

  cat /var/lib/jenkins/.ssh/id_rsa.pub | ssh -p29418 USER@review.local gerrit create-account --email 'jenkins@jenkins.local' --ssh-key - --full-name Jenkins jenkins

In the Gerrit web ui, add the ``jenkins`` user to the "Non-Interactive Users" group so it will have the appropriate permissions.

From the command line on the CD server, become the jenkins user and run the following command to accept the ssh key configuration:

.. code-block:: ruby

  sudo su - jenkins
  ssh -p29148 jenkins
  yes



Build the First Project into Gerrit
-----------------------------------

The first project that will be managed by Gerrit and built with Jenkins will be the project to continuously deploy the continuous deployment tools, Gerrit and Jenkins themselves, along with all of the dependencies.

Log into the Gerrit web ui, as the administrative user you created earlier.  

Create a new project, called ``cd-tools``. 

Click Projects -> Create New -> Inherit Rights

You won't need to make an initial commit; you already have a repository to start from.

Select ``merge if necessary``.  This setting allows Gerrit to make simpler merges on behalf of the developers without manual intervention.  You can change this setting for other projects, but it should be fine for the ``cd-tools`` project.

Choose ``Require Change ID``. This setting allows you to compress all commits that fail basic syntax and foodcritic tests into the final good commit.  Reviewers will not have to approve all failed commits, only the last good commit that works and passes these tests.  The patchsets will still be recorded, but only the working commits will be passed on to review stage.


