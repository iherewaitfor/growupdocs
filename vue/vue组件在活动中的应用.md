# vue 组件在活动需求中的开发技巧

总体来说：一般的活动一般分为任务条和榜单：
任务条的位置一般是视频下方或者中屏右下角，是 CefWebWidget，是控件；
榜单（弹窗），是一个弹出窗口 CefCocosPopWnd，可以在任何位置，如果要跟随窗口移动就要加上 entTemplate.keepWndPos 属性；
下面具体介绍开发技巧：

## 1. v-if 和 v-show

    由于v-if在条件不成立时候是不会创建出来的，所以一些在js里面用到的根据div或者id查找元素的组件会报错
    例如：svga、qrcode等。
    解决办法：把v-if换成v-show

## 2. 圆形进度条

![Image](./img/kid.png)

```html
<svg viewBox="0 0 200 200" :width="40" :height="40">
  <circle
    cx="100"
    cy="100"
    :r="90"
    fill="none"
    stroke="#fdec47"
    :stroke-width="15"
    :stroke-dasharray="dashLen"
    :stroke-dashoffset="dashLen_per"
    transform="rotate(-90,100,100)"
    style="transition: stroke-dashoffset 0.4s"
  />
</svg>
```

## 3. 任务条 tip 实现方法

    情况一：如果tip的尺寸不大，可以把tip的尺寸加在banner里面，这样使用 @mouseover/ @mouseout，速度快省时间
    情况二：如果tip的尺寸较大，则只能把tip做成弹窗形式，而且考虑到效率问题，@mouseout的时候要稍作延时处理，避免频繁显示隐藏弹窗导致的刷新慢的问题

## 4. 字体和效果太小的实现方法

    由于部分字体只支持最小12px的大小，小于此像素的时候要实现，只能使用svg
    ![svg教程](https://www.runoob.com/svg/svg-tutorial.html)

## 5. 判断协议先后和结算状态

- 由于服务器协议一般都会带上时间戳和状态

```javascript
if (
  本次协议结构.时间戳 <= 上次协议结构.时间戳 ||
  本次协议结构.状态 <= 上次协议结构.状态
) {
  return;
} //丢弃过期协议
```

```javascript
if (本次协议结构.当前时间 > 本次协议结构.结束时间) {
  显示;
} //显示结算状态
```

## 6. 防止数据 undefined 和 null 的处理

```javascript
数据结构 || 默认值; //数据结构不存在，正确
数据结构.子结构 || 默认值; //数据结构不存在，错误
数据结构.子结构 || 默认值; //数据结构存在，正确
```

## 7. 类型转换

```javascript
isFinite(); //检查某个值是否为有穷大的数。
isNaN(); //检查某个值是否是数字。
Number(); //把对象的值转换为数字。
parseFloat(); //解析一个字符串并返回一个浮点数。
parseInt(); //解析一个字符串并返回一个整数。
String(); //把对象的值转换为字符串。
```

## 8. 写配置

    如果想本地只弹一次提示这种要记录状态的操作

```javascript
//写配置
const key = `key_${用户id}`;
if (!window.localStorage.getItem(key)) {
  弹框代码;
  window.localStorage.setItem(key, "1");
}

//监听
window.addEventListener(
  "storage",
  e => {
    //e只是一个传参
    //获取被修改的键值
    console.log("storagechange", e);
    if (e.key == `key_${用户id}`) {
      if (e.newValue == "1") {
        console.log("doing");
      } else if (e.newValue == "0") {
        console.log("undoing");
      }
    }
  },
  false
);
```

## 9. css 相关

```css
 {
  line-height: 30px; //设置行高避免字体显示不全
  user-select: none; //用户不能选择
  pointer-events: auto/none; //事件穿透 none-不接事件 auto-接受事件
}

//css循环样式
@for $i from 1 through 9 {
  .lv-#{$i} {
    background: url(~img/level/lv-#{$i}.png) center no-repeat;
  }
}

//继承样式
@extend .样式;
```

## 10. video

```html
<video :src="video_path" autoplay loop object-fit="fill" class="see"></video>
```

```css
.see {
  position: absolute;
  left: 50px;
  top: 177px;
  width: 340px;
  height: 340px;
  // background-color: #25252f;
  object-fit: fill;
}
```
