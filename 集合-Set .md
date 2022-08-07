## 集合 Set
备注：基本借鉴 MDN

[集合Set---MDN文档链接](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set)

**因为是个新的对象，还比较好用，经常使用有 去重、取交集、并集、差集等等。 
这里重点学习下，并学学常用场景**

<br>

### 一、描述
>Set 对象允许你存储任何类型的唯一值，无论是原始值或者对象引用

>Set 对象是值的集合，可以按照插入的顺序迭代它的元素。<br>
Set 中的元素只会出现一次，即 Set 中的元素时唯一的

<br>

**特殊点:**<br>
**NaN 和 undefined 都可以被存储在 Set 中， NaN 之间被视为相同的值（虽然在 js 中， NaN !== NaN）**

<br>

### 二、语法
语法
```javascript
new Set([iterable])

[iterable]:
    若 [iterable] 为一个可迭代对象，比如数组 [1, 2, 3, 3, 4]---({a: 1, b: 1} 是不可迭代的)
    它的所有元素将不重复的添加到新的 Set 中
    若不指定或为 null，则新的 Set 为空
```

使用
```javascript
let set1 = new Set() // Set {}
let set2 = new Set([1, 2, 3]) // Set {1, 2, 3}
let set3 = new Set('hello') // Set {"h", "e", "l", "o"}
```

<br>

### 三、属性
**常用属性 size: 表示 Set 对象值的个数**
```javascript
let set1 = new Set([1, 2, 3]) // Set {1, 2, 3}
set1.size // 3
```

<br>

### 四、方法
1、 add: 在 Set 对象尾部添加一个元素。返回改 Set 对象
```javascript
语法:
Set.prototype.add(value)

使用:
let set = new Set([0, 1])
set.add(2) // Set {0, 1, 2}
```

2、 clear: 移除 Set 对象内的所有元素
```javascript
语法:
Set.prototype.clear()

使用:
let set = new Set([0, 1])
set.clear() // Set {}
```

3、delete: 移除 Set 的中与这个值相等的元素
```javascript
语法:
Set.prototype.delete(value)

使用:
let set = new Set([0, 1])
set.delete(0) // 删除元素 存在且删除成功返回 true，否则返回 false。 (Set {1})
```

4、 entries: 返回一个新的迭代器对象<br>
该对象包含 Set 对象中的按插入顺序排列的所有元素的值的[value, value]数组。每个值的键和值相等<br>

**迭代器对象，必须使用 [...iterableObj] 或者 for (let key of obj) 等方式才能使用**
```javascript
语法:
Set.prototype.entries()

使用:
let set = new Set('hello')

// set.entries()
// SetIterator 
//     0: {key: "h", value: "h"}
//     1: {key: "e", value: "e"}
//     2: {key: "l", value: "l"}
//     3: {key: "o", value: "o"}
let entriesSet = set.entries() 

这里即可看到上面所说的 [value, value] 数组
// [...entriesSet]
// [
//  ["h", "h"]
//  ["e", "e"]
//  ["l", "l"]
//  ["o", "o"]
// ]
let enrriesArr = [...entriesSet]

// for of
for (let set of entriesSet) { // 也可以这么写 for (let [key, value] of entriesSet) {
    console.log('set:', set)
    // set:["h", "h"]
    // set:["e", "e"]
    // set:["l", "l"]
    // set:["o", "o"]
}
```

5、 forEach: 按照插入顺序，为 Set 对象中的每一个值调用一次 callBackFn。<br>
如果提供了 thisArg 参数，回调中的 this 会是这个参数。
```javascript
语法:
Set.prototype.forEach(callbackFn, [thisArg])

使用:
let set = new Set('hello')
set.forEach(s => {
    console.log('s:', s)
    // s:h
    // s:e
    // s:l
    // s:o
})
```

6、 has: 返回一个布尔值，表示该值在Set中存在与否。
```javascript
语法:
Set.prototype.has(value)

使用:
let set = new Set('hello')
set.has('o') // true
set.has('a') // false
```

7、 keys: 与 values() 方法相同，返回一个新的迭代器对象<br>
该对象包含 Set 对象中的按插入顺序排列的所有元素的值。

**迭代器对象，必须使用 [...iterableObj] 或者 for (let key of obj) 等方式才能使用**
```javascript
语法:
Set.prototype.keys()

使用:
let set = new Set('hello')

// keysSet
// SetIterator 
//     0: "h"
//     1: "e"
//     2: "l"
//     3: "o"
let keysSet = set.keys()

// ["h", "e", "l", "o"]
[...keysSet]

for (let set of keysSet) {
    console.log(set)
    // h
    // e
    // l
    // o
}
```

8、 values: 返回一个新的迭代器对象，该对象包含 Set 对象中的按插入顺序排列的所有元素的值。

**迭代器对象，必须使用 [...iterableObj] 或者 for (let key of obj) 等方式才能使用**
```javascript
语法:
Set.prototype.values()

使用:
let set = new Set('hello')

// valuesSet
// SetIterator 
//     0: "h"
//     1: "e"
//     2: "l"
//     3: "o"
let valuesSet = set.values()

// ["h", "e", "l", "o"]
[...valuesSet] 

for (let set of valuesSet) {
    console.log(set)
    // h
    // e
    // l
    // o
}
```

<br>

### 五、常用数据类型转换
1、 数组转 Set
```javascript
let arr = [1, 2, 3, 3]
let set = new Set(arr) // 可以去重 Set {1, 2, 3}
```
<br>

2、 Set 转数组
```javascript
let set = new Set([1, 2, 3])
let arr = [...set] // [1, 2, 3]

或者 
let set = new Set([1, 2, 3])
let arr = Array.from(set) // [1, 2, 3]
```

<br>

3、字符串转 Set
```javascript
let string = 'hello'
let set = new Set(string) // Set {'h', 'e', 'l', 'o'}
```

<br>

### 六、实际应用场景

1、数组去重
```javascript
let arr = [1, 2, 3, 3, 5, 5, 7, 8, 8, 9]
let uniArr = [...new Set(arr)] // [1, 2, 3, 5, 7, 8, 9]
```

<br>

2、对于集合的操作

2、1 并集
```javascript
let set1 = new Set([1, 2, 4])
let set2 = new Set([1, 2, 3, 4])
for (let set of set2) {
    // 遍历 set2，添加到 set1 中，得到并集
    set1.add(set)
}
// 输出并集
console.log(set1) // Set {1, 2, 4, 3}

或者
let set1 = new Set([1, 2, 4])
let set2 = new Set([1, 2, 3, 4])
[...set2].map(set => set1.add(set))
let unionSet = [...set1]  //  [1, 2, 3, 4]
```

2、2 交集
```javascript
let set1 = new Set([1, 2, 4])
let set2 = new Set([1, 2, 3, 4])
let set3 = new Set()
for (let set of set2) {
    // 遍历 set2，判断若 set1 中有 set2 当前遍历的元素
    // 则添加到新的 set3 中，得到交集
    if (set1.has(set)) {
        set3.add(set)
    }
}
// 输出交集
console.log(set3) // Set {1, 2, 4}

或者
let set1 = new Set([1, 2, 4])
let set2 = new Set([1, 2, 3, 4])
let finalSet = [...set2].filter(set => set1.has(set)) // [1, 2, 4]
```

2、3 差集

求 set1 相对于 set2 的差集
```javascript
let set1 = new Set([1, 2, 4, 5])
let set2 = new Set([1, 2, 3, 4])
let set3 = new Set()
for (let set of set1) {
    // 遍历 set1，若 set2 中没有 set1 当前遍历的元素
    // 则添加到新的 set3 中，得到 set1 相对于 set2 的差集
    if (!set2.has(set)) {
        set3.add(set)
    }
}
// 输出 set1 的差集
console.log(set3) // Set {5}

或者
let set1 = new Set([1, 2, 4, 5])
let set2 = new Set([1, 2, 3, 4])
let finalSet = [...set1].filter(set => !set2.has(set)) // [5]
```

求 set2 相对于 set1 的差集
```javascript
let set1 = new Set([1, 2, 4, 5])
let set2 = new Set([1, 2, 3, 4])
let set3 = new Set()
for (let set of set2) {
    // 遍历 set2，若 set1 中没有 set2 当前遍历的元素
    // 则添加到新的 set3 中，得到 set2 相对于 set1 的差集
    if (!set1.has(set)) {
        set3.add(set)
    }
}
// 输出 set2 的差集
console.log(set3) // Set {3}

或者
let set1 = new Set([1, 2, 4, 5])
let set2 = new Set([1, 2, 3, 4])
let finalSet = [...set2].filter(set => !set1.has(set)) // [3]
```

求 set1 和 set2 的差集总和
```javascript
let set1 = new Set([1, 2, 4, 5])
let set2 = new Set([1, 2, 3, 4])
for (let set of set2) {
    // 遍历 set2
    // 若 set1 有 set2 当前遍历的元素，则删除 set1 中与该元素相等的值
    // 若没有，则添加进 set1 中
    // 最后的 set1 既是差集总和
    if (set1.has(set)) {
        set1.delete(set)
    } else {
        set1.add(set)
    }
}
// 输出 set1 和 set2 的差集总和
console.log(set1) // Set {5, 3}
```