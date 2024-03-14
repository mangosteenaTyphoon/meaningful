> 继续接着Css基础进行记录。

## CSS定位

### 为什么需要定位？

在我们需要将某些特定元素自由的在一个盒子内移动位置，并且压住其他盒子。

在我们需要制作导航栏时，并希望导航栏可以固定在顶部。

以上情况标准流和浮动都无法帮我们实现，所以我们需要定位来帮我们实现。

### 定位组成

将盒子定在某一个位置，所以定位也是在摆放盒子， 按照定位的方式移动盒子。

```
定位 = 定位模式 + 边偏移
```

- 定位模式用于指定一个元素在文档中的定位方式
- 边偏移则决定了该元素的最终位置

#### 定位模式

定位模式决定元素的定位方式，它通过 CSS 的 `position` 属性来设置，其值可以分为四个。

| 值         | 语义     |
| ---------- | -------- |
| `static`   | 静态定位 |
| `relative` | 相对定位 |
| `absolute` | 绝对定位 |
| `fixed`    | 固定定位 |

#### 边偏移

边偏移就是定位的盒子移动的最终位置。有 `top`、`bottom`、`left` 和 `right` 4 个属性。

| 边偏移属性 | 实例           | 描述                                           |
| ---------- | -------------- | ---------------------------------------------- |
| `top`      | `top: 80px`    | 顶端偏移量，定义元素相对于其父元素上边线的距离 |
| `bottom`   | `bottom: 80px` | 底部偏移量，定义元素相对于其父元素下边线的距离 |
| `left`     | `left: 80px`   | 左侧偏移量，定义元素相对于其父元素左边线的距离 |
| `rigth`    | `right: 80px`  | 右侧偏移量，定义元素相对于其父元素右边线的距离 |

#### 静态定位

静态定位是元素的默认定位方式，无定位的意思。

```CSS
选择器 { position: static; }
```

1. 静态定位按照标准流特性摆放位置，它没有边偏移
2. 静态定位在布局时很少用到

#### 相对定位

相对定位是元素在移动位置的时候**相对于它原来的位置**来说的定位（自恋型）。

```CSS
选择器 { position: relative; }
```

1. 它是相对于自己原来的位置来移动的（移动位置的时候参照点是自己原来的位置点）
2. 原来在**标准流的位置继续占有**，后面的盒子仍然以标准流的方式对待它

因此，**相对定位并没有脱标**。它最典型的应用是给绝对定位当爹的。

```HTML
<!doctype html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>相对定位</title>
    <style>
        .box1 {
            position: relative;
            top: 100px;
            left: 100px;
            width: 200px;
            height: 200px;
            background-color: pink;
        }

        .box2 {
            width: 200px;
            height: 200px;
            background-color: deeppink;
        }
    </style>
</head>

<body>
    <div class="box1">

    </div>
    <div class="box2">

    </div>

</body>

</html>
```

#### 绝对定位absolute

绝对定位是元素在移动位置的时候**相对于它祖先元素**来说的定位（拼爹型）

```CSS
选择器 { position: absolute; }
```

1. 如果没有祖先元素或者祖先元素没有定位，则以浏览器为准定位（Document 文档）
2. 如果祖先元素有定位（相对、绝对、固定定位），则以**最近一级的有定位祖先元素为参考点**移动位置
3. 绝对定位**不再占有原先的位置**（脱标），并且**脱标的程度大于浮动**（会压住浮动）

所以绝对定位是**脱离标准流**的。

```HTML
<!doctype html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>绝对定位-无父亲或者父亲无定位</title>
    <style>
        .father {
            width: 500px;
            height: 500px;
            background-color: skyblue;
        }

        .son {
            position: absolute;
            /* top: 10px;
            left: 10px; */
            /* top: 100px;
            right: 200px; */
            left: 0;
            bottom: 0;
            width: 200px;
            height: 200px;
            background-color: pink;
        }
    </style>
</head>

<body>
    <div class="father">
        <div class="son"></div>
    </div>
</body>

</html>
```

### 子绝父相的由来

弄清楚这个口诀，就明白了绝对定位和相对定位的使用场景。

这个 “子绝父相” 太重要了，是我们学习定位的口诀，是定位中最常用的一种方式这句话的意思是：子级是绝对定位的话，父级要用相对定位。

### 固定定位

固定定位是元素固定于浏览器可视区的位置。

主要使用场景： 可以在浏览器页面滚动时元素的位置不会改变。

```CSS
选择器 { position: fixed; }
```

1. 以

   浏览器的可视窗口为参照点

   移动元素

   - 跟父元素没有任何关系
   - 不随滚动条滚动

2. 固定定位不再占有原先的位置

   - 固定定位也是**脱标**的，其实固定定位也可以看做是一种**特殊的绝对定位**

### 固定定位小技巧：固定在版心右侧位置

让固定定位的盒子 `left: 50%`，走到浏览器可视区（也可以看做版心） 一半的位置

让固定定位的盒子 `margin-left: 版心宽度的一半距离`，多走版心宽度的一半位置

```HTML
<!doctype html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>固定定位小技巧-固定到版心右侧</title>
    <style>
        .w {
            width: 800px;
            height: 1400px;
            background-color: pink;
            margin: 0 auto;
        }

        .fixed {
            position: fixed;
            /* 1. 走浏览器宽度的一半 */
            left: 50%;
            /* 2. 利用 margin 走版心盒子宽度的一半距离（为了美观多加了 5px）*/
            margin-left: 405px;
            width: 50px;
            height: 150px;
            background-color: skyblue;
        }
    </style>
</head>

<body>
    <div class="fixed"></div>
    <div class="w">版心盒子 800像素</div>

</body>

</html>
```

### 粘性定位 Sticky（了解）

粘性定位可以被认为是相对定位和固定定位的混合。

```CSS
选择器 { position: sticky; top: 10px; }
```

1. 以浏览器的**可视窗口为参照点**移动元素（固定定位特点）
2. 粘性定位**占有原先的位置**（相对定位特点）
3. 必须添加 top 、left、right、bottom 其中一个才有效

### 总结定位

------

| 定位模式          | 是否脱标         | 移动位置           | 是否常用   |
| ----------------- | ---------------- | ------------------ | ---------- |
| static 静态定位   | 否               | 不能使用边偏移     | 很少       |
| relative 相对定位 | 否（占有位置）   | 相对于自身位置移动 | 常用       |
| absolute 绝对定位 | 是（不占有位置） | 带有定位的父级     | 常用       |
| fixed 固定定位    | 是（不占有位置） | 浏览器可视区       | 常用       |
| sticky 粘性定位   | 否（占有位置）   | 浏览器可视区       | 当前阶段少 |

1. 一定记住，相对定位、固定定位、绝对定位 两个大的特点： 1. 是否占有位置（脱标否） 2. 以谁为基准点移动位置。
2. 学习定位重点学会子绝父相

## 定位叠放次序 z-index

------

·在使用定位布局时，可能会出现盒子重叠的情况。此时，可以使用 z-index 来控制盒子的前后次序（z轴）。

```CSS
选择器 { z-index: 1; }
```

- 数值可以是正整数、负整数或 0，默认是 auto，数值越大，盒子越靠上
- 如果属性值相同，则按照书写顺序，后来居上
- 数字后面不能加单位
- 只有定位的盒子才有 z-index 属性

## 装饰

浏览器里的文字类型元素排版中存在用于对齐的基线（baseline）

### 垂直对齐方式

**属性名**：`vertical-align`

| 属性值   | 效果           |
| -------- | -------------- |
| baseline | 默认，基线对齐 |
| top      | 顶部对齐       |
| middle   | 中部对齐       |
| bottom   | 底部对齐       |

这里需要记住一个属性：box-sizing: border-box; 取掉标签的内外边距。

### 设置鼠标形状

```CSS
 cursor: 属性值;
```

| 属性值  | 效果     |
| ------- | -------- |
| default | 默认值   |
| pointer | 小手效果 |
| text    | 工字型   |
| move    | 十字光标 |