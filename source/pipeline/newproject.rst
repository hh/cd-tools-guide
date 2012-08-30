Set Up An Application Project
=============================

Any application project you add to your CD server will require these set up steps.  Putting in your Jenkins tasks is documented elsewhere.

Add the Project in Gerrit
-------------------------

Add your new project to Gerrit in the same way you added the ``cd-tools`` project in the :doc:`manual </installation/manual>` steps.

In addition to the Gerrit project permissions for Foring User Identity, if you are importing an existing git-based project, you may need to also set ``Push Merge Commit`` for projects with historical merges.

The errors may look like:

.. code-block:: ruby

  [mandiwalls@bistromath game-of-life]$ git push gerrit master
  Counting objects: 7235, done.
  Delta compression using up to 4 threads.
  Compressing objects: 100% (2106/2106), done.
  Writing objects: 100% (7235/7235), 17.02 MiB | 274 KiB/s, done.
  Total 7235 (delta 4710), reused 7137 (delta 4618)
  remote: Resolving deltas: 100% (4710/4710)
  remote: Processing changes: refs: 1, done    
  To ssh://lnxchk@review.local:29418/Game_Of_Life
   ! [remote rejected] master -> master (you are not allowed to upload merges)
  error: failed to push some refs to 'ssh://lnxchk@review.local:29418/Game_Of_Life'

The settings are required to be under the References for ``refs/for/refs/*``.  You can add this to ``All-Projects``, which will simplify future additions of existing projects.

.. image:: ../images/gerrit_merge_commit.jpg
Initialize the Code Repo for Gerrit
-----------------------------------

.gitreview file

Add the chef-repo to the Code Repo
----------------------------------

blank PROJECT cookbook

blank PROJECT role

Set up knife.rb
---------------

change the cookbooks directory

Check in and git review
-----------------------

go into Jenkins
PROJECT infrastructure repo

add the jenkins check jobs to your repo for the cookbook syntax and foodcritic

add the app_pipelines databag

  sets up all the jobs in jenkins
  whatever you normally do for the first round of tests in your project -- maven build and testing? ok!


