### HEAD
git 恢复文件到初始状态的命令：

```ruby
$ git reset HEAD <file>
```

git 展示提交日志命令：

```bash
$ git log
commit c4f9d71863ab78cfca754c78e9f0f2bf66a2bd77 (HEAD -> master)
```

在这些命令中常常会看到`HEAD`这个名词，它指的是什么呢？

### 回答

这要从git的分支说起，git 中的分支，其实本质上仅仅是个指向 commit 对象的可变指针。git 是如何知道你当前在哪个分支上工作的呢？  
其实答案也很简单，它保存着一个名为 HEAD 的特别指针。在 git 中，它是一个指向你正在工作中的本地分支的指针，可以将 HEAD 想象为当前分支的别名。
![[Pasted image 20220214112844.png]]
  

HEAD 指向当前所在的分支——master

所以，

-   `git reset HEAD <file>` 指的是恢复到当前分支中文件的状态。
-   `git log` 日志展示中`HEAD -> master`指的是：当前分支指向的是master分支

### Index
  Git Index是工作目录和仓库的一个临时区域，或者称之为暂存区域。你可以用这个Index去建立你要一起提交的变更。当你做一个Commit，这个Commit的当前是在Index的，而不是在工作目录中内容。

要查看什么是Index，最好的办法是用Git Status这条命令。这条显示可以显示哪些文件被staged(在index里面)，哪些修改还没有被staged，哪些还没有被跟踪。  
