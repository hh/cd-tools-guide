Set Up An Application Project
=============================

Any application project you add to your CD server will require these set up steps.  Putting in your Jenkins tasks is documented elsewhere.

Add the Project in Gerrit
-------------------------

Add your new project to Gerrit in the same way you added the ``cd-tools`` project in the :doc:`manual </installation/manual>` steps.

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


