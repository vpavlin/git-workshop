# git-workshop
Example repo for git workshop

# Get account on Github

You will need a Github account for this exercise. If you do not have it yet, you can go to https://github.com/join and sign up.

# Install

## Fedora

Install git: 

```
dnf install git 
```

## RHEL

Install git:

```
yum install git
```

## Both systems

Install Visual Studio Code: https://code.visualstudio.com/docs/setup/linux#_rhel-fedora-and-centos-based-distributions

# Git Configuration

To make sure your commits are correctly marked with your name and email, edit and call following commands to configure your name and email

```
git config --global user.name "<MY NAME>"
git config --global user.email "<MY_EMAIL>
```

# Fork

Go to https://github.com/vpavlin/git-workshop and click [**Fork**](https://github.com/vpavlin/git-workshop/fork). This will create a copy of the repository in your organization. Repositories will be still linked in Github, but until you submit a Pull Request (PR) changes in your fork will not influence the upstream (original) repository.

# Clone

To download your fork to your computer, you need to perform a **clone** operation. Replace `$USER` with your Github username:

```
git clone git@github.com:$USER/git-workshop.git
```

Depending on your system and Github configuration, you might be prompted for password. Once cloned, you can enter the repository

```
cd git-workshop
```

# Upstream and Rebase

To stay in sync with the upstream (original) repository and to prevent conflicts when submitting PRs, it is important to **rebase** before you start working on a new branch or before you submit the PR. 

We first need to add the upstream repository as a new **remote** server:

```
git remote add upstream https://github.com/vpavlin/git-workshop
```

Then we need to **fetch** it to get latest revision of that repository

```
git fetch upstream
```

**Rebase** itself then takes the latest revision from the upstream repository, finds it in your cloned repository and intelligently merges changes from upstream and your local changes:

```
git rebase upstream/master
```

This might result in a **conflict**. We will look at conflict resolution in later section.

# Branching

When you start working on a new feature or fixing a new issue, you want to start your work in a new **branch**.

```
git checkout -b feature/awesome-new-feature
```

Let's add a file now

```
echo "New creature" > feature.txt
```

If we look at **diff** now, we won't see anything

```
git diff
```

And **status** explains why

```
git status
```

It is because the newly created file is **untracked**. We need to **stage** it first and then **commit** it to the repository

```
git add feature*
git commit -m "New feature"
```

We can **show** our latest commit by

```
git show
```

Changes are now persisted in your local clone of the repository and we need to get them to some remote server for backup and sharing - in other words, we need to **push** the new commit.

```
git push
```

You will get an error message `fatal: The current branch feature/awesome-new-feature has no upstream branch. To push the current branch and set the remote as upstream...`. Simply call the command git suggests

```
git push --set-upstream origin feature/awesome-new-feature
```

This will make sure the branch and the changes will be pushed to your repository on Github.

Git will also suggest to create a **pull request** - i.e. to propose the changes to be merged to the repository - you will see something like

```
remote: 
remote: Create a pull request for 'feature/awesome-new-feature' on GitHub by visiting:
remote:      https://github.com/vpavlin/git-workshop/pull/new/feature/awesome-new-feature
remote: 
```

The link will get to *Open a new pull request* page. It will automatically try to create a PR against the upstream repository. Normally that would be exactly what you want, but at this point you want to work only in your fork. 

To do that, select your `usrename/repository` in **base repository** drop down and then select `master` as a **base**. Add a description and hit **Create pull request**.

Github will check if the PR can be merged and if everything is ok, it will show **Merge pull request** button. Assuming we are happy with the PR, click the merge button nd then click confirm.

Your change is now in the `master` branch of your repository and you should see a new file in the *Code* section. You can also view the content of the file by clicking on it.

## Fixing an issue

We have promoted an issue to `master` branch - the new file `feature.txt` actually contains text `New creature` instead of `New feature`, which is not good. We need to fix it now. First, we need to create an issue on Github. 

As forks do not allow issue creation by default, we need to go to **Settings** and select **Issues** in **Features** section. You can create an issue now - do not forget to describe what do you plan to do.

Now create a new branch in your repository - make sure you first **checkout** to `master` branch and **pull** latest revision from **your** repository.

```
git checkout master
git pull
git checkout -b bug/fix-creature
```

Now let's fix the issue

```
sed -i 's/creature/feature/' feature.txt
```

Look at the **diff** to see the changes

```
git diff
```

Let's commit a change and submit a PR as we did above. We can now use parameter `-a` with commit command which will automatically add all changed tracked files

```
git commit -a -m "Fix the creature (resolves #2)"
```

Now push the branch to Github and create a PR with your new fix the same way as before.

Notice the part `resolves #2` mentioned in the commit message. This links the PR and the issue together which means that the issue gets updated with the PR link and also gets automatically closed when the PR is merged.

# Resolving conflicts

It can happen that your changes get into conflict with some other changes by other contributors. It is not hard to resolve those conflicts, so let's take a look at it.

Create a new branch and switch to it

```
git branch conflict
git checkout conflict
```

or

```
git checkout -b conflict
```

Change the `existing-file.txt`

```
sed -i 's/#CHANGEME/Changed!/' existing-file.txt
```

Look at the changes you just made:

```
git diff
```

And commit the changes

```
git commit -a -m "Changed the file"
```

Now switch back to `master` branch and change the file there as well

```
git checkout master
```

```
sed -i 's/#CHANGEME/Also changed it here/' existing-file.txt
```

And commit the changes as well

```
git commit -a -m "Changed the file in here as well"
```

Now it's time to try to merge the branch `conflict` to `master`

```
git merge conflict
```

You will see an error `Automatic merge failed; fix conflicts and then commit the result.` This means git cannot resolve the problem automatically and you will need to perform some manual steps

Go back to Visual Studio code and you'll see this in the `existing-file.txt`:

```
<<<<<<< HEAD
Also changed it here
=======
Changed!
>>>>>>> conflict

```

You can manually edit this to your liking, or simply click the `Accept...` buttons displayed by the editor. Let's accept the **incoming** change (i.e. the change made in the branch `conflict`). Do not forget to save the file. Look at the diff again after you clicked the button

```
git diff
```

Before letting git continue the merge, we need to stage the changed file

```
git add existing-file.txt
```

We can now continue with the merge

```
git merge --continue
```

Yay! You have successfully resolved a merge conlict!

# Going Back in Time

We were changing the `master` branch in previous section. To be able to submit a clean PR to upstream repository in the next section, we need to get our local repo to the same state as the upstream `master` branch. First, let's take a look at the history of the repository

```
git log
```

You can now look for `upstream/master` annotation or just print log for that specific remote and branch

```
git log upstream/master
```

Then take the commit hash from there and add it to the **reset** command

```
git reset --hard <HASH>
```

When you create a new branch now, it will be based on upstream master branch.

# Working in a Team

While working on a project as a team you will follow similar workflows as we tried above - issue, branch, code, commit, PR, review, resolving conflicts, merge. So let's try something like this here.

1. Create a new branch
2. Open the file `names.txt`
3. Add your username in the file
4. Commit, push and submit a PR
 * *This time do not change the base repository, but submit the PR to the upstream repository (`vpavlin/git-workshop`)*
5. I will merge your PRs

As many people are editing same file, it is highly probable there will be conflicts, so you will have to rebase your change on a new upstream revision (use `git fetch` and `git rebase` commands from the top of this README). WHen you rebase your change on top of new revision, you will need to push the new content of the branch. This will fail

```
 ! [rejected]        add-name -> add-name (non-fast-forward)
error: failed to push some refs to 'https://github.com/vpavlin/git-workshop.git'
```

This is because hashes for your commits changed. To resolve this you will have to do a force push to your branch

```
git push --force
```

This will overwrite contents of your branch - always be careful with force pushing and make sure you know what you are doing (and never force push to master:))

Github PR will be automatically updated and the mergeability will be reevaluated.

# Blame others!

Git offers tools to learn about the history and changes in the repository. One of them (**log**) we tried before. Another one is **blame**.

```
git blame names.txt
```

Blame shows who is the author of each line in a given file. This is specifically useful if something about the content of the file is not clear and you need to find the author. You can also view output of both commands directly in Github UI.