**3.** git常用命令记录。



使用git pull ：





使用git pull --rebase ：



```
添加到暂存区
	git add --all :添加所有目录的中的已修改文件到暂存区。
	git rm [file] :删除工作区文件，并且将这次删除放入暂存区。
	git mv [file-original] [file-renamed] :改名文件，并且将这个改名放入暂存区。
	
​提交到本地仓库
	git commit -m [message] :提交暂存区到仓库区。
	git commit --amend -m [message] :使用一次新的commit，替代上一次提交;如果代码没有任何新变化，则     用来改写上一次commit的提交信息。
	
​分支
	git branch : 列出所有本地分支。
	git branch -r : 列出所有远程分支
	git checkout -b [branch] : 新建一个分支，并切换到该分支
	git branch --set-upstream-to=origin/remote_branch  local_branch : 将本地分支与远程分支关     联，这样在执行git pull, git push操作时就不需要指定对应的远程分支。
	git merge [branch] : 合并指定分支到当前分支。
	
​忽略/恢复跟踪
	git update-index --no-assume-unchanged  [file]  #恢复跟踪
	git update-index --assume-unchanged   [file]    #忽略跟踪
	
​撤销与版本回退
	git checkout [file] : 恢复暂存区的指定文件到工作区。
	//git checkout [commit] [file] : 恢复某个commit的指定文件到暂存区和工作区。
	//git reset --hard : 置暂存区与工作区，与上一次commit保持一致。
	//git reset --hard [commit] : 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定    	commit一致。
	git stash
	git stash pop : 暂时将未提交的变化移除，稍后再移入。
	
	========================================================
	git reset HEAD readme.txt  :将readme.txt从暂存区移出，恢复到工作区；
	git reset --hard HEAD^ : 回退到上一个版本（也可以用指定版本代替HEAD^）
	git reset --soft HEAD^ ：撤销了commit操作，但是写的代码依然保留(用于不小心commit);
		git log:查看版本库的状态；
		git reflog:记录你的每一次命令;
	
	
	
	
	
	
	
	
​
​
​
​
​
​










git add . : 只提交当前目录或者它后代目录下相应文件。

​	git add readme.txt : 将单个文件加入到暂存区。

​	git add readme.txt ant.txt：将多个文件加入到暂存区。

​	git add *.html ：使用通配符方式批量提交文件。

（1）.git add all无论在哪个目录执行都会提交相应文件。

（2）.git add .只能够提交当前目录或者它后代目录下相应文件。
```

```
git commit -m"[ADD] A.java" : 提交所有文件到本地仓库。

​	git push origin master : 提交所有文件到中央仓库。（master分支下）
```

```
	git branch : 列出本地所有分支.

​	git branch -r : 列出所有远程分支.

​	git branch -a : 列出所有本地分支和远程分支。

​	git branch dev : 列出分支dev。

​	git checkout : 切换分支或恢复工作树文件

​	git checkout bug : 切换到指定分支，并更新工作区

​	git checkout -b bug : 新建一个分支，并切换到该分支
```

​	

```
	git config user.name/user.email : 查看用户名/邮箱。

​	git remote - v : 显示所有远程仓库

​	git merge [branch] ：合并指定分支到当前分支。eg：git merge 	bug : 合并bug分支到当前分支。

​	git pull --rebase origin master : 拉取合并本地分支和远程分支。
```

​	git diff  readme.txt： 看看readme.txt文件上次修改了什么内容。diff即difference

4.mvn clean install -U : 强制更新

-------------------

**不熟悉的命令：**git branch -vv

​							

----------

-------------------------------------------------------个人记录-------------------------------------------

1.git checkout -b feature-yufutian.  创建并切换到你的分支。（git branch feature_yufutian + git checkout feature_yufutian）

2.git checkout master.	切换回主分支

3.git pull --rebase oringin master. (拉取更新后的代码，这一步一定要做，先拉再推(先更新再修改)).

4.git checkout feature-yufutian. 切换回自己的分支。（**本地**自己的分支，修改完后再合并到**本地**自己的master分支。）

5.git merge master.将自己的分支合并到主分支。（我们本地上的主分支，我们在自己电脑上操作的都是自己电脑上的本地分支，只有最后push的时候才会到远程分支。）

(也可以先切换到master分支，再合并本地分支 // git checkout master.   git merge feature_yufutian)

//git branch  查看当前分支。  git branch -d dev. 合并完成后，可以删除dev分支.

- 我们注意到，切换分支使用git checkout <branch>,而撤销修改也用checkout：git checkout --<file>.同一个命令，有两种作用，确实有点令人迷惑。

  实际上，切换分支这个动作，我们可以用 switch 更加科学。//git switch -c dev. 创新并切换到dev分支。  git switch master.   直接切换到已有的master分支。

6.git pull --rebase origin master. (在推送之前，需要先拉取一下最新的代码，不拉取应该也行（据说还是要先拉一下，预防冲突。）)

7.git push -u origin feature-yufutian.(推送到远程自己的远程分支。注意：我们不能推送到远程的主线分支，在实际开发中我们没有这个权限，因为远程主线master分支是受保护的).





//$ git push -u origin master 上面命令将本地的master分支推送到origin主机，同时指定origin为默认主机，后面就可以不加任何参数使用git push了。

不带任何参数的git push，默认只推送当前分支，这叫做simple方式。





**注意：**将本地分支与远程分支创建关联：

 	git push --set-upstream origin yto-yft

**git创建一个远程分支并将本地分支推送远程分支：**

​	git push origin localbranch:localbranch

**注意**：git push的时候提示找不到远程分支，这时候可以git branch -a查看所有分支；查看远程分支是否存在，不存在的话就可以用上面的命令新建一个远程分支；而且有时候显示推送成功了，进git网页界面却找不到对应的代码，这是因为你一开始进去的时候一般默认再master分支，当你推送到其他远程分支，当然一进去看不到了，这时候选择一下分支就可以看到，或者将远程分支合并到master分支；



----

-------------------git 忽略/恢复跟踪

你要整哪样？:
git update-index --no-assume-unchanged  <file>  #恢复跟踪

你要整哪样？:
git update-index --assume-unchanged   <file>    #忽略跟踪



- git版本回退！

  ​	**git reset --hard HEAD^**   表示回退到上一个版本    HEAD^^表示上上一个版本，依次类推；

   

  1.如果不知道自己要回退到哪个版本；可以使用

  ​      **git log** 命令显示最近到最远的提交日志；如果觉得信息太多，眼花缭乱的；还可以加上参数：

  ​      **git log --pretty=online**   这样只显示一行。

  

  2.然后commit之后，如果想撤销commit；重新提到猪齿鱼上，就需要再回退版本，如下：

  ​	**git reset --soft HEAD^**这样就成功的撤销了你的commit（注意，仅仅是撤回commit操作，您写的代码仍然保留）

  ​	此处，--soft也可以改成 --hard；soft不删除工作空间的改动代码，撤销了commit，不撤销add； hard删除工作空间的改动代码；

  

  顺便说一下：如果commit注释写错了，只是想改一下注释，只需要：git commit --amend

  

------------------------------------------------------------------------------

# - 简单对比git pull和git pull --rebase的使用

使用下面的关系区别这两个操作：
git pull = git fetch + git merge
git pull --rebase = git fetch + git rebase

现在来看看git merge和git rebase的区别。

假设有3次提交A,B,C。

![img](http://images2015.cnblogs.com/blog/907596/201609/907596-20160922155014777-999552544.png)

在远程分支origin的基础上创建一个名为"mywork"的分支并提交了，同时有其他人在"origin"上做了一些修改并提交了。

![img](http://images2015.cnblogs.com/blog/907596/201609/907596-20160922155038152-1733703139.png)

其实这个时候E不应该提交，因为提交后会发生冲突。如何解决这些冲突呢？有以下两种方法：

1、git merge
用git pull命令把"origin"分支上的修改pull下来与本地提交合并（merge）成版本M，但这样会形成图中的菱形，让人很困惑。

![img](http://images2015.cnblogs.com/blog/907596/201609/907596-20160922155107949-1520786903.png)

2、git rebase
创建一个新的提交R，R的文件内容和上面M的一样，但我们将E提交废除，当它不存在（图中用虚线表示）。由于这种删除，小李不应该push其他的repository.rebase的好处是避免了菱形的产生，保持提交曲线为直线，让大家易于理解。

![img](http://images2015.cnblogs.com/blog/907596/201609/907596-20160922155132715-596060966.png)

在rebase的过程中，有时也会有conflict，这时Git会停止rebase并让用户去解决冲突，解决完冲突后，用git add命令去更新这些内容，然后不用执行git-commit,直接执行git rebase --continue,这样git会继续apply余下的补丁。
在任何时候，都可以用git rebase --abort参数来终止rebase的行动，并且mywork分支会回到rebase开始前的状态。

================================

- 管理修改

git为何优秀？

​		因为Git跟踪并管理的是修改，而非文件。

什么是修改？比如说你新增了一行，这就是一个修改，删除了一行，也是一个修改，更改了某些字符，也是一个修改，删了一些又增加了一些，也是一个修改，甚至创建一个新文件，也是一个修改；

============

如果不想合并整个分支，git也提供了一个专门的命令    cherry-pick; 让我们能够复制一个特定的提交到当前分支；





============

**版本回退：**

​	回退到上一个版本，可以用：

​	**git reset --hard HEAD^**

  如果想回退到某一个指定的版本,可以先

```
git reflog   或   git log  看看想回退到那一次提交；
git reflg 记录了你的每一次命令，git log 记录了你的每一次提交；
```

如果只是想撤销当前提交，比如其他都是对的，只有配置文件不能提交上去；