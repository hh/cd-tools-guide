CD Tools User Workflow
======================

These are the basic steps all users will need to take to interact with the CD Server.

Install Git
-----------

Create a User on the Central Github Server
------------------------------------------

Create a Gerrit User
--------------------

Set Up .gitreview
-----------------

An essential part of the user set up is to make sure all the user's repos that will be using Gerrit have two remotes.

One remote will be the replicated repo on the central server, from which the user will pull updates from.

The other remote will be the Gerrit server, which is where all pushes will be sent.

Every project should have a ``.gitreview`` file at the top of its repository.  It will need to be created by hand for each repo.  

TODO: add .gitreview file sample

Set Up SSH Keys
---------------

If the user's ssh keys in the central git server are different from those in the Gerrit server, set up both keys in the ``~/.ssh/config`` file:

.. code-block:: ruby

  Host review.local
  User bobo
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/review_rsa

  Host github.mycorp.com
  User git
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/github_rsa


never be pushing to gerrit master
  have to take those permissions away from anyone who doesn't explicitly need it
  make a group, put yourself in, run the command you need, take yourself out
  tests run not matter how you get those changes into the repo, but they're not looking for things like 
    "rm -rf /" pushed by someone malicious

Making A Commit
---------------

Users won't be running ``git push`` to their git server anymore, they will running ``git review`` instead.  ``git review`` will return a url to the user so they can find their review request on the Gerrit server.

Under the hood, Gerrit is creating a reference in its internal git repo, and making a branch that includes the new code and all of the current master branch. It will run the initial unit tests on that code - so, checking the code as if it were already merged.

For cookbooks, this means running ``foodcritic`` in particular.  For other code, you include whatever initial unit tests you have for the code.


Review A Commit
---------------

Jenkins is likely going to be your first reviewer. You'll want some sort of lint tool and syntax checking to run before asking a human to review code, so those jobs will kick off as soon as Jenkins sees a new commit in Gerrit's internal git repo.

If those initial checks pass, the next step is often going to be a human code reviewer, or multiple reviewers, taking a look at the code and deciding if it is acceptable for promotion.  

TODO: screen cap the first review screen

A reviewer can view a diff of the new code vs. the original code, make comments on particular lines, and either approve or deny the code change.  When the commit has had sufficient positive reviews, it is merged to the master branch and is ready for promotion.

TODO: screen cap diff

When There is an Error
----------------------
gerrit magic
  console output  
  make the fixing - code? set up?
  set up : rerun the jenkins tests if you don't; new jenkins job on the same event because it was 
    environmental and not code based problem
    retriggers all of the jobs needed for the current task
  you'll look at the console a lot
  new commits when you run "git review"
  you don't need the first review by itself to be reviewed, you need both of them 
    git rebase -i HEAD-N
      where N is the number of piled up commits

      take the last two commits and squash them; change a pick to an s
  now run git review with the new one
  now see patchset 2

  go back to jenkins to see the new jobs

Jenkins Cookbook Tasks
-----------------------
what it's doing
  checking the current shit vs what's in the chef server
  bump the metadata for cookbooks with changes
  jenkins can update the gerrit master
  have an environment for the servers that are the targets for the build
  set the environment constraints for the cookbook versions
    change the pins before the upload so preconditions fail waiting for uploads
    we want the failure so we can guarantee that the clients fail with the download not a run 
 

Pulling Updates
---------------
now go to your workstation and pull the updates

  on your local master branch
  git pull --rebase origin master 
  

