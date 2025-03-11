## Structure

![](https://pic2.zhimg.com/80/v2-af3bf6fee935820d481853e452ed2d55_720w.jpg?source=1940ef5c)

**Working Directory** is simply your work area. For git, it's your local working directory. The content of the working directory includes content committed to the staging area and repository (current commit point), as well as your own modifications.

**Stage Area (also known as index area)** is a very important concept in git. It's a transition phase before we commit modifications to the repository. When viewing the GIT built-in help manual, the staging area is usually referred to as index. Under the working directory, there is a .git directory with an index file that stores the content of the staging area. The git add command adds working directory content to the staging area.

**Local Repository** is the version control system's repository that exists locally. When executing the git commit command, the content in the staging area will be committed to the repository. Under the working directory, there is a .git directory whose content does not belong to the working directory. It contains the repository's data information, including staging area-related content. Here you can also use merge or rebase to merge the **Remote Repository Copy** into the local repository. The diagram only shows merge, but note that rebase can also be used here.

**Remote Repository** is basically the same concept as the local repository, with the difference being that one exists remotely for remote collaboration, while the other exists locally. Interaction between local and remote can be achieved through push/pull.

**Remote Repository Copy** can be understood as a cache of the remote repository that exists locally. To update, you can use the git fetch/pull command to get remote repository content. When using fetch to retrieve, it has not been merged into the local repository. At this point, you can use git merge to merge the remote repository copy with the local repository. Depending on the configuration, git pull can be git fetch + git merge or git fetch + git rebase. You can find resources online to understand the difference between rebase and merge.

Reference Article:

[What's the difference between git pull and git fetch?](https://www.zhihu.com/question/38305012)

## git clone

To clone a repository from a remote host, you use the `git clone` command.

> ```javascript
> $ git clone <repository URL>
> ```

This command will generate a directory on your local machine with the same name as the remote repository. If you want to specify a different directory name, you can pass it as the second parameter to the `git clone` command.

> ```javascript
> $ git clone <repository URL> <local directory name>
> ```

## git remote

For management purposes, Git requires each remote host to have a specified hostname. The `git remote` command is used to manage hostnames.

Without options, the `git remote` command lists all remote hosts.

> ```javascript
> $ git remote
> origin
> ```

Using the `-v` option, you can view the remote host's URL.

> ```javascript
> $ git remote -v
> origin  git@github.com:jquery/jquery.git (fetch)
> origin  git@github.com:jquery/jquery.git (push)
> ```

The above command shows that there is currently only one remote host called origin, along with its URL.

## git fetch

Once the remote host's repository has updates (called commits in Git terminology), you need to fetch these updates locally, which is when you use the `git fetch` command.

> ```javascript
> $ git fetch <remote hostname>
> ```

The above command fetches all updates from a specific remote host to local.

The `git fetch` command is typically used to view others' progress because **the code it fetches has no impact on your local development code**.

By default, `git fetch` retrieves updates for all branches. If you only want to fetch updates for a specific branch, you can specify the branch name.

> ```javascript
> $ git fetch <remote hostname> <branch name>
> ```

For example, to fetch the `master` branch from the `origin` host:

> ```javascript
> $ git fetch origin master
> ```

## git pull

The `git pull` command's function is to fetch updates from a remote host's branch and merge them with the specified local branch. Its complete format is slightly complex.

> ```javascript
> $ git pull <remote hostname> <remote branch>:<local branch>
> ```

For example, to fetch the `next` branch from the `origin` host and merge it with the local `master` branch, you would write:

> ```javascript
> $ git pull origin next:master
> ```

**If a remote host deletes a branch, by default, `git pull` won't delete the corresponding local branch when pulling remote branches.** This is to prevent unintentional deletion of local branches due to others' operations on the remote host.

**git pull = git fetch + git merge**

## git push

The `git push` command is used to push local branch updates to the remote host. Its format is similar to the `git pull` command.

> ```javascript
> $ git push <remote hostname> <local branch>:<remote branch>
> ```

Note that the branch push order is written as <source>:<destination>, so `git pull` is <remote branch>:<local branch>, while `git push` is <local branch>:<remote branch>.

If you omit the remote branch name, it means pushing the local branch to its "tracking" remote branch (usually they have the same name). If that remote branch doesn't exist, it will be created.

> ```javascript
> $ git push origin master
> ```

The above command means pushing the local `master` branch to the `master` branch on the `origin` host. If the latter doesn't exist, it will be created.

If you omit the local branch name, it means deleting the specified remote branch, because this is equivalent to pushing an empty local branch to the remote branch.

> ```javascript
> $ git push origin :master
> # equivalent to
> $ git push origin --delete master
> ```

## Global Environment Configuration

Configure personal username and email

```
git config --global user.name "runoob"
git config --global user.email text@runoob.com
```

After configuration, you can view all configuration information using the `$ git config --list` command. You can also directly query specific environment variable information.

```
git config user.name
git config user.email
```

## Check Working Directory Status

```
git status
```

- Status 1: Modified but not added to staging area (red). At this point, you can use git diff to view the modified content. "-" shows before modification, "+" shows after modification. The first plus sign shows the line before the modification, the second plus sign shows the modified content.
- Status 2: Modified and added to staging area (green)
- Status 3: "On branch master nothing to commit, work tree clean" indicates no modifications

[Git Remote Operations Explained](https://www.ruanyifeng.com/blog/2014/06/git_remote.html)

## git merge and git rebase

Basic principles for using `rebase` and `merge`:

1. Use `rebase` when downstream branches update upstream branch content
2. Use `merge` when upstream branches merge downstream branch content
3. Always use the `--rebase` parameter when updating current branch content

For example, if you have an upstream branch master and create a development branch dev based on it, after developing on dev for a while and wanting to update new content from the master branch to dev, switch to the dev branch and use `git rebase master`

When dev branch development is complete and needs to be merged into the upstream master branch, switch to the master branch and use `git merge dev`