# git-workshop
Example repo for git workshop

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

## Both

Install Visual Studio Code: https://code.visualstudio.com/docs/setup/linux#_rhel-fedora-and-centos-based-distributions

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