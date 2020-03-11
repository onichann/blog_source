---
layout: post
title: git 子模块
date: 2020-03-10 20:43:50
tags:
    - git
    - 子模块
categories: [git , 子模块]
---
## 子模块
有种情况我们经常会遇到：某个工作中的项目需要包含并使用另一个项目。 也许是第三方库，或者你独立开发的，用于多个父项目的库。 现在问题来了：你想要把它们当做两个独立的项目，同时又想在一个项目中使用另一个。

我们举一个例子。 假设你正在开发一个网站然后创建了 Atom 订阅。 你决定使用一个库，而不是写自己的 Atom 生成代码。 你可能不得不通过 CPAN 安装或 Ruby gem 来包含共享库中的代码，或者将源代码直接拷贝到自己的项目中。 如果将这个库包含进来，那么无论用何种方式都很难定制它，部署则更加困难，因为你必须确保每一个客户端都包含该库。 如果将代码复制到自己的项目中，那么你做的任何自定义修改都会使合并上游的改动变得困难。
<!--more-->
Git 通过子模块来解决这个问题。 子模块允许你将一个 Git 仓库作为另一个 Git 仓库的子目录。 它能让你将另一个仓库克隆到自己的项目中，同时还保持提交的独立。



### 开始使用子模块

我们将要演示如何在一个被分成一个主项目与几个子项目的项目上开发。

我们首先将一个已存在的 Git 仓库添加为正在工作的仓库的子模块。 你可以通过在 **<font color=red>git submodule add</font>** 命令后面加上想要跟踪的项目的相对或绝对 URL 来添加新的子模块。在本例中，我们将会添加一个名为 “DbConnector” 的库。

> $ git submodule add https://github.com/chaconinc/DbConnector
Cloning into 'DbConnector'...
remote: Counting objects: 11, done.
remote: Compressing objects: 100% (10/10), done.
remote: Total 11 (delta 0), reused 11 (delta 0)
Unpacking objects: 100% (11/11), done.
Checking connectivity... done.

默认情况下，子模块会将子项目放到一个与仓库同名的目录中，本例中是 “DbConnector”。 <font color=red>如果你想要放到其他地方，那么可以在命令结尾添加一个不同的路径</font>。

如果这时运行 **git status**，你会注意到几件事。

> $ git status
On branch master
Your branch is up-to-date with 'origin/master'.
.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)
  .
	new file:   .gitmodules
	new file:   DbConnector

首先应当注意到新的 **.gitmodules** 文件。 该配置文件保存了项目 URL 与已经拉取的本地目录之间的映射：	

> [submodule "DbConnector"]
	path = DbConnector
	url = https://github.com/chaconinc/DbConnector

如果有多个子模块，该文件中就会有多条记录。 要重点注意的是，该文件也像 .gitignore 文件一样受到（通过）版本控制。 它会和该项目的其他部分一同被拉取推送。 这就是克隆该项目的人知道去哪获得子模块的原因。

>Note: 由于 .gitmodules 文件中的 URL 是人们首先尝试克隆/拉取的地方，因此请尽可能确保你使用的 URL 大家都能访问。 例如，若你要使用的推送 URL 与他人的拉取 URL 不同，那么请使用他人能访问到的 URL。 你也可以根据自己的需要，通过在本地执行 **git config submodule.DbConnector.url <私有URL>** 来覆盖这个选项的值。 如果可行的话，一个相对路径会很有帮助。

在 **git status** 输出中列出的另一个是项目文件夹记录。如果你运行 **git diff**，会看到类似下面的信息：

>$ git diff - -cached DbConnector
diff - -git a/DbConnector b/DbConnector
new file mode 160000
index 0000000..c3f01dc
\- - - /dev/null
\+++ b/DbConnector
\@@ -0,0 +1 @@
\+Subproject commit c3f01dc8862123d317dd46284b05b6892c7b29bc

虽然 **DbConnector** 是工作目录中的一个子目录，但 Git 还是会将它视作一个子模块。当你不在那个目录中时，Git 并不会跟踪它的内容， 而是将它看作子模块仓库中的某个具体的提交。

如果你想看到更漂亮的差异输出，可以给 **git diff** 传递 **- -submodule** 选项。

>$ git diff - -cached - -submodule
diff - -git a/.gitmodules b/.gitmodules
new file mode 100644
index 0000000..71fc376
\- - - /dev/null
\+++ b/.gitmodules
\@@ -0,0 +1,3 @@
\+[submodule "DbConnector"]
\+       path = DbConnector
\+       url = https://github.com/chaconinc/DbConnector
Submodule DbConnector 0000000...c3f01dc (new submodule)

当你提交时，会看到类似下面的信息：

>$ git commit -am 'added DbConnector module'
[master fb9093c] added DbConnector module
 2 files changed, 4 insertions(+)
 create mode 100644 .gitmodules
 create mode 160000 DbConnector
 
注意 DbConnector 记录的 160000 模式。 这是 Git 中的一种特殊模式，它本质上意味着你是将一次提交记作一项目录记录的，而非将它记录成一个子目录或者一个文件。

最后，推送这些更改：

>$ git push origin master

### 克隆含有子模块的项目

接下来我们将会克隆一个含有子模块的项目。 当你在克隆这样的项目时，默认会包含该子模块目录，但其中还没有任何文件：

> $ git clone https://github.com/chaconinc/MainProject
Cloning into 'MainProject'...
remote: Counting objects: 14, done.
remote: Compressing objects: 100% (13/13), done.
remote: Total 14 (delta 1), reused 13 (delta 0)
Unpacking objects: 100% (14/14), done.
Checking connectivity... done.
$ cd MainProject
$ ls -la
total 16
drwxr-xr-x   9 schacon  staff  306 Sep 17 15:21 .
drwxr-xr-x   7 schacon  staff  238 Sep 17 15:21 ..
drwxr-xr-x  13 schacon  staff  442 Sep 17 15:21 .git
-rw-r- -r- -   1 schacon  staff   92 Sep 17 15:21 .gitmodules
drwxr-xr-x   2 schacon  staff   68 Sep 17 15:21 DbConnector
-rw-r- -r- -   1 schacon  staff  756 Sep 17 15:21 Makefile
drwxr-xr-x   3 schacon  staff  102 Sep 17 15:21 includes
drwxr-xr-x   4 schacon  staff  136 Sep 17 15:21 scripts
drwxr-xr-x   4 schacon  staff  136 Sep 17 15:21 src
$ cd DbConnector/
$ ls
$

其中有 **DbConnector** 目录，不过是空的。 你必须运行两个命令：**<font color=red>git submodule init</font>** 用来初始化本地配置文件，而 **<font color=red>git submodule update</font>** 则从该项目中抓取所有数据并检出父项目中列出的合适的提交。

> $ git submodule init
Submodule 'DbConnector' (https://github.com/chaconinc/DbConnector) registered for path 'DbConnector'
$ git submodule update
Cloning into 'DbConnector'...
remote: Counting objects: 11, done.
remote: Compressing objects: 100% (10/10), done.
remote: Total 11 (delta 0), reused 11 (delta 0)
Unpacking objects: 100% (11/11), done.
Checking connectivity... done.
Submodule path 'DbConnector': checked out 'c3f01dc8862123d317dd46284b05b6892c7b29bc'

现在 DbConnector 子目录是处在和之前提交时相同的状态了。

不过还有更简单一点的方式。 如果**<font color=red>给 git clone 命令传递 - -recurse-submodules</font>** 选项，它就会自动初始化并更新仓库中的每一个子模块， 包括可能存在的嵌套子模块。

> $ git clone - -recurse-submodules https://github.com/chaconinc/MainProject
Cloning into 'MainProject'...
remote: Counting objects: 14, done.
remote: Compressing objects: 100% (13/13), done.
remote: Total 14 (delta 1), reused 13 (delta 0)
Unpacking objects: 100% (14/14), done.
Checking connectivity... done.
Submodule 'DbConnector' (https://github.com/chaconinc/DbConnector) registered for path 'DbConnector'
Cloning into 'DbConnector'...
remote: Counting objects: 11, done.
remote: Compressing objects: 100% (10/10), done.
remote: Total 11 (delta 0), reused 11 (delta 0)
Unpacking objects: 100% (11/11), done.
Checking connectivity... done.
Submodule path 'DbConnector': checked out 'c3f01dc8862123d317dd46284b05b6892c7b29bc'

如果你已经克隆了项目但忘记了 **- -recurse-submodules**，那么可以运行 **<font color=red>git submodule update - -init</font>** 将 git submodule init 和 git submodule update 合并成一步。如果还要初始化、抓取并检出任何**嵌套的子模块**， 请使用简明的 **<font color=red>git submodule update - -init - -recursive</font>**。

### 在包含子模块的项目上工作

现在我们有一份包含子模块的项目副本，我们将会同时在主项目和子模块项目上与队员协作。

#### 从子模块的远端拉取上游修改

**在项目中使用子模块**的最简模型，就是只使用子项目并不时地获取更新，而并不在你的检出中进行任何更改。 我们来看一个简单的例子。

如果想要在子模块中查看新工作，可以进入到目录中运行 git fetch 与 git merge，合并上游分支来更新本地代码。

> $ git fetch
From https://github.com/chaconinc/DbConnector
   c3f01dc..d0354fc  master     -> origin/master
$ git merge origin/master
Updating c3f01dc..d0354fc
Fast-forward
 scripts/connect.sh | 1 +
 src/db.c           | 1 +
 2 files changed, 2 insertions(+)
 
如果你现在返回到主项目并运行 **git diff - -submodule**，就会看到子模块被更新的同时获得了一个包含新添加提交的列表。 如果你不想每次运行 git diff 时都输入 - -submodle，那么可以将 diff.submodule 设置为 “log” 来将其作为默认行为。

>$ git config - -global diff.submodule log
$ git diff
Submodule DbConnector c3f01dc..d0354fc:
  > more efficient db routine
  > better connection routine

如果在此时提交，那么你会将子模块锁定为其他人更新时的新代码。

如果你不想在子目录中手动抓取与合并，那么还有种更容易的方式。 运行 **<font color=red>git submodule update - -remote</font>**，Git 将会进入子模块然后抓取并更新。

> $ git submodule update - -remote DbConnector
remote: Counting objects: 4, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 4 (delta 2), reused 4 (delta 2)
Unpacking objects: 100% (4/4), done.
From https://github.com/chaconinc/DbConnector
   3f19983..d0354fc  master     -> origin/master
Submodule path 'DbConnector': checked out 'd0354fc054692d3906c85c3af05ddce39a1c0644'

**<font color=red>此命令默认会假定你想要更新并检出子模块仓库的 master 分支</font>**。 不过你也可以设置为想要的其他分支。 例如，你想要 DbConnector 子模块跟踪仓库的 “stable” 分支，那么既可以在 .gitmodules 文件中设置 （这样其他人也可以跟踪它），也可以只在本地的 .git/config 文件中设置。 让我们在 .gitmodules 文件中设置它：

>$ git config -f .gitmodules submodule.DbConnector.branch stable
.
$ git submodule update - -remote
remote: Counting objects: 4, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 4 (delta 2), reused 4 (delta 2)
Unpacking objects: 100% (4/4), done.
From https://github.com/chaconinc/DbConnector
   27cf5d3..c87d55d  stable -> origin/stable
Submodule path 'DbConnector': checked out 'c87d55d4c6d4b05ee34fbc8cb6f7bf4585ae6687'
 
如果不用 **-f .gitmodules** 选项，那么它只会为你做修改。但是在仓库中保留跟踪信息更有意义一些，因为其他人也可以得到同样的效果。

这时我们运行 git status，Git 会显示子模块中有“新提交”。

>$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
.
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout - - <file>..." to discard changes in working directory)
.
  modified:   .gitmodules
  modified:   DbConnector (new commits)
.
no changes added to commit (use "git add" and/or "git commit -a")

如果你设置了配置选项 status.submodulesummary，Git 也会显示你的子模块的更改摘要：

> $ git config status.submodulesummary 1
.
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
.
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
.
	modified:   .gitmodules
	modified:   DbConnector (new commits)
.
Submodules changed but not updated:
.
*DbConnector c3f01dc...c87d55d (4):
  catch non-null terminated lines
  
这时如果运行 git diff，可以看到我们修改了 .gitmodules 文件，同时还有几个已拉取的提交需要提交到我们自己的子模块项目中。

>$ git diff
diff --git a/.gitmodules b/.gitmodules
index 6fc0b3d..fd1cc29 100644
--- a/.gitmodules
+++ b/.gitmodules
@@ -1,3 +1,4 @@
 [submodule "DbConnector"]
        path = DbConnector
        url = https://github.com/chaconinc/DbConnector
       branch = stable
 Submodule DbConnector c3f01dc..c87d55d:
  \> catch non-null terminated lines
  \> more robust error handling
  \> more efficient db routine
  \> better connection routine

这非常有趣，因为我们可以直接看到将要提交到子模块中的提交日志。 提交之后，你也可以运行 git log -p 查看这个信息。

> $ git log -p --submodule
commit 0a24cfc121a8a3c118e0105ae4ae4c00281cf7ae
Author: Scott Chacon <schacon@gmail.com>
Date:   Wed Sep 17 16:37:02 2014 +0200
\
    updating DbConnector for bug fixes
\
diff --git a/.gitmodules b/.gitmodules
index 6fc0b3d..fd1cc29 100644
--- a/.gitmodules
+++ b/.gitmodules
@@ -1,3 +1,4 @@
 [submodule "DbConnector"]
        path = DbConnector
        url = https://github.com/chaconinc/DbConnector
\+       branch = stable
Submodule DbConnector c3f01dc..c87d55d:
  \> catch non-null terminated lines
 \> more robust error handling
  \> more efficient db routine
  \> better connection routine

当运行 **git submodule update - -remote** 时，Git 默认会尝试更新 所有 子模块， 所以如果有很多子模块的话，你可以传递想要更新的子模块的名字。比如：子模块路径：themes/material-x ,那么执行 **git submodule update - -remote themes/material-x**  即可。


#### 从项目远端拉取上游更改

现在，**让我们站在协作者的视角**，他有自己的 MainProject 仓库的本地克隆， 只是执行 git pull 获取你新提交的更改还不够：

> $ git pull
From https://github.com/chaconinc/MainProject
   fb9093c..0a24cfc  master     -> origin/master
Fetching submodule DbConnector
From https://github.com/chaconinc/DbConnector
   c3f01dc..c87d55d  stable     -> origin/stable
Updating fb9093c..0a24cfc
Fast-forward
 .gitmodules         | 2 +-
 DbConnector         | 2 +-
 2 files changed, 2 insertions(+), 2 deletions(-)
. 
 $ git status
 On branch master
Your branch is up-to-date with 'origin/master'.
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout - - <file>..." to discard changes in working directory)
.
	modified:   DbConnector (new commits)
.
Submodules changed but not updated:
.
*DbConnector c87d55d...c3f01dc (4):
  < catch non-null terminated lines
  < more robust error handling
  < more efficient db routine
  < better connection routine
.
no changes added to commit (use "git add" and/or "git commit -a")

默认情况下，git pull 命令会递归地抓取子模块的更改，如上面第一个命令的输出所示。 然而，它不会 更新 子模块。这点可通过 git status 命令看到，它会显示子模块“已修改”，且“有新的提交”。 此外，左边的尖括号（<）指出了新的提交，表示这些提交已在 MainProject 中记录，但尚未在本地的 DbConnector 中检出。 为了完成更新，你需要运行 **git submodule update**：

>$ git submodule update - -init - -recursive
Submodule path 'vendor/plugins/demo': checked out '48679c6302815f6c76f1fe30625d795d9e55fc56'
.
$ git status
 On branch master
Your branch is up-to-date with 'origin/master'.
nothing to commit, working tree clean

**请注意，为安全起见，如果 MainProject 提交了你刚拉取的`新子模块`，那么应该在 git submodule update 后面添加 - -init 选项，如果子模块有嵌套的子模块，则应使用 - -recursive 选项。**  **<font color='red'>git submodule update - -init - -recursive</font>**

**如果你想自动化此过程，那么可以为 git pull 命令添加 - -recurse-submodules 选项（从 Git 2.14 开始）。 这会让 Git 在拉取后运行 git submodule update，将子模块置为正确的状态。 此外，如果你想让 Git 总是以 - -recurse-submodules 拉取，可以将配置选项 submodule.recurse 设置为 true （从 Git 2.15 开始可用于 git pull）。此选项会让 Git 为所有支持 - -recurse-submodules 的命令使用该选项（除 clone 以外）。**

在为父级项目拉取更新时，还会出现一种特殊的情况：在你拉取的提交中， 可能 .gitmodules 文件中记录的子模块的 URL 发生了改变。 比如，若子模块项目改变了它的托管平台，就会发生这种情况。 此时，若父级项目引用的子模块提交不在仓库中本地配置的子模块远端上，那么执行 git pull - -recurse-submodules 或 git submodule update 就会失败。 为了补救，`git submodule sync` 命令需要：

>#将新的 URL 复制到本地配置中
$ git submodule sync - -recursive
#从新 URL 更新子模块
$ git submodule update - -init - -recursive

### 在子模块上工作
**你很有可能正在使用子模块，因为你确实想在子模块中编写代码的同时，还想在主项目上编写代码（或者跨子模块工作）。** 否则你大概只能用简单的依赖管理系统（如 Maven 或 Rubygems）来替代了。

现在我们将通过一个例子来演示如何在子模块与主项目中同时做修改，以及如何同时提交与发布那些修改。

**到目前为止，当我们运行 git submodule update 从子模块仓库中抓取修改时， Git 将会获得这些改动并更新子目录中的文件，但是会将子仓库留在一个称作“游离的 HEAD”的状态。 这意味着没有本地工作分支（例如 “master” ）跟踪改动。 如果没有工作分支跟踪更改，也就意味着即便你将更改提交到了子模块，这些更改也很可能会在下次运行 git submodule update 时丢失。如果你想要在子模块中跟踪这些修改，还需要一些额外的步骤。**

**<font color=red>这一部分个人建议还是cd 到对应子模块 ,单独更新提交比较容易,个人使用的话，基本上就master分支,参考提示进行更新提交即可。如果子模块还需要进行切其他分支，这个就参考官方介绍,比较繁琐</font>**

### 子模的块技巧

你可以做几件事情来让用子模块工作轻松一点儿。

#### 子模块遍历

todo...



