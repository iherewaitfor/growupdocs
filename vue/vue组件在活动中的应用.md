# vue组件在活动需求中的开发技巧
总体来说：一般的活动一般分为任务条和榜单：
任务条的位置一般是视频下方或者中屏右下角，是CefWebWidget，是控件；
榜单（弹窗），是一个弹出窗口CefCocosPopWnd，可以在任何位置，如果要跟随窗口移动就要加上entTemplate.keepWndPos属性；
下面具体介绍开发技巧：

## 1. v-if 和 v-show
	由于v-if在条件不成立时候是不会创建出来的，所以一些在js里面用到的根据div或者id查找元素的组件会报错
	例如：svga、qrcode等。
	解决办法：把v-if换成v-show


## 2. 圆形进度条

![Image](./img/kid.png)

```html
<svg viewBox="0 0 200 200" :width="40" :height="40">
    <circle cx="100" cy="100" :r="90" fill="none" stroke="#fdec47" :stroke-width="15" :stroke-dasharray="dashLen" :stroke-dashoffset="dashLen_per" transform="rotate(-90,100,100)" style="transition: stroke-dashoffset 0.4s" />
</svg>
```

## 3. 任务条tip实现方法
	情况一：如果tip的尺寸不大，可以把tip的尺寸加在banner里面，这样使用 @mouseover/ @mouseout，速度快省时间
	情况二：如果tip的尺寸较大，则只能把tip做成弹窗形式，而且考虑到效率问题，@mouseout的时候要稍作延时处理，避免频繁显示隐藏弹窗导致的刷新慢的问题

## 4. 字体和效果太小的实现方法
	由于部分字体只支持最小12px的大小，小于此像素的时候要实现，只能使用svg
	![svg教程](https://www.runoob.com/svg/svg-tutorial.html)

## 5. 判断协议先后和结算状态
- 由于服务器协议一般都会带上时间戳和状态
```javascript
if (本次协议结构.时间戳<=上次协议结构.时间戳 || 本次协议结构.状态<=上次协议结构.状态) {return;} //丢弃过期协议
```
```javascript
if (本次协议结构.当前时间 > 本次协议结构.结束时间) {显示;} //显示结算状态
```

## 6. 防止数据undefined和null的处理
```javascript
数据结构 || 默认值; //数据结构不存在，正确
数据结构.子结构 || 默认值; //数据结构不存在，错误
数据结构.子结构 || 默认值; //数据结构存在，正确
```

## 7. 类型转换
```javascript
isFinite();//检查某个值是否为有穷大的数。
isNaN();//检查某个值是否是数字。
Number();//把对象的值转换为数字。
parseFloat();//解析一个字符串并返回一个浮点数。
parseInt();//解析一个字符串并返回一个整数。
String();//把对象的值转换为字符串。
```