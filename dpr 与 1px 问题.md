### dpr 与 移动端 1px 问题

#### 名词概念
首先明白下具体名词概念：<br>
>设备出厂自带 `硬件像素` ，称之为 `物理像素`<br>
页面 css  `布局像素` ，称之为 `逻辑像素`

<br>

#### dpr 与两者关系

web 开发中， css 写了 1px，在设备中显示的 `物理像素` 不是简单 1 对 1 关系<br>
这取决于设备的 `dpr`
```javascript
dpr（devicePixelRatio - 设备像素比例）

dpr = 物理像素 / 逻辑像素
```
<br>

若一个 安卓/iOS 设备的 `dpr` 是 `2`
>那么 一个 `逻辑像素` 等于 `2` 个 `物理像素`

<br>

假如该设备宽度是 `750px`，即 `物理像素 750px` 。此时 UI 出了一个 `750px` 的 PSD 图片<br>
这样意味着在 `750px` 设计稿上的 `1px` ，这个 `1px` 是 `物理像素`<br>
用 `css` 单位 `1px` 显示的话，实际是 `2` 个 `物理像素` ，就导致看起来比UI图偏大

<br>

#### 解决方法
有很多其他的解决方法<br>
这里使用 transform 来实现
```SCSS
.element-bottom-top {
    position: relative;
    &:after {
        content: '';
        position: absolute;
        height: 1px;
        width: 100%;
        background: blue;
    }
    // dpr = 2
    @media screen and (-webkit-min-device-pixel-ratio: 2) {
        &:after {
            transform: scaleY(0.5); // 将高度 height 的 1px 缩小 0.5
            transform-origin: 0 0; // scale 缩小操作会引起位移差，需要 transform-origin 属性做调整
        }
    }
    // dpr = 3
    @media screen and (-webkit-min-device-pixel-ratio: 3) {
        &:after {
            transform: scaleY(0.33); // 将高度 height 的 1px 缩小 0.5
            transform-origin: 0 0; // scale 缩小操作会引起位移差，需要 transform-origin 属性做调整
        }
    }
}
```