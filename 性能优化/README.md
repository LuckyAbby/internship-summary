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