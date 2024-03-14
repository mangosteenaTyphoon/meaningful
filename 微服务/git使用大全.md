> git在我们平时工作无处不在，本文简单的记录一下git的基础使用方法。

首先，下载好git这个插件，看清楚电脑是32位还是64位。直接傻瓜式安装就好。下一步 --> 下一步的点击就可以。

安装好之后右键点击设置可以看到“git bash here” 就是安装成功了

进入页面后输入创建一个新的仓库：

```powershell
git init
```


然后，添加你的远程仓库地址：

~~~powershell
git remote add origin https://.................(你的地址)
~~~


注意：地址是要去仓库复制下来然后直接粘贴的，有http和ssh,我们复制http格式的哦！

第三步：把你的代码放入暂存区：

~~~powershell
git add .
~~~

或

~~~powershell
git add <文件名>
~~~


第一个是“空格 点” 代表把所有文件都放入暂存区，第二个是指定文件放入暂存区。

第四步：提交到head,并给一个备注信息（方便看）

~~~powershell
git commit -m "暂存"
~~~


现在已经提交完毕，但是还没有到远程仓库中。

一般不提交到master中，都是提交到分支中。

但默认是`master`，所以`checkout`切换分支（如果没有会自动创建的）

```powershell
git checkout -b <分支名>
```

checkout完毕会显示：

```powershell
switched to a new branch "分支名"
```

切换到你自己的分支之后就可以推送代码了！但在此之前，，最好需要pull一下，也就是更新一下，即使没有什么改动也要更新一下哦。

~~~powershell
git pull origin <分支名>
~~~


更新成功了！最后一步了！

推送！

~~~powershell
git push origin <分支名> -f
~~~


 我加了一个`“-f”` 是可以强行推送的，管他三七二十一，直接成功。

以上，就是提交代码到远程仓库的步骤，大家试一下吧。

另外补充一些命令：

```powershell
git branch -a
```


可以查看目前有哪些分支。

```powershell
git log
```


可以查看有提交id.

```powershell
git merge
```


可以合并分支，常用的我目前了解的暂时就这么多了，后续如果有我用到其他的会更新的。

如果我们在拉取代码的时候出现如下错误

~~~ powershell
fatal: refusing to merge unrelated histories
~~~

可以使用以下指令进行解决

~~~ powershell
git pull origin 分支名 --allow-unrelated-histories
~~~

## 本地代码与远程代码冲突

### 强行拉取远程覆盖本地代码

依次执行下面命令

~~~sh
git fetch --all
git reset --hard origin/所在分支名
git pull
~~~

