## 记录下来实习期间踩的 其他 类的一些坑或者总结
### 1.learn RESTful api

Learn RESTful api and write a blog about it.

Here is it : [我所理解的RESTful api](http://luckyabby.com/2017/07/10/%E6%88%91%E6%89%80%E7%90%86%E8%A7%A3%E7%9A%84RESTful-api/)

### 2.IE 不支持 window.location.origin

兼容性的写法：
```
if (!window.location.origin) {
  window.location.origin = window.location.protocol + '//' + window.location.hostname + (window.location.port ? (':' + window.location.port) : '');
}
```

### 3.git rebase

假设我的远程分支是 origin, 我新建了一个分支是 mywork,然后在分支上做了一些修改，做了两个提交 C3 C4。

这个时候有人已经基于 origin 也做了两个提交 C5 C6，这个时候样子大概是：

![](http://ojzeprg7w.bkt.clouddn.com/rebase1.png)

这个时候如果我们想整合这两个分支，最快的方法就是 git merge 命令，它会把两个分支最新的快照以及二者最新的共同祖先（C2）进行三方合并，合并的结果是产生一个新的提交对象（C7），如下图：

![](http://ojzeprg7w.bkt.clouddn.com/rebase2.png)

这个时候分支看上去就很乱，有个更好用的命令叫 git rebase master。

这个命令会把 C5 C6 里面产生的变化补丁在 C4 的基础上再打一遍。它会回到两个分支最近的祖先元素就是 C2，根据 mywork 的后续若干次提交生成一系列文件补丁（这里是 C5' C6'），以 origin 分支最后一个提交对象（C4）为新的出发点，逐个应用之前的文件补丁，最后生成一个新的合并对象（C7'），从而改写 mywork 的提交历史，使它成为 origin 分支的直接下游。最后的结果如图：

![](http://ojzeprg7w.bkt.clouddn.com/rebase4.png)

使用 rebase 我们能使得远程分支更加干净，解决分支补丁同最新主干代码之间冲突的责任。

使用 rebase 和 merge 得到的结果最后其实是一样的，只是提交的历史不一样。 rebase 是按照每次的修改次序重演一遍修改，合并是把最终的结果合并在一起。

在使用 rebase 的时候可能会提示我们有冲突，这个时候需要我们手动去解决冲突，解决完之后直接使用 git add 去更新索引，之后使用`git rebase --continue`使得 git 使用余下的补丁。

可以在任意时候通过`git rebase --abort`来停止rebase,这个时候 mywork 分支会回到 rebase 开始前的状态。

使用`git rebase`要住一个准则就是 **一旦分支中的提交对象发布到公共仓库，就千万不要对该分支进行衍合操作。** 因为此时可能有别人在你的分支的基础上开发新的需求，如果此时再对分支进行变基，会使得别人的提交一团糟。

### 4.Linux 系统用户账号管理
#### 1.添加新的用户账号

```
useradd 选项 用户名
其中各选项含义如下：
-c comment 指定一段注释性描述。
-d 目录 指定用户主目录，如果此目录不存在，则同时使用-m选项，可以创建主目录。
-g 用户组 指定用户所属的用户组。
-G 用户组，用户组 指定用户所属的附加组。
-s Shell文件 指定用户的登录Shell。
-u 用户号 指定用户的用户号，如果同时有-o选项，则可以重复使用其他用户的标识号。
```
例如：
```
# useradd -s /bin/sh -g group –G adm,root gem
此命令新建了一个用户gem，该用户的登录Shell是/bin/sh，它属于group用户组，同时又属于adm和root用户组，其中group用户组是其主组
```
#### 2.删除账号

如果一个用户的账号不再使用，可以从系统中删除。删除用户账号就是要将/etc/passwd等系统文件中的该用户记录删除，必要时还删除用户的主目录。删除一个已有的用户账号使用userdel命令，其格式如下：

```
userdel 选项 用户名
常用的选项是-r，它的作用是把用户的主目录一起删除。
例如：
# userdel sam
此命令删除用户sam在系统文件中（主要是/etc/passwd, /etc/shadow, /etc/group等）的记录，同时删除用户的主目录。
```

#### 3.修改账号
参数同增加新用户

```
usermod -s /bin/ksh -d /home/z –g developer sam
此命令将用户sam的登录Shell修改为ksh，主目录改为/home/z，用户组改为developer。
```
### 5.jQuery 使用 FormData 上传文件

使用 jQuery 通过FormData对象上传文件时需要注意:

(1）processData设置为false。因为data值是FormData对象，不需要对数据做处理

(2）contentType设置为false。因为是由<form>表单构造的FormData对象，将 contentType 设置为 false 是为了避免 jQuery 对其操作，从而失去分界符，而使服务器不能正常解析文件。

### 6.clientHeight srcollHeight offsetHeight

![](http://ojzeprg7w.bkt.clouddn.com/clientheight.png)

clientHeight: 内部可视区域大小。

offsetHeight：整个可视区域大小，包括border和scrollbar在内。

scrollHeight：元素内容的高度，包括溢出部分。

scrollTop：元素内容向上滚动了多少像素，如果没有滚动则为0。

innerHeight是DOM视口的大小，包括滚动条。

outerHeight是整个浏览器窗口的大小，包括窗口标题、工具栏、状态栏等。

```
var height = window.innerHeight
    || document.documentElement.clientHeight
    || document.body.clientHeight;
```
事实上后两种方式获取的高度和window.innerHeight是不一样的，这3个属性的值逐个变小。

具体说来，window.innerHeight包括整个DOM：内容、边框以及滚动条。

documentElement.clientHeight不包括整个文档的滚动条，但包括<html>元素的边框。

body.clientHeight不包括整个文档的滚动条，也不包括<html>元素的边框，也不包括<body>的边框和滚动条。

offsetTop: 当前元素顶部距离最近父元素顶部的距离,和有没有滚动条没有关系。单位px，只读元素。
