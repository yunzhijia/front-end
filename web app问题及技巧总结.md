## 一些问题：
1. **问题**：手机端 `click` 事件会有大约 `300ms` 的延迟  
<p>**原因**：手机端事件 `touchstart` --\> `touchmove` --> `touchend` or `touchcancel` --> `click`，因为在touch事件触发之后，浏览器要判断用户是否会做出双击屏幕的操作，所以会等待300ms来判断，再做出是否触发click事件的处理，所以就会有300ms的延迟
<p>**解决方法**：使用touch事件来代替click事件，如 [zepto.js](https://github.com/madrobby/zepto) 的tap事件和[fastClick](https://github.com/ftlabs/fastclick)，还有我自己也写了个移动端手势操作库 [mTouch](https://github.com/DMQ/mTouch)，都有相应的事件可以代替click事件解决这个问题
&nbsp;
2. **问题**：在部分机型下（如小米4、小米2s、中兴） `body` 设置的 `font-size` 是用 `rem `单位的话，若其他元素没有设置font-size，该font-size值继承于body，则会很高概率出现**字体异常变大**的情况
<p>**原因**：估计是跟app的webview默认设置有关，body的font-size使用rem单位，就是相对于当前根节点的font-size来确定的，可能在某些webview的设置下，body用的是webview设置的默认字体大小，因为在我给html设置了一个px单位的默认font-size时，还是会出现字体异常变大的情况，具体webview的一些细节就没有再研究了
<p>**解决方法**：body设置一个`px`单位的默认font-size值，不用rem，或者给字体会异常变大的元素设定一个px单位的font-size值
&nbsp;
3. **问题**：使用zepto的 `tap` 事件时会出现“点透”bug，比如：一个元素A绑定了tap事件，紧跟其后的元素B绑定了click事件，A触发tap事件时将自己remove掉，B就会自动“掉”到A的位置，接下来就是不正常的情况，因为这个时候B的click事件也触发了
<p>**原因**：因为tap事件是通过 `touchstart` 、`touchmove` 、 `touchend` 这三个事件来模拟实现的，在手机端事件机制中，触发touch事件后会紧接着触发touch事件坐标元素的click事件，因为B元素在300ms内刚好“掉”回来A的位置，所以就触发了B的click事件，还有zepto的tap事件都是代理到body的，所以想通过e.preventDefault()阻止默认行为也是不可行的
<p>**解决方法**：（1）A元素换成click事件；（2）使用我写的库 [mTouch](https://github.com/DMQ/mTouch) 来给A绑定tap事件，然后在事件回调中通过e.preventDefault()来阻止默认行为，或者换用其他的支持tap事件的库
&nbsp;
4. 待续...

## 一些有用技能点
1. 通过设置css属性 `-webkit-tap-highlight-color: rgba(0, 0, 0, 0);`取消掉手机端webkit浏览器 点击按钮或超链接之类的 默认灰色背景色
<p>
2. 设置css属性 `-webkit-user-select:none;` 控制用户不可选择文字
<p>
3. 区域性 `overflow: scroll | auto` 滚动时使用原生效果：`-webkit-overflow-scrolling: touch` （ios8+，Android4.0+）
<p>
4. 单行、多行溢出省略

```
/* 单行省略 */
.ellipsis {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
}

/* 多行省略 */
.ellipsis-2l {
    overflow : hidden;
    text-overflow: ellipsis;
    display: -webkit-box;
    -webkit-line-clamp: 2;    /* 数值代表显示几行 */
    -webkit-box-orient: vertical;
}
```
<p>
5. 垂直居中常用方法：
```
<!-- html结构 -->
<body>
    <div class="wrap">
        <div class="box"></div>
    </div>
</body>

/* css样式 */

/* (1) 模仿单行文字居中的方式 */
.wrap {
    width: 200px;
    height: 80px;
    line-height: 80px;
}

.box {
    display: inline-block;
    vertical-align:middle;
}

/* (2) 已知宽高，通过position:absolute; */
.wrap {
    width: 200px;
    height: 200px;
    position: relative;
}

.box {
    width: 100px;
    height: 80px;
    position: absolute;
    left: 50%;
    top: 50%;
    margin: -50px 0 0 -40px;
}

/* (3) 未知宽高，通过css3属性 transfrom */
.wrap {
    width: 200px;
    height: 200px;
    position: relative;
}

.box {
    position: absolute;
    left: 50%;
    top: 50%;
    -webkit-transform: translateX(-50%) translateY(-50%);
    transform: translateX(-50%) translateY(-50%);
}

/* (4) 通过flex布局 */
<!-- html结构 -->
<body>
    <div class="wrap flexbox flexbox-center flexbox-middle">
        <div class="box"></div>
    </div>
</body>

/* css样式 */

.flexbox {
	display: -webkit-box; 
    display: -moz-box; 
	display: -ms-flexbox; 
	display: -webkit-flex;
	display: flex;
}

/* 水平居中 */
.flexbox-center {
	-webkit-box-pack: center; 
	-moz-box-pack: center; 
	-ms-flex-pack: center; 
	-webkit-justify-content: center;
	justify-content: center;
}

/* 垂直居中 */
.flexbox-middle {
	-webkit-box-align: center; 
	-moz-box-align: center;
	-ms-flex-align: center; 
	-webkit-align-items: center;
	align-items: center;
}
```
6. retina屏幕实现真正的1px边框
```
<!-- html结构 -->
<body>
    <div class="box retina-border rt-bd-all"></div>
</body>

/* css样式 */

.box {
    width: 200px;
    heigth: 100px;
    box-sizing: border-box;
    border: 1px solid #aaa;
}

/* 去掉元素原有的边框 */
.retina-border {
    position: relative;
    border: none;
}

/* 通过设置伪元素放大到2倍的宽高，设置1px边框，再缩小1倍，以达到0.5px边框的效果*/
.retina-border:after {
    content: '';
    display: block;
    width: 200%;
    height: 200%;
    position: absolute;
    left: 0;
    top: 0;
    box-sizing: border-box;
    border: 0px solid #aaa;
    -webkit-transform-origin: left top;
    transform-origin: left top;
    -webkit-transform: scale(.5);
    transform: scale(.5);
}

.rt-bd-all:after {
    border-width: 1px;
}

/* 如果只是想设置一条边框，可以这样改一下，以此类推 */

<!-- html结构 -->
<body>
    <div class="box retina-border rt-bd-b"></div>
</body>

/* css样式 */

.tr-bd-b:after {
    border-bottom-width: 1px;
}

.tr-bd-t:after {
    border-top-width: 1px;
}

.tr-bd-l:after {
    border-left-width: 1px;
}

.tr-bd-r:after {
   border-right-width: 1px;
}

```
7. 关于水平、垂直、多列布局的多种实现参照：《[利用HTML和CSS实现常见的布局](http://segmentfault.com/a/1190000003931851)》
