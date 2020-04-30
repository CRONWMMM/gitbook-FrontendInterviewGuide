---
description: 这两个东西，你肯定听过，就是两种优化浏览器性能的手段。相关文章你肯定也看过，如果还是不太清楚，没关系，看完这篇短文，相信你能轻松理解其中差别。
---

# 防抖和节流

### 防抖（deounce）

我们先说防抖吧，这里有个小笑话，看完你应该就秒懂了：

> 小明军训，教官发令：向左转！向右转！向后转！大家都照着做，唯有小明坐下来休息，教官火的一批，大声斥问他为啥不听指挥？小明说，我准备等你想好到底往哪个方向转，我再转。

虽然是个笑话，却很好地说明了防抖的定义：**给一个固定时间，如果你开始触发动作，并且在这个固定时间内不再有任何动作，我就执行一次，否则我每次都会重新开始计时**。我们可以用极端情况理解它：如果给定时间间隔足够大，并且期间一直有动作触发，那么回调就永远不会执行。放在笑话的语境里就是，只有教官最后一次发号后，小明才会转。

### 节流（throttle）

如果你理解了防抖，关于节流，就更好理解了

> 学生上自习课，班主任五分钟过来巡视一次，五分钟内随便你怎么皮，房顶掀了都没事，只要你别在五分钟的时间间隔点上被班主任逮到，逮不到就当没发生，逮到她就要弄你了。

在这里，班主任就是节流器；你搞事，就是用户触发的事件；你被班主任逮住弄，就是执行回调函数。

节流的定义：**用户会反复触发一些操作，比如鼠标移动事件，此时只需要指定一个“巡视”的间隔时间，不管用户期间触发多少次，只会在间隔点上执行给定的回调函数。**，我们同样可以用极端情况来理解：如果给定的间隔时间是 `240毫秒`，用户永不间断地在屏幕上疯狂移动鼠标，那么你的回调函数会分别在 `240毫秒`、 `480毫秒`、 `720毫秒`... 就这么一直执行下去

### 这俩有啥用？

* 防抖（deounce）:
  * 可用于**input.change**实时输入校验，比如输入实时查询，你不可能摁一个字就去后端查一次，肯定是输一串，统一去查询一次数据。
  * 可用于 **window.resize** 事件，比如窗口缩放完成后，才会重新计算部分 `DOM` 尺寸
* 节流（throttle），用于监听 `mousemove`、 鼠标滚动等事件，通常可用于：**拖拽动画**、**下拉加载**。

> 节流通常用在比防抖刷新更频繁的场景下，而且大部分是需要涉及动画的操作。

### Talking is cheap show me the code

#### 防抖

```javascript
function debounce (fn, delay = 200) {
  let timeout;
  return function() {
    // 重新计时
    timeout && clearTimeout(timeout);
    timeout = setTimeout(fn.bind(this), delay, ...arguments);
  }
}

const handlerChange = debounce(function () {alert('更新触发了')})

// 绑定监听
document.querySelector("input").addEventListener('input', handlerChange);
```

#### 节流

```javascript
function throttle (fn, threshhold = 200) {
  let timeout;
  // 计算开始时间
  let start = new Date();
  return function () {
    // 触发时间
    const current = new Date() - 0;
    timeout && clearTimeout(timeout);
    // 如果到了时间间隔点，就执行一次回调
    if (current - start >= threshhold) {
      fn.call(this, ...arguments);
      // 更新开始时间
      start = current;
    } else {
      // 保证方法在脱离事件以后还会执行一次
      timeout = setTimeout(fn.bind(this), threshhold, ...arguments);
    }
  }
}

let handleMouseMove = throttle(function(e) {
  console.log(e.pageX, e.pageY);
})

// 绑定监听
document.querySelector("#panel").addEventListener('mousemove', handleMouseMove);
```

> 从代码上看不难发现，节流只是在防抖的基础上加了时间差的判断，多穿了件马甲而已。

### 小兄弟，你是否还有很多问号？？

1. 防抖用在 `input.change` 上我能理解，鼠标拖拽功能为啥不能用防抖？用的话会发生什么？

   > 仔细想一想，简单来说，防抖其实是只会触发一次，明显不能用在这种拖拽场景下，如果硬用上去，会出现用户拖了弹框它不动，过了一会儿“啪”地就跳过去了。出现这种情况，老板是要请你喝茶的。

2. 节流的代码，为什么要加上那句 `setTimeout` ？我不加会怎么样？

   > 这句代码的主要作用，说白了其实就是“兜底”，它确保了你的回调不管怎么样，一定会执行一次。还是用鼠标拖拽弹框功能举例，这个兜底解决了拖拽弹框在速度很快的情况下弹框不动的问题、也确保你最后拖拽完成，放开鼠标，弹框能回到你鼠标的位置，就是代码注释里写的：保证方法在脱离事件以后还会执行一次。

其实这两个问题，是我初见防抖和节流的时候所疑惑的，希望也能帮助到你。

### 再深入一点

本来是没有写这块内容的，但是评论区有朋友指出了节流函数存在的问题，我就顺带着又了解了一些防抖和节流的细节。

我们可以从上面的节流函数入手，你能否看出存在什么问题呢？这个也是评论区朋友指出的：**这个节流函数第一次不会立即执行，而是会等待一段时间执行，并且这个等待时间越长，延迟越是明显**，我们用这个思路再审视一下防抖函数，也同样有这个问题。

那么怎么解决呢？针对防抖和节流，其实有**立即执行**和**非立即执行**两种版本，上面写的就是非立即执行版本。想解决上面的问题也很简单，把函数改造一下，使用立即执行版本就可以了~只需要改变一行代码。（同理，节流也是类似）

```javascript
// 立即执行的防抖函数
function debounce (fn, delay = 200) {
  let timeout;
  return function() {
    // 如果 timeout == null 说明是第一次，直接执行回调，否则重新计时
+   timeout == null ? fn.call(this, ...arguments) : clearTimeout(timeout);
    timeout = setTimeout(fn.bind(this), delay, ...arguments);
  }
}

const handlerChange = debounce(function () {alert('更新触发了')})

// 绑定监听
document.querySelector("input").addEventListener('input', handlerChange);
```

### 可视化

如果你还是无法直观感受二者差别，进入 [传送门](http://demo.nimius.net/debounce_throttle/)

### 相关参考

[知乎：函数防抖与函数节流](https://zhuanlan.zhihu.com/p/38313717) 文章是大佬司徒正美写的，谨以技术缅怀大佬。

### 总结

关于节流和防抖这两个概念，我也看过很多解说，说实话，很少有文章能真的说清楚的。基本上就是官腔+代码流的一套组合拳，都在说 `Talking is cheap show me the code`，但是很多时候，往往是 `Fucking code is so difficult, talk to me please.` 真正能让人豁然开朗的，应该是大白话。

真心希望这篇文章能帮到你，帮到了点个赞，感谢！

