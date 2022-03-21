简单来说，带上-u 参数其实就相当于记录了push到远端分支的默认值，这样当下次我们还想要继续push的这个远端分支的时候推送命令就可以简写成git push即可。

示例展示：
下面展示一个示例来说明这一点。

andy@AndyMacBookPro:/usr/local/github/andy/php-examples$ git pull
There is no tracking information for the current branch.
Please specify which branch you want to merge with.
See git-pull(1) for details.

git pull <remote> <branch>

If you wish to set tracking information for this branch you can do so with:

git branch --set-upstream-to=origin/<branch> test

这个就是如果你之前未使用 -u 参数，后面省略了想要pull的分支参数而产生的结果。pull 因为没有track for the current branch. 所以他不知道你要从哪里pull，所以这也就是 -u 参数的意义，指定trach branch。

其实你可以在指定完-u之后，去.git/config看GIT配置文件，可以看到下面有了branch "test"的分支的记录：
<code>
[branch "master"] 
 	remote = origin 	
	merge = refs/heads/master 
[branch "test"] 	
	remote = origin 	
	merge = refs/heads/test
<code>

这样git才能知道当前test下的remote和merge的信息，如果你在git push的时候没有带入-u参数，那么config中就不会有branch "test"这一项。
<code>
 [branch "master"]
    remote = origin
    merge = refs/heads/master 
	<code>

配置说明，这告诉Git 2件事：

当您在master分支上时，默认的遥控器是origin。
在git pullmaster分支上使用时（未指定任何远程和分支），请使用默认的remote（源）并合并来自remote master分支的更改。
配置修改
您可以手动去.git/config修改GIT配置文件内容，也可以使用命令行设置这些选项。

 $ git config branch.master.remote origin
 $ git config branch.master.merge refs/heads/master
1
2
如果使用命令进行配置，它将有一定的纠错能力。比如您键入了一个不存在的分支或者您没有执行git remote add 操作。在较新的git中，希望您使用 git branch --set-upstream-to=origin/master master

其实，执行添加了-u 参数的命令 git push -u origin master就相当于是执行了

git push origin master 和
git branch --set-upstream master origin/master。

所以，在进行推送代码到远端分支，且之后希望持续向该远程分支推送，则可以在推送命令中添加 -u 参数，简化之后的推送命令输入。

说明1：
Git can set a specific branch in the remote repository as the default “upstream” branch for that particular branch. For example, if you clone an existing repository, GIT by default associates your master branch with the branch in the origin repository from which you want to clone. This means that git can provide useful default values, such as only git pull is used when it is opened, and master does not have to specify the repository and branch from which to get and merge.

However, if you do not want to clone from an existing repository, but you want to create a new origin remote control, which represents the newly created GitHub library, you must manually notify git’s associated master and master about the new origin library. In - U and git push “as well as pulling” You only need to do it once to record the association in .git / config.
