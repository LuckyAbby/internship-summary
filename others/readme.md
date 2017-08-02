## 记录下来实习期间踩的 其他 类的一些坑或者总结
### learn RESTful api

Learn RESTful api and write a blog about it.

Here is it : [我所理解的RESTful api](http://luckyabby.com/2017/07/10/%E6%88%91%E6%89%80%E7%90%86%E8%A7%A3%E7%9A%84RESTful-api/)

### IE 不支持 window.location.origin

兼容性的写法：
```
if (!window.location.origin) {
  window.location.origin = window.location.protocol + '//' + window.location.hostname + (window.location.port ? (':' + window.location.port) : '');
}
```

### git rebase

假设我的远程分支是 origin, 我新建了一个分支是 mywork,然后在分支上做了一些修改，做了两个提交 C3 C4。

这个时候有人已经基于 origin 也做了两个提交 C5 C6，这个时候样子大概是：

![](http://ojzeprg7w.bkt.clouddn.com/rebase1.png)

这个时候如果我们想整合这两个分支，最快的方法就是 git merge 命令，它会把两个分支最新的快照以及二者最新的共同祖先（C2）进行三方合并，合并的结果是产生一个新的提交对象（C7），如下图：

![](http://ojzeprg7w.bkt.clouddn.com/rebase2.png)

这个时候分支看上去就很乱，有个更好用的命令叫 git rebase。

这个命令会把 C5 C6 里面产生的变化补丁在 C4 的基础上再打一遍。它会回到两个分支最近的祖先元素就是 C2，根据 mywork 的后续若干次提交生成一系列文件补丁（这里是 C5' C6'），以 origin 分支最后一个提交对象（C4）为新的出发点，逐个应用之前的文件补丁，最后生成一个新的合并对象（C7'），从而改写 mywork 的提交历史，使它成为 origin 分支的直接下游。最后的结果如图：

![](http://ojzeprg7w.bkt.clouddn.com/rebase4.png)

使用 rebase 我们能使得远程分支更加干净，解决分支补丁同最新主干代码之间冲突的责任。

使用 rebase 和 merge 得到的结果最后其实是一样的，只是提交的历史不一样。 rebase 是按照每次的修改次序重演一遍修改，合并是把最终的结果合并在一起。

在使用 rebase 的时候可能会提示我们有冲突，这个时候需要我们手动去解决冲突，解决完之后直接使用 git add 去更新索引，之后使用`git rebase --continue`使得 git 使用余下的补丁。

可以在任意时候通过`git rebase --abort`来停止rebase,这个时候 mywork 分支会回到 rebase 开始前的状态。


