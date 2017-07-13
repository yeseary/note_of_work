## 2017年6月30号
### 目标
- [x] **学习MarkDown语法**  
    主要是学习了MarkDown的基础标签，诸如标题、列表、引用、
    图片和链接、粗体与斜体、代码框等的表示和使用。
- [x] **将糯扎渡四号竖井的竖井导入至工程中**
    1. 首先是将生成的ProjectDB.sql文件和RawData.sql文件修改一下，然后运行将数据导入进来。
    2. 然后是打开原始工程文档，修改一下VC++目录中的库目录和包含目录，然后
    修改一下DBSettings.xml中的数据库参数即可。
- [ ] **看三个小时的《程序员面试宝典》**
    这个并没有完成，大概看了半个小时多吧。
-----------------------------------------------------------------
## 2017年7月1号
### 目标
- [ ] **看两个小时的《程序员面试宝典》，加上昨天剩下的一共是4个半小时**
    今天看了一个半小时吧，这样还剩下3个小时。
    <br>第五章快看完了，打算接下来先把第五章看完，然后看第三部分，也就是
    第13章到第15章。
-------------------------------------------------------------
## 2017年7月2号
### 目标
- [ ] **再看两个小时的面试宝典吧，加上昨天剩下的一共是5个小时**
上午的话，把第五章看完了，第五章讲得比较泛，讲了一下运算符、类型转换、位运算、等方面的内容，用时1小时，今日目标还剩4小时。
- [ ] **看看糯扎渡的程序能否优化一下内存，尝试从以下几个方面尝试以下：**
    1. 是不是能够用cv::Mat类型来代替IplImage对象  
    这个是不行的，在cvtest.exe里面尝试了一下，分别对CvMat和IplImage生成了一个30000\*3000\*3的一副图像，据测算，CvMat对象占用了257.5MB内存，而IplImage对象占用了257.1MB的内存，因此两者占用内存差不多。对于三通道的30000\*3000\*3的图像来说，其需要的内存为`30000*3000*3/2^20=257.49MB`，因此CvMat和IplImage占用的空间实际上和原始数据差不了多少。
    2. 优化函数传递时候的不必要的临时对象复制，采用const或者&等方式使得内存占用减小
    3. 有没有办法生成64位的程序，64位的程序是不是可以使用到更多的内存？  
    从图中可以看到64位的测试程序可以申请5G的内存都不会导致程序崩溃，因此64位程序确实可以申请到更多的内存空间。但是64位程序生成好像有很多麻烦，貌似是MFC下的很多函数和64位都有些冲突。
    ![64位程序内存测试](imgs\64位程序内存测试.png)
    4. 能不能考虑采用多线程的方式，一个线程的占用内存不超过1G的话，搞几个线程是不是能够调用到更多的内存？
--------------------------------------------------
## 2017年7月3号
### 目标
- [ ] **面试书累计到现在的话共计6小时，今天务必保证能看2-3小时**  
      今天一共看了3小时22分钟，剩余2小时38分钟
- [x] **优化内存，我觉得目前来看可行性较高的是，7月2号提出的第2、3个办法**  
    优化内存的工作已经完成，程序内存奔溃的主要原因是因为`pano_image = blendImageofTwoHeight2(pano_image,low_image);`这段代码，这里面pano_image是一个结构体，结构体里面有`IplImage *`指针，当`blendImageofTwoHeight2`结束之后，`pano_image`被释放了，但是指针指向的图像数据没有被释放，导致了占用内存越积越多，最终内存崩溃。能够发现这一问题的主要原因是重开了一个测试程序，申请了7226\*11064\*3的图像，一开始程序占用内存为：
![20170703_use_memory](imgs\20170703_use_memory.png)
    然后申请图像之后，占用内存为：
![  20170703_use_memory_large_image](imgs\20170703_use_memory_large_image.png)
    当申请完如此之大的图像之后，内存占用仅有230MB，根本不需要1GB的内存，说明原始程序肯定是内存没有清干净，于是排查到了上面那段代码，从而问题解决。因此也不需要重新编译64位版本的程序了。  
    这个例子给了我一个结论，就是如果x是很大的对象的话，`x=f(x,y)`这样的函数形式是很不靠谱的，这样会导致程序分配两倍的x对象的内存，如果采用`f(x,y)`这样的形式更加节省内存。
---------------------------------------------------------
## 2017年7月4号
### 目标
- [ ] **今天的主要任务先把几个遥感影像的处理功能集成在一起吧**  
    用控制台的方式好了，用命令行的方式实现这些功能，主要是要把Matlab的代码集成到C++里面，这个需要查一查。  
    retinex、image naturalization、Mask和Wallis需要转到C++里面  
    #### **这里正好小结一下Matlab和C++互操作一些方法**  
    这次用到的就是在C++程序里面调用Matlab写好的函数，首先是设置编译器，然后是编译.m文件，用mcc编译文件就行。我这边写.m文件的话，给定输入进行处理，然后程序结束。
    ``` Matlab
    mbuild -setup
    mcc -W cpplib:retinex -T link:lib retinex.m
    ```
    看一下retinex.m文件的函数定义及参数列表：
    ![retinex_function](imgs\retinex_function.png)   

    编译之后我们能得到很多文件，其中retinex.h、retinex.lib和retinex.dll这三个文件是我们需要的，把这三个文件分别放到程序的包含目录、库目录和dll目录（我是放到和.exe文件夹下的）里面，附加依赖库加入以下文件，第一个是生成的lib文件，后面几个是Matlab运行的依赖库。  
    ```
    retinex.lib
    libmx.lib
    libmex.lib
    libmat.lib
    libeng.lib
    mclmcrrt.lib
    mclmcr.lib
    ```
    然后设置Matlab里面的目录，如下。第一个是包含目录，第二个是库目录。
    ```
    C:\Program Files\MATLAB\R2015b\extern\include
    C:\Program Files\MATLAB\R2015b\extern\lib\win64\microsoft
    ```
    具体调用代码如下，另外在程序的头文件中需要包含'retinex.h'
![C++_USE_Matlab](imgs\c++_USE_Matlab.png)   

  我们看到在C++里面调用的形式为`retinex(1,mwOutPath,mwPath)`在C++中调用MATLAB的函数如果有返回参数的话，则输入形式为:`retinex(输出参数数量,输出参数,...,输出参数,输入参数,输入参数)`
- [ ] **考虑一下毕业设计的方向吧，虽然可能说是改变的可能不太大了，还是有空能好好想一下吧**  
- [ ] **工作岗位的选择，再看看吧**
-------------------------------------------------------------
## 2017年7月5号
### 目标  
- [x] **调试遥感影像的几个增强功能**  
    现在的话已经是集成了几个功能了，目前去雾好像有一些问题。  
    去雾的问题在于没有添加highgui那个lib，从而导致没有具体代码实现imread函数，使得返回的Mat数据为空。
    现在已经实现的功能有：retinex、dodging(Mask,Wallis)、2% linear stretch、Histgram Equlization、DeHaze based dark channel prior、Dehaze based on dark light vedio、
------------------------------------------------------------
## 2017年7月6号
### 目标  
- [x] **完成程序演示**
- [x] **修改遥感影像处理文档**
- [x] **精简程序**  
精简程序过程中，发现无法生成包含Matlab函数的部分，其如果要移植的话必须要安装MCRinstall.exe，而且Matlab2015a版本的MCR还巨大，有700多兆，以前的32位的貌似只有两三百兆，于是再找其他的方法。
在Matlab中输入`coder`能够弹出如下界面，然后选择要生成的.M文件。这种方式可以生成独立的C或C++代码，而不必依赖于Matlab运行库，但是我选择了retinex进去，发现很多函数都不支持生成，因此作罢。
![MatlabCoder](imgs\matlabcoder.png)
------------------------------------------------------------
## 2017年7月7号
### 目标  
- [x] **通过[高德API](http://lbs.amap.com/api/webservice/guide/api/direction#driving)，获取两个GPS轨迹点之间的红绿灯数目**   
直接用以前爬自行车数据的代码应该可以了，不过换了新的线程对象，一个线程对象里面设置10个线程去获取数据，线程对象可以返回得到线程函数得到的返回值，当线程执行完毕之后，返回获取到的值，将值写入到一个数据写入缓存区也就是一个list对象，检测list对象数目超过指定数目的时候将数据写入文件，每写入一条就把对应的数据读取缓存区list对象里面的数据记录删除掉。

 ------------------------------------------------------------
## 2017年7月8号
### 目标  
- [x] **优化了轨迹点之间获取红绿灯数目的数据爬取程序**  
解决了一些bug，比如获取当前是否进入下一天出错；在填充data_read的时候，设置的是`for i in range(0,MAX_COUNT)`因此当文件剩余的行数没有那么多的时候会导致程序出错，不过这个还没有改。

 ------------------------------------------------------------
## 2017年7月9号
### 目标  
- [ ] **今天貌似没做什么**  
看了半个下午和一晚上的LPL比赛...看了一个小时的面试书籍。

------------------------------------------------------------
## 2017年7月10号
### 目标  
- [ ] **糯扎渡程序，大图的显示问题 **  
**可以通过几个方式来解决这个问题：**
1.  第一种是可以采用类似于ENVI的方式，有两到三个窗口，分别是Scroll、Image和Zoom，其中Scroll显示的全图或者是缩略图，上面有一个矩形框显示的Image的显示范围，Image窗口显示的就是矩形框中图像的原始尺寸，Zoom窗口显示的是放大图像。
2. 第二是对图像进行切片，然后按照层级逐层显示。
3.
- [ ] **继续看书啊，不要停**  
欠下15个小时多了... 抓紧

------------------------------------------------------------
## 2017年7月11号
### 目标  
- [ ] **尝试了几个网上的代码**
1. [ofxGiantImage](https://github.com/timknapen/ofxGiantImage)  
这个工具依赖于OpenFramework开源库，从昨天开始一直安装这个OpenFramework开源库，但是一开始安装的是V0.7.1_VS2010_Release，一直无法编译，在iconic.h中一直报编码错误，无法解决。于是尝试最新版V0.9.8版，官网上说明这个版本是在VS2015下编译的，于是再电脑上装VS2015 Community版，下完iso文件，打开发现还需要装7G的Visual C++支持包，醉了。最终是找到了V0.7.4_vs2010_release这个版本的OpenFramework库，下载解压之后打开projectGenerator\projectGenerator.exe文件生成新项目，打开界面是这样的：  
![of_projectGenerator](imgs\of_projectgenerator.png)
运行生成的项目，导入ofxGiantImage.h和cpp文件发现是可以运行的，运行界面如下，自带的2万\*1万5分辨率的图像无法加载出来，在图像读取的时候就已经崩溃了，加载10000\*7500像素的图片是没有问题的，而且占用内存非常低。图像的显示是挺流畅的，但是无法进行图像的缩放和鼠标拖拽平移。
![ofxGiantImage_Dialog](imgs\ofxgiantimage_dialog.png)
![ofxGiantImage_Memory](imgs\ofxgiantimage_memory.png)
在查看图像显示的源码的时候，看到显示了图像的格网。
![ofxGiantImage_Grid](imgs\ofxgiantimage_grid.png)
通过查看代码，我们可以发现程序将图像分成15\*20大小的格网，然后把每个格网的图像放到一个`vector <ofTexture*> tiles`的向量中，其中分带格网分配内存`tile->allocate(tileWidth, tileHeight, GL_RGB);`并不会增加图像内存消耗，然后当图像清空之后，内存占用就非常低了，挺奇怪的。
占用内存降低可能的原因是该程序用了纹理来显示图片，而且其图像貌似还没有完整覆盖整个图像区域，每个纹理是512\*512的区域，转成纹理会导致图像的质量下降，如下进行对比：
![ofxGiantImage_Image](imgs\ofxgiantimage_image.png)
![IrfanView_Image](imgs\irfanview_image.png)
同样的一块区域，原始图像的清晰度显然要更高，而在程序之中的清晰度不够，因此数据质量得到了损失。
2. Simple2D
基于OpenGL实现的图像显示程序，其中有一个auto关键字，好像在VS2010下面支持并不好，于是转到VS2015设置一下项目的平台工具集属性就可以直接编译运行了，但是显示的图片无法平移缩放，显示的图像貌似也有一些问题。
3. DeepZoom
在VS2010中编译的话，一直缺少DirectXMath.h，在VS2015中编译没有这个问题，但是`<hashtable.h>`文件中用到模板的又出现了问题，所以并没有调通这个程序。
------------------------------------------------------------
## 2017年7月12号
### 目标  
- [ ] **按照ENVI显示的方式来显示竖井全图**
- [x] **催审**  
7月13号上午询问论文进展。

------------------------------------------------------------
## 2017年7月13号
### 目标  
- [ ] **竖井全图显示模块**  
首先我们来看一下三个打开全图的程序的耗费内存情况：

| 程序名称 |    程序阶段  |耗费内存情况|
| :------------- | :------------- | :------------- |
| irfanView 32bit|完成加载、显示| 334.2MB|
| ENVI 4.7 32bit|完成数据加载、读取| 357.6MB |
| ENVI 4.7 32bit|进行数据浏览| 410.1MB-->359.5MB |
| cvtest 64bit|分配7226*15712大小的图像|327.0MB|
| cvtest 64bit|用cvShowImage显示图像|659.6MB|
| mySketch 64bit|加载数据阶段|665.9MB|
| mySketch 64bit|纹理创建完成|40.1MB|
| mySketch 64bit|数据浏览阶段|40.1MB+显存占用多出446MB|
| mfcDrawCrack 32bit|分配7226*15712大小的图像|334.1MB|
| mfcDrawCrack 32bit|分配压缩比例0.5的缩略图|415.4MB|
| mfcDrawCrack 32bit|分配500*500的局部放大图|416.1MB|
| mfcDrawCrack 32bit|压缩比例0.1,500*500的局部放大图|338.6MB|
* 值得注意的是irfanView已经是实现了整个功能，占用内存比较稳定，这个程序肯定是没有分块的。在浏览缩放的过程中，浏览非常稳定。
* 另外ArcMap中的机制也是比较奇怪的，感觉其占用内存的一直特别低，一直稳定在100MB左右，其硬盘读写速度也很低，创建完金字塔也是如此。但是在浏览缩放的过程中，非常卡顿，经常是出现空白，要等很一会才能出现影像。
* mySketch的话，利用纹理加载影像，在一开始占用内存较多，从341.1MB跳到665.9MB再降到341.1MB，但是之后一直稳定在40多兆的样子吧，但是观察显存占用从824MB涨至1170MB，多出了446MB，我估计是将图像读取至显存中了，导致内存占用变低。
* ENVI的话，读取完图像之后程序占用内存稳定在357.6MB，在显示图像的时候瞬间增加到410多兆，随后降低至359.5MB，显存占用没有发生变换，说明ENVI是将图片完整读取到内存中的，在显示的过程进行不断的切分，在浏览过程中程序并没有卡顿。
![irfanView_Windows_contrast](imgs\irfanview_windows_contrast.png)
irfanview、Windows照片查看器、myStetch的局部细节对比
![imgs_contrast](imgs\imgs_contrast.png)
irfanview、Envi的局部细节对比
![irfanView_envi_contrast](imgs\irfanview_envi_contrast.png)
-----------------------------------------------
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
这样就能完成同步了，打开自己的GitHub主页，进入到刚才的Repository中，可以看到刚才加载的文件都已经提交上来了。
------------------------------------------------------------
## 2017年7月14号
### 目标  
- [ ] ** **  
