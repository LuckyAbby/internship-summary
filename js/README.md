## 记录下来实习期间踩的 JS 的坑
### 1.注意mouseenter/mouseover mouseleave/mouseout的区别
这个问题当初是由 轮播条鼠标移上去之后无法暂停导致的，后来查看原因发现是因为 mouseover 会冒泡的原因。

#### 进入
mouseenter：不冒泡
mouseover: 冒泡
把事件绑定在被选元素上，不论鼠标指针穿过被选元素或其子元素，都会触发 mouseover 事件
只有在鼠标指针穿过被选元素时，才会触发 mouseenter 事件

#### 移出
mouseleave: 不冒泡
mouseout：冒泡
把事件绑定在被选元素上面，不论鼠标指针离开被选元素还是任何子元素，都会触发 mouseout 事件
只有在鼠标指针离开被选元素时，才会触发 mouseleave 事件

mouseover/mouseout在原生JS中一直都是有的，也就是onmouseover和onmouseout事件，而如今chrome以及firefox、safari也都支持onmouseenter和onmouseleave事件。详细的兼容性可以参考MDN [mouseleave](https://developer.mozilla.org/en-US/docs/Web/Events/mouseleave)

#### 阻止 mouseover/mouseout 反复触发
这里要用到event对象的一个属性relatedTarget，这个属性就是用来判断 mouseover和mouseout事件目标节点的相关节点的属性。简单的来说就是当触发mouseover事件时，relatedTarget属性代表的就是鼠标刚刚离开的那个节点，当触发mouseout事件时它代表的是鼠标移向的那个对象。由于MSIE不支持这个属性，不过它有代替的属性，分别是 fromElement和toElement。

有了这个属性，我们就能够清楚的知道我们的鼠标是从哪个对象移过来，又是要移动到哪里去了。这样我们就能够通过判断这个相关联的对象是否在我们要触发事件的对象的内部，或者是不是就是这个对象本身。通过这个判断我们就能够合理的选择是否真的要触发事件。

这里我们还用到了一个用于检查一个对象是否包含在另外一个对象中的方法，contains方法。

#### 执行的顺序
##### 1. 鼠标移入时，触发mouseover、mouseenter和mousemove事件
浏览器先触发mouseover和mouseenter事件，再触发多次mousemove事件（IE还未测试）

##### 2. 鼠标移出时，触发mousemove、mouseleave和mouseout事件
所有浏览器的顺序都是(1)mousemove、(2)mouseout和(3)mouseleave事件

### 2.数组、对象、字符串常用转换

因为实习做的产品主要是 B 端产品，与后端之间数据交互的数据结构一般比较复杂，涉及到数组、对象以及字符串之间互相转换等等。

#### 1.记录一种将数组转换成对象的方法：

一般对于一个表格的数据后端一般是返回一个对象数组给前端，但是如果前端需要的是一个大对象做另外一些处理。这个时候可以：

```
// res是返回来的数组
const data = {};
Object.keys(res).forEach(item => {
data[item] = res[item];
});
```
这里介绍一下 Object.keys 这个方法。根据 MDN

> Object.keys() 方法会返回一个由一个给定对象的自身可枚举属性组成的数组，数组中属性名的排列顺序和使用 for...in 循环遍历该对象时返回的顺序一致 （两者的主要区别是 一个 for-in 循环还会枚举其原型链上的属性）。

举个例子：
```
const arr = ["abby", "node"];
console.log(Object.keys(arr)); //["0", "1"]
```
数组也是对象，因此对数组使用这个方法就会得到数组的索引，使用循环可以给一个空对象添加上属性(数组的索引)与对应的值，这个时候数组就转化成对象啦。也是小 tips。

至于对象实例转换成数组，只要遍历对象实例上面的属性，然后赋值给空数组的每个值。

```
const data = [];
let i = 0;
for (let prop in selected) {
  if (selected.hasOwnProperty(prop)) {
    data[i++] = selected[prop];
  }
}
```

#### 4.记录一些经常容易忘记的 api

对于字符串

##### 1.slice(beginSlice[, endSlice])

1.索引（以 0 为基数）截取一个字符串并返回新的字符串，参数是负数的时候相当于 length+endSlice
```
var str1 = 'The morning is upon us.';
var str2 = str1.slice(4, -2); // 不包含倒数第二个字符串

console.log(str2); // OUTPUT: morning is upon u
```

对于数组

##### 1.slice()

包含开始位置不包含结束位置地浅拷贝一份数组，并返回新数组
```
var fruits = ['Banana', 'Orange', 'Lemon', 'Apple', 'Mango'];
var citrus = fruits.slice(1, 3);

// fruits contains ['Banana', 'Orange', 'Lemon', 'Apple', 'Mango']
// citrus contains ['Orange','Lemon']
```

##### 2.splice

splice() 方法通过删除现有元素和/或添加新元素来更改一个数组的内容。不返回新的数组，在原数组上面操作
```
var myFish = ['angel', 'clown', 'mandarin', 'sturgeon'];

myFish.splice(2, 0, 'drum'); // 在索引为2的位置插入'drum'
// myFish 变为 ["angel", "clown", "drum", "mandarin", "sturgeon"]

myFish.splice(2, 1); // 从索引为2的位置删除一项（也就是'drum'这一项）
// myFish 变为 ["angel", "clown", "mandarin", "sturgeon"]

```

##### 3.find ／ findIndex

对数组进行遍历找到满足测试函数的第一个元素的值/索引

##### 4.filter

方法创建一个新数组, 其包含通过所提供函数实现的测试的所有元素。

### 3.js异步编程
js异步编程主要分为回调函数、promise、以及async、await

回调最大的问题是无法完全信任回调函数的操作，而promise 主要解决控制反转的问题

async await 可以像写同步代码一样写异步

#### 注意异步的问题

之前碰到一个问题是两个异步请求拿到数据，同时第二个异步的数据操作是依赖第一个异步的。之前在线下测试的时候没有问题，但是在预上线的版本上面拿到的数据一直存在问题，后来发现是拿到数据的时候依赖关系没有处理好，后来使用 Promise.all 将两个异步操作完成之后再进行数据的操作，这样就保证执行数据正确了。

### 4.使用fetch的时候踩坑

fetch默认不会把cookie带上，不管是同域还是跨域；因此对于有一些需要权限验证的请求就可能会出现403等等问题，导致无法正常获取数据，这是只需要配置其credentials项。

这个属性有三个值，omit: 默认值，表示忽略cookie的发送；same-origin: 表示cookie只能同域发送，不能跨域发送；include: cookie既可以同域发送，也可以跨域发送。

因此只需要使用fetch(url,{credentials: 'include'})就可以使得发出的请求带上cookie信息了。

### 5.将数字转换成货币

自己用循环写了一个函数
```
function toThousands(num) {
  let count = 0
  let result = []
  const indexPoint = String(num).indexOf('.')
  const interPart= String(num).split('.')[0]
  const interArray = interPart.split('')
  for(let i = interArray.length - 1; i >= 0; i--) {
    count++
    result.unshift(interArray[i])
    if(count % 3 === 0 && i != 0) {
        result.unshift(',')
    }
  }
  if (indexPoint > 1) {
    const pointPart = String(num).split('.')[1]
    return `${result.join('')}.${pointPart}`
  }
  return result.join('')
}
```
或者还有另一种循环的写法:
```
function toThousands(num) {
  let result = ''
  const indexPoint = String(num).indexOf('.')
  let interPart = String(num).split('.')[0]
  while(interPart.length > 3) {
    result = `,${interPart.slice(-3)}${result}`
    interPart = interPart.slice(0, interPart.length - 3)
  }
  if (indexPoint > 1) {
    const pointPart = String(num).split('.')[1]
    return `${interPart}${result}.${pointPart}`
  }
  return `${interPart}${result}`
}
```
其实更简单的还有正则，但是最简单的是原生的api:`Number.prototype.toLocaleString()`，可以快速转换成各个国家的货币表示。

### 6.前端实现一键复制

复制的基本思路就是选中+复制，其中选中可以使用表单元素的select()方法，注意只有 input 和 textarea 可以执行 select() 方法，复制可以使用 API`document.execCommand()`。

这样最简单的思路就是在页面中创建一个 input 元素，但是次元素不能设置成`display:none`，可以设置成`position:relative;left:-9999px`，这样DOM结构也存在。这样再将此input元素的值设置成需要复制的内容，再使用input.select()方法选中，最后使用`document.execCommand("copy")`即可执行复制命令。


### 7.将字符串描述的数组变成对象

使用 JSON.Parse 可以很方便地将 JSON 字符串变成 由字符串描述的 JS 值或者对象。举个例子比如将``'[a, [b], c]'``变成一个对象。可以用如下的方法：

```
const str = '[a, [b], c]'
const arr = JSON.parse(str.replace(/\w+/g, match => `"${match}"`))
console.log(arr) //  ["a", ["b"], "c"]
```
这样就可以轻松地将字符串变成对象，相比于进行字符串操作要简单的多。

### 8.使用FormData上传文件

**踩坑**
使用 FormData post 上传文件的时候注意不要设置 headers,这样容易造成`no multipart boundary`的问题，因为会为请求头正确设置边界，但如果你设置了，你的服务器都没法预知你的边界是什么（因为边界是被自动加到请求头的）

不论是 fetch 还是 ajax 发的请求都需要注意这个问题
