## JavaScript 错误类型

<br>


### 一、基础错误类型
`JavaScript` 中可以通过 `Error` 构造函数创建一个错误对象<br>
当运行时出错，Error 的实例对象将被抛出

```javascript
语法：
/**
* @param message: 错误信息
* @param fileName: 发生错误所在文件名
* @param lineNumber: 发生错误所在行号
*/
new Error([message[, fileName[,lineNumber]]])
```

<br>

### 二、扩展错误类型

<br>

还有其他 6 个错误构造函数

#### 1、语法错误：
JavaScript 解析代码时候，JavaScript 引擎发现不符合语法规范的 token 或者 token 顺序时抛出 SyntaxError<br>
一般此类语法错误都会在开发期间，通过 IDE 或者检测工具如 eslint 提示解决
```javascript
function(){
Uncaught SyntaxError: Function statements require a function name

function fn(){
Uncaught SyntaxError: Unexpected end of input

bar foo
Uncaught SyntaxError: Unexpected identifier

let bar foo
Uncaught SyntaxError: Unexpected identifier

decodeURI('a'b')
Uncaught SyntaxError: missing ) after argument list
```
<br>

#### 2、引用错误
尝试引用一个未被定义的变量时，将会抛出 ReferenceError
```javascript
console.log(a)
Uncaught ReferenceError: a is not defined
```
<br>

#### 3、类型错误
表示值的类型，非预期的类型时，将会抛出 TypeError
```javascript
Array.test() // 调用了 Array 上不存在的 test，值为 undefined，作为函数执行，则会抛出类型错误
Uncaught TypeError: Array.test is not a function
```
<br>

#### 4、范围错误
传递一个 number 参数给一个范围内不包含该 number 的函数时，将会抛出 RangeError
```javascript
new Array(12221312312)
Uncaught RangeError: Invalid array length

(1).toFixed(111)
Uncaught RangeError: toFixed() digits argument must be between 0 and 100
    at Number.toFixed (<anonymous>)
```
<br>

#### 5、eval 错误
eval 函数出错，将会抛出 EvalError<br>
（此异常不在新版 ECMAScript 规范中使用，所以运行时不再被 JavaScript 抛出，但 EvalError 仍可以使用）
```javascript
new EvalError('Hello', 'someFile.js', 10);
EvalError: Hello
```
<br>

#### 6、uri 错误
向全局 URI 处理函数传递一个不合法的 URI 时，将会抛出 URIError
```javascript
decodeURI('%')
Uncaught URIError: URI malformed
```
<br>

#### 7、内部错误
JavaScript 引擎内部发生错误时，抛出 InternalError

表示 JavaScript 引擎内部的错误：如递归过深<br>
! 这个特性是非标准的，尽量不要在生产中使用
