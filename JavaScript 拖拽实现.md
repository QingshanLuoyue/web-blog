### 一、普通实现

>拖拽实现原理：
1. `鼠标点击` 后 ，`记录当前鼠标位置`
2. `鼠标移动` 时
   1. 拖拽元素 `左侧` 定位位置 = 拖拽元素 与 `定位参考父级` 左侧的距离 + `移动鼠标位置 X 轴值` 减去 `之前记录的鼠标位置 X 轴值`
   2. 拖拽元素 `顶部` 定位位置 = 拖拽元素 与 `定位参考父级` 顶部的距离 + `移动鼠标位置 Y 轴值` 减去 `之前记录的鼠标位置 Y 轴值`

<br>

>鼠标 与 拖拽元素 位置变化关系
1. 第二次点击 left 位置 = 第一次当时的 offsetLeft + （第二次 client.X - 第一次 client.X）
2. 每次 拖拽元素 移动之后， oldX 会变化，需要重新赋值
3. 运动条件： 按下后再移动
   1. onmousedown
      1. onmousemove（在 onmousedown 中注册 onmousemove 事件）
   2. onmouseup
   

<br>

>备注：<br>
实现拖拽功能，一开始的想法是，鼠标移动到哪里，拖拽元素就移动到哪里<br>
那么是否可以直接设置元素的 left 和 top 为 鼠标的 clientX 和 clientY 呢<br>
然而这种做法仅限于鼠标点击元素左上角进行移动；<br>
若点击在元素中间部位，那么此刻的 clientX 和 clientY 设置成 left 和 top 后<br>
元素将会闪退部分长度，而后跟随鼠标移动

<br>

#### 全局拖拽

[全局拖拽测试用例](https://github.com/QingshanLuoyue/simple-drag/blob/master/demo/普通实现-全局拖拽.html)

css
```css
#dragson {
    position: absolute;
    width: 100px;
    height: 100px;
    background-color: blue;
}
```

html
```html
<div id="dragson"></div>
```

js
```javascript
let dragSon = document.getElementById('dragson')
// 鼠标按下
dragSon.onmousedown = function(e) {
    // 兼容 IE
    let ev = e || window.event,
        // 存储当前鼠标位置
        oldX = ev.clientX,
        oldY = ev.clientY

    // 阻止默认事件 ，即鼠标悬停在拖拽元素上，系统选中内容
    if (window.event) {
        // 兼容 IE
        window.event.returnValue = false;
    } else {
        ev.preventDefault()
    }

    // 因为鼠标移动过快，可能会移出拖拽元素的范围
    // 这里使用 document.documentElement.onmousemove 来解决
    document.documentElement.onmousemove = function(e) {
        let ev = e || window.event,
            newX = ev.clientX,
            newY = ev.clientY,
            // 拖拽元素 距离 `定位参考父级` 左侧的距离 + `移动鼠标位置 X 轴值` 减去 `之前记录的鼠标位置 X 轴值`
            // offsetLeft: 拖拽元素与父级左侧的距离长度（不含父级边框）
            endX = dragSon.offsetLeft + newX - oldX,
            // 拖拽元素 距离 `定位参考父级` 顶部的距离 + `移动鼠标位置 Y 轴值` 减去 `之前记录的鼠标位置 Y 轴值`
            // offsetTop: 拖拽元素与父级顶部的距离长度（不含父级边框）
            endY = dragSon.offsetTop + newY - oldY
        
        // 阻止默认事件 ，即鼠标悬停在拖拽元素上，系统选中内容
        if (window.event) {
            // 兼容 IE
            window.event.returnValue = false;
        } else {
            ev.preventDefault()
        }

        // 设置拖拽元素位置
        dragSon.style.left = endX + 'px'
        dragSon.style.top = endY + 'px'

        // 新旧值交换
        oldX = newX
        oldY = newY
    }
}
// 鼠标松开
document.documentElement.onmouseup = function(e) {
    // 函数赋值为 null，让函数失效，便于浏览器垃圾回收
    document.documentElement.onmousemove = null
}
```

<br>


#### 限制父级内拖拽 

[限制父级内拖拽测试用例](https://github.com/QingshanLuoyue/simple-drag/blob/master/demo/普通实现-限制父级内拖拽.html)

css
```css
#dragparent {
    position: relative;
    width: 500px;
    height: 500px;
    border: 1px solid red
}
#dragson {
    position: absolute;
    width: 100px;
    height: 100px;
    background-color: blue;
}
```

html
```html
<div id="dragparent">
    <div id="dragson"></div>
</div>
```

js
```javascript
let dragParent = document.getElementById('dragparent'),
    dragSon = document.getElementById('dragson')
// 鼠标按下
dragSon.onmousedown = function(e) {
    // 兼容 IE
    let ev = e || window.event,
        // 存储当前鼠标位置
        oldX = ev.clientX,
        oldY = ev.clientY

    // 阻止默认事件 ，即鼠标悬停在拖拽元素上，系统选中内容
    if (window.event) {
        // 兼容 IE
        window.event.returnValue = false;
    } else {
        ev.preventDefault()
    }

    // 因为鼠标移动过快，可能会移出拖拽元素的范围
    // 这里使用 document.documentElement.onmousemove 来解决
    document.documentElement.onmousemove = function(e) {
        let ev = e || window.event,
            newX = ev.clientX,
            newY = ev.clientY,
            // 拖拽元素 距离 `定位参考父级` 左侧的距离 + `移动鼠标位置 X 轴值` 减去 `之前记录的鼠标位置 X 轴值`
            // offsetLeft: 拖拽元素与父级左侧的距离长度（不含父级边框）
            endX = dragSon.offsetLeft + newX - oldX,
            // 拖拽元素 距离 `定位参考父级` 顶部的距离 + `移动鼠标位置 Y 轴值` 减去 `之前记录的鼠标位置 Y 轴值`
            // offsetTop: 拖拽元素与父级顶部的距离长度（不含父级边框）
            endY = dragSon.offsetTop + newY - oldY
        
        // 阻止默认事件 ，即鼠标悬停在拖拽元素上，系统选中内容
        if (window.event) {
            // 兼容 IE
            window.event.returnValue = false;
        } else {
            ev.preventDefault()
        }

        // 限制在 定位参考父级元素 内移动
        // 左边界
        if (endX <= 0) {
            endX = 0
        }
        // 右边界
        // X 轴方向可移动距离 = 父级内部宽度 - 拖拽元素宽度
        if (endX >= dragParent.clientWidth - dragSon.clientWidth) {
            endX = dragParent.clientWidth - dragSon.clientWidth
        }
        
        // 上边界
        if (endY <= 0) {
            endY = 0
        }
        // 下边界
        // Y 轴方向可移动距离 = 父级内部高度 - 拖拽元素高度
        if (endY >= dragParent.clientHeight - dragSon.clientHeight) {
            endY = dragParent.clientHeight - dragSon.clientHeight
        }

        // 设置拖拽元素位置
        dragSon.style.left = endX + 'px'
        dragSon.style.top = endY + 'px'

        // 新旧值交换
        oldX = newX
        oldY = newY
    }
}
// 鼠标松开
document.documentElement.onmouseup = function(e) {
    // 函数赋值为 null，让函数失效，便于浏览器垃圾回收
    document.documentElement.onmousemove = null
}
```
<br>

### 二、封装

[封装测试用例](https://github.com/QingshanLuoyue/simple-drag/blob/master/demo/拖拽-封装.html)

> 实现对 `拖拽` 功能的 `类封装`

```javascript
class Drag {
    constructor(option) {
        this.oldX = 0
        this.oldY = 0
        this.newX = 0
        this.newY = 0
        this.maxMoveWidth = 0
        this.maxMoveHeight = 0

        if (!option.dragEle) throw '拖拽元素 dragEle 必须传递'
        this.dragSon = option.dragEle

        if (option.parent) {
            this.dragParent = option.parent
            this.maxMoveWidth = this.dragParent.clientWidth - this.dragSon.clientWidth
            this.maxMoveHeight = this.dragParent.clientHeight - this.dragSon.clientHeight
        }

        this.init()
    }
    // 初始化
    init() {
        // 鼠标按下
        this.dragSon.onmousedown = e => {
            // 兼容 IE
            let ev = e || window.event

            // 阻止默认事件 ，即鼠标悬停在拖拽元素上，系统选中内容
            if (window.event) {
                // 兼容 IE
                window.event.returnValue = false;
            } else {
                ev.preventDefault()
            }

            // 存储当前鼠标位置
            this.oldX = ev.clientX
            this.oldY = ev.clientY

            this.dragMove()
        }

        // 鼠标松开
        document.documentElement.onmouseup = function(e) {
            // 函数赋值为 null，让函数失效，便于浏览器垃圾回收
            document.documentElement.onmousemove = null
        }
    }
    // 鼠标移动
    dragMove() {
        // 因为鼠标移动过快，可能会移出拖拽元素的范围
        // 这里使用 document.documentElement.onmousemove 来解决
        document.documentElement.onmousemove = e => {
            let ev = e || window.event, endX, endY

            // 阻止默认事件 ，即鼠标悬停在拖拽元素上，系统选中内容
            if (window.event) {
                // 兼容 IE
                window.event.returnValue = false;
            } else {
                ev.preventDefault()
            }

            this.newX = ev.clientX,
            this.newY = ev.clientY,
            // 拖拽元素 距离 `定位参考父级` 左侧的距离 + `移动鼠标位置 X 轴值` 减去 `之前记录的鼠标位置 X 轴值`
            // offsetLeft: 拖拽元素与父级左侧的距离长度（不含父级边框）
            endX = this.dragSon.offsetLeft + this.newX - this.oldX,
            // 拖拽元素 距离 `定位参考父级` 顶部的距离 + `移动鼠标位置 Y 轴值` 减去 `之前记录的鼠标位置 Y 轴值`
            // offsetTop: 拖拽元素与父级顶部的距离长度（不含父级边框）
            endY = this.dragSon.offsetTop + this.newY - this.oldY

            // 若存在父级，限制在父级范围内移动
            if (this.dragParent) {
                let { limitEndX, limitEndY } = this.limitRange(endX, endY)
                endX = limitEndX
                endY = limitEndY
            }

            // 设置拖拽元素位置
            this.dragSon.style.left = endX + 'px'
            this.dragSon.style.top = endY + 'px'

            // 新旧值交换
            this.oldX = this.newX
            this.oldY = this.newY
        }
    }
    // 限制移动范围
    limitRange(endX, endY) {
        // 限制在 定位参考父级元素 内移动
        // 左边界
        if (endX <= 0) {
            endX = 0
        }
        // 右边界
        // X 轴方向可移动距离 = 父级内部宽度 - 拖拽元素宽度
        if (endX >= this.maxMoveWidth) {
            endX = this.maxMoveWidth
        }
        
        // 上边界
        if (endY <= 0) {
            endY = 0
        }
        // 下边界
        // Y 轴方向可移动距离 = 父级内部高度 - 拖拽元素高度
        if (endY >= this.maxMoveHeight) {
            endY = this.maxMoveHeight
        }
        return { limitEndX: endX, limitEndY: endY }
    }
}
```

<br>

### 三、支持移动端

[支持移动端测试用例](https://github.com/QingshanLuoyue/simple-drag/blob/master/demo/拖拽-封装-支持移动端.html)

```javascript
class Drag {
    constructor(option) {
        this.oldX = 0
        this.oldY = 0
        this.newX = 0
        this.newY = 0
        this.maxMoveWidth = 0
        this.maxMoveHeight = 0

        this.isMobile = false
        if ('ontouchstart' in window) {
            this.isMobile = true
        }

        this.eventType = {
            start: this.isMobile ? 'ontouchstart' : 'onmousedown',
            move: this.isMobile ? 'ontouchmove' : 'onmousemove',
            end: this.isMobile ? 'ontouchend' : 'onmouseup',
        }

        if (!option.dragEle) throw '拖拽元素 dragEle 必须传递'
        this.dragSon = option.dragEle

        if (option.parent) {
            this.dragParent = option.parent
            this.maxMoveWidth = this.dragParent.clientWidth - this.dragSon.clientWidth
            this.maxMoveHeight = this.dragParent.clientHeight - this.dragSon.clientHeight
        }

        this.init()
    }
    // 初始化
    init() {
        // 鼠标按下
        this.dragSon[this.eventType['start']] = e => {
            // 兼容 IE
            let ev = e || window.event

            // 阻止默认事件 ，即鼠标悬停在拖拽元素上，系统选中内容
            if (window.event) {
                // 兼容 IE
                window.event.returnValue = false;
            } else {
                ev.preventDefault()
            }

            if (this.isMobile) {
                ev = ev.touches[0]
            }

            // 存储当前鼠标位置
            this.oldX = ev.clientX
            this.oldY = ev.clientY

            this.dragMove()
        }
        
        // 鼠标松开
        document.documentElement[this.eventType['end']] = e => {
            // 函数赋值为 null，让函数失效，便于浏览器垃圾回收
            document.documentElement[this.eventType['move']] = null
        }
    }
    // 鼠标移动
    dragMove() {
        // 因为鼠标移动过快，可能会移出拖拽元素的范围
        // 这里使用 document.documentElement.onmousemove 来解决
        document.documentElement[this.eventType['move']] = e => {
            let ev = e || window.event, endX, endY

            // 阻止默认事件 ，即鼠标悬停在拖拽元素上，系统选中内容
            if (window.event) {
                // 兼容 IE
                window.event.returnValue = false;
            } else {
                ev.preventDefault()
            }
            
            if (this.isMobile) {
                ev = ev.touches[0]
            }

            this.newX = ev.clientX,
            this.newY = ev.clientY,
            // 拖拽元素 距离 `定位参考父级` 左侧的距离 + `移动鼠标位置 X 轴值` 减去 `之前记录的鼠标位置 X 轴值`
            // offsetLeft: 拖拽元素与父级左侧的距离长度（不含父级边框）
            endX = this.dragSon.offsetLeft + this.newX - this.oldX,
            // 拖拽元素 距离 `定位参考父级` 顶部的距离 + `移动鼠标位置 Y 轴值` 减去 `之前记录的鼠标位置 Y 轴值`
            // offsetTop: 拖拽元素与父级顶部的距离长度（不含父级边框）
            endY = this.dragSon.offsetTop + this.newY - this.oldY
            
            // 若存在父级，限制在父级范围内移动
            if (this.dragParent) {
                let { limitEndX, limitEndY } = this.limitRange(endX, endY)
                endX = limitEndX
                endY = limitEndY
            }

            // 设置拖拽元素位置
            this.dragSon.style.left = endX + 'px'
            this.dragSon.style.top = endY + 'px'

            // 新旧值交换
            this.oldX = this.newX
            this.oldY = this.newY
        }
    }
    
    // 限制移动范围
    limitRange(endX, endY) {
        // 限制在 定位参考父级元素 内移动
        // 左边界
        if (endX <= 0) {
            endX = 0
        }
        // 右边界
        // X 轴方向可移动距离 = 父级内部宽度 - 拖拽元素宽度
        if (endX >= this.maxMoveWidth) {
            endX = this.maxMoveWidth
        }
        
        // 上边界
        if (endY <= 0) {
            endY = 0
        }
        // 下边界
        // Y 轴方向可移动距离 = 父级内部高度 - 拖拽元素高度
        if (endY >= this.maxMoveHeight) {
            endY = this.maxMoveHeight
        }
        return { limitEndX: endX, limitEndY: endY }
    }
}
```

<br>