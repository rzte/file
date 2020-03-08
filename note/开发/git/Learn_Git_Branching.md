# [Learn Git Branching](https://learngitbranching.js.org/?NODEMO)

# 基础篇

## 分支(git branch)

```
git branch bugFix			# 新建一个bugFix的分支
git checkout bugFix 		# HEAD指向bugFix

git checkout -b bugFix		# 新建一个bugFix分支并且HEAD指向bugFix
```

## 合并分支（git merge）

```
git checkout master			# 移动到master分支上
git merge bugFix			# bugFix分支与master分支合并
```

## 合并分支2（git rebase）

第二种合并分支的方法是 git rebase。Rebase 实际上就是取出一系列的提交记录，“复制”它们，然后在另外一个地方逐个的放下去

![](/images/Sat-Dec--8-12:32:17-2018_303097.jpg)

# 高级篇

## 分离HEAD

分离的 HEAD 就是让其指向了某个具体的提交记录而不是分支名。在命令执行之前的状态如下所示：

![](/images/Sat-Dec--8-13:10:38-2018_576177.jpg)

## 相对引用 ^

```
git checkout master^				# 相当于 master 的第一个父节点
git checkout master^^ 				# 相当于 master 的第二个父节点
git checkout HEAD^					# 当前位置的第一个父节点
```

^ 后面也可以加数字，不过不同于 ~，并不是用来指定向上返回几代，而是指定合并提交记录的某个父提交。

![](/images/Sat-Dec--8-18:32:46-2018_330008.jpg)

## 相对引用2 ~

使用 `~<num>` 可以一次后退 num 个节点

```
git checkout HEAD~4					# 后退4步
```

**强制修改分支位置**

```
git branch -f master HEAD~3				# 将 master 分支强制指向 HEAD 的第三级父提交
```

![](/images/Sat-Dec--8-13:21:08-2018_414948.jpg)

## 撤销变更

- git reset
	git reset 通过把分支记录回退几个提交记录来实现撤销改动。你可以将这想象成“改写历史”。git reset 向上移动分支，原来指向的提交记录就跟从来没有提交过一样。
	![](/images/Sat-Dec--8-16:37:28-2018_430179.jpg)
- git revert
	git reset 对远程分支是无效的，为了撤销更改并分享给别人，需要使用 git revert
	![](/images/Sat-Dec--8-16:46:56-2018_347050.jpg)

# 移动提交记录

- git cherry-pick <记录1> <记录2> <记录3> ...

	![](/images/Sat-Dec--8-17:02:04-2018_726323.jpg)

- 交互式 rebase（git rebase --interactive）简写 (git rebase -i)

	当你知道你所需要的提交记录（并且还知道这些提交记录的哈希值）时, 用 cherry-pick 再好不过了 —— 没有比这更简单的方式了。但是如果不清楚要提交记录的哈希值时，可以使用交互式的rebase

	![](/images/Sat-Dec--8-17:22:43-2018_524833.jpg)

# 杂项

## git tag

git tag，为历史中某个提交打标签

![](/images/Sat-Dec--8-17:59:51-2018_775689.jpg)

## git Describe

由于标签在代码库中起着“锚点”的作用，Git 还为此专门设计了一个命令用来描述离你最近的锚点（也就是标签），它就是 git describe！

`git describe` 的语法是：

```bash
git describe <ref>		# <ref> 可以是任何能被 Git 识别成提交记录的引用，如果你没有指定的话，Git 会以你目前所检出的位置（HEAD）。
```

它输出的结果是这样的：（当 ref 提交记录上有某个标签时，则只输出标签名称）

```bash
<tag>_<numCommits>_g<hash>	# tag 表示的是离 ref 最近的标签， numCommits 是表示这个 ref 与 tag 相差有多少个提交记录， hash 表示的是你所给定的 ref 所表示的提交记录哈希值的前几位。
```

![](/images/Sat-Dec--8-18:07:20-2018_236685.jpg)

## 远程仓库

远程分支有一个命名规范：`<remote name>/<branch name>`

也就是说，如果看到一个名为 `origin/master` 的分支，那么这个分支就叫做 `master`，远程仓库的名称就是 `origin`。（当用`git clone`某个仓库时，Git已经把远程仓库的名称设置为`origin`了）

- **git fetch**

	- 从远程仓库下载本地仓库中缺失的提交记录
	- 更新远程分支指针(如 origin/master)

	**要注意，git fetch 并不会改变你本地仓库的状态。它不会更新你的 master 分支，也不会修改你磁盘上的文件**

- **git pull**

	在 git fetch 后，你可以像合并本地分支那样来合并远程分支。也就是说就是你可以执行以下命令:

	- git cherry-pick o/master
	- git rebase o/master
	- git merge o/master
	- 等等

	由于先抓取更新再合并本地分支这个流程很常用，所以git提供了一个专门的命令来完成这两个操作，这就是`git pull`

- **git pull --rebase**

	相当于
	```
	git fetch
	git rebase origin/master
	git pull
	```

## merge 与 rebase

在开发社区里，有许多关于 merge 与 rebase 的讨论。以下是关于 rebase 的优缺点：

- 优点:
	Rebase 使你的提交树变得很干净, 所有的提交都在一条线上

- 缺点:
	Rebase 修改了提交树的历史
	比如, 提交 C1 可以被 rebase 到 C3 之后。这看起来 C1 中的工作是在 C3 之后进行的，但实际上是在 C3 之前。

一些开发人员喜欢保留提交历史，因此更偏爱 merge。而其他人（比如我自己）可能更喜欢干净的提交树，于是偏爱 rebase。仁者见仁，智者见智。 :D

## 追踪远程分支

默认情况下，使用的`master`分支追踪远程仓库`origin/master`（git pull、git push等命令默认都是操作的这个master分支），可以通过下面方式修改，让任意分支跟踪`origin/master`

- 通过远程分支检出一个新的分支

	```bash
	git checkout -b totallyNotMaster origin/master				# 这样就创建了一个 totallyNotMaster 分支跟踪 origin/master 分支了
	```

- 设置远程追踪分支

	```bash
	git branch -u origin/master foo					# 这样 foo 就会跟踪 origin/master 了

	# 如果当前就在 foo 分支上，还可以省略 foo
	git checkout -b foo
	git branch -u origin/master
	```

## git push

- git push <remote> <place>
	```bash
	git push origin master				# 将本地的master推送到远程的master（可以用上面的“追踪远程分支”修改本地master对应的远程分支）
	git push origin foo					# 将本地的foo推送到远程的foo（可以用上面的“追踪远程分支”修改本地foo对应的远程分支）
	```
- git push <remote> <source>:<destination>
	```bash
	git push origin foo^:master			# 将本地的 foo 之前的一个提交记录（foo^)推送到远程的master分支
	git push origin master:newBranch	# 将本地的master分支提交到远程的一个新分支（远程没有newBranch这个分支会自己创建）
	git push origin :foo				# <source> 为空，它会删除远程仓库中的 foo 分支
	```

## git fetch

git pull同样适用，git pull相当于 git fetch 后 git merge

```bash
git fetch						# 如果 git fetch 没有参数，它会下载所有的提交记录到各个远程分支
git fetch origin foo			# Git 会到远程仓库的 foo 分支上，然后获取所有本地不存在的提交，放到本地的 origin/foo 上。
git fetch origin foo~1:bar		# Git 将远程的foo~1位置的提交记录抓取到本地 bar 这个分支上（若bar不存在则创建）
git fetch origin :bar			# <source>为空，则它会在本地创建一个新分支 bar
```












