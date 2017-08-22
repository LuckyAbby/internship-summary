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

### js异步编程
js异步编程主要分为回调函数、promise、以及async、await

回调最大的问题是无法完全信任回调函数的操作，而promise 主要解决控制反转的问题

async await 可以像写同步代码一样写异步



