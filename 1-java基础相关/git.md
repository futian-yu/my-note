**git**

//1.为什么git需要公钥？  因为你每次pull/push的时候，git需要确认是你推送的，而不是别人冒充的。就像家里的门一样，需要你有钥匙才可以开，并且钥匙可以同时多人拥有，即pub_rsa可以配置多个。

​		补充：其实用钥匙打比方还不太准确，因为git添加的公钥是不同的，相当于不用的钥匙，只要经过我认证，你就可以开这个门。有点类似于楼下门禁的人脸识别。   git私钥的作用是在你本地经过你配置之后，可以不用每次推/拉代码都输入密码。    



- **git常用命令**

```java
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
	
​git merge合并
	git merge [branch] ：合并指定分支到当前分支。eg：git merge 	bug : 合并bug分支到当前分支。

​挑拣另一个分支的指定提交到当前分支
	git checkout test
	git cherry-pick 62ecb3：挑拣develop的62ecb3提交到test分支(62ecb3是develop的分支)
        
​合并某个分支上的一系列commits
	在一些特性情况下，合并单个commit并不够，你需要合并一系列相连的commits。这种情况下就不要选择cherry-pick了，rebase 更适合。还以上例为例，假设你需要合并feature分支的commit76cada ~62ecb3 到master分支。
	首先需要基于feature创建一个新的分支，并指明新分支的最后一个commit：
	git checkout -b newbranch 62ecb3
	然后，rebase这个新分支的commit到master（--ontomaster）。76cada^ 指明你想从哪个特定的commit开始。
	git rebase --onto master 76cada^
	得到的结果就是feature分支的commit 76cada   ~ 62ecb3  都被合并到了master分支。        

```

