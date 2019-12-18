---
typora-copy-images-to: assets
---

## 一. CSS

### 1. `margin-top`塌陷

在两个盒子嵌套时候，内部的盒子设置的`margin-top`会加到外边的盒子上，导致内部的盒子`margin-top`设置失败，解决方法如下：

1. 外部盒子设置一个边框

2. 外部盒子设置 `overflow:hidden`

3. 使用伪元素类：

   ```css
   .clearfix:before{
       content: '';
       display:table;
   }
   ```

### 2. 元素溢出

当子元素的尺寸超过父元素的尺寸时，需要设置父元素显示溢出的子元素的方式，设置的方法是通过overflow属性来设置。overflow的设置项： 

1. visible 默认值。内容不会被修剪，会呈现在元素框之外。
2. hidden 内容会被修剪，并且其余内容是不可见的，此属性还有清除浮动、清除margin-top塌陷的功能。
3. scroll 内容会被修剪，但是浏览器会显示滚动条以便查看其余的内容。
4. auto 如果内容被修剪，则浏览器会显示滚动条以便查看其余的内容。
5. inherit 规定应该从父元素继承 overflow 属性的值。

### 3.  块元素、内联元素、内联块元素

元素就是标签，布局中常用的有三种标签，块元素、内联元素、内联块元素，了解这三种元素的特性，才能熟练的进行页面布局。

#### 3.1 块元素

块元素，也可以称为行元素，布局中常用的标签如：div、p、ul、li、h1~h6、dl、dt、dd等等都是块元素，它在布局中的行为：

1. 支持全部的样式
2. 如果没有设置宽度，默认的宽度为父级宽度100%
3. 盒子占据一行、即使设置了宽度

#### 3.2 内联元素

内联元素，也可以称为行内元素，布局中常用的标签如：a、span、em、b、strong、i等等都是内联元素，它们在布局中的行为：

1. 支持部分样式（不支持宽、高、margin上下、padding上下）
2. 宽高由内容决定
3. 盒子并在一行
4. 代码换行，盒子之间会产生间距
5. 子元素是内联元素，父元素可以用text-align属性设置子元素水平对齐方式

**解决内联元素间隙的方法**
1、去掉内联元素之间的换行
2、将内联元素的父级设置font-size为0，内联元素自身再设置font-size

#### 3.3 内联块元素

内联块元素，也叫行内块元素，是新增的元素类型，现有元素没有归于此类别的，img和input元素的行为类似这种元素，但是也归类于内联元素，我们可以用display属性将块元素或者内联元素转化成这种元素。它们在布局中表现的行为：

1. 支持全部样式
2. 如果没有设置宽高，宽高由内容决定
3. 盒子并在一行
4. 代码换行，盒子会产生间距
5. 子元素是内联块元素，父元素可以用text-align属性设置子元素水平对齐方式。

这三种元素，可以通过display属性来相互转化，不过实际开发中，块元素用得比较多，所以我们经常把内联元素转化为块元素，少量转化为内联块，而要使用内联元素时，直接使用内联元素，而不用块元素转化了。

### 4. 浮动

浮动特性

1. 浮动元素有左浮动(float:left)和右浮动(float:right)两种
2. 浮动的元素会向左或向右浮动，碰到父元素边界、其他元素才停下来
3. 相邻浮动的块元素可以并在一行，超出父级宽度就换行
4. 浮动让行内元素或块元素自动转化为行内块元素(此时不会有行内块元素间隙问题)
5. 浮动元素后面没有浮动的元素会占据浮动元素的位置，没有浮动的元素内的文字会避开浮动的元素，形成文字饶图的效果
6. 父元素如果没有设置尺寸(一般是高度不设置)，父元素内整体浮动的元素无法撑开父元素，父元素需要清除浮动
7. 浮动元素之间没有垂直margin的合并

#### 4.1 清除浮动

- 父级上增加属性overflow：hidden

- 在最后一个子元素的后面加一个空的div，给它样式属性 clear:both（不推荐）

- 使用成熟的清浮动样式类，clearfix

  ```css
  .clearfix:after,.clearfix:before{ content: "";display: table;}
  .clearfix:after{ clear:both;}
  .clearfix{zoom:1;}
  ```

  清除浮动的使用方法：

  ```css
  .con2{... overflow:hidden}
  或者
  <div class="con2 clearfix">
  ```

### 5. 定位

文档流，是指盒子按照html标签编写的顺序依次从上到下，从左到右排列，块元素占一行，行内元素在一行之内从左到右排列，先写的先排列，后写的排在后面，每个盒子都占据自己的位置。

#### 5.1 关于定位 

我们可以使用css的position属性来设置元素的定位类型，postion的设置项如下：

1. relative 生成相对定位元素，元素所占据的文档流的位置保留，元素本身相对自身原位置进行偏移。
2. absolute 生成绝对定位元素，元素脱离文档流，不占据文档流的位置，可以理解为漂浮在文档流的上方，相对于上一个设置了定位的父级元素来进行定位，如果找不到，则相对于body元素进行定位。
3. fixed 生成固定定位元素，元素脱离文档流，不占据文档流的位置，可以理解为漂浮在文档流的上方，相对于浏览器窗口进行定位。
4. static 默认值，没有定位，元素出现在正常的文档流中，相当于取消定位属性或者不设置定位属性。
5. inherit 从父元素继承 position 属性的值。

#### 5.2 定位元素的偏移

定位的元素还需要用left、right、top或者bottom来设置相对于参照元素的偏移值。

#### 5.3 定位元素层级

定位元素是浮动的正常的文档流之上的，可以用z-index属性来设置元素的层级

伪代码如下:

```
.box01{
    ......
    position:absolute;  /* 设置了绝对定位 */
    left:200px;         /* 相对于参照元素左边向右偏移200px */
    top:100px;          /* 相对于参照元素顶部向下偏移100px */
    z-index:10          /* 将元素层级设置为10 */
}
```

#### 5.4 定位元素特性 

绝对定位和固定定位的块元素和行内元素会自动转化为行内块元素

### 6. 权重

可以把样式的应用方式分为几个等级，按照等级来计算权重：

1. !important，加在样式属性值后，权重值为 10000
2. 内联样式，如：style=””，权重值为1000
3. ID选择器，如：#content，权重值为100
4. 类，伪类和属性选择器，如： content、:hover 权重值为10
5. 标签选择器和伪元素选择器，如：div、p、:before 权重值为1
6. 通用选择器（*）、子选择器（>）、相邻选择器（+）、同胞选择器（~）为0

### 7.  新增选择器

1. `E:nth-child(n)`：匹配元素类型为E且是父元素的第n个子元素

   ```css
   <style type="text/css">            
       .list div:nth-child(2){
           background-color:red;
       }
   </style>
   ```

2. `E:first-child`：匹配元素类型为E且是父元素的第一个子元素
3. `E:last-child`：匹配元素类型为E且是父元素的最后一个子元素
4. `E > F`：E元素下面第一层子集
5. `E ~ F`：E元素后面的兄弟元素
6. `E + F`：紧挨着的后面的兄弟元素

#### 7.1 属性选择器

1. `E[attr]`：含有attr属性的元素

   ```css
   <style type="text/css">
       div[data-attr='ok']{
           color:red;
       }
   </style>
   ......
   <div data-attr="ok">这是一个div元素</div>
   ```

2. `E[attr='ok']`：含有attr属性的元素且它的值为“ok”
3. `E[attr^='ok']`： 含有attr属性的元素且它的值的开头含有“ok”
4. `E[attr$='ok']`： 含有attr属性的元素且它的值的结尾含有“ok”
5. `E[attr*='ok']`： 含有attr属性的元素且它的值中含有“ok”

### 8. transition动画

1. transition-property 设置过渡的属性，比如：width height background-color
2. transition-duration 设置过渡的时间，比如：1s 500ms
3. transition-timing-function 设置过渡的运动方式，常用有 linear(匀速)|ease(缓冲运动)
4. transition-delay 设置动画的延迟
5. transition: property duration timing-function delay 同时设置四个属性

### 9. transform变换

1. translate(x,y) 设置盒子位移
2. scale(x,y) 设置盒子缩放
3. rotate(deg) 设置盒子旋转
4. skew(x-angle,y-angle) 设置盒子斜切
5. perspective 设置透视距离
6. transform-style flat | preserve-3d 设置盒子是否按3d空间显示
7. translateX、translateY、translateZ 设置三维移动
8. rotateX、rotateY、rotateZ 设置三维旋转
9. scaleX、scaleY、scaleZ 设置三维缩放
10. tranform-origin 设置变形的中心点
11. backface-visibility 设置盒子背面是否可见

### 10. animation动画

1. @keyframes 定义关键帧动画
2. animation-name 动画名称
3. animation-duration 动画时间
4. animation-timing-function 动画曲线 linear(匀速)|ease(缓冲)|steps(步数)
5. animation-delay 动画延迟
6. animation-iteration-count 动画播放次数 n|infinite
7. animation-direction 动画结束后是否反向还原 normal|alternate
8. animation-play-state 动画状态 paused(停止)|running(运动)
9. animation-fill-mode 动画前后的状态 none(缺省)|forwards(结束时停留在最后一帧)|backwards(开始时停留在定义的开始帧)|both(前后都应用)
10. animation:name duration timing-function delay iteration-count direction;同时设置多个属性

### 11. 浏览器样式前缀

为了让CSS3样式兼容，需要将某些样式加上浏览器前缀：

1. -ms- 兼容IE浏览器
2. -moz- 兼容firefox
3. -o- 兼容opera
4. -webkit- 兼容chrome 和 safari

比如：

```
div
{    
    -ms-transform: rotate(30deg);        
    -webkit-transform: rotate(30deg);    
    -o-transform: rotate(30deg);        
    -moz-transform: rotate(30deg);    
    transform: rotate(30deg);
}
```

#### 11.1 自动添加浏览器前缀

目前的状况是，有些CSS3属性需要加前缀，有些不需要加，有些只需要加一部分，这些加前缀的工作可以交给插件来完成，比如安装： autoprefixer

可以在Sublime text中通过package control 安装 autoprefixer

Autoprefixer在Sublime text中的设置：

1. preferences/key Bindings-User

```
{ "keys": ["ctrl+alt+x"], "command": "autoprefixer" }
```

2. Preferences>package setting>AutoPrefixer>Setting-User

```
{
    "browsers": ["last 7 versions"],
    "cascade": true,
    "remove": true
}
```

last 7 versions：最新的浏览器的7个版本
cascade：缩进美化属性值
remove：是否去掉不必要的前缀

### 12. HTML5新增标签

#### 12.1 新增语义标签

1. `<header>` 页面头部、页眉
2. `<nav>` 页面导航
3. `<article>` 一篇文章
4. `<section>` 文章中的章节
5. `<aside>` 侧边栏
6. `<footer>` 页面底部、页脚

#### 12.2 音频视频

1. `<audio>``
2. ``<video>`

PC端兼容h5的新标签的方法，在页面中引入以下js文件:

```
<script type="text/javascript" src="//cdn.bootcss.com/html5shiv/r29/html5.js"></script>
```

#### 12.3 HTML5 新增表单控件

新增类型：网址 邮箱 日期 时间 星期 数量 范围 电话 颜色 搜索

```html
<label>网址:</label><input type="url" name="" required><br><br> 
<label>邮箱:</label><input type="email" name="" required><br><br> 
<label>日期:</label><input type="date" name=""><br><br> 
<label>时间:</label><input type="time" name=""><br><br> 
<label>星期:</label><input type="week" name=""><br><br> 
<label>数量:</label><input type="number" name=""> <br><br>
<label>范围:</label><input type="range" name=""><br><br> 
<label>电话:</label><input type="tel" name=""><br><br> 
<label>颜色:</label><input type="color" name=""><br><br> 
<label>搜索:</label><input type="search" name=""><br><br>
```

新增常用表单控件属性：

1. placeholder 设置文本框默认提示文字
2. autofocus 自动获得焦点
3. autocomplete 联想关键词

### 13. 透明

1、盒子透明度表示法：

```
    .box
    {
        opacity:0.1;
        /* 兼容IE */
        filter:alpha(opacity=10); 
    }
```

2、rgba(0,0,0,0.1) 前三个数值表示颜色，第四个数值表示颜色的透明度

### 14. 移动端与PC端页面布局区别

#### 14.1 视口 viewport

视口是移动设备上用来显示网页的区域，一般会比移动设备可视区域大，宽度可能是980px或者1024px，目的是为了显示下整个为PC端设计的网页，这样带来的后果是移动端会出现横向滚动条，为了避免这种情况，移动端会将视口缩放到移动端窗口的大小。这样会让网页不容易观看，可以用 meta 标签，name=“viewport ” 来设置视口的大小，将视口的大小设置为和移动设备可视区一样的大小。

```html
<head>  <!--  快捷方式：meta:vp + tab  -->
......
<meta name="viewport" content="width=device-width, user-scalable=no,
 initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
......
</head>
```

#### 14.2 Retina屏幕解决

视网膜屏幕指的是屏幕的物理像素密度更高的屏幕，物理像素可以理解为屏幕上的一个发光点，无数发光的点组成的屏幕，视网膜屏幕比一般屏幕的物理像素点更小，常见有2倍的视网膜屏幕和3倍的视网膜屏幕，2倍的视网膜屏幕，它的物理像素点大小是一般屏幕的1/4,3倍的视网膜屏幕，它的物理像素点大小是一般屏幕的1/9。

图像在视网膜屏幕上显示的大小和在一般屏幕上显示的大小一样，但是由于视网膜屏幕的物理像素点比一般的屏幕小，图像在上面好像是被放大了，图像会变得模糊，为了解决这个问题，可以使用比原来大一倍的图像，然后用css样式强制把图像的尺寸设为原来图像尺寸的大小，就可以解决模糊的问题。

### 15. 屏幕适配

设备屏幕有多种不同的分辨率，页面适配方案有如下几种：

1. 全适配：响应式布局+流体布局
2. 移动端适配：
   1. 流体布局+少量响应式
   2. 基于rem的布局

#### 15.1 流体布局

流体布局，就是使用百分比来设置元素的宽度，元素的高度按实际高度写固定值，流体布局中，元素的边线无法用百分比，可以使用样式中的计算函数 calc() 来设置宽度，或者使用 box-sizing 属性将盒子设置为从边线计算盒子尺寸。

`calc()`： 可以通过计算的方式给元素加尺寸，比如： width：calc(25% - 4px);

`box-sizing`：content-box 是默认的盒子尺寸计算方式；`border-box`设置盒子的尺寸计算方式为从边框开始，边框和内填充算在盒子尺寸内。

#### 15.2 响应式布局

响应式布局就是使用媒体查询的方式，通过查询浏览器宽度，不同的宽度应用不同的样式块，每个样式块对应的是该宽度下的布局方式，从而实现响应式布局。响应式布局的页面可以适配多种终端屏幕（pc、平板、手机）。

响应式布局的伪代码如下：

```css
@media (max-width:960px){
    .left_con{width:58%;}
    .right_con{width:38%;}
}
@media (max-width:768px){
    .left_con{width:100%;}
    .right_con{width:100%;}
}
```

#### 15.3 基于rem的布局

首先了解em单位，em单位是参照元素自身的文字大小来设置尺寸。

rem指的是参照根节点的文字大小，根节点指的是html标签，设置html标签的文字大小，其他的元素相关尺寸设置用rem，这样，所有元素都有了统一的参照标准，改变html文字的大小，就会改变所有元素用rem设置的尺寸大小。

##### 15.3.1 cssrem安装

cssrem插件可以动态地将px尺寸换算成rem尺寸

下载本项目，比如：git clone https://github.com/flashlizi/cssrem ；进入packages目录：`Sublime Text -> Preferences -> Browse Packages...` 复制下载的cssrem目录到刚才的packges目录里。 重启Sublime Text。

配置参数参数配置文件：`Sublime Text -> Preferences -> Package Settings -> cssrem px_to_rem - px`转rem的单位比例，默认为40。 `max_rem_fraction_length - px`转rem的小数部分的最大长度。默认为6。 `available_file_types` 启用此插件的文件类型。默认为：[".css", ".less", ".sass"]。

### 98. 常用CSS列表

1. `color`：设置文字的颜色，如： `color:red;`
2. `font-size`：设置文字的大小，如：`font-size:12px;`
3. `font-family`：设置文字的字体，如：`font-family:'微软雅黑';`
4. `font-style`：设置字体是否倾斜，如：`font-style:'normal';` 设置不倾斜，`font-style:'italic';`设置文字倾斜
5. `font-weight`：设置文字是否加粗，如：`font-weight:bold;` 设置加粗 `font-weight:normal` 设置不加粗
6. `line-height`：设置文字的行高，设置行高相当于在每行文字的上下同时加间距， 如：`line-height:24px;`
7. `font`：同时设置文字的几个属性，写的顺序有兼容问题，建议按照如下顺序写： `font：是否加粗 字号/行高 字体;`，如： `font:normal 12px/36px '微软雅黑';`
8. `text-decoration`：设置文字的下划线，如：`text-decoration:none;` 将文字下划线去掉
9. `text-indent`： 设置文字首行缩进，如：`text-indent:24px;` 设置文字首行缩进24px
10. `text-align`：设置文字水平对齐方式，如`text-align:center`设置文字水平居中
11. `text-overflow`：设置一行文字宽度超过容器宽度时的显示方式，如：`text-overflow:clip` 将多出的文字裁剪掉，`text-overflow:ellipsis`将多出的文字显示成省略号
12. `white-space`：一般用来设置文本不换行，如：`white-space:nowrap`设置文本不换行，一般与text-overflow和overflow属性配合使用来让一行文字超出宽度时显示省略号
13. `list-style`：一般用来设置去掉ul或者ol列表中的小圆点或数字 如：`list-style:none`
14. `width`：设置盒子内容的宽度，如：`width：100px;`
15. `height`：设置盒子内容的高度，如：`height：100px;`
16. `border-top`：设置盒子顶部边框的三个属性 如：`border-top:5px solid red;`设置盒子顶部边框为3像素宽的红色的实线
17. `border`：同时设置盒子的四个边框，如果四个边的样式统一就使用它。如：`border:1px solid #000`，设置盒子四个边都是1像素宽的黑色实线
18. `padding`：设置盒子四个边的内边距，如：`padding:10px 20px 30px 40px`分别设置盒子上、右、下、左的内边距(顺时针)，
19. `margin`：设置盒子四个边的外边距，如：`margin:10px 20px 30px 40px`分别设置盒子上、右、下、左的外边距(顺时针)
20. `overflow`：设置当子元素的尺寸超过父元素的尺寸时，盒子及子元素的显示方式。如：`overflow:hidden`，超出的子元素被裁切
21. `display`：设置盒子的显示类型及隐藏，如：`display:block`将盒子设置为以块元素显示，`display:none`将元素隐藏
22. `float`：设置元浮动，如：`float:left`设置左浮动，`float:right`设置右浮动
23. `clear`：在盒子两侧清除浮动，如：`clear:both`在盒子两侧都不允许浮动
24. `position`：设置元素定位，如：`position:relative`设置元素相对定位
25. `background`：设置元素的背景色和背景图片，如：`background:url(bg.jpg) cyan`;设置盒子的背景图片为`bg.jpg`，背景色为`cyan`
26. `background-size`：设置盒子背景图的尺寸，如：`background-size:30px 40px;`设置背景图的尺寸宽为30px，高为40px，这个属性不能合到background属性中
27. `opacity`：设置元素整体透明度，一般为了兼容需要加上filter属性设置。如：·、`opacity:0.1;filter:alpha(opacity=10)`
28. `cursor`：设置鼠标悬停在元素上时指针的形状。如：`cursor:pointer`，设置为手型
29. `outline`：设置文本输入框周围凸显的蓝色的线，一般是设为没有。如：`outline:none`
30. `border-radius`：设置盒子的圆角 如：`border-radius:10px`设置盒子的四个角为10px半径的圆角
31. `box-shadow`：设置盒子的阴影，如：`box-shadow:10px 10px 5px 2px pink;`设置盒子有粉色的阴影
32. `transition`：设置盒子的过渡动画，如：`transition:all 1s ease;`设置元素过渡动画为1秒完成，所有变动的属性都做动画
33. `animation`：设置盒子的关键帧动画
34. `transform`：设置盒子的位移、旋转、缩放、斜切等变形。如：`transform:rotate(45deg);`设置盒子旋转45度
35. `box-sizing`：设置盒子的尺寸计算方式，如：`box-sizing:border-box` 将盒子的尺寸计算方法设置为按边框计算，此时width和height的值就是盒子的实际尺寸
36. `border-collapse`：设置表格边框是否合并，如：`border-collapse:collapse`将表格边框合并，这样就可以制作1px边框的表格。

### 99. 练习

见assets中practice.html

1、特征布局：翻页（所需知识点：盒模型、内联元素）

![layout05](.\assets\layout05.jpg)

2、特征布局：导航条01（所需知识点：盒模型、行内元素布局）

![layout02](.\assets\layout02.jpg)

3、特征布局：导航条02（所需知识点：盒模型、浮动、定位、字体对齐）

![layout01](.\assets\layout01.jpg)

4、特征布局：图片列表（所需知识点：盒模型、浮动）

![layout03](.\assets\layout03.jpg)

5、特征布局：新闻列表（所需知识点：盒模型、浮动）

![layout04](.\assets\layout04.jpg)

**课后练习**

![layout06](.\assets\layout06-1562829929614.jpg)

![layout07](.\assets\layout07-1562829939832.jpg)