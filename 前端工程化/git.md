## 结构

![](https://pic2.zhimg.com/80/v2-af3bf6fee935820d481853e452ed2d55_720w.jpg?source=1940ef5c)

**工作区(working directory)，**简言之就是你工作的区域。对于git而言，就是的本地工作目录。工作区的内容会包含提交到暂存区和版本库(当前提交点)的内容，同时也包含自己的修改内容。

**暂存区(stage area, 又称为索引区index)，**是git中一个非常重要的概念。是我们把修改提交版本库前的一个过渡阶段。查看GIT自带帮助手册的时候，通常以index来表示暂存区。在工作目录下有一个.git的目录，里面有个index文件，存储着关于暂存区的内容。git add命令将工作区内容添加到暂存区。

**本地仓库(local repository)，**版本控制系统的仓库，存在于本地。当执行git commit命令后，会将暂存区内容提交到仓库之中。在工作区下面有.git的目录，这个目录下的内容不属于工作区，里面便是仓库的数据信息，暂存区相关内容也在其中。这里也可以使用merge或rebase将**远程仓库副本**合并到本地仓库。图中的只有merge，注意这里也可以使用rebase。

**远程版本库(remote repository)，**与本地仓库概念基本一致，不同之处在于一个存在远程，可用于远程协作，一个却是存在于本地。通过push/pull可实现本地与远程的交互；

**远程仓库副本，**可以理解为存在于本地的远程仓库缓存。如需更新，可通过git fetch/pull命令获取远程仓库内容。使用fech获取时，并未合并到本地仓库，此时可使用git merge实现远程仓库副本与本地仓库的合并。git pull 根据配置的不同，可为git fetch + git merge 或 git fetch + git rebase。rebase和merge的区别可以自己去网上找些资料了解下。

参考文章：

[git pull 和 git fetch的区别？](https://www.zhihu.com/question/38305012)

## git clone

从远程主机克隆一个版本库，这时就要用到`git clone`命令。

> ```javascript
> $ git clone <版本库的网址>
> ```

该命令会在本地主机生成一个目录，与远程主机的版本库同名。如果要指定不同的目录名，可以将目录名作为`git clone`命令的第二个参数。

> ```javascript
> $ git clone <版本库的网址> <本地目录名>
> ```

## git remote

为了便于管理，Git要求每个远程主机都必须指定一个主机名。`git remote`命令就用于管理主机名。

不带选项的时候，`git remote`命令列出所有远程主机。

> ```javascript
> $ git remote
> origin
> ```

使用`-v`选项，可以参看远程主机的网址。

> ```javascript
> $ git remote -v
> origin  git@github.com:jquery/jquery.git (fetch)
> origin  git@github.com:jquery/jquery.git (push)
> ```

上面命令表示，当前只有一台远程主机，叫做origin，以及它的网址。

## git fetch

一旦远程主机的版本库有了更新（Git术语叫做commit），需要将这些更新取回本地，这时就要用到`git fetch`命令。

> ```javascript
> $ git fetch <远程主机名>
> ```

上面命令将某个远程主机的更新，全部取回本地。

`git fetch`命令通常用来查看其他人的进程，因为**它取回的代码对你本地的开发代码没有影响**。

默认情况下，`git fetch`取回所有分支（branch）的更新。如果只想取回特定分支的更新，可以指定分支名。

> ```javascript
> $ git fetch <远程主机名> <分支名>
> ```

比如，取回`origin`主机的`master`分支。

> ```javascript
> $ git fetch origin master
> ```

## git pull

`git pull`命令的作用是，取回远程主机某个分支的更新，再与本地的指定分支合并。它的完整格式稍稍有点复杂。

> ```javascript
> $ git pull <远程主机名> <远程分支名>:<本地分支名>
> ```

比如，取回`origin`主机的`next`分支，与本地的`master`分支合并，需要写成下面这样。

> ```javascript
> $ git pull origin next:master
> ```

**如果远程主机删除了某个分支，默认情况下，`git pull` 不会在拉取远程分支的时候，删除对应的本地分支。**这是为了防止，由于其他人操作了远程主机，导致`git pull`不知不觉删除了本地分支。

**git pull = git fetch + git merge**

## git push

`git push`命令用于将本地分支的更新，推送到远程主机。它的格式与`git pull`命令相仿。

> ```javascript
> $ git push <远程主机名> <本地分支名>:<远程分支名>
> ```

注意，分支推送顺序的写法是<来源地>:<目的地>，所以`git pull`是<远程分支>:<本地分支>，而`git push`是<本地分支>:<远程分支>。

如果省略远程分支名，则表示将本地分支推送与之存在"追踪关系"的远程分支（通常两者同名），如果该远程分支不存在，则会被新建。

> ```javascript
> $ git push origin master
> ```

上面命令表示，将本地的`master`分支推送到`origin`主机的`master`分支。如果后者不存在，则会被新建。

如果省略本地分支名，则表示删除指定的远程分支，因为这等同于推送一个空的本地分支到远程分支。

> ```javascript
> $ git push origin :master
> # 等同于
> $ git push origin --delete master
> ```



## 全局配置环境

配置个人用户名和电子邮箱

```
git config –globle user.name “runoob”
git config –globle user.email text@runoob.com
```

配置完毕后，可以通过$ git config –list命令查看所有的配置信息。 也可直接查询某个环境变量的信息。

```
git config user.name
git config user.email
```

## 查看工作区状态

```
git status
```

- 状态一：修改了没有添加到缓存区（红色），此时可以通过git diff 查看修改了的内容，“-”号是修改前，“+”号是修改后，第一个加号后修改的前一行。第二个加号是修改的内容。
- 状态二：修改了添加到缓存区（绿色）
- 状态三：On branch master nothing to commit, work tree clean 表明无修改内容

[Git远程操作详解](https://www.ruanyifeng.com/blog/2014/06/git_remote.html)

## git merge 和 git rebase

使用 `rebase` 和 `merge` 的基本原则：

1. 下游分支更新上游分支内容的时候使用 `rebase`
2. 上游分支合并下游分支内容的时候使用 `merge`
3. 更新当前分支的内容时一定要使用 `--rebase` 参数

例如现有上游分支 master，基于 master 分支拉出来一个开发分支 dev，在 dev 上开发了一段时间后要把 master 分支提交的新内容更新到 dev 分支，此时切换到 dev 分支，使用 `git rebase master`

等 dev 分支开发完成了之后，要合并到上游分支 master 上的时候，切换到 master 分支，使用 `git merge dev`