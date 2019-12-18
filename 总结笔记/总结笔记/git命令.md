问题：为什么是--？

- git init：初始化一个Git仓库。
- git add <file>：添加文件到Git仓库。
- git commit -m "distribute"：提交文件到Git仓库。
- git status：查看当前仓库的状态。
- git diff <file>：查看修改的内容。
- git log：查看提交日志（按时间从最近到最远排序）。
- git log --pretty=oneline：一行显示提交日志。
- git reflog：查看命令历史。
- git reset --hard 版本号
	- HEAD：表示当前版本。
	- HEAD^：上一个版本。
	- HEAD^^：上上一个版本。
	- HEAD~n：n个版本以前。
	- 提交ID
- git checkout -- <file>：丢弃工作区的修改。让文件回到最后一次git commit或git add的状态。
- git reset HEAD file：把暂存区的修改回退到工作区，HEAD表示最新版本。
- git rm：删除一个文件。
- ssh-keygen -t rsa -C "jctonly66@gmail.com"：生成一个SSH key。
- git remote add origin git库地址：本地库与远程库关联。
- git push -u origin master：推送master的所有分支。
- git clone：克隆
- git checkout -b 分支名：创建并切换分支。
- git branch 分支名：创建分支。
- git checkout 分支名：切换分支。
- git branch：查看所有分支。
- git merge <name>：合并某分支到当前分支。
- git merge --no-ff：可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而fast forward合并就看不出来曾经做过合并。
- git branch -d <name>：删除分支。
- git log --graph：查看合并分支图。
	-  git log --graph --pretty=oneline --abbrev-commit
- git stash：将当前工作现场储存起来。
- git stash list：查看储存起来的工作现场。
- git stash apply：恢复工作现场。
- git stash drop：删除储存起来的工作现场。
- git stash pop：恢复并删除工作现场。
- git branch -D <name>：强行删除未被合并的分支。
- git remote：查看远程库的信息。
- git remote -v：显示远程库更详细的信息。
- git remote show [remote-name]：查看远程库的所有信息。

工作区与暂存区：

我们的文件目录就是工作区，隐藏目录.git时Git的版本库（版本库中最重要的就是一个称为stage的暂存区，还有主分支master以及指向master的指针HEAD）。

add实际上就是把文件修改添加到暂存区，commit是把暂存区的所有内容提交到当前分支。