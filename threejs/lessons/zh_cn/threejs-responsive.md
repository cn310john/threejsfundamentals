Title: Three.js响应式设计
Description: 如何让你的three.js适应不同尺寸的显示器。
TOC: 响应式设计

这是three.js系列文章的第二篇。
第一篇是[关于基础](threejs-fundamentals.html)。
如果你还没有阅读第一篇那你应该从第一篇开始。

本篇文章是关于如何让你的three.js应用自适应各种情况。
设计响应式网页一般是指让其在桌面、平板及手机等不同尺寸的屏幕上显示良好。

对three.js来说，要考虑的情况更多。例如，我们可能需要处理控件在左侧、右侧、顶部或底部的三维编辑器。本文的中间部分展示了另一个例子。

上一个例子中我们使用了一个没有CSS和尺寸的最简画布（canvas）。

```html
<canvas id="c"></canvas>
```

那个canvas默认300x150像素。

网页开发的通行做法是使用CSS来设置尺寸。现在就添加CSS，以使canvas填充整个页面。

```html
<style>
html, body {
   margin: 0;
   height: 100%;
}
#c {
   width: 100%;
   height: 100%;
   display: block;
}
</style>
```

HTML中的body默认有5个像素的边界（margin）。为移除边界，将之设置为0。
设置html和body的高度为100%以整个窗口，否则他们的大小只会和其中的填充内容一样。

然后我们让`id=c`的元素的尺寸是容器的100%。此例中的容器即为页面的body元素。  

最后将`display`设置为`block`。canvas的display默认为`inline`。
对于设为行内显示的元素，显示内容的末端会自动添加空格。而将canvas设为块级元素就能将此空格消除。

以下是结果。

{{{example url="../threejs-responsive-no-resize.html" }}}

你可以看到canvas充满了整个页面，但是有两个问题。
第一是我们的立方体被拉伸了。他们不是立方体了更像是个盒子，太高或者太宽。 在新标签中打开它然后改变尺寸你就能看到立方体是怎么在宽高上被拉伸的。

<img src="resources/images/resize-incorrect-aspect.png" width="407" class="threejs_center nobg">

另一个问题是立方体看起来分辨率太低或者说块状化或者有点模糊。
将窗口拉伸的非常大你就能看到问题。

<img src="resources/images/resize-low-res.png" class="threejs_center nobg">

我们先解决拉伸的问题。为此我们要将相机的宽高比设置为canvas的宽高比。
我们可以通过canvas的`clientWidth`和`clientHeight`属性来实现。

我们需要将渲染循环变成这样。

```js
function render(time) {
  time *= 0.001;

+  const canvas = renderer.domElement;
+  camera.aspect = canvas.clientWidth / canvas.clientHeight;
+  camera.updateProjectionMatrix();

  ...
```

现在立方体就不会变形了。

{{{example url="../threejs-responsive-update-camera.html" }}}

在新标签页中打开示例，可以看到立方体不再会拉伸得过宽或者过高了。
无论窗口尺寸如何，他们都会保持正确的比例。

<img src="resources/images/resize-correct-aspect.png" width="407" class="threejs_center nobg">

我们现在来解决块状化的问题。

canvas元素有两个尺寸。一个是canvas在页面上的显示尺寸，
是我们用CSS来设置的。另一个尺寸是canvas本身像素的数量。这和图片一样。
比如我们有一个128x64像素的图片，我们可以通过CSS让它显示为400x200像素。

```html
<img src="some128x64image.jpg" style="width:400px; height:200px">
```

一个canvas的内部尺寸，它的分辨率，通常被叫做绘图缓冲区(drawingbuffer)尺寸。
在three.js中我们可以通过调用`renderer.setSize`来设置canvas的绘图缓冲区。
我们应该选择什么尺寸? 最显而易见的是"和canvas的显示尺寸一样"，
即可以直接用canvas的`clientWidth`和`clientHeight`属性。

我们写一个函数来检查渲染器的canvas尺寸是不是和其显示尺寸不同。
如果不一样，就设置一下。

```js
function resizeRendererToDisplaySize(renderer) {
  const canvas = renderer.domElement;
  const width = canvas.clientWidth;
  const height = canvas.clientHeight;
  const needResize = canvas.width !== width || canvas.height !== height;
  if (needResize) {
    renderer.setSize(width, height, false);
  }
  return needResize;
}
```

注意我们检查了canvas是否真的需要调整大小。 
调整画布大小是canvas规范的一个有趣部分，如果它已经是我们想要的大小，最好不要按同样大小重复设置。
 
一旦我们知道了是否需要调整大小，我们就调用`renderer.setSize`，然后传入新的宽高。
在末尾传入`false`很重要。`render.setSize`默认会设置canvas的CSS尺寸，但我们并不需要。
我们希望浏览器能继续正常运行：three.js所使用的canvas和其他元素应该一样，保持使用CSS来确定元素尺寸。

注意，如果canvas大小变化，函数就会返回true。我们可以据此检查是否要更新其他东西。
我们可以修改一下渲染循环，把这个新函数用上。

```js
function render(time) {
  time *= 0.001;

+  if (resizeRendererToDisplaySize(renderer)) {
+    const canvas = renderer.domElement;
+    camera.aspect = canvas.clientWidth / canvas.clientHeight;
+    camera.updateProjectionMatrix();
+  }

  ...
```

因为只有canvas的显示尺寸变化时宽高比才变化，所以我们仅在`resizeRendererToDisplaySize`函数返回`true`时才设置摄像机的宽高比。

{{{example url="../threejs-responsive.html" }}}

现在渲染的分辨率就和canvas的显示尺寸一样了。

为了更清楚地体现调整尺寸是由CSS完成的，我们将代码放进[一个单独的js文件](../threejs-responsive.js)。
这里还有一些由CSS确定尺寸大小的示例。请注意，我们并没有改变任何代码，但示例仍运行良好。

我们将立方体放在文字段落的中间。

{{{example url="../threejs-responsive-paragraph.html" startPane="html" }}}

我们在编辑器样式布局中使用了相同代码，右侧的控制区域可以调整大小。

{{{example url="../threejs-responsive-editor.html" startPane="html" }}}

重点注意我们的代码并没有改变，只有HTML和CSS变了。

## 应对HD-DPI显示器

HD-DPI代表每英寸高密度点显示器(视网膜显示器)。它指的是当今大多数的Mac、相当一部分Windows主机以及几乎所有的智能手机。

浏览器中的工作方式是，无论屏幕的分辨率有多高，浏览器总是通过“CSS像素”把尺寸设置为相同大小。
只是对于字体，高分辨率显示的浏览器会渲染出更多的细节，但实际尺寸仍然相同。

使用three.js有多种方法来应对HD-DPI。

第一种就是不做任何特别的事情。这可以说是最常见的。
渲染三维图形需要大量的GPU处理能力。至少截止到2018年，手机在GPU能力比桌面的要弱。而手机屏幕又有着相当高的分辨率。
目前最高端手机的HD-DPI比例为3x，非分辨率显示器上的一个像素在高分辨率屏幕上是9个像素，这就意味着9倍的渲染工作。

计算9倍的像素是个大工程。所以如果保持代码不变，我们将计算一个像素然后浏览器将以三倍大小绘制(3x3=9像素)。

对于大型的three.js应用来说，这可能就是一个妥善的方案，否侧你的帧速率会很低。

尽管如此，如果你确实想用设备的分辨率来渲染，three.js中有两种方法来实现。

一种是使用renderer.setPixelRatio来告诉three.js分辨率的倍数：从浏览器获取CSS-设备像素倍数，然后传递给three.js。

     renderer.setPixelRatio(window.devicePixelRatio);

之后任何对`renderer.setSize`的调用都会神奇地使用您请求的大小乘以您传入的像素比例。
**强烈不建议这样操作**，原因后述。

另一种方法是在调整canvas的大小时自己处理。

```js
    function resizeRendererToDisplaySize(renderer) {
      const canvas = renderer.domElement;
      const pixelRatio = window.devicePixelRatio;
      const width = canvas.clientWidth * pixelRatio | 0;
      const height = canvas.clientHeight * pixelRatio | 0;
      const needResize = canvas.width !== width || canvas.height !== height;
      if (needResize) {
        renderer.setSize(width, height, false);
      }
      return needResize;
    }
```

第二章方法从客观上来说更好。为什么？因为这种方法更便于其他操作。
在使用three.js时，我们有时需要知道canvas绘图缓冲区的确切尺寸。
比如制作后期处理滤镜，比如操作着色器访问`gl_FragCoord`变量，又比如截屏，抑或是将像素读取到GPU绘制二维canvas，等等。
如果采用`setPixelRatio`，实际尺寸和变量中的尺寸就会不同，我们往往得猜测什么时候得用变量中的尺寸，什么时候又得采用实际尺寸。
通过我们自己处理，我们就始终知悉两种尺寸并无不同，幕后并没有发生什么特殊的魔法。

以下是一个使用上述代码的示例。

{{{example url="../threejs-responsive-hd-dpi.html" }}}

可能很难看出区别。但是如果你有一个HD-DPI显示器，和上面的例子做个对比，你就能发现边角更清晰。

这篇文章涵盖了一个非常基础但是很有必要的主题。接下来我们快速[过一遍three.js提供的基本图元](threejs-primitives.html)。

