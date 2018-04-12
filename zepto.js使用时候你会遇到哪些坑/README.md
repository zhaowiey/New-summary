## ES6
### 面向对象的继承
	移动端开发类库为什要用zepto，为什么不是 jQuery 呢？因为 jQuery 的目标是兼容所有主流浏览器，
这就意味着它的大量代码对移动端的浏览器是无用或者低效的。 而 Zepto 只针对移动端浏览器编写，因此体积更小、
效率更高，更重要的是，它的 API 完全仿照 jQuery ，所以学习成本也很低。
	
	但是在开发过程中，我发现 Zepto 还远未成熟，其中包含了一些或大或小的“坑”，与 jQuery 的差距还
是很明显的，所以写篇文章记录下，希望对后来者有帮助
注意，本文撰写时 Zepto 版本为 1.0 正式版

	*从哪里下载 Zepto
这个问题看起来很蠢，从官网下载不就行了嘛！可是你有没有发现下载链接上面有行小字呢？

	There are more modules; a list of all modules is available in the 
README.
在这个 README 里面你会惊奇地发现，Zepto 源码中有 14 个模块，而官网提供的标准版里面只有 7 个模块！
而且居然不包含对移动端开发非常重要的 touch 模块（提供对触摸事件的支持）！

所以我的建议是，不要从官网下载，而是从 Github 下载了源代码之后自己 Build 一个版本，这样你可以自行
挑选适合的模块。比如我挑选的模块是这么几个：

polyfill，zepto，detect，event，ajax，form，fx 这7个就是标准版包含的模块 fx_methods 
有了这个模块之后，.show() .hide() 等几个方法才能支持动画了，比如 .show(‘fast’) data 提供
对 .data() 方法的完整支持，像 jQuery 一样用内存对象存储 assets 移除 img 元素后做一些特殊处理，
用来清理内存 selector 更多的选择器的支持，后面会提到 touch 对触摸事件的支持，比如 tap 事件

###不要用 click 事件，用 tap 代替
这个估计已经广为人知了，因为 click 事件有 200~300 ms 的延迟，为了更快的响应，最好用 Zepto 提供
的 tap 事件
不相信的话，可以用以下代码测试一下
```
	var t1,t2;
	$('#id').tap(function () {
	    t1 = Date.now();
	});
	$('#id').click(function () {
	    t2 = Date.now();
	    alert(t2 - t1);
	});
```

###Zepto 对 CSS 选择器的支持
郑重提醒，:text :checkbox :first 等等在 jQuery 里面很常用的选择器，Zepto 不支持！
原因很简单，jQuery 通过自己编写的 sizzle 引擎来支持 CSS 选择器，而 Zepto 是直接通过浏览器提
供的 document.querySelectorAll 接口。
这个接口只支持标准的 CSS 选择器，而上面提到的那些属于 jQuery 选择器扩展，所以仔细看看这个网页，
注意一下这些选择器。

当然也有好消息，就是上面提到的 selector 模块，如果有这个模块的话，能够支持 部分 的 jQuery 选择
器扩展，列举如下：

:visible :hidden :selected :checked :parent :first :last :eq :contains :has

###元素的尺寸计算
首先 Zepto 没有 .innerHeight() .outerWidth() 等四个方法，其次，它的 .height()/.width() 
方法也不完善，对于 display:none 的元素，计算出的高宽都是 0
而这在 jQuery 里面是没有问题的，因为 jQuery 针对这种元素，会先设置其 css 样式设置
为 position: “absolute”, visibility: “hidden”, display: “block” 。
计算完高宽后再恢复，参见
https://github.com/jquery/jquery/blob/master/src/css.js#L460
如果遇到这种特殊情况，可以参考 jQuery 写一个类似的方法

###.prop() 方法的陷阱
有次我要把一个文本框置为只读，写了这么一行 (‘#text’).prop(‘readonly’,true)结果死活不工作
找了半天才发现，正确的写法是这样(‘#text’).prop(‘readOnly’, true) ，如果你居然看不出两者
的差别，那么悄悄提示你：注意大小写！
翻了一下相关的文档，原来只读属性的正确拼法确实是 readOnly，可是在 jQuery 里面上一段代码却能正常工作。
于是到 jQuery 源码里面一找才发现，还有这么一段 https://github.com/jquery/jquery/blob/master/src/attributes.js#L466
```
	jQuery.each([
    "tabIndex",
    "readOnly",
    "maxLength",
    "cellSpacing",
    "cellPadding",
    "rowSpan",
    "colSpan",
    "useMap",
    "frameBorder",
    "contentEditable"
	], function() {
	    jQuery.propFix[ this.toLowerCase() ] = this;
	});
```
从这里也能看到，jQuery 的成熟度真是难以超越，因为他把我们都惯坏了……
考虑到这段代码比较简单，我厚颜无耻地抄袭了一下然后给 Zepto 提了一个 pull request ，如果你们喜欢
这种无脑的用法，可以去评论表达支持（记得用英文）

###.show() 的动画效果
如果没有 fx_mehods 模块的话，.show() 方法是不支持动画的，不过有了这模块后，动画的支持还是有点小问题，
如果你调用 $(‘p’).show(‘fast’) ，那么动画完成后你看到的不会是一个半透明的元素，而是全黑不透明的
因为 Zepto 的 .show() 动画实现的很简单，没有高宽的变化，而是将透明度从 0 逐渐变为 1，所以元素上原来设
置的透明度就被替代了。
这种情况下，可以用 .fadeIn() 方法来替代 .show()

###结语
看到这里相信你已经了解为什么我说” Zepto 还远未成熟“，目前它其实还仅仅处于“能用”，远未达到 jQuery “好用”的地步

zeptojs的注意点 不要用 click 事件用 tap 代替 Zepto 对 CSS 选择器的支持 元素的尺寸计算 prop 方法的陷阱 show 的动画效果。