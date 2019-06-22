WEB页面实现等比例缩放自适应 - 通过 rem 和 vw 实现
---

## 一、rem 和 vw 简介

### 1. rem

`rem` 是相对长度单位，是指相对于根元素(即html元素)font-size(字号大小)的倍数。

#### **浏览器支持：[Caniuse](https://caniuse.com/#search=rem)**

![rem浏览器支持](http://md.ws65535.top/xsj/2019_5_21_rem浏览器支持.jpg)

#### **示例**

* 若根元素 `font-size` 为 12px
```css
html {
	font-size: 12px;
}
h1 { 
	font-size: 2rem;  /* 2 × 12px = 24px */
} 
p {
	font-size: 1.5rem;   /* 1.5 × 12px = 18px */
}
div {
	width: 10rem;  /* 10 × 12px = 120px */
} 
```

* 若根元素 `font-size` 为 16px
```css
html {
	font-size: 16px;
}
h1 { 
	font-size: 2rem;  /* 2 × 16px = 32px */
} 
p {
	font-size: 1.5rem;   /* 1.5 × 16px = 24px */
}
div {
	width: 10rem;  /* 10 × 16px = 160px */
} 
```

### 2. vw

`vw` 是相对长度单位，相对于浏览器窗口的宽度，浏览器窗口宽度被均分为100个单位的vw

#### **浏览器支持：[Caniuse](https://caniuse.com/#search=vw)**

![rvw浏览器支持](http://md.ws65535.top/xsj/2019_5_21_vw浏览器支持.jpg)

* Opera Mini不支持该属性

#### 示例

* 当浏览器窗口宽度为320px时，1vw = 3.2px
```css
p {
	font-size: 5vw;   /* 5 × 3.2px = 16px */
}
div {
	width: 20vw;  /* 20 × 3.2px = 64px*/
}
```

* 当浏览器窗口宽度为375px时，1vw = 3.75px
```css
p {
	font-size: 5vw;   /* 5 × 3.75px = 18.75px */
}
div {
	width: 20vw;  /* 20 × 3.75px = 75px*/
}
```

## 二、rem 和 vw 结合实现WEB页面等比例缩放自适应

### 1. 选择基准窗口宽度及

示例：  
以 iPhone 6/7/8/X 的屏幕宽度 375px 作为基准窗口宽度；  
以 15px 最为 `html` 元素的 `font-size`，即rem单位的基本长度。

```css
html {
	font-size: 15px;
}
h1 { 
	font-size: 2rem;  /* 2 × 15px = 30px */
} 
p {
	font-size: 1.2rem;   /* 1.2 × 15px = 18px */
}
div {
	width: 10rem;  /* 10 × 15px = 150px*/
} 
```

> 注意：`html` 元素的 `font-size` 不宜过大，也不宜过小。
> 当 `font-size` 过大时，以其为基准的 rem 数值会出现精度丢失，造成较大的误差。
> 当 `font-size` 过小时，由于很多主流浏览器 `font-size` 不能小于12px，当 `font-size` 小于12px 时，会以 12px 展示。此时，rem 单位会以 12px 为基准进行计算，页面就会整个跑偏。

### 2. 将 html 元素的 font-size 替换为使用 vw 表示

	窗口宽度：375px 
	
	=> 1vw  = 3.75px
	=> 15px = ( 15 / 3.75 )vw = 4vw

因此， html 元素的 font-size 可以替换为 4vw 

```css
html {
	font-size: 4vw;
}
h1 { 
	font-size: 2rem;  /* 2 × 4vw × 3.75px = 30px */
} 
p {
	font-size: 1.2rem;   /* 1.2 × 4vw × 3.75px = 18px */
}
div {
	width: 10rem;  /* 10 × 4vw × 3.75px = 150px*/
}
```

**当窗口宽度调整为320px时**

	1vw = 3.2px
	4vw = 4 × 3.2px = 12.8px

```css
html {
	font-size: 4vw;
}
h1 { 
	font-size: 2rem;  /* 2 × 4vw × 3.2px = 25.6px */
} 
p {
	font-size: 1.2rem;   /* 1.2 × 4vw × 3.2px = 15.36px */
}
div {
	width: 10rem;  /* 10 × 4vw × 3.2px = 128px*/
}
```

可见，此时所有以rem为单位的字号和长度都会随着屏幕宽度的放大和缩小而进行等比例缩放。


> **重要的事情说第二遍**
> 注意：`html` 元素的 `font-size` 不宜过大，也不宜过小。
> 当 `font-size` 过大时，以其为基准的 rem 数值会出现精度丢失，造成较大的误差。
> 当 `font-size` 过小时，由于很多主流浏览器 `font-size` 不能小于12px，当 `font-size` 小于12px 时，会以 12px 展示。此时，rem 单位会以 12px 为基准进行计算，页面就会整个跑偏。

### 3. 为页面设置最大宽度和最小宽度

#### 当页面小于300px时，不再等比例缩小，当页面大于500px时，不再等比例放大

**窗口宽度300px时**

	1vw  = 3px
	4vw = 4 × 3px = 12px

**窗口宽度500px时**

	1vw  = 5px
	4vw = 4 × 5px = 20px


```css
@media screen and (max-width: 300px) {
    html {
        width: 300px;
        font-size: 12px;
    }
}

@media screen and (min-width: 500px) {
    html {
        width: 500px;
        font-size: 20px;
        margin: 0 auto;  /* 让窗口水平居中展示 */
    }
}
```

## 三、根据浏览器宽度切换PC和WAP页面

### 1. 当页面宽度大于阈值时，自动切换到PC页面，当小于阈值时，切换回WAP页面

#### **WAP页面**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>WAP页面</title>
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
</head>
<body id="wap">
    我是WAP页面
<script src="https://mat1.gtimg.com/libs/jquery2/2.2.0/jquery.js"></script>
<script>
function adapt() {
    var agent;
    var clientWidth = document.body.clientWidth;
    console.log(clientWidth);
    if (clientWidth < 800) {
        agent = 'wap';
    } else {
        agent = 'pc'
    }

    if ($('body').attr('id') !== agent) {
        location.href = 'pc.html';
    }
}
adapt();
window.onresize = function(){
    adapt();
};
</script>
</body>
</html>
```

#### **PC页面**
```
<!DOCTYPE html>
<html lang="en">
<head>
    <title>我是PC页面</title>
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
</head>
<body id="pc">
    我是PC页面
<script src="https://mat1.gtimg.com/libs/jquery2/2.2.0/jquery.js"></script>
<script>
function adapt() {
    var agent;
    var clientWidth = document.body.clientWidth;
    console.log(clientWidth);
    if (clientWidth < 800) {
        agent = 'wap';
    } else {
        agent = 'pc'
    }

    if ($('body').attr('id') !== agent) {
        location.href = 'wap.html';
    }
}
adapt();
window.onresize = function(){
    adapt();
};
</script>
</body>
</html>
```
