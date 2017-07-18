## 记录下来实习期间踩的 JS 的坑

### 注意mouseenter/mouseover mouseleave/mouseout的区别

这个问题当初是由 轮播条鼠标移上去之后无法暂停导致的，后来查看原因发现是因为 mouseover 会冒泡的原因。

##### 进入

mouseenter：不冒泡
mouseover: 冒泡
把事件绑定在被选元素上，不论鼠标指针穿过被选元素或其子元素，都会触发 mouseover 事件
只有在鼠标指针穿过被选元素时，才会触发 mouseenter 事件

##### 移出

mouseleave: 不冒泡
mouseout：冒泡
把事件绑定在被选元素上面，不论鼠标指针离开被选元素还是任何子元素，都会触发 mouseout 事件
只有在鼠标指针离开被选元素时，才会触发 mouseleave 事件

mouseover/mouseout在原生JS中一直都是有的，也就是onmouseover和onmouseout事件，而如今chrome以及firefox、safari也都支持onmouseenter和onmouseleave事件。详细的兼容性可以参考MDN [mouseleave](https://developer.mozilla.org/en-US/docs/Web/Events/mouseleave)

#### 阻止 mouseover/mouseout 反复触发

这里要用到event对象的一个属性relatedTarget，这个属性就是用来判断 mouseover和mouseout事件目标节点的相关节点的属性。简单的来说就是当触发mouseover事件时，relatedTarget属性代表的就是鼠标刚刚离开的那个节点，当触发mouseout事件时它代表的是鼠标移向的那个对象。由于MSIE不支持这个属性，不过它有代替的属性，分别是 fromElement和toElement。

有了这个属性，我们就能够清楚的知道我们的鼠标是从哪个对象移过来，又是要移动到哪里去了。这样我们就能够通过判断这个相关联的对象是否在我们要触发事件的对象的内部，或者是不是就是这个对象本身。通过这个判断我们就能够合理的选择是否真的要触发事件。

这里我们还用到了一个用于检查一个对象是否包含在另外一个对象中的方法，contains方法。

