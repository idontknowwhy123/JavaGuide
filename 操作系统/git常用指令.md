1.git abandon某条提交导致后续提交无法merge,
原因:Git提交代码后发现编译无法通过，后面把该提交abandon掉结果后续提交编译通过却无法merge,因为后面的commit都是基于abandon掉的那条生成的
（1）git status 查看当前状态，将修改过但不必提交的进行git checkout恢复
（2）git log --graph 找到abandon的那条commit以及上传一次commit的ID
（3）git reset ID  将代码回退到abandon之前的那个节点
（4）git status 查看当前状态
（5）git add .
（6）git commit -m ""
 (7) git pull origin master --rebase(直接git pull会把远程分支上的修改拉下来和你的修改合并，这样提交曲线会产生菱形，rebase能够避免菱形的产生
 保证提交的曲线直线)
 rebase的原理:首先将你本地当前分支的提交(commit)取消掉，并且把他们临时保存为补丁（这些补丁放".git/rebase"目录中），然后把本地当前分支更新为最新的远程
 分支，最后把保存的这些补丁应用到本地当前分支上
 注：这个过程可能会产生冲突，解决冲突后，再git add ,git rebase --continue
 （8）git push origin HEAD:refs/for/master
 
 2.git push出现[remote rejected]you are not allowed to upload merges错误提示
 错误原因：你基于origin远程创建了一个叫“mywork”分支，然后在mywork分支上进行了修改，与此同时，有人在origin分支上做了一些修改并提交，这就意味着
 "origin"和"mywork"这两个分支“分叉”了，之后你在git pull,再git push就会出现这种问题。
 解决办法:git pull origin master --rebase
 
 3.一些基本回退
 git reset ID:将提交到本地仓库的代码回退到暂存区（对应的是git commit）
 git checkout file:撤销对工作区的修改。
 git reset HEAD file：撤销从本地提交到暂存区的代码（对应的是git add）
 
 4.missing change -ID
 1.首先通过git push提示的错误信息获取该文件
 2.如果缺失change -ID的是最后一个commit,则直接git commit --amend生成change -ID再push即可
 3.如果缺失change -ID的不是最后一个commit,则用git reset commit-ID回退到该commit,再git commit --amend生成change -ID
 4.最后把git reset撤销掉的代码重新commit，然后push即可。
 注：git commit --amend当我们想对上一次的提交进行修改时，我们可以使用git commit --amend（既可以对上一次提交的内容进行修改，也可以修改提交说明）
 
 5.提交代码之前git  pull保证本地代码是最新的，就不会有冲突
 
 6代码冲突git pull之后报错error:your local changes to the following files would be overwritten by merge
 1.git stash
 2.git pull origin master
 3.git stash pop
 4（然后在IDEA上根据提示进行修改）
 解决了冲突之后一定要提交，否则后面的代码无法提交
 
 7.报错[remote  rejectd](change http:......closed)
 git log发现有重复的commit ID和change ID
 解决办法:git reset到重复的commit ID之前的那次commit，再重复提交
 
 8.报错[remote rejected](cannot upload review)没有权限提交
 
 9.git submit including parents
 出现问题原因：commit互相依赖，gerrit上已经存在了commitA,commitA还没merge入库，然后在commitA基础上进行了修改，并做了新的commitB,
 commitB包含了commitA的修改，然后在gerrit库上abandon commitA，只留下commitB在gerrit库上，这样commitB review通过后就会出现上述问题
 
 解决办法：1.从远程分支上重新创建一个新的工作分支：git fetch origin master(远程分支)：new_work(新分支)
          2.切换到新的工作分支：git checkout new_work
          3.将commitB移到新的分支（gerrit页面右上角download直接复制cherry-pick命令）：git fetch ssh:xxx && git cherry-pick xxx
          4.正常解决冲突流程
          5.正常提交代码：git push origin HEAD:refs/for/master(需要提交的分支)
          6.刷新gerrit.重新做code view
          
 解决办法2（待验证）:1.abandon掉commitB
                   2.git reset commitA之前的那次提交
                   3.再重新提交本地代码
                   
 10.git出现（master|rebase）
    取消master分支上的rebase操作
    git rebase --abort
    
 11.git出现commit已经push上去了，但是无法merge的问题
    原因：上次commit被abandon掉了，这次提交节点的父节点对应这次anandon的节点
    解决办法：reset到abandon那次节点的上次一次提交，然后再重新提交push

 12.matser分支切换到其他分支报错
    error:pathspec 'master' did not match any file(s)know to git 
    
 13.git commit --amend出现报错you are in the middle of a merge --connot amend问题解决办法
    1. 取消合并
    git reset --merge
    2.将当前分支重新设置基线
    git rebase 
    3.在idea里面解决代码冲突
    4.让rebase继续
    git rebase --continue
    5.追加修改后的文件
    git add .
    6.提交修改后的代码
    git commit --amend
    7.git push

14.git stash 引发的问题
原本是：git stash save -u "描述"
写成了：git stash save -a "描述"
-u： 会把没有记录到的文件也保存下来(比如你新建了一个文件，但是还没有git add，stash也会把这个文件保存下来)
-a： 会把忽略的文件也保存下来（.gitignore中的）
https://blog.csdn.net/weixin_30700977/article/details/97360968

