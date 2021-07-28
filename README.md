# html5 drag api

参考：

[html5的拖拽dragAPI（如果看了API不懂，看看那三个案例就会恍然大悟）](https://www.cnblogs.com/yangguoe/p/9681692.html)

[html5 drag api详解](https://www.cnblogs.com/wuya16/p/DragApi.html)

[drag - Web API 接口参考 | MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/drag_event)

![api图](https://img-blog.csdnimg.cn/b526dd80c648438cbdb666580783d310.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTIyMTAzNg==,size_16,color_FFFFFF,t_70)

### 1】拖拽和释放定义

- 拖拽：Drag 
- 释放：Drop

### 2】源对象和目标对象

　　![img](https://img2018.cnblogs.com/blog/1284345/201809/1284345-20180920162225387-258905329.png)

### 3】相关API

- `dragstart`：事件主体是被拖放元素，在开始拖放被拖放元素时触发。
- `darg`：事件主体是被拖放元素，在正在拖放被拖放元素时触发。
- `dragenter`：事件主体是目标元素，在被拖放元素进入某元素时触发。
- `dragover`：事件主体是目标元素，在被拖放在某元素内移动时触发。
- `dragleave`：事件主体是目标元素，在被拖放元素移出目标元素是触发。
- `drop`：事件主体是目标元素，在目标元素完全接受被拖放元素时触发。
- `dragend`：事件主体是被拖放元素，在整个拖放操作结束时触发。

### 4】让元素支持drag api

 要让一个元素支持拖拽，首先我们需要在标签上标示出来：

```html
<div draggable="true"></div>
```

对于Safari，还必须要在CSS中对能拖拽的元素如下设置：

```css
*[draggable = true] {
    -khtml-user-drag: element;
}    
```

有了HTML5 drag API，判断拖拽元素跟其他dom元素相交变得容易起来了，只需要：

```js
$('.drop-area').on('dragover',function(e){
    console.log('dragover');
    // 阻止默认动作以启用drop
    e.preventDefault();
});
$('.drop-area').on('drop',function(e){
    console.log('drop');
});
```

需要注意的是，必须写上红色部分，**如果不阻止默认事件，drop事件将不会触发**。关于这点，可以去[MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/drag_event)查看。

### 5】[dataTransfer对象](https://developer.mozilla.org/zh-CN/docs/Web/API/DataTransfer)保存数据

dataTransfer对象有`getData()`和`setData()`两个主要方法，操作dataTransfer中携带的数据。

```js
//手动设置数据，format是指格式
event.dataTransfer.setData(format, data);
//获取数据
event.dataTransfer.getData(format);
//清楚数据
event.dataTransfer.clearData([format])
//以下是format支持传入的值：
//text/html：HTML代码格式
//text/plain：纯文本文字格式
//text/xml：XML字符格式
//text/url-list：URL格式列表

//用于设置自定义的拖动图像
event.dataTransfer.setDragImage(img, xOffset, yOffset)

//给被拖放元素设置效果
event.dataTransfer.effectAllowed = "uninitialized";
//uninitialized：没有该被拖动元素放置行为。
//none：被拖动的元素不能有任何行为。
//copy：只允许值为“copy”的dropEffect。
//link：只允许值为“link”的dropEffect。
//move：只允许值为“move”的dropEffect。
//copyLink：允许值为“copy”和“link”的dropEffect。
//copyMove：允许值为“copy”和”link”的dropEffect。
//linkMove：允许职位“link”和”move”的dropEffect。
//all：允许任意dropEffect。

//给目标元素设置效果
event.dataTransfer.dropEffect = "move";
//none：不能把拖动的元素放在这里。这是除文本框之外所有元素的默认值。
//move：应该把拖动的元素移动到放置目标
//copy：应该把拖动的元素复制到放置目标
//link：表示放置目标会打开拖动的元素（但拖动的元素必须是一个链接，有URL）。
```

保存在dataTransfer对象中的**数据只能在drop事件处理程序中读取**。如果在`ondrop`处理程序中没有读到数据，那就是dataTransfer对象已经被销毁，数据也丢失了。

在拖动文本框中的文本时，**浏览器会自动调用`setData()`方法**，将拖动的文本以“text”格式保存在dataTransfer对象中。类似地，在拖放链接或图像时，会调用`setData()`方法并保存URL。然后，在这些元素被拖放到放置目标时，就可以通过getData()读到这些数据。当然，作为开发人员，你也可以在`ondragstart`事件处理程序中调用`setData()`，手工保存自己要传输的数据，以便将来使用。

将数据保存为**文本和URL是有区别的**。如果将数据保存为文本格式，那么数据不会得到任何特殊处理。而如果将数据保存为URL，浏览器会将其当成网页中的链接。换句话说，如果你把它放置到另一个浏览器窗口中，浏览器就会打开该URL。

### 6】[setDragImage 方法](https://developer.mozilla.org/zh-CN/docs/Web/API/DataTransfer/setDragImage)设置拖放图标

发生拖动时，从拖动目标(`dragstart (en-US)`事件触发的元素)生成**半透明图像，并在拖动过程中跟随鼠标指针，这个图片是自动创建的，你不需要自己去创建它**。然而，如果想要设置为自定义图像，那么 `DataTransfer.setDragImage()` 方法就能派上用场。

在HTML5中，一个元素在被拖放时，还可以自定义拖放元素的鼠标图标。调用格式如下：

```js
dataTransfer.setDragImage(img, xOffset, yOffset)
```

图像通常是一个`<image>`元素，但也可以是`<canvas>`元素，或任何其他图像元素。该方法的x和y坐标是**图像应该相对于鼠标指针出现的偏移量**。

### 7】effectAllowed 和 dropEffect 设置拖放效果

dataTransfer对象的两个属性：`dropEffect`和`effectAllowed`，能通过它来确定被拖动的元素以及作为放置目标的元素能够接受什么操作。结`effectAllowed`和`dropEffect`这两个属性，可以自定义拖放过程中的效果。两个属性虽然都是为了实现同一功能，但绑定的元素不同：

- `effectAllowed`属性作用于被拖放元素；
- `dropEffect`属性作用于目标元素。

#### dropEffect属性

其中，通过dropEffect属性可以知道被拖动的元素能够执行哪种放置行为。这个属性有下列4个可能的值。

- none：不能把拖动的元素放在这里。这是除文本框之外所有元素的默认值。
- move：应该把拖动的元素移动到放置目标
- copy：应该把拖动的元素复制到放置目标
- link：表示放置目标会打开拖动的元素（但拖动的元素必须是一个链接，有URL）。

在把元素拖动到放置目标上时，以上每一个值都会导致光标显示为不同的符号。然而，要怎样实现光标所指示的动作完全取决于你。换句话说，如果你不介入，没有什么会自动地移动、复制，也不会打开链接。总之，浏览器只能帮你改变光标的样式，而其它的都要靠你自己来实现。要使用dropEfect属性，必须在ondraggenter事件处理程序中针对放置目标来设置它。

#### effectAllowed属性

dropEffect属性只有搭配effectAllowed属性才有用。effectAllowed属性表示允许拖放元素的哪种dropEffect，effectAllowed属性可能的值如下。

1. uninitialized：没有该被拖动元素放置行为。
2. none：被拖动的元素不能有任何行为。
3. copy：只允许值为“copy”的dropEffect。
4. link：只允许值为“link”的dropEffect。
5. move：只允许值为“move”的dropEffect。
6. copyLink：允许值为“copy”和“link”的dropEffect。
7. copyMove：允许值为“copy”和”link”的dropEffect。
8. linkMove：允许职位“link”和”move”的dropEffect。
9. all：允许任意dropEffect。

必须在ondraggstart事件处理程序中设置effectAllowed属性。

假设你想允许用户把文本框中的文本拖放到一个`<div>`元素中。首先，必须将dropEffect和effectAllowed设置为”move”。但是，由于`<div>`元素的放置事件的默认行为是什么也不做，所以文本不可能自动移动。重写这个默认行为，就能从文本框中移走文本。然后你就可以自己编写代码将文本插入到`<div>`中，这样整个拖放操作就完成了。如果将dropEffect和effectAllowed的值设置为”copy”，那就不会自动移走文本框中的文本。