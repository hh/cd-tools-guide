Manual Installation Steps
=========================


Once you have your CD server up and running, there are a few things that you'll need to do manually to get all the pieces of the workflow knitted together.  

Eventually these may end up being automated, of course!

Adding the First User to Gerrit
-------------------------------
The Gerrit server needs an initial administrative user to get started.  

In a web browser, go to the Gerrit server using the hostname included in your ``jenkins`` role as the ``['cd-tools']['gerrit']['front_end_url']``. Click "register" in the upper right corner. The current version of the toolset integrates with OpenID for authentication, so you can use an OpenID linked account to establish your user on the Gerrit server.

.. image:: ../images/gerrit_reg_firstuser.jpg
   :alt: Register the first Gerrit user

The first user becomes the administrative user for the Gerrit system. This user is able to create additional users and delegate administrative privileges to them. Choose a local user name for this user account:

.. image:: ../images/gerrit_username.jpg
   :alt: Local Gerrit User Name

Add an ssh key to the administrative user's account on Gerrit. You can create a new key or use an existing key.

.. image:: ../images/gerrit_ssh_key.jpg
   :alt: Add SSH Key to the Gerrit User

Click "Add":

.. image:: ../images/gerrit_sshkey_added.jpg
   :alt: SSH Key Summary

And then "Continue".  You'll be returned to the main Gerrit page.

You can test the key from the command line, where USER is the username you created above:

.. code-block:: ruby

  [root@review.local .ssh]# ssh -i ~/.ssh/admin_gerritkey -p29418 USER@review.local

  ****    Welcome to Gerrit Code Review    ****

  Hi Bobo T. Clown, you have successfully connected over SSH.

  Unfortunately, interactive shells are disabled.
  To clone a hosted Git repository, use:

  git clone ssh://USER@review.local:29418/REPOSITORY_NAME.git

  Connection to review.local closed.


Create the Non-Interactive Users Group
--------------------------------------

Create a new authorization group in Gerrit.  This group will be for non-interactive users like the Jenkins server to access the Gerrit repos.

In the Gerrit web UI, click on Admin->Groups.  In the ``Create New Group`` box, add "Non-Interactive Users". The Owners are the "Administrators" group.

.. image:: ../images/gerrit_nonintusers_add.jpg
  :alt: Adding the Non-Interactive Users group

Set the Default Permissions for "All-Projects" in Gerrit
--------------------------------------------------------

Part of the initial set up of Gerrit requires that the default permissions be set for "All-Projects".  The following image shows how those permissions should be set for your Gerrit installation:

.. image:: ../images/gerrit_all_projects_perms.jpg
  :alt: Initial Gerrit permissions for All-Projects

Other permissions combinations are available. Gerrit has an extensive authorization system for projects it manages.  For more discussion of Gerrit's settings, see the Gerrit website.

http://code.google.com/p/gerrit/

Add git Keys for Git Repo Mirroring
-----------------------------------

Adding github keys to your Gerrit server will allow it to replicate its repositories to a central github server or a public server like github.com. Developers will be able to clone and pull from the github repository, but only Gerrit will push to these repositories.  

On the Gerrit server, create a new ssh key for the ``gerrit2`` user to be used as a deploy key for github.  Github.com only allows deploy keys to be linked to a single project.  If you want to replicate each project on your Gerrit server to a different project on github.com, the consensus seems to be that you add a new account as a collaborator to the project for your jenkins server. The following section, however, documents a single project mirrored to a single github project.

.. code-block:: ruby

  [root@ip-10-120-241-168 ~]# sudo su - gerrit2
  -bash-4.1$ ssh-keygen -t rsa
  Generating public/private rsa key pair.
  Enter file in which to save the key (/srv/gerrit/site_path/.ssh/id_rsa): 
  Enter passphrase (empty for no passphrase): 
  Enter same passphrase again: 
  Your identification has been saved in /srv/gerrit/site_path/.ssh/id_rsa.
  Your public key has been saved in /srv/gerrit/site_path/.ssh/id_rsa.pub.
  The key fingerprint is:
  e7:eb:49:5a:df:3b:be:f5:fb:f6:43:58:ba:e4:a6:3a gerrit2@ip-10-120-241-168
  The key's randomart image is:
  +--[ RSA 2048]----+
  |                 |
  |                 |
  |                 |
  |               . |
  |        S .   +  |
  |         o   + . |
  |          + o o .|
  |         E + =.oo|
  |        .o*.+.=*O|
  +-----------------+
  -bash-4.1$ cat id_rsa.pub 
  ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAtuuEtEGVuzOpSEpX9jNJd6u1k8YfePc3Miyfwk9bLhPSn7vPELVMUXBObOSaThNQ0a4ZK9qoMxTtDoKSlGYz48IxCETyARPkDZfvB14HmNb3movwCdm+134snMmsEUKmbj+FYrTj+zVVX2KCYkCQjJU8ySPP+YWBX2K4FdtY+JsThHDpceuAPmvAjfw+Sa1CkRBIYNOgkRSG4UoL97zcRZ1byeo8bJUDHWpSbMXe8+2zycYhNx5bYHYRQf0xm26NBn0Z2YCJ1ncsxBOYk29FGdF3k8A4BHP/HP8T9y/UgOjIJ4k5BmSnS+qBcw2tLime+72LjUJjvQU9j9wMufPUAQ== gerrit2@ip-10-120-241-168
  

In your github repository, click on "Admin":

.. image:: ../images/github_mirror_1.jpg

In the left rail, click "Deploy Keys":

.. image:: ../images/github_mirror_2.jpg

Add the new key:

.. image:: ../images/github_mirror_3.jpg

Add ``github.com`` as a host in ``gerrit2``'s ``~/.ssh/config`` file:

.. code-block:: ruby

  Host github.com
  User git
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/cd-tools


Add the ssh host key for git@github.com to Gerrit's ssh known_hosts. You can test ssh for the ``gerrit2`` user the same way you would test ssh access to github.com for your own user, with ``ssh -T git@github.com``. This will also prompt you to add the host key to ``known_hosts``.

To start mirroring, add a new remote to ``~gerrit2/etc/replication.config``:

.. code-block:: ruby

    [remote "github"]
      url = git@github.com:/GITHUBUSER/${name}.git
      push = +refs/heads/*:refs/heads/*
      push = +refs/tags/*:refs/tags/*

The ``${name}`` is significant to Gerrit.  

There are also manual ways to mirror git repositories to various remotes, like `this suggestion at Sourceforge <http://cweiske.de/tagebuch/mirror-sourceforge-git-github.htm>`_.

Creating a client.pem for the Jenkins Server
--------------------------------------------

In order for your Continuous Deployment server to talk to your Chef server, it needs to have a client key within the organization it will be working with.  This will be a separate client id from the server's own chef-client configuration. This piece is actually managed by Jenkins, and not Gerrit, so we create a client in the organization called "jenkins" with knife:

Run the following command:

.. code-block:: ruby
  
  knife node create jenkins

This will open a buffer in your EDITOR with the following content:

.. code-block:: javascript

  1 {
  2   "chef_type": "client",
  3   "public_key": null,
  4   "admin": false,
  5   "name": "jenkins",
  6   "json_class": "Chef::ApiClient"
  7 }
  
Save and quit your editor, and you should get some output on your console:

.. code-block:: ruby

  [bobo@bistromath cd-tools]$ knife client create jenkins
  Created client[jenkins]
  -----BEGIN RSA PRIVATE KEY-----
  MIIEpQIBAAKCAQEA3ctE8qupNPeQU8muhIuwb7uDX6BOCi9IWxPREw6W/pny5VRM
  bsVtnromlWSyBeEY/Crcp8eNrp5zXDoKFjlLH86eJHHQn34hA0brmN8vSRenPfXG
  wtm7X78W5rzKKvxygzVLRK8eFPzIf+BuiEJbVIeb7w11XTijDZ9MnbJ3JQ8DnwuK
  BSBgcaBW9qJbD4LrY/Xql9RA5DqCM7WJc4v5XW6eb5xL/ZndWsZtjUIQrNlpLHC5
  ..SNIP..
  -----END RSA PRIVATE KEY-----


When your buffer session ends, the Chef server will return a public key.  Save this key into the file :file:`/var/lib/jenkins/.chef`. **Note that this location is different from the system's chef-client configuration**.

Then on the Chef server, add ``jenkins`` to the Admin group of the org the server will be working in.  Make sure you are logged in on the Chef server as an Admin of the correct organization, then click "Groups -> Admins -> Edit":

.. image:: ../images/jenkins_chef_admin.jpg
  :alt: Add the Jenkins client to the Admins group on Chef Server

Initialize the Jenkins User for Gerrit
--------------------------------------

Gerrit and Jenkins will communicate using Gerrit's ssh daemon, which runs on port 29418.  To create the jenkins user's Gerrit account, run the following command.  Substitute the administrative user you created above for the ``USER@review.local`` and change the hostname if you have altered what is set in the ``jenkins`` role. You'll need that key available from wherever you run this command from; if you are on your workstation, you'll need to have access to the Gerrit ``sshd`` port, 29418.  If you are on the CD Server itself, you'll want to make sure whatever keypair you used is in the account you're using.

If the key isn't the default key for that account, you can use the standard ``-i /path/to/key`` in the ``ssh`` command below.

.. code-block:: ruby

  cat /var/lib/jenkins/.ssh/id_rsa.pub | ssh -p29418 USER@review.local gerrit create-account --email 'jenkins@jenkins.local' --ssh-key - --full-name Jenkins jenkins

In the Gerrit web ui, add the ``jenkins`` user to the "Non-Interactive Users" group so it will have the appropriate permissions.

.. image:: ../images/gerrit_addjenkins.jpg
  :alt: Add Jenkins to the Non-Interactive Users Group

From the command line on the CD server, become the jenkins user and run the following command to accept the ssh key configuration:

.. code-block:: ruby

  [jenkins@review.local ~]$ ssh -p29418 review.local
  The authenticity of host '[review.local]:29418 ([10.120.241.168]:29418)' can't be established.
  RSA key fingerprint is c9:bc:ac:a4:f4:e8:ec:d6:33:85:2a:2c:ef:67:6b:b4.
  Are you sure you want to continue connecting (yes/no)? yes
  Warning: Permanently added '[review.local]:29418,[10.120.241.168]:29418' (RSA) to the list of known hosts.

  ****    Welcome to Gerrit Code Review    ****

  Hi Jenkins, you have successfully connected over SSH.

  Unfortunately, interactive shells are disabled.
  To clone a hosted Git repository, use:

  git clone ssh://jenkins@review.local:29418/REPOSITORY_NAME.git

  Connection to review.local closed.



Build the First Project into Gerrit
-----------------------------------

The first project that will be managed by Gerrit and built with Jenkins will be the project to continuously deploy the continuous deployment tools, Gerrit and Jenkins themselves, along with all of the dependencies.

Log into the Gerrit web ui, as the administrative user you created earlier.  

Create a new project, called ``cd-tools``. 

Click Admin -> Projects -> Create New Project

.. image:: ../images/gerrit_new_project.jpg

You won't need to make an initial commit; you already have a repository to start from.

In the "General" settings, select ``merge if necessary``.  This setting allows Gerrit to make simpler merges on behalf of the developers without manual intervention.  You can change this setting for other projects, but it should be fine for the ``cd-tools`` project.

Choose ``Require Change ID``. This setting allows you to compress all commits that fail basic syntax and foodcritic tests into the final good commit.  Reviewers will not have to approve all failed commits, only the last good commit that works and passes these tests.  The patchsets will still be recorded, but only the working commits will be passed on to review stage.


.. image:: ../images/gerrit_cdtools_project.jpg
  :alt: General options for the cd-tools project

Check In the Project to Gerrit Using git review 
-----------------------------------------------

Make sure your repository has a ``.gitreview`` file in it.  As a short cut, you can run ``git review`` first, and git will initialize a remote for you in your ``.git/config`` file.  It will return an error, because the git repo on the gerrit server is empty.

Push to the ``gerrit`` remote:

.. code-block:: ruby

  [mandiwalls@bistromath cd-tools]$ git push gerrit master
  Counting objects: 1322, done.
  Delta compression using up to 4 threads.
  Compressing objects: 100% (469/469), done.
  Writing objects: 100% (1322/1322), 3.00 MiB | 160 KiB/s, done.
  Total 1322 (delta 712), reused 1322 (delta 712)
  remote: Resolving deltas: 100% (712/712)
  remote: Processing changes: refs: 1, done    
  To ssh://lnxchk@review.local:29418/cd-tools
   * [new branch]      master -> master

If you have cloned your project, you may receive an error similar to:

.. code-block:: ruby

  remote: 
  remote: ERROR:  In commit 3d4fe24dd104570487ec46c9b7311d9d915b0928
  remote: ERROR:  committer email address adam@opscode.com
  remote: ERROR:  does not match your user account.
  remote: ERROR:
  remote: ERROR:  The following addresses are currently registered:
  remote: ERROR:    mandi.walls@gmail.com
  remote: ERROR:
  remote: ERROR:  To register an email address, please visit:
  remote: ERROR:  http://review.local/#/settings/contact
  remote: 
  remote: 
  To ssh://lnxchk@review.local:29418/cd-tools
   ! [remote rejected] HEAD -> refs/for/master (invalid committer)
  error: failed to push some refs to 'ssh://lnxchk@review.local:29418/cd-tools'


This means that the gerrit server doesn't know about the users who created the upstream commits.  You'll need to add some permissions to the Gerrit server to allow you to check this project in.

In the Gerrit web ui, click on "Admin -> Projects -> All-Projects -> Access".  Click the "Edit" button.  Under the heading ``Reference: refs/heads/*``, add permissions for "Forge Author Identity" and "Forge Committer Identity":

.. image:: ../images/gerrit_forge_identity.jpg
