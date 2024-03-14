> 前端才是兴趣爱好，后端才是饭碗。这篇笔记是为了加强自己的复习，为了日后查漏补缺做出准备。刚好秋招也找好工作了，可以放心玩前端了。

# HTMl简介

这里简单的东西我就不记录了，直接从标签走起了。

## HTML标签

### 基本标签

- 标题标签 <h1>~<h6>
- 段落和换行标签 <p>…</p>、<br/>
- 水平线标签 <hr/>
- 斜体 <em>…</em>
- 字体加粗 <strong>…</strong>

### 图片标签

<img src="D:\typora\学习笔记\图片路径" alt="替换文本" width="宽度" height="高度" title="鼠标悬停"/>

`src`为图片的路径，`alt`为如果图片丢失显示的文字。

### 超链接

<a href**="链接地址" target="目标窗口位置" name= "名称" >文本或图像</a>**

**target="目标窗口位置**"  _self:自身窗口，_blank 新建窗口

### 特殊符号

| 特殊符号  | 字符实体 | 示例                                       |
| --------- | -------- | ------------------------------------------ |
| 空格      |          | <a href="#">百度</a>                       |
| 大于号(>) | >        | 如果时间>晚上6点，就坐车回家               |
| 小于号(<) | <        | 如果时间<早上7点，就走路去上学             |
| 引号(")   | "        | W3C规范中，HTML的属性值必须用成对的"引起来 |
| 版权符号@ | ©        | © 2003-2013北大青鸟                        |

### 媒体元素

#### 视频video

<video src="视频路径"   controls>  </video>

`controls`：提供播放，暂停和音量的控件                    `loop`：循环播放

`autoplay`**：自动播放 ——**<!-- **muted** 由于浏览器兼容性或者缓存原因，可能无法自动播放使用muted加上autoplay可实现自动播放-->

演示代码 自动循环播放视频

```HTML
<video muted autoplay  controls loop >
    <source src="video/bdqn.mp4" type="video/mp4"/>
    <source src="video/bdqn.webm" type="video/webm"/>
    <source src="video/bdqn.avi" type="video/webm"/>
</video>
```

#### 音频元素：audio

<audio src="音频路径" controls="controls"></ audio >

```HTML
<audio controls>

    <source src="music/music.mp3" type="audio/mpeg"/>

    <source src="music/music.ogg" type="audio/ogg"/>

    你的浏览器不支持audio元素

</audio>
```

# CSS：层叠样式表

css就是帮助我们把html写出来的简单网页进行美化。

## CSS引入方式

**内嵌式**：Css写在style标签中，通常写在head标签中。

```HTML
<style>
        h1{
            color: blue;
        }
</style>
</head>
<body>

    <h1>山竹</h1>
  
</body>
```

**外联式**：CSS写在一个单独的.css文件中，通过link标签进行引入

先写一个单独的CSS文件

```CSS
h1{
    color:yellow;
}
```

然后通过link标签进行引入

```HTML
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="day.css" />
    <title>Document</title>
</head>
<body>
    <h1>山竹</h1>
</body>
```

**行内式**：CSS直接写在HTML标签中

```HTML
<body>
    <h1 style="color:red">山竹</h1>
</body>
```

## CSS选择器

选择器(选择符)就是根据不同需求把不同的标签选出来这就是选择器的作用。 简单来说，就是选择标签用的。

### 标签选择器

标签选择器（元素选择器）是指用 HTML 标签名称作为选择器，按标签名称分类，为页面中某一类标签指定统一的 CSS 样式。

```CSS
h1{
    color:yellow;
}
```

### 类选择器

如果想要差异化选择不同的标签，单独选一个或者某几个标签，可以使用类选择器.类可以很多标签同时使用。

```CSS
.name{
    font-size: 50px;
}
```

### id选择器

id 选择器可以为标有特定 id 的 HTML 元素指定特定的样式。id不能重复。

```CSS
#title{
    color: red;
}
```

### 通配符选择器

```CSS
*{
 color: red;
}
```

## CSS字体属性

#### 字体大小

CSS 使用 font-size 属性定义字体大小

```CSS
p{
  font-size: 30px; 

}
```

px（像素）大小是我们网页的最常用的单位

#### 字体粗细

CSS 使用 font-weight 属性设置文本字体的粗细

```CSS
p {
  font-weight: bold;     normal：默认值。 bold：加粗。   400等同于normal   700等同于bold
}
```

#### 字体样式

CSS 使用 font-style 属性设置文本的风格。

```CSS
p {
  font-style: normal;
}
```

属性选择：

| 属性值 | 作用     |
| ------ | -------- |
| normal | 默认值   |
| italic | 斜体样式 |

#### 字体选择

CSS使用font-family设置字体。

```CSS
p{
    font-family: 'Courier New', Courier, monospace; 
}
```

#### 字体属性连写

语法：font: style weight size family;

省略要求：只能省略前两个。

```CSS
#fonts{
    font: italic bold 100px 'Courier New', Courier, monospace ;

}
```

## CSS文本属性

#### 文本颜色

color 属性用于定义文本的颜色

```CSS
*{
 color: red;
}
```

#### 文本对齐

text-align 属性用于设置元素内文本内容的水平对齐方式

```CSS
div {
text-align: center;  待选属性：left right
}
```

text-align能让哪些元素水平居中：

- 文本
- span标签 a标签
- input标签  img 标签

如果需要让以上元素进行水平居中,该属性应加给其父元素。

#### 文本缩进

text-indent是用来定义文本缩进

```CSS
p{
    text-indent: 32px;
}
```

em 是一个相对单位，就是当前元素（font-size) 1 个文字的大小, 如果当前元素没有设置大小，则会按照父元素的 1 个文字大小。在缩进的时候，我们可以根据字体大小进行缩进，使用em就可以。

比如要缩进两个字体

```CSS
p{
    text-indent: 2em;
}
```

#### 文本修饰属性

该属性可以对文本添加下划线，上划线 ，删除线等。

```CSS
div {
  text-decoration：underline；
}


none：没有装饰线。 underline:下划线 。 overline:上划线 。  line-through:删除线。
```

重点记住如何添加下划线 如何删除下划线 其余了解即可.

#### 行间距

line-height 属性用于设置行间的距离（行高）。可以控制文字行与行之间的距离

```CSS
p {
  line-height: 26px;
}
```

行高的文本分为 上间距 文本高度 下间距 = 行间距

## 复合选择器

### 什么是复合选择器

在 CSS 中，可以根据选择器的类型把选择器分为**基础选择器**和**复合选择器**，复合选择器是建立在基础选择器之上，对基本选择器进行组合形成的。 复合选择器是由两个或多个基础选择器，通过不同的方式组合而成的，可以更准确、更高效的选择目标元素（标签） 常用的复合选择器包括：后代选择器、子选择器、并集选择器、伪类选择器等等

### 后代选择器

后代选择器又称为包含选择器，可以选择父元素里面子元素。其写法就是把外层标签写在前面，内层标签写在后面，中间用空格分隔。当标签发生嵌套时，内层标签就成为外层标签的后代。

```CSS
元素1 元素2{ 样式声明 }
```

上述语法表示选择元素 1 里面的所有元素 2 (后代元素)

- 元素1 和 元素2 中间用空格隔开
- 元素1 是父级，元素2 是子级，最终选择的是元素2
- 元素2 可以是儿子，也可以是孙子等，只要是元素1 的后代即可
- 元素1 和 元素2 可以是任意基础选择器

### 子选择器

子元素选择器（子选择器）只能选择作为某元素的最近一级子元素。

```CSS
元素1 > 元素2 {样式声明}
```

上述语法表示选择元素1里面的所有直接后代（子元素）元素2.

语法说明：

- 元素1 和 元素2 中间用 大于号 隔开
- 元素1 是父级，元素2 是子级，最终选择的是元素2
- 元素2 必须是亲儿子，其孙子、重孙之类都不归他管. 你也可以叫他 亲儿子选择器

### 并集选择器

并集选择器可以选择多组标签, 同时为他们定义相同的样式，通常用于集体声明。并集选择器是各选择器通过英文逗号（,）连接而成，任何形式的选择器都可以作为并集选择器的一部分。

```CSS
元素1，元素2{ 样式声明 }
```

**语法说明**：

- 元素1 和 元素2 中间用逗号隔开
- 逗号可以理解为和的意思
- 并集选择器通常用于集体声明

### 伪类选择器

伪类选择器用于向某些选择器添加特殊的效果，比如给链接添加特殊效果，或选择第1个，第n个元素

#### 结构伪类选择器

根据元素在HTML中的结构关系查找元素

| 选择器              | 说明                        |
| ------------------- | --------------------------- |
| E:first-child       | 选择父元素的第一个子元素    |
| E:last-child        | 选择父元素的最后一个子元素  |
| E:nth-child(n)      | 选择父元素下的第 n 个子元素 |
| E:nth-last-child(n) | 选择父元素中倒数第n个子元素 |

## emmet语法

### 简介

Emmet语法的前身是Zen coding,它使用缩写,来提高html/css的编写速度, Vscode内部已经集成该语法。

它能够快速生成HTML结构语法，快速生成CSS样式语法。

### 快速生成HTML结构

- 生成标签 直接输入标签名 按`tab`键即可 比如 `div `然后`tab `键， 就可以生成
- 如果想要生成多个相同标签 加上 * 就可以了 比如 `div`*3 就可以快速生成3个`div`
- 如果有父子级关系的标签，可以用 `> `比如` ul > li`就可以了
- 如果有兄弟关系的标签，用 `+` 就可以了 比如` div+p`
- 如果生成带有类名或者id名字的， 直接写 `.demo `或者 `#two tab` 键就可以了
- 如果生成的`div `类名是有顺序的， 可以用 自增符号 `$`
- 如果想要在生成的标签内部写内容可以用`{ }`表示

### 快速生成CSS样式语法

CSS 基本采取简写形式即可

- 比如 w200 按tab 可以 生成 width: 200px;
- 比如 lh26px 按tab 可以生成 line-height: 26px;

## CSS的背景

通过 CSS 背景属性，可以给页面元素添加背景样式。  背景属性可以设置背景颜色、背景图片、背景平铺、背景图片位置、背景图像固定等。

### 背景颜色

使用方式：`background-color:颜色值`。

其他说明：元素背景颜色默认值是 transparent（透明）

### 背景图片

使用方式：`background-image：none | url(url)`

### 背景平铺

使用方式： background-repeat ：repeat | no-repeat | repeat-x | repeat-y

| 参数值    | 作用                              |
| --------- | --------------------------------- |
| repeat    | 背景图像在纵向和横向上平铺(默认） |
| no-repeat | 背景图像不平铺                    |
| repeat-x  | 背景图像在横向上平铺              |
| repeat-y  | 背景图像在纵向平铺                |

### 背景图片的位置

使用方式：`background-position ：x y；`

参数代表的意思是：x 坐标和 y 坐标。 可以使用 方位名词 或者 精确单位

| 参数值   | 说明   |
| -------- | ------ |
| length   | 百分数 |
| position | top    |

其他说明

- 参数是方位词

  如果指定的两个值都是方位名词，则两个值前后顺序无关，比如left top 和top left 效果一致。

  如果只指定了一个方位名词，另一个值省略，则第二个值默认居中对齐

- 参数是精确单位

  如果参数值是精确坐标，那么第一个肯定是x坐标，第二个一定是y坐标

  如果只指定一个数值，那该数值一定是x坐标，另一个默认垂直居中

- 参数是混合单位

  如果指定的两个值是精确单位和方位名词混合使用，则第一个值是x坐标，第二个值是y坐标。

### 背景图片固定

`background-attachment` 属性设置背景图像是否固定或者随着页面的其余部分滚动。

`background-attachment` 后期可以制作 `视差滚动` 的效果。

```CSS
background-attachment : scroll | fixed
```

| 参数   | 作用                                                       |
| ------ | ---------------------------------------------------------- |
| scroll | 背景图像是随对象内容滚动的（可见区域取决于背景图像的高度） |
| fixed  | 背景图像固定                                               |

### 背景复合写法

为了简化背景属性的代码，我们可以将这些属性合并简写在同一个属性 `background` 中，从而节约代码量。 当使用简写属性时，没有特定的书写顺序，一般习惯约定顺序为： `background`: `背景颜色` `背景图片地址` `背景平铺` `背景图像滚动` `背景图片位置`

```CSS
background: transparent url(image.jpg) no-repeat fixed top;
```

实际开发中，我们更提倡这样。

### 背景色半透明

CSS3 为我们提供了背景颜色半透明的效果。

```CSS
background: rgba(0, 0, 0, 0.3);
```

- 最后一个参数是 `alpha` 透明度，取值范围在 `0~1` 之间
- 习惯把 0.3 的 0 省略掉，写为 `background: rgba(0, 0, 0, .3);`
- 注意：背景半透明是指盒子背景半透明，盒子里面的内容不受影响
- CSS3 新增属性，是 IE9+ 版本浏览器才支持的
- 但是现在实际开发，我们不太关注兼容性写法了，可以放心使用

## CSS三大特性

CSS 有三个非常重要的特性：`层叠性`、`继承性`、`优先级`。

### 层叠性

给同一个选择器设置相同的样式，此时一个样式就会**覆盖**（层叠）另一个冲突的样式，**层叠性主要解决样式冲突的问题**。

层叠性原则：

- 样式冲突，遵循的原则是 `就近原则`，哪个样式距离结构近，就执行哪个样式
- 样式不冲突，不会层叠

就近的标准是：后 > 前

### 继承性

现实中的继承：我们继承了父亲的姓。

CSS 中的继承：**子标签会继承父标签的某些样式**，如：文本颜色和字号，简单的理解就是：子承父业。

- 恰当地使用继承可以简化代码，降低 CSS 样式的复杂性
- 子元素可以继承父元素的样式（ `text-`、`font-`、`line-`、`color` ） 文本、字体、段落、颜色

```HTML
<!doctype html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS三大特性之继承性</title>
    <style>
        div {
            color: pink;
            font-size: 14px;
        }
    </style>
</head>

<body>
    <div>
        <p>山竹山竹</p>
    </div>
</body>

</html>
```

**行高的继承**

```CSS
body {
    font: 12px/1.5 'Microsoft YaHei';
}
```

- 行高可以跟单位也可以不跟单位
- 如果子元素没有设置行高，则会继承父元素的行高为 1.5
- 此时子元素的行高是：**当前元素**的**文字大小** * 1.5
- body 行高 1.5 这样写法最大的优势就是**里面的子元素可以根据自己文字的大小自动调整行高**

```HTML
<!doctype html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS三大特性之继承性——行高的继承</title>
    <style>
        body {
            color: pink;
            /* font: 12px/18px; */
            font: 12px/1.5;    /* 12 + 12 * 0.5 = 18 */
        }

        div {
            /* 子元素继承了父元素 body 的行高 1.5 */
            /* 这个 1.5 就是当前元素文字大小 font-size 的 1.5 倍 */
            /* 所以当前 div 的行高就是：14 + 14 * 0.5 = 21px */
            font-size: 14px;
        }

        p {
            /* 1.5 * 16 = 24 当前行高 */
            font-size: 16px;
        }

        /* li 没有手动指定文字大小，则会继承父亲（此处的父亲可以层层向上推）的文字大小  */
        /* 即：body 12px 所有 li 的文字大小为 12px */
        /* 当前 li 的行高就是 12 * 1.5 = 18  */
    </style>
</head>

<body>
    <div>山竹山竹</div>
    <p>JERRY</p>
    <ul>
        <li>没有指定文字大小</li>
    </ul>
</body>

</html>
```

### 优先级

- 选择器相同，则执行层叠性
- 选择器不同，则根据选择器权重执行

**选择器权重如下表所示：**

| 选择器               | 选择器权重 |
| -------------------- | ---------- |
| 继承 或 *            | 0,0,0,0    |
| 元素选择器           | 0,0,0,1    |
| 类选择器、伪类选择器 | 0,0,1,0    |
| ID 选择器            | 0,1,0,0    |
| 行内样式 style=""    | 1,0,0,0    |
| !important 重要的    | ∞ 无穷大   |

```HTML
<!doctype html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS三大特性之优先级</title>
    <style>
        div {
            color: pink;
        }

        .test {
            color: red;
        }

        #demo {
            color: green;
        }
    </style>
</head>

<body>
    <div class="test">你笑起来真好看</div> <!-- red -->
    <div class="test" id="demo">你笑起来真好看</div> <!-- green -->
    <div class="test" id="demo" style="color: blue;">你笑起来真好看</div> <!-- blue -->
<!-- 
    假如在 css 选择器 某一个属性值后面跟上 !important，那么这个属性一定会执行！
    例如：div {
             color: pink !important;  
          }
          ...
    <div class="test" id="demo" style="color: blue;">你笑起来真好看</div> -- pink --
-->

</body>

</html>
```

- 权重是由 4 组数字组成的，但是不会有进位！
- 可以理解为：`类选择器` 永远大于 `元素选择器`，`ID 选择器` 永远大于 `类选择器`，以此类推！
- 等级判断 `从左到右`，如果某一位数值相同，则判断下一位数值
- 可以简单的记忆：`通配符` 和 `继承` 权重为 0，`标签选择器` 为 1，`类`（`伪类`）选择器为 10，`ID` 选择器为 100，`行内样式表` 为 1000，`!important` 无穷大
- 继承的权重是 0，不管父元素权重多高，子元素得到的权重都是 0 ！
- `a` 链接浏览器默认指定了一个样式，所以它不参与继承，所以设置样式需要选中单独设置

```HTML
<!doctype html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS三大特性之优先级——注意问题</title>
    <style>
        /* 父亲的权重是 100 */
        #father {
            color: red !important;
        }

        /* p 继承的权重为 0 */
        /* 所以以后我们看标签到底执行哪一个样式，就先看这个标签有没有直接被选出来
           如果直接被选择出来了，那么就与父亲无关了！*/
        p {
            color: pink;
        }
    </style>
</head>

<body>
    <!-- 继承的权重是 0，不管父元素权重多高，子元素得到的权重都是 0 -->
    <div id="father">
        <p>你还是很好看</p> <!-- pink -->
    </div>
    <!-- a 链接浏览器默认指定了一个样式，所以它不参与继承，所以给 a 改样式必须直接把 a 选出来 -->
    <a href="#">我是单独的样式</a>
</body>

</html>
```

**权重的叠加：**

如果是复合选择器，则会有权重叠加，需要计算权重。

注意：再次强调！权重虽然会叠加，但一定不会进位！（1万个元素选择器也比不过一个类选择器）。

从左到右逐位比较，只有左一位同样大，才比较右边一位！

**例如：**

- `div ul li` ——> `0,0,0,3`
- `.nav ul li` ——> `0,0,1,2`
- `a:hover` ——> `0,0,1,1`
- `.nav a` ——> `0,0,1,1`

如果要对某一元素设置样式，那么就必须给它足够高的权重（注意：是给他，而不是他的父亲！）。

> 提高选择器权重的技巧之一：

- 多写几层类选择器

```HTML
<!doctype html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS三大特性之优先级——权重叠加</title>
    <style>
        /* 复合选择器会有权重叠加的问题 */
        /* 权重虽然会叠加，但是永远不会有进位 例如：十个 0,0,0,1 相加为 0,0,0,10 而不是 0,0,1,0 */
        /* ul li 权重 0,0,0,1 + 0,0,0,1 = 0,0,0,2 */
        ul li {
            color: green;
        }

        /* li 的权重是 0,0,0,1 */
        li {
            color: red;
        }

        /* .nav li 权重 0,0,1,0 + 0,0,0,1 = 0,0,1,1 */
        .nav li {
            color: pink;
        }
    </style>
</head>

<body>
    <ul class="nav">
        <li>大猪蹄子</li>  <!-- pink -->
        <li>大肘子</li>  <!-- pink -->
        <li>猪尾巴</li>  <!-- pink -->
    </ul>
</body>

</html>
```

## CSS盒子模型

页面布局要学习**三大核心**：**盒子模型**、**浮动**和**定位**。学习好盒子模型能非常好的帮助我们布局页面。

### 盒子模型组成

所谓盒子模型：就是把 HTML 页面中的布局元素看作是一个矩形的盒子，也就是一个盛**装内容的容器**。 CSS 盒子模型本质上是一个盒子，封装周围的 HTML 元素，它包括：`边框`、`外边距`、`内边距`、和 `内容`。

### 边框

border可以设置元素的边框

边框有三部分组成：`边框宽度（粗细）`、`边框样式`、`边框颜色`。

语法：

```CSS
border: border-width || border-style || border-color
```

| 属性         | 做用                  |
| ------------ | --------------------- |
| border-width | 定义边框粗细 单位是px |
| border-style | 边框的样式            |
| border-color | 边框颜色              |

边框样式 border-style 可以设置如下值：

- `none`：没有边框，即忽略所有边框的宽度（默认值）
- `solid`：边框为单实线（最为常用的）
- `dashed`：边框为虚线
- `dotted`：边框为点线

**边框简写：**

```CSS
border: 1px solid red;   /* 没有顺序 */
```

### 表格的细线边框

表格中两个单元格相邻的边框会重叠在一起，呈现出加粗的效果。

`border-collapse` 属性控制浏览器绘制表格边框的方式。

它控制相邻单元格的边框。

```CSS
border-collapse: collapse;
```

- `collapse` 单词是合并的意思
- `border-collapse: collapse;` 表示相邻边框合并在一起

### 边框会影响盒子实际大小

边框会额外增加盒子的实际区域大小。因此我们有两种方案解决：

- 测量盒子大小的时候，不量边框
- 如果测量的时候包含了边框，则需要 width、height 减去边框宽度（注意减单边还是双边）

注意：盒子实际区域大小 = 内容区大小 + 内边距大小 + 边框大小 + 外边距大小

### 内边距

padding 属性用于设置**内边距**，即**边框与内容之间的距离**。

| 属性             | 作用     |
| ---------------- | -------- |
| `padding-left`   | 左内边距 |
| `padding-rigth`  | 右内边距 |
| `padding-top`    | 上内边距 |
| `padding-bottom` | 下内边距 |

padding 属性（简写属性）可以有一到四个值。

| 值的个数                       | 表达意思                                                     |
| ------------------------------ | ------------------------------------------------------------ |
| `padding: 5px;`                | 1 个值，代表上下左右都有 5 像素内边距                        |
| `padding: 5px 10px;`           | 2 个值，代表上下内边距是 5 像素，左右内边距是 10 像素        |
| `padding: 5px 10px 20px;`      | 3 个值，代码上内边距 5 像素，左右内边距 10 像素，下内边距 20 像素 |
| `padding: 5px 10px 20px 30px;` | 4 个值，上是 5 像素，右 10 像素，下 20 像素，左是 30 像素（顺时针） |

以上 4 种情况，我们实际开发都会遇到。

当我们给盒子指定 `padding` 值之后，发生了 2 件事情：

- 内容和边框有了距离，添加了内边距
- `padding` 影响了盒子实际区域大小

也就是说，如果盒子已经有了宽度和高度，此时再指定内边距，会撑大盒子区域！

解决方案：

- 如果保证盒子跟效果图大小保持一致，则让 width、height 减去多出来的内边距大小即可
- 如果盒子本身没有指定 width、height 属性，则此时 padding 不会撑开盒子区域大小

### 外边距

`margin` 属性用于设置**外边距**，即控制**盒子和盒子之间的距离**。

| 属性            | 作用     |
| --------------- | -------- |
| `margin-left`   | 左外边距 |
| `margin-right`  | 右外边距 |
| `margin-top`    | 上外边距 |
| `margin-bottom` | 下外边距 |

`margin` 简写方式代表的意义跟 `padding` 完全一致。

外边距典型应用：

外边距可以让**块级盒子水平居中**，但是必须满足两个条件：

- 盒子必须指定了宽度 `width`
- 盒子左右的外边距都设置为 `auto`

```CSS
.header { width: 960px; margin: 0 auto;}
```

常见的写法，以下三种都可以：

- `margin-left: auto; margin-right: auto;`
- `margin: auto;`
- `margin: 0 auto;`

注意：以上方法是让块级元素水平居中，行内元素或者行内块元素水平居中给其父元素添加 `text-align: center` 即可。

### 外边距合并

使用 `margin` 定义块元素的垂直外边距时，可能会出现外边距的合并。

注意：**内边距没有合并一说！浮动的盒子不会发生外边距合并！**

主要有两种情况:

- **相邻块元素垂直外边距的合并**
- **嵌套块元素垂直外边距的塌陷**

#### 相邻块元素垂直外边距的合并

当上下相邻的两个块元素（兄弟关系）相遇时，如果上面的元素有下外边距 `margin-bottom`，下面的元素有上外边距 `margin-top` ，则他们之间的垂直间距不是 `margin-bottom` 与 `margin-top` 之和。而是取两个值中的**较大者**，这种现象被称为相邻块元素垂直外边距的合并（准确的描述应该是：**大的外边距覆盖小的**）。

#### 嵌套块元素垂直外边距的塌陷

对于两个嵌套关系（父子关系）的块元素，当子元素有上外边距，此时父元素会塌陷较大的外边距值（**外边距效果显示在父元素之上**）。

### 清除内外边距

网页元素很多都带有默认的内外边距，而且不同浏览器默认的也不一致。因此我们在布局前，首先要清除下网页元素的内外边距。

```CSS
* {
  padding:0;   /* 清除内边距 */
  margin:0;   /* 清除外边距 */
}
```

注意：行内元素为了照顾兼容性，尽量只设置左右内外边距，不要设置上下内外边距（某些时候，行内元素对上下内外边距不生效）。但是转换为块级和行内块元素就可以了

## 圆角边框、盒子阴影、文字阴影

### 圆角边框

CSS 3 新增了圆角边框样式。

border-radius 属性用于设置元素的外边框圆角。

```CSS
border-radius: length;
```

![img](https://secure2.wostatic.cn/static/jtvAFnhUx8QdhUXfFt64Qh/image.png)

- 参数值可以为数值或百分比的形式
- 如果是正方形，想要设置为圆形，那么只需要把数值修改为高度或者宽度的一半即可，或者直接写为 50%
- 如果是个矩形，设置为高度的一半就可以做 “胶囊” 效果了
- 该属性是一个简写属性，可以跟多个值
  - 四个值：左上角、右上角、右下角、左下角（从左上开始顺时针）
  - 三个值：左上、右上+左下、右下（对角为一组）
  - 两个值：左上+右下、右上+左下（对角为一组）
- 同时可以对特定角单独设置
  - 左上角：`border-top-left-radius`
  - 右上角：`border-top-right-radius`
  - 右下角：`border-bottom-right-radius`
  - 左下角：`border-bottom-left-radius`

### 盒子阴影

box-shadow 属性用于为盒子添加阴影。

```CSS
box-shadow: h-shadow v-shadow blur spread color inset;
```

| 值         | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| `h-shadow` | 必须。水平阴影的位置，允许负值。                             |
| `v-shadow` | 必须。垂直阴影的位置，允许负值。                             |
| `blur`     | 可选。模糊距离（虚实程度）。                                 |
| `spread`   | 可选。阴影的尺寸（大小）。                                   |
| `color`    | 可选。阴影的颜色，请参阅 CSS 颜色值（阴影多为半透明颜色）。  |
| `inset`    | 可选。将外部阴影（outset）改为内部阴影（outset 不能指定，默认为空即可） |

```CSS
<!doctype html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>盒子阴影</title>
    <style>
        div {
            width: 200px;
            height: 200px;
            background-color: salmon;
            margin: 100px auto;
            /* box-shadow: 10px 10px; */
        }

        /* 伪类不仅仅可以用于 a 链接，还能用于其他标签 */
        /* 原先盒子没有影子,当我们鼠标经过盒子就添加阴影效果 */
        div:hover {
            box-shadow: 10px 10px 10px -4px rgba(0, 0, 0, .3);
        }
    </style>
</head>

<body>
    <div></div>
</body>

</html>
```

![img](https://secure2.wostatic.cn/static/hFUtFZcnQ9zNstW2n9WvyW/20210407181220613.gif)

**三边阴影、双边阴影、单边阴影的设置方法：**

```HTML
<!doctype html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>盒子阴影 三边阴影、双边阴影、单边阴影</title>
    <style>
        div {
            width: 100px;
            height: 100px;
            background-color: #000;
            margin: 25px auto;
            color: white;
        }

        .a {
            box-shadow: 0 0 25px 5px red;
        }

        /* 三边阴影就是直接把整个阴影部分往下边移 */
        .b {
            box-shadow: 0 6px 10px 0 red;
        }

        /* 两边阴影要用盒子嵌套来解决 */
        .c1 {
            box-shadow: 0 10px 10px -5px red;
        }

        .c2 {
            width: 100%;
            box-shadow: 0 -10px 10px -5px red;
        }

        /* 单边阴影就是直接把整个阴影部分往下边移，并且减小阴影大小 */
        .d {
            box-shadow: 0 10px 10px -5px red;
        }
    </style>
</head>

<body>
    <div class="a">四边阴影</div>
    <div class="b">三边阴影</div>
    <div class="c1">
        <div class="c2">两边阴影</div>
    </div>
    <div class="d">一边阴影</div>
</body>

</html>
```

### 文件阴影

text-shadow 属性用于为文本添加阴影。

```CSS
text-shadow: h-shadow v-shadow blur color;
```

| 值         | 描述                                |
| ---------- | ----------------------------------- |
| `h-shadow` | 必须。水平阴影的位置。允许负值。    |
| `v-shadow` | 必须。垂直阴影的位置。允许负值。    |
| `blur`     | 可选。模糊的距离（虚实程度）。      |
| `color`    | 可选。阴影的颜色。参阅 CSS 颜色值。 |

```HTML
<!doctype html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>文字阴影</title>
    <style>
        div {
            font-size: 50px;
            color: salmon;
            font-weight: 700;
            text-shadow: 5px 5px 6px rgba(0, 0, 0, .3);
        }
    </style>
</head>

<body>
    <div>
        你是阴影,我是火影
    </div>
</body>

</html>
```

![img](https://secure2.wostatic.cn/static/au6VddBjRGvGB4ikKsVSgs/image.png)

## 浮动

### 传统网页布局的三种方式

网页布局的本质：用 CSS 来摆放盒子，把盒子摆放到相应位置。

CSS 提供了三种传统布局方式（简单说就是盒子如何进行排列）。

- 普通流（标准流）
- 浮动
- 定位

> 这里指的只是传统布局，其实还有一些特殊高级的布局方式。

### 标准流（普通流/文档流）

所谓的标准流：就是标签按照规定好的默认方式排列。

1. **块级元素会独占一行，从上向下顺序排列。**
2. **行内元素会按照顺序，从左到右顺序排列，碰到父元素边缘则自动换行。**

以上都是标准流布局，我们前面学习的就是标准流，标准流是最基本的布局方式。

这三种布局方式都是用来摆放盒子的，盒子摆放到合适位置，布局自然就完成了。

### 为什么需要浮动

**如何让多个块级盒子（div）水平排列成一行？**

这是标准流很难完成的事情，因此需要浮动流来进行润色。

**浮动最典型的应用：可以让多个块级元素一行内排列显示。**

**网页布局第一准则：多个块级元素纵向排列找标准流，多个块级元素横向排列找浮动！**

```HTML
<!doctype html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>多个块级元素横向排列找浮动</title>
    <style>
        div {
            float: left;
            width: 150px;
            height: 200px;
            background-color: #d87093;
        }
    </style>
</head>

<body>
    <div>1</div>
    <div>2</div>
    <div>3</div>
</body>

</html>
```

浮动的盒子不会发生外边距合并

### 什么是浮动

`float` 属性用于创建浮动框，将其移动到一边，直到左边缘或右边缘触及包含块或另一个浮动框的边缘。

```CSS
选择器 { float: 属性值;}
```

| 属性  | 描述                 |
| ----- | -------------------- |
| none  | 元素不浮动（默认值） |
| left  | 元素向左浮动         |
| right | 元素向右浮动         |

```HTML
<!doctype html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>什么是浮动</title>
    <style>
        .left,
        .right {
            float: left;
            width: 200px;
            height: 200px;
            background-color: pink;
        }
    </style>
</head>

<body>
    <div class="left">左青龙</div>
    <div class="right">右白虎</div>
</body>

</html>
```

### 浮动的特性

加了浮动之后的元素，会具有很多特性，需要我们掌握。

- 浮动元素会脱离标准流（脱标）

  脱离标准普通流的控制（浮） 移动到指定位置（动），（俗称脱标）

  浮动的盒子不再保留原先的位置

  ![img](https://secure2.wostatic.cn/static/htnVzvoCTvJwdJVNTX2DXK/image.png)

```HTML
<!doctype html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>浮动特性1</title>
    <style>
        /* 设置了浮动（float）的元素会：
        1.脱离标准普通流的控制（浮）移动到指定位置（动）。
        2.浮动的盒子不再保留原先的位置 */
        .box1 {
            float: left;
            width: 200px;
            height: 200px;
            background-color: pink;
        }

        .box2 {
            width: 300px;
            height: 300px;
            background-color: gray;
        }
    </style>
</head>

<body>
    <div class="box1">浮动的盒子</div>
    <div class="box2">标准流的盒子</div>
</body>

</html>
```

- 浮动的元素会一行内显示并且元素顶部对齐

  如果多个盒子都设置了浮动，则它们会按照属性值一行内显示并且顶端对齐排列

  浮动的元素是互相贴靠在一起的（不会有缝隙），如果父级宽度装不下这些浮动的盒子，多出的盒子会另起一行对齐

```HTML
<!doctype html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>浮动元素特性-浮动元素一行显示</title>
    <style>
        div {
            float: left;
            width: 200px;
            height: 200px;
            background-color: pink;
        }

        .two {
            background-color: skyblue;
            height: 249px;
        }

        .four {
            background-color: skyblue;
        }
    </style>
</head>

<body>
    <div>1</div>
    <div class="two">2</div>
    <div>3</div>
    <div class="four">4</div>
</body>

</html>
![](https://secure2.wostatic.cn/static/x2yV7vtFna88G32a95R5pF/image.png)
```

- 浮动的元素会具有行内块元素的特性

  任何元素都可以浮动。不管原先是什么模式的元素，添加浮动之后具有行内块元素相似的特性。

  块级盒子：没有设置宽度时默认宽度和父级一样宽，但是添加浮动后，它的大小根据内容来决定

  行内盒子：宽度默认和内容一样宽，直接设置高宽无效，但是添加浮动后，它的大小可以直接设置

  浮动的盒子中间是没有缝隙的，是紧挨着一起的

  **即：默认宽度由内容决定，同时支持指定高宽，盒子之间无空隙**

```HTML
<!doctype html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>浮动的元素具有行内块元素特点</title>
    <style>
        /* 任何元素都可以浮动。不管原先是什么模式的元素，添加浮动之后具有行内块元素相似的特性。 */
        span,
        div {
            float: left;
            width: 200px;
            height: 100px;
            background-color: pink;
        }

        /* 如果行内元素有了浮动，则不需要转换块级\行内块元素就可以直接给高度和宽度 */
        p {
            float: right;
            height: 200px;
            background-color: skyblue;
        }
    </style>
</head>

<body>
    <span>span1</span>
    <span>span2</span>

    <div>div</div>
    <p>pppppppppppppp</p>
</body>

</html>
```

### 浮动元素经常和标准流父级搭配使用

为了约束浮动元素位置，我们网页布局一般采取的策略是：

**先用标准流的父元素排列上下位置，之后内部子元素采取浮动排列左右位置。符合网页布局第一准侧。**

![img](https://secure2.wostatic.cn/static/5xDBUXRAUiV8CVBfNXYkfg/image.png)

举个例子：

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        .box{
            width: 1500px;
            height: 800px;
        
            margin: 0 auto;
        }
        .left{
            width: 20%;
            height: 800px;
            background-color: blue;
            float: left;
        }
        .right{
            width: 80%;
            height: 800px;
        
            float: left;
        }
        .right > div{  
  
            margin-left: 2%;
            width: 18%;
            height: 389px;
            background-color: yellow;
            float: left;
            margin-bottom: 2% ;
            border-radius: 15px;
        }
     

        
    </style>
</head>
<body>
    <div class="box">
        <div class="left"></div>
        <div class="right">
            <div></div>
            <div></div>
            <div></div>
            <div></div>
            <div></div>
            <div></div>
            <div></div>
            <div></div>
            <div></div>
            <div></div>
        </div>
    </div>
</body>
</html>
```

![img](https://secure2.wostatic.cn/static/xvWd2NJLEhaGU49duFWs5X/image.png)

### 浮动布局注意

- **浮动和标准流的父盒子搭配**

先用标准流的父元素排列上下位置，之后内部子元素采取浮动排列左右位置。

- **一个元素浮动了，理论上其余的兄弟元素也要浮动**

一个盒子里面有多个子盒子，如果其中一个盒子浮动了，那么其他兄弟也应该浮动，以防止引起问题

浮动的盒子只会影响浮动盒子后面的标准流，不会影响前面的标准流。

## 清除浮动

我们前面浮动元素有一个标准流的父元素，他们有一个共同的特点，都是有高度的。但是，所有的父盒子都必须有高度吗？

不一定！比如，一个产品列表，随着时期的不同，产品数量也不同，所需的盒子大小也会随之改变，那么直接固定盒子高度的形式显然就是不行的。再比如，文章之类的盒子，不同的文章字数是不相同的，那么显然盒子也不能直接固定高度。

**理想中的状态，让子盒子撑开父亲。有多少孩子，我父盒子就有多高。但是不给父盒子高度又会出现问题，但有方法解决。这个时候就要请我们清除浮动出手了。**

### 为什么需要清除浮动

由于父级盒子很多情况下不方便给高度，但是子盒子浮动又不占有位置，最后父级盒子高度为 0 时，就会影响下面的标准流盒子。

![img](https://secure2.wostatic.cn/static/3J8hfw7kt97vBmFYD2p1Kh/image.png)

- 由于浮动元素不再占用原文档流的位置，所以它会对后面的元素排版产生影响
- 此时一但父盒子下面有其他盒子，那么布局就会发生严重混乱！

### 清除浮动本质

- 清除浮动的本质是清除浮动元素造成的影响
- 如果父盒子本身有高度，则不需要清除浮动
- 清除浮动之后，父级就会根据浮动的子盒子自动检测高度。父级有了高度，就不会影响下面的标准流了

### 清除浮动的几种方法(四种）

#### 额外标签法

额外标签法也称为隔墙法，是 W3C 推荐的做法。

额外标签法会在浮动元素末尾添加一个空的标签。例如 `<div style="clear: both"></div>`，或者其他标签（如 `<br>` 等）。

- 优点： 通俗易懂，书写方便
- 缺点： 添加许多无意义的标签，结构化较差

注意： 要求这个新的空标签必须是**块级元素**。

```HTML
<!doctype html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>清除浮动之额外标签法</title>
    <style>
        .box {
            width: 800px;
            border: 3px solid black;
            margin: 0 auto;
        }

        .damao {
            float: left;
            width: 300px;
            height: 200px;
            background-color: salmon;
        }

        .ermao {
            float: left;
            width: 200px;
            height: 200px;
            background-color: skyblue;
        }

        .footer {
            height: 200px;
            background-color: gray;
        }

        .clear {
            clear: both;
        }
    </style>
</head>

<body>
    <div class="box">
        <div class="damao">大毛</div>
        <div class="ermao">二毛</div>
        <div class="ermao">二毛</div>
        <div class="ermao">二毛</div>
        <div class="ermao">二毛</div>
        <div class="clear"></div>
        <!-- 这个新增的盒子要求必须是块级元素不能是行内元素 -->
        <!-- <span class="clear"></span> -->
    </div>
    <div class="footer"></div>

</body>

</html>
```

#### 父级添加overflow

可以给父级添加 `overflow` 属性，将其属性值设置为 `hidden`、 `auto` 或 `scroll`。

子不教，父之过，注意是给父元素添加代码。

- 优点：代码简洁
- 缺点：无法显示溢出的部分

```HTML
<!doctype html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>overflow清除浮动</title>
    <style>
        .box {
            /* 清除浮动 */
            overflow: hidden;
            width: 800px;
            border: 1px solid blue;
            margin: 0 auto;
        }

        .damao {
            float: left;
            width: 300px;
            height: 200px;
            background-color: purple;
        }

        .ermao {
            float: left;
            width: 200px;
            height: 200px;
            background-color: pink;
        }

        .footer {
            height: 200px;
            background-color: black;
        }
    </style>
</head>

<body>
    <div class="box">
        <div class="damao">大毛</div>
        <div class="ermao">二毛</div>
    </div>
    <div class="footer"></div>

</body>

</html>
```

#### :afterfa伪元素法

`:after` 方式是额外标签法的升级版，也是给父元素添加代码。

原理：自动在父盒子里的末尾添加一个 行内盒子，我们将它转换为 块级盒子，就间接实现了额外标签法。

```CSS
.clearfix:after {
  content: "";
  display: block;
  height: 0;
  clear: both;
  visibility: hidden;
}

.clearfix { 
    /* IE6、7 专有 */
  *zoom: 1;
}
```

注意：类名不一定非要是 clearfix，但是还是推荐这么写以提高可读性。

- 优点：没有增加标签，结构更简单
- 缺点：需要单独照顾低版本浏览器
- 代表网站： 百度、淘宝网、网易等

```CSS
<!doctype html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>伪元素清除浮动</title>
    <style>
        .clearfix:after {
            content: "";
            display: block;
            height: 0;
            clear: both;
            visibility: hidden;
        }

        .clearfix {
            /* IE6、7 专有 */
            *zoom: 1;
        }

        .box {
            width: 800px;
            border: 1px solid blue;
            margin: 0 auto;
        }

        .damao {
            float: left;
            width: 300px;
            height: 200px;
            background-color: purple;
        }

        .ermao {
            float: left;
            width: 200px;
            height: 200px;
            background-color: pink;
        }

        .footer {
            height: 200px;
            background-color: black;
        }
    </style>
</head>

<body>
    <div class="box clearfix">
        <div class="damao">山竹一号</div>
        <div class="ermao">山竹二号</div>
    </div>
    <div class="footer"></div>

</body>

</html>
```

#### 双伪元素清楚浮动

额外标签法的升级版，也是给父元素添加代码。

原理：自动在父盒子里的两端添加两个行内盒子，并把它们转换为 表格，间接实现了额外标签法

```CSS
.clearfix:before,
.clearfix:after {
  content: "";
  display: table;
}

.clearfix:after {
  clear: both;
}

.clearfix {
    /* IE6、7 专有 */
  *zoom:1;
}
```

注意：类名不一定非要是 clearfix，但是还是推荐这么写以提高可读性。

- 优点：代码更简洁
- 缺点：需要单独照顾低版本浏览器
- 代表网站：小米、腾讯等

```HTML
<!doctype html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>双伪元素清除浮动</title>
    <style>
        .clearfix:before,
        .clearfix:after {
            content: "";
            display: table;
        }

        .clearfix:after {
            clear: both;
        }

        .clearfix {
            *zoom: 1;
        }

        .box {
            width: 800px;
            border: 1px solid blue;
            margin: 0 auto;
        }

        .damao {
            float: left;
            width: 300px;
            height: 200px;
            background-color: purple;
        }

        .ermao {
            float: left;
            width: 200px;
            height: 200px;
            background-color: pink;
        }

        .footer {
            height: 200px;
            background-color: black;
        }
    </style>
</head>

<body>
    <div class="box clearfix">
        <div class="damao">大毛</div>
        <div class="ermao">二毛</div>
    </div>
    <div class="footer"></div>

</body>

</html>
```

### 清楚浮动总结

| 清除浮动的方式         | 优点               | 缺点                                 |
| ---------------------- | ------------------ | ------------------------------------ |
| 额外标签法（隔墙法）   | 通俗易懂，书写方便 | 添加许多无意义的标签，结构化较差     |
| 父级 overflow: hidden; | 书写简单           | 溢出隐藏                             |
| 父级 after 伪元素      | 结构语义化正确     | 由于 IE6~7 不支持 :after，兼容性问题 |
| 父级双伪元素           | 结构语义化正确     | 由于 IE6~7 不支持 :after，兼容性问题 |