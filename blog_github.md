
### Git使用记录
#### 1. 安装Git
电脑系统是win10.因此下载的是[Git for windows](https://git-for-windows.github.io/)。安装过程都是采用默认的参数，一路下一步。
#### 2. 创建Repository和SSH key
##### 2.1 创建Repository
* 打开GitHub网站-->进入个人主页-->Repositories-->New-->在[Repository name]输入名称HelloGit，在[Description]文本框中输入项目描述，免费用户只能选择Public-->勾选Initialize this repository with a README
##### 2.2 创建SSH key
* 打开安装好的Git Bash，输入命令设置ssh秘钥：`ssh-keygen`，运行之后需要输入存储秘钥的文件名以及查看密码，我这边都是没输，直接Enter按照默认进行下去，命令提示窗口输出：  
```
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/Administrator/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/Users/Administrator/.ssh/id_rsa.
Your public key has been saved in /c/Users/Administrator/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:ndKHdah92wO0YBl0LFHZwwK1FYLW4aeolrgSh3QijQo Administrator@westchen-PC
The key's randomart image is:
+---[RSA 2048]----+
|   .+o*=o.       |
|    =o=++        |
| o . +o...       |
|Eo..+..o .       |
|=.o..o..S .      |
|+..+ .s. = .     |
|.s= o o.. o      |
|.o . . o..       |
|o     . ..       |
+----[SHA256]-----+
```
* 生成完秘钥之后输入`ls`应该会有如下结果显示：
![ls_git](imgs/ls_git.png)  
其中id_rsa.pub就是秘钥，输入`cat id_rsa.pub`查看秘钥，复制这段秘钥从`ssh-rsa`开始一直到`Administrator@Ws-PC`，然后进入GitHub个人中心-->Edit Profile-->点击 SSH and GPG keys--> New SSH key-->输入标题，把刚才复制的秘钥粘贴到key文本框下-->点击Add SSH key完成秘钥添加。  
* 输入`ssh -T git@github.com`测试SSH秘钥是否建立，如果成功则有`Hi ***! You've successfully authenticated,but GitHub does not proviede shell access`的提示，这说明你已经连接上你的。
#### 3. 利用Git进行文件同步
* 我们现在在Github上创建了一个空的Repository，接下来我们要把本地的文件提交到这个空的Repository中，首先我们需要在本地创建一个git项目，然后将本地的git项目同步到git服务器上去。
##### 3.1 进行本地git文件的创建
* 进入到需要同步的文件目录下:`cd /d/code/note`。这是指进入'D:/code/note'目录下（注意Windows下文件目录是反斜杠，这边是斜杠）。  
* 输入`git init`进行本地git初始化，控制台提示`Initialized empty Git repository in D:/code/note/.git/`，说明已经初始化完毕。  
* 输入`git add diary.md`，是把当前目录下的diary.md文档加入到本地的git项目下了，如果嫌这样添加太慢，则输入`git add -A`则会将本目录及子目录下所有的文件都加入git项目中。  
* 输入`git show`可以查看本地git项目添加了多少文件以及文件的状态。如果你不知道还有什么命令以及命令的用法，输入`git --help`可以查看帮助文档。  
* 输入`git commit -m 'first commit'`则把当前的修改进行了首次的提交，当然此时还没有提交到GitHub的服务器上。需要注意的是，只有进行commit之后才会把更改保存到本地的git项目中。
##### 3.2 将本地的git文件同步到服务器上
* 当我们把本地的git项目编辑好之后，可以开始进行和服务器的同步了。这里面有很多的操作，在这边我们是想把本地的文件提交到服务器上，输入`git remote add origin git@github.com:用户名/Repository项目名称.git`其中用户名为你自己的用户名，Repository名称填写你刚才创建的空的Repository，这样的话你已经建立了此电脑和服务器上创建的项目的连接。  
* 输入`git pull origin master --allow-unrelated-histories`，这句话是取回服务器的master更新，然后输入`git push origin master`将本地的git项目同步到服务器上，可以看到控制台输出如下结果：  
```
Counting objects: 5, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (5/5), 7.14 KiB | 0 bytes/s, done.
Total 5 (delta 0), reused 0 (delta 0)
To github.com:*****/******.git
   5267a88..b8419dd  master -> master
```
* 这样就能完成同步了，打开自己的GitHub主页，进入到刚才的Repository中，可以看到刚才加载的文件都已经提交上来了。
### 参考资料
* [Git远程操作详解](http://www.ruanyifeng.com/blog/2014/06/git_remote.html)
* [git2.9.2使用总结](http://blog.csdn.net/u010853261/article/details/51935503)
* [4.3 服务器上的 Git - 生成 SSH 公钥](https://git-scm.com/book/zh/v1/%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A%E7%9A%84-Git-%E7%94%9F%E6%88%90-SSH-%E5%85%AC%E9%92%A5)
* [windows下使用git管理github项目（入门）](http://blog.csdn.net/emaste_r/article/details/19560321)
* [git中Please enter a commit message to explain why this merge is necessary.](http://www.cnblogs.com/wei325/p/5278922.html)
