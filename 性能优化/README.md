性能优化的方案太多太多了，不过归纳起来也就是雅虎前端优化35条。

不过这里我还是谈谈具体的性能优化方案，总结一些我目前知道的或者正在使用的。

## 懒加载

对于图片数量较多的网站这是一种很不错的方案，在长页面中延迟加载图片会帮助减少服务器负载，提升用户的体验。

懒加载的实现一般是将页面上的图片的 src 属性设为 loading.gif，而图片的真实路径则设置在 data-src 属性中，页面滚动的时候计算图片的位置与滚动的位置，当图片出现在浏览器视口内时，将图片的 src 属性设置为 data-src 的值

```
function lazyload() {
	var images = document.getElementsByTagName('img');
	var len = images.length;
	var n = 0;      //存储图片加载到的位置，避免每次都从第一张图片开始遍历		
	return function() {
	    var seeHeight = document.documentElement.clientHeight;
		var scrollTop = document.documentElement.scrollTop || document.body.scrollTop;
		for(var i = n; i < len; i++) {
		    if(images[i].offsetTop < seeHeight + scrollTop) {
		        if(images[i].getAttribute('src') === 'images/loading.gif') {
			     images[i].src = images[i].getAttribute('data-src');
			}
			n = n + 1;
		    }
	    }
	}
}
var loadImages = lazyload();
loadImages();          //初始化首页的页面图片
window.addEventListener('scroll', loadImages, false);

```

## 防抖

函数防抖与节流也是比较常见的，尤其是在一些高频触发的事件上面，比如实时搜索框等等。

防抖就比如有一个电梯，十秒钟的周期，十秒之内如果有人按就不走，从0开始重新计时等到十秒。但是节流就是电梯周期是十秒，不管十秒之内有多少人按了，十秒后电梯就上楼。

比如在上面那个懒加载里面，如果直接绑定在 scroll 事件上面就会导致触发的频率十分高,因此我们使用防抖来优化。

```
function throttle(fn, delay, atleast) {
	var timeout = null,
    startTime = new Date();
	return function() {
		var curTime = new Date();
		clearTimeout(timeout);
		if(curTime - startTime >= atleast) {
		    fn();
		    startTime = curTime;
		}else {
		    timeout = setTimeout(fn, delay);
		}
	}
}
function lazyload() {
    var images = document.getElementsByTagName('img');
    var len = images.length;
    var n = 0;      //存储图片加载到的位置，避免每次都从第一张图片开始遍历		
    return function() {
	    var seeHeight = document.documentElement.clientHeight;
	    var scrollTop = document.documentElement.scrollTop || document.body.scrollTop;
	    for(var i = n; i < len; i++) {
	        if(images[i].offsetTop < seeHeight + scrollTop) {
	            if(images[i].getAttribute('src') === 'images/loading.gif') {
		        images[i].src = images[i].getAttribute('data-src');
	        }
		    n = n + 1;
	     }
	    }
    }
}
var loadImages = lazyload();
loadImages();          //初始化首页的页面图片
window.addEventListener('scroll', throttle(loadImages, 500, 1000), false);

```

## localStorage

提到 localStorage 更多人想到的其实是数据存储，相比于 cookie ,localStorage 最大的数据存储量是 5M , 适合存储一些数据量特别大的数据，比如一些静态页面。localStorage 通过调用 Window.localStorage 创建 Storage 对象，通过 Storage 对象，可以设置、获取和移除数据项。对于每个源（origin）sessionStorage 和 localStorage 使用不同的 Storage 对象——独立运行和控制。通过使用 localStorage.setItem 与 getItem 等一系列 API 可以存储自己想储存的数据。其实也有很多缺点，需要自己根据自己的项目是否真的需要，引入的成本等等去衡量。

因为之前在听猫眼技术栈分享的时候有听到C端有个项目的数据就是存放在 localStorage 里面，对于静态的不会经常发生变化的数据可以尝试使用这种方式。但是我在 B 端实习的时候是没有接触过 localStorage 存放数据的。所以具体的实现自己目前还并没有接触过。

## 再谈性能优化

提到性能优化，更多的关注点应该是放在首屏上面，毕竟如果首页打开需要十秒钟的话，用户多半都走了。 

提到首屏优化的话，如果从现在的我的角度来想，先想到的会是这个页面出现在我的眼前经历了哪些步骤，在这些步骤中有哪些可以进行优化的。

好了，这个问题又涉及到了一个最经典的问题就是：从输入 URL 到页面渲染在用户的面前经历了哪些过程。

才疏学浅的我今天就粗陋地回答一些，然后结合这些过程说下我对性能优化的理解。有些地方因为自己也不熟悉或者没有接触过的自己留坑，希望各位大佬们能释疑。

### 1.DNS解析

当你在访问 www.google.com 的时候其实是在访问谷歌的某台机器，而这台机器的唯一的标识是 IP 地址，但是对于用户来说，不可能记住这个长串的数字，因此需要 DNS 做一次域名的解析。

用下面的图来表示这个访问的过程。
![](http://ojzeprg7w.bkt.clouddn.com/%E4%BC%98%E5%8C%961.png)

当要查询 www.google.com 的 IP 的时候，会先向本地域名服务器发起请求，如果没有的话会再向根域名服务器发送一个请求，如果根域名服务器也不存在该域名时，本地域名会向 com 顶级域名服务器发送一个请求，依次类推下去。直到最后本地域名服务器得到 google 的 IP 地址。


好了，每次 DNS 解析都需要经历上面 8 个步骤，有点太麻烦了。其实真正的 DNS 解析是有涉及到缓存的，每次查询在找到之后再之后的查找中只需要从缓存中查询就可以了。

具体的过程是：

1.浏览器缓存：浏览器会缓存已经查询到的域名一阵子时间。

2.系统缓存：系统也会缓存已经查询的域名。

3.路由器缓存：当请求经过你的路由器的时候也会有缓存。

4.ISP 缓存： ISP 也会缓存。

5.最后才是之前说的递归查询。

缓存其实也是用的比较常见的手段，在这里的缓存感觉不太好受我们的控制，机制就是如此。自己对于这块其实也不是特别了解。

### 2.建立 TCP 连接

接着要开始建立 TCP 连接了，其实这一块自己并不熟悉。关于这一块的优化自己几乎不了解，如果有很了解的可以探讨。

值得一提的是 TCP 建立连接是需要三次握手的，这三次握手需要花费一些时间。但是目前的 HTTP 1.1 都已经通过一个 connection 头来保持同一个 TCP 连接。

### 3.发送 HTTP 请求

之后就是向服务器发送 HTTP 请求，比如我们还是输入 www.google.com，在 chrome 的 NetWork 里面我们能看到第一个请求信息是：

![](http://ojzeprg7w.bkt.clouddn.com/%E4%BC%98%E5%8C%962-2.png)

之后在第一个 HTML 页面的渲染之中会遇到很多资源的加载，因此还会不断地发送请求，在这张图里面也能看到浏览器其实在一直发送请求。

优化的重点也就来了，其实请求是十分耗费时间的。如果对于并行请求还好，对于一些串行请求，必须等到前面的加载完了才能加载下面的就更好费时间。

所以这里很明显就能想到如果我们减少一些请求呢？

因此我们看到雅虎 35 条军规前面提及的就是让我们减少 HTTP 请求的数目。 具体的做法里面其实也有提到：

-合并文件：如合并js，合并css都能减少请求数。如果页面间脚本和样式差异很大，合并会更具挑战性。

-雪碧图：可以合并多个背景图片，通过background-image 和 background-position 来显示不同部分。