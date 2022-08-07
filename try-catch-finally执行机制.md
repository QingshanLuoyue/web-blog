## try catch finally 执行顺序与抛错机制

### 一、结论
1. `try 先执行` ，若有报错，执行 catch

2. `不管有没有异常 finally 肯定会执行`
   
3. try 和 catch 中 `存在 return 或者 throw` ， `finally 也会执行`

4. catch 中若报错：<br>
   猜测先 `暂存 catch 错误` ，等待 finally 执行完毕后，抛出错误

5. catch 中若报错， finally 也报错：
   1. 猜测先 `暂存 catch 错误` ，等待 finally 执行，发现 finally 存在错误，`直接抛出 finally 的错误`
   2. 抛出 catch 错误的程序不会执行，所以最终表示为 `抛出 finally 错误，catch 错误不显示`

6. 在 try catch 中，如果碰到 return 语句:
   1. 先执行 `return 语句表达式` ，但是执行结果会被 `局部变量` 暂存
   2. 之后执行 finally 内容，再执行 return 操作，`返回 局部变量暂存 的结果`

7. try 或 catch 若有 return ：<br>
   `暂存 return 的执行结果` ，等待 finally 执行完毕后，才返回 try 或 catch 中 `return 的执行结果`

8. try 或 catch 若有 return ，finally 也有 return：
   1. `暂存 try 和 catch return 的执行结果` ，等待 finally 执行完毕
   2. finally 存在 return ，`直接返回 finally 的 return 值，程序提前退出`

<br>

### 二、场景

>throw 的作用是手动中断程序，抛出一个异常，会被捕获

>以下的 `制造一个异常` 场景可以视为 `throw 一个常量或者其他数据`，所以 throw 的场景等同异常场景

<br>

#### 1、无错误情况
```
1、正常执行
2、在 try 中 return 一个 常量
3、在 try 中 return 一个 常量, 在 finally 中 return 一个 常量
```

<br>

#### 2、有错误情况
```
1、在 try 中 制造一个异常
2、在 try、catch 中 各制造一个异常
3、在 try、catch、finally 中 各制造一个异常

4、在 try 中 制造一个异常， catch return 一个 常量
5、在 try 中 制造一个异常， catch return 一个 常量，finally return 一个常量
```

<br>

### 三、测试代码

以下测试用例都执行
```javascript
console.log(test())
```

#### 1、正常执行
```javascript
let test = function() {
    try {
        console.log('try:>>', 1)
    } catch (e) {
        console.log('catch:>> ', e)
    } finally {
        console.log('finally:>> ')
    }
}
// 没有报错，正常输出
// try:>> 1
// finally:>> 
// undefined
```
<br>

#### 2、在 try 中 return 一个 常量

1. try 中 return 的结果被 局部变量 保存起来，执行 finally 之后再输出
```javascript
let test = function() {
    try {
        // 未定义 a
        console.log('try:>>', 1)
        return 2
    } catch (e) {
        console.log('catch:>> ', e)
    } finally {
        console.log('finally:>> ')
    }
}

// try:>> 1
// finally:>> 
// 2
```
<br>

#### 3、在 try 中 return 一个 常量, 在 finally 中 return 一个 常量

1. try return 的值被 局部变量 保存起来，执行 finally 之后,
2. 发现 finally 中存在 return ，直接 返回 finally 的 return 结果，程序提前退出
```javascript
let test = function() {
    try {
        console.log('try:>>', 1)
        return 2
    } catch (e) {
        console.log('catch:>> ', e)
    } finally {
        console.log('finally:>> ')
        return 3
    }
}

// try:>> 1
// finally:>> 
// 3
```
<br>

#### 4、在 try 中 制造一个异常

1. try 异常被 catch 捕获，输出 try 异常，继续执行 finally
```javascript
let test = function() {
    try {
        console.log('try:>>', a)
    } catch (e) {
        console.log('catch:>> ', e)
    } finally {
        console.log('finally:>> ')
    }
}
// catch:>>  ReferenceError: a is not defined
// finally:>> 
// undefined
```
<br>

#### 5、在 try、catch 中 各制造一个异常

1. try 异常被 catch 捕获，此时catch 异常，猜测先 `暂存 catch 错误` ，等待 finally 执行完毕后，抛出 catch 错误
```javascript
let test = function() {
    try {
        console.log('try:>>', a)
    } catch (e) {
        console.log('catch:>> ', e)
        console.log('catch:>> ', b)
    } finally {
        console.log('finally:>> ')
    }
}

// catch:>>  ReferenceError: a is not defined
// finally:>> 
// ReferenceError: b is not defined
```
<br>

#### 6、在 try、catch、finally 中 各制造一个异常

1. try 异常被 catch 捕获，此时 catch 异常，猜测先 `暂存 catch 错误` ，等待 finally 执行，发现 finally 存在错误，`直接抛出 finally 的错误`
2. 抛出 catch 错误的程序不会执行，所以最终表示为 `抛出 finally 错误，catch 错误不显示`
```javascript
let test = function() {
    try {
        console.log('try:>>', a)
    } catch (e) {
        console.log('catch:>> ', e)
        console.log('catch:>> ', b)
    } finally {
        console.log('finally:>> ')
        console.log('finally:>> ', c)
    }
}

// catch:>>  ReferenceError: a is not defined
// finally:>> 
// ReferenceError: c is not defined

```
<br>

#### 7、在 try 中 制造一个异常， catch return 一个 常量

1. try 异常被 catch 捕获， `局部变量暂存 catch return 的执行结果` ，
2. 执行 finally 之后，返回 `局部变量暂存的结果`
```javascript
let test = function() {
    try {
        console.log('try:>>', a)
    } catch (e) {
        console.log('catch:>> ', e)
        return 1
    } finally {
        console.log('finally:>> ')
    }
}

// catch:>>  ReferenceError: a is not defined
// finally:>> 
// 1

```
<br>

#### 8、在 try 中 制造一个异常， catch return 一个 常量，finally return 一个常量

1. try 异常被 catch 捕获， `局部变量暂存 catch return 的执行结果` ，等待 finally 执行完毕 
2. finally 存在 return ，`直接返回 finally 的 return 值，程序提前退出`
```javascript
let test = function() {
    try {
        console.log('try:>>', a)
    } catch (e) {
        console.log('catch:>> ', e)
        return 1
    } finally {
        console.log('finally:>> ')
        return 2
    }
}

// catch:>>  ReferenceError: a is not defined
// finally:>> 
// 2
```
<br>

#### 9、验证 try return 后，结果暂存，执行 finally 后，最后返回暂存结果
```javascript
let test = function() {
    let x = 1
    try {
        // 此处 执行 ++x 表达式， x = 2，但是程序不会直接返回结果
        // 而是会使用 局部变量 缓存 return 的值 2 
        // 等待 finally 执行完成后，再 return 缓存的 局部变量值
        return ++x
    } catch (e) {
        
    } finally {
        // 这里 x = 3
        // x 变成了 3，但是最终函数执行的返回值是 2
        ++x
    }
    // 因为上面 return ，此处不执行
    console.log('x :>> ', x);
    return x
}
// 最后输出结果为： 2
// 分析：
// 1、
// try 执行 ++x 表达式， x = 2，但是程序不会直接返回结果
// 而是会使用 局部变量 缓存 return 的值 2 

// 2、
// 执行 finally
// x = 3

// 3、
// finally 执行完成后， return 缓存的 局部变量值 2
// 最后输出结果为 2
```