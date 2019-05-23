title: 浅谈 js 拖拽
date: 2017-01-02
author: LiuLingyang
categories: 其他
---

## 前言

本文依据半年前本人的分享《浅谈js拖拽》撰写，算是一篇迟到的文章。

## 基本思路

虽然现在关于拖拽的组件库到处都是，HTML5也把拖放纳入了标准。但考虑到兼容问题，我们还是从最古老的方式开始讲起。

onmousedown：模拟开始拖拽事件。鼠标按键按下即发生onmousedown事件。获取鼠标位置，获取被拖拽元素的位置，记录两者之间的纵横坐标的差值。对document元素绑定onmousemove,onmouseup事件。 

为什么是对document绑定而不是对被拖动的元素绑定呢？原来是如果对被拖动元素绑定的话当鼠标拖动过快时，会导致鼠标与被拖动元素的脱离。 

onmousemove：模拟拖拽中事件。鼠标拖动即发生onmousemove事件。将被拖拽元素的position改成绝对位置，这个可以通过left和top改变该元素的位置，从而使得该元素随着鼠标的拖拽而移动。获取鼠标位置，将鼠标x坐标（e.clientX）减去第2步储存的横坐标差作为被拖动元素的left值，将鼠标y坐标（e.clientY）减去第2步储存的纵坐标差作为被拖动元素的top值。实现元素跟随鼠标拖动的效果。 

onmouseup：模拟拖拽结束事件。鼠标按键弹起即发生onmouseup事件。可以回收onmousemove和onmousedown中的事件和变量，一次拖拽至此结束。

（nej也提供了拖拽的一个简单案例，位于util/dragger/下，代码就不贴出来了，有兴趣的童鞋可以自行查阅，毕竟看文字实在是过于枯燥了。）

## HTML5拖放

有古老的方式自然有潮流的方式，如果你无需考虑兼容性问题的话，笔者强烈建议你使用HTML5提供的拖放API。如果你还未曾了解，提供你一个简单的HTML5拖放实例[http://www.w3school.com.cn/tiy/t.asp?f=html5_draganddrop](http://www.w3school.com.cn/tiy/t.asp?f=html5_draganddrop)。

让我们一起来看看HTML5拖放相关的一些知识点。

**7个事件：**

dragstart：当用户开始拖动对象时触发。

dragenter： 当鼠标第一次经过目标元素，且有拖动发生时触发。此事件的监听者应指明在这个位置上是否允许drop，或者监听者不执行任何操作，那么drop默认是不允许的。

dragover：当鼠标经过一个元素时，且有拖动发生时触发 。

dragleave：当鼠标离开一个元素，且有拖动在发生时触发。

drag： 当对象被拖动，每次移动鼠标时触发。

drop：在drag操作的最后发生drop时，在元素上触发此事件。监听者应该负责检索拖动的数据，并插入drop的位置。

dragend： 在拖动对象时放开鼠标按键时触发。

**draggable属性：**

如果网页元素的draggable元素为true，这个元素就是可以拖动的。

<pre><div draggable="true">Draggable Div</div></pre>

**dataTransfer对象：**

拖动过程中，回调函数接受的事件参数，有一个dataTransfer属性。它指向一个对象，包含了与拖动相关的各种信息。

dataTransfer对象的属性:

*   dropEffect：拖放的操作类型，决定了浏览器如何显示鼠标形状，可能的值为copy、move、link和none。
*   effectAllowed：指定所允许的操作，可能的值为copy、move、link、copyLink、copyMove、linkMove、all、none和uninitialized（默认值，等同于all，即允许一切操作）。
*   files：包含一个FileList对象，表示拖放所涉及的文件，主要用于处理从文件系统拖入浏览器的文件。
*   types：储存在DataTransfer对象的数据的类型。
*   dataTransfer对象的方法：
*   setData(format, data)：在dataTransfer对象上储存数据。第一个参数format用来指定储存的数据类型，比如text、url、text/html等。
*   getData(format)：从dataTransfer对象取出数据。
*   clearData(format)：清除dataTransfer对象所储存的数据。如果指定了format参数，则只清除该格式的数据，否则清除所有数据。
*   setDragImage(imgElement, x, y)：指定拖动过程中显示的图像。默认情况下，许多浏览器显示一个被拖动元素的半透明版本。参数imgElement必须是一个图像元素，而不是指向图像的路径，参数x和y表示图像相对于鼠标的位置。

dataTransfer对象，允许在其上存储数据，这使得在被拖动元素与目标元素之间传送信息成为可能。

**e.preventDefault():**

ondragover有一个默认行为，那就是当ondragover触发时，ondrop会失效！

如何阻止？

<pre data-type="javascript">ondragover = function(e){</pre>

<pre data-type="javascript">    e.preventDefault();</pre>

<pre data-type="javascript">    ....</pre>

<pre data-type="javascript">}</pre>

附一张整理好的图片供大家理解：

![image](http://upload-images.jianshu.io/upload_images/13276697-e3f1caa17193f21c?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 案例

扯了这么多，最后再举个栗子来结束本文。先上图片...

![image](http://upload-images.jianshu.io/upload_images/13276697-f9a4a35e17da8815?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这是笔者在网易有数项目中的一个需求，实现文件拖拽上传。

实现步骤：

  1.监听事件

 ![image](http://upload-images.jianshu.io/upload_images/13276697-34c74f5bf800a8f1?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  其中dragenter和dragleave只是处理了外框高亮的效果，如上面图片所示。

  需要注意的是一定要将dragover的默认事件取消掉，不然无法触发drop事件。可以用preventDefault()，也可以用nej的_v._$stop(_event)。如需拖拽页面里的元素，需要给其添加属性draggable=”true”。这些上文已经有所提及。

  2.处理drop事件

  在drop事件回调函数中通过_event.dataTransfer.files获取拖拽文件列表。dataTransfer对象真是个好东西。

  3.发送文件数据

  使用FormData模拟表单数据AJAX提交文件流。

 ![image](http://upload-images.jianshu.io/upload_images/13276697-f1a888eccb80074b?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  至此，HTML5的文件拖拽上传就实现了，是不是很easy？
