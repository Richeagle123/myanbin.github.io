---
layout: post
title: '我的开发日常 Git 命令清单'
---


从 2011 年开始接触 Git 到现在，我的使用时间不算太短，但是却只限于 `git add`、`git commit`、`git push` 这几个简单命令。最近因为工作需要，我将 [Pro Git](https://git-scm.com/book/zh/v2) 通读了一遍，故写此篇。


## 前言

![Git 工作流](https://infp.github.io/blogimages/git-status.png){:.center}

Git 是一个分布式的版本控制系统，是指 Git 的远程仓库（Remote）和本地仓库（Repository）具有同等的地位，保存了代码的所有历史记录。上面的图来自阮一峰的[博客](http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html)，表示在 Git 的各个状态间相互切换的命令。

## 创建代码仓库

有两种方法来建立 Git 代码仓库。第一种是在现有目录下直接生成：

~~~sh
$ git init
~~~

另外一种是从服务器上直接克隆一个远程的仓库，比如：

~~~sh
$ git clone https://github.com/myanbin/jsterm.git
~~~

## 配置 Git

在初次运行 Git 前，需要配置一下用户信息和编辑器偏好：

~~~sh
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
~~~

配置你喜欢的编辑器，这样当 Git 需要输入信息时，便会调用它：

~~~sh
$ git config --global core.editor vim
~~~

Git 有三个级别的配置文件，分别为系统级的 `/etc/gitconfig`，用户级的 `~/.gitconfig` 和代码仓库级的 `.git/config`，每一个级别覆盖上一级别的配置。一般地，我们只需要读取用户级和代码仓库级的配置即可。

另外，我们可以通过设置别名，将较长的 Git 命令简写：

~~~sh
$ git config --global alias.co checkout
$ git config --global alias.br branch
$ git config --global alias.ci commit
$ git config --global alias.st status
~~~

这样，当要输入 `git status` 时，只需要输入 `git st` 即可。


## 添加

本地的代码仓库由 Git 维护的三颗 Tree 组成。第一个是工作目录（Workspace），它持有实际的文件；第二个是暂存区（Index/Stage），它像一个缓存区，临时保存即将要提交的改动；第三个是代码仓库（Repository），它有一个 HEAD 指针，用于指向你最后一次提交的结果。

![Git 的三个区域](https://infp.github.io/blogimages/git-areas.png){:.center}

当你再工作目录中修改了某个文件后，使用 `git add` 命令，可以将你在工作目录下的改动添加到暂存区，以便准备提交。比如：

~~~sh
$ echo hello,world > README.md
$ git add README.md
~~~

这个时候再查看 Git 仓库状态，应该类似于下面：

~~~sh
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   README.md

~~~

如果要添加多个文件到暂存区，也可以使用 `git add -i` 进行交互式提交。

`git add` 的本质是维护一个准备提交的改动清单。所以执行 `git add` 时，添加到暂存区的是改动而不是文件。比如上面的例子，当用 `git add` 添加 `README.md` 之后，`git commit` 会将这次改动提交。但是假如没来得及 `git commit`，你又在 `README.md` 文件末尾添加了一行，如果你没有再次 `git add README.md`，这次修改是不会被提交的。

其实在 Git 中，每次运行 `git add <filename>` 时，便会计算该文件的 SHA-1 哈希值作为本次改动的唯一标识。


## 提交和推送

在完成上一步的添加之后，使用如下命令以便将本次修改动永久记录下来：

~~~sh
$ git commit -m "first commit"
~~~

现在，改动已经提交到了本地仓库的 HEAD，但是还没有到远程仓库。继续执行:

~~~sh
$ git push origin master
~~~

便可以把这些改动推送到远程仓库的 master 分支上。


## 撤销操作

有时候当提交完后发现漏掉几个文件没有添加，或者提交信息写错时，可以使用带有 `--amend` 选项的提交明亮重新提交：

~~~sh
$ git commit -m "initial commit"
$ git add forgotten_file
$ git commit --amend -m "second commit"
~~~

## 分支和 Git 工作流

Git 鼓励在工作流程中频繁地使用分支和合并，甚至是一天之内进行多次。

Git 的分支，其实本质上仅仅是指向提交对象的可变指针。Git 的默认分支名字是 `master`，它会在每次的提交操作中自动向前移动。

Git 可以通过 `git branch` 命令创建分支：

~~~sh
$ git branch test
~~~

分支建好之后，你当前仍然在 `master` 分支上，这时需要切换分支到 `test`：

~~~sh
$ git checkout test
~~~

这时，Git 内部的 `HEAD` 指针便会指向到 `test`。

> 上面创建和切换分支的命令，可以简写成 `git checkout -b test`。

下面我们来看一个使用 Git 分支的例子：假如你正在为某个网站实现一个新的需求，正在此时，线上网站有一个严重的问题需要紧急修补，上级把这个任务指派给了你。这时，你需要把当前手头的工作暂停，来解决这个优先级更高的问题。



## 查看日志

使用 `git log` 命令可以查看代码的提交历史，不带任何参数的话，它会列出每个提交的 SHA-1 校验和、作者的名字和邮箱地址、提交时间以及提交说明。

下面的表格列出了 `git log` 的常用选项：

| 选项   | 说明   |
|-------------------|-------------------|
| `-p`              | 按补丁格式显示每个更新之间的差异   |
| `--stat`          | 显示每次更新的文件修改统计信息   |
| `--abbrev-commit` | 仅显示 SHA-1 的前几个字符，而非所有的 40 个字符   |
| `--graph`         | 显示 ASCII 图形表示的分支合并历史   |
| `--pretty`        | 使用其他格式显示历史提交信息，如 format   |
| `-<n>`            | 仅显示最近的 n 条提交   |
| `--since=<date>`  | 仅显示指定时间之后的提交   |
| `--until=<date>`  | 仅显示指定时间之前的提交   |
| `--author=<name>` | 仅显示指定作者相关的提交   |
| `--grep=<patten>` | 仅显示含指定关键字的提交   |

甚至你通过下面的命令，来个性化 `git log` 的输出格式：

~~~sh
git log --graph --pretty=format:'%Cred%h%Creset - %C(yellow)%d%Creset %s %Cgreen (%cr) %C(blue)<%an>%Creset' --abbrev-commit
~~~


## 打标签

Git 可以给历史中的某一个提交打上标签，以示重要。想要列出 Git 中的所有标签，只需输入：

~~~sh
$ git tag
v0.1
v0.9
v1.0
~~~

Git 可以创建两种不同类型的标签：轻量标签（lightweight）与附注标签（annotated）。用法如下：

~~~sh
$ git tag v1.8
$ git tag -a v2.0 -m "public version"
~~~

目前为止，我们所打的标签只在本次仓库中。如果想要与其他人共享这些标签，可以使用如下命令：

~~~sh
$ git push origin v2.0
~~~

当其他人从仓库中克隆或拉取时，便能得到这些标签。