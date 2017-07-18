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