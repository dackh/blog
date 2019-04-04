#git分支
###查看分支
	git branch						//查看本地分支
	git branch -r					//查看远端分支
	git branch -a					//查看所有分支
###创建分支
	git branch	[branch name]		//创建本地分支
###切换分支
	git checkout [branch name]		//切换分支
	git checkout -b [branch name]	//创建+切换分支，相当于git branch & git checkout
###推送新分支
	git push origin [branch name]
###删除分支
	git branch -d [branch name]		//删除本地分支
	git push origin :[branch name]	//删除远端分支，分支前面的冒号代表删除

	

#git处理冲突

当拉取下来的文件与本地修改的文件有冲突，先提交你的改变，或者先将你的改变暂时存储起来

###1、将本地修改存储起来
	git stash
###2、pull内容
 	git pull 
###3、还原暂存的内容
	git stash pop stash@{0}
也可以简写

	git stash pop


#git放弃修改文件

###本地修改了文件但还未add
	git checkout --filename		//单个文件

	git checkout .				//全部文件

###本地新增了文件还未add
	rm filename / rm dir -rf    //单个文件 //直接删除文件


	git clean -xdf				//全部文件

	// 删除新增文件，如果文件已经git add到缓存区，并不会删除
###已经git add提交到了暂存区
	git reset HEAD filename 	//单个文件
	git reset HEAD .			//全部文件
###git add以及git commit之后
	git reset commit_id			//commit_id是你回到的那个节点，可通过git log查看，可以只选前几位

	//撤销之后，已经commit的修改还在工作区
	
	git reset --hard commit_id
	
	//撤销之后，已经commit的修改将会删除，仍在工作区/暂存区的代码不会清除

#回滚远程分支


> 原理：先将本地分支退回某个commit，删除远程分支，再重新push本地分支

	git log //查看要回退的commit_id
	git branch the_branch_backup  	//备份当前这个分支的情况
	git reset --hard commit_id		//本地分支回滚到对应版本
	git push origin :the_branch 	//删除远端分支
	git push origin the_branch   	//用回滚后的本地分支建立远端分支

#git另一个进程还在运行

###问题描述
出现这种情况可能是`git`在执行的过程中，你中止之后异常，进程一直停留


	Another git process seems to be running in this repository, e.g.
	an editor opened by 'git commit'. Please make sure all processes
	are terminated then try again. If it still fails, a git process
	may have crashed in this repository earlier:
	remove the file manually to continue.

###问题原因
因为进程的互斥，所以资源被上锁，但是由于进程突然崩溃，所以未来得及解锁，导致其他进程访问不了。

###问题解决
打开隐藏文件夹选项，进入工作区文件目录的隐藏文件`.git`，把其中的`index.lock`问价删除掉

#git本地版本回退
- 通过tortoiseGit查看日志show log
- 选中需要回退的代码版本
- 右键选择Reset "master to this...
- 在弹出的窗口Reset Type选择Hard:Reset working tree and index(discard all local changes)


#git lfs的使用
操作步骤：

###1、安装lfs
安装包地址：[https://git-lfs.github.com/](https://git-lfs.github.com/ "https://git-lfs.github.com/")

###2、把项目从gitlab clone下来
	F:\镜像文件>git clone https://gitlab-sz.dianchu.cc/dc/vmware-image.git
	Cloning into 'vmware-image'...
	remote: Enumerating objects: 3, done.
	remote: Counting objects: 100% (3/3), done.
	remote: Total 3 (delta 0), reused 0 (delta 0)
	Unpacking objects: 100% (3/3), done.

###3、初始化git lfs项目
	F:\镜像文件\vmware-image>git lfs install
	Updated git hooks.
	Git LFS initialized.

###4、选择要作为大文件处理的文件扩展名
	F:\镜像文件\vmware-image>git lfs track "*.wim"
	Tracking "*.wim"

###5、提交文件
	F:\镜像文件\vmware-image>git add install.wim
	Possibly malformed conversion on Windows, see `git lfs help smudge` for more det
	ails.
	
	F:\镜像文件\vmware-image>git commit -am "添加install.wim"
	[master 43c28b8] 添加install.wim
	 1 file changed, 3 insertions(+)
	 create mode 100644 install.wim
	
	F:\镜像文件\vmware-image>git push origin huangzhibiao
	Locking support detected on remote "origin". Consider enabling it with:
	  $ git config lfs.https://gitlab-sz.dianchu.cc/dc/vmware-image.git/info/lfs.loc
	ksverify true
	Uploading LFS objects: 100% (1/1), 11 GB | 23 MB/s, done
	Counting objects: 3, done.
	Delta compression using up to 4 threads.
	Compressing objects: 100% (3/3), done.
	Writing objects: 100% (3/3), 407 bytes | 0 bytes/s, done.
	Total 3 (delta 0), reused 0 (delta 0)
	remote:
	remote: To create a merge request for huangzhibiao, visit:
	remote:   https://gitlab-sz.dianchu.cc/dc/vmware-image/merge_requests/new?merge_
	request%5Bsource_branch%5D=huangzhibiao
	remote:
	To https://gitlab-sz.dianchu.cc/dc/vmware-image.git
	 * [new branch]      huangzhibiao -> huangzhibiao
	
	


#参考
- [https://blog.csdn.net/ustccw/article/details/79068547](https://blog.csdn.net/ustccw/article/details/79068547 "https://blog.csdn.net/ustccw/article/details/79068547")
- [https://blog.csdn.net/top_code/article/details/51931916](https://blog.csdn.net/top_code/article/details/51931916 "https://blog.csdn.net/top_code/article/details/51931916")
- [https://www.cnblogs.com/wteam-xq/p/4122163.html](https://www.cnblogs.com/wteam-xq/p/4122163.html "https://www.cnblogs.com/wteam-xq/p/4122163.html")