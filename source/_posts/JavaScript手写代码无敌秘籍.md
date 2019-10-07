---
title: JavaScript手写代码无敌秘籍
thumbnail: 
date: 2019-10-07 23:19:44
updated: 2019-10-07 23:19:44
layout:
tags: 
    - 面试
    - JavaScript
categories:
    - 面试
    - JavaScript
comments:
---
中高级前端面试之手写代码

<!-- more -->

- [实现一个new操作符](#new)
+ [实现一个JSON.stringify](#stringify)
* [实现一个JSON.parse](#parse)
- [实现一个call或 apply](#apply)
+ [实现一个Function.bind](#bind)
* [实现一个继承](#extend)
- [实现一个JS函数柯里化](#kelihua)
+ [手写一个Promise(中高级必考)](#promise)
* [手写防抖(Debouncing)和节流(Throttling)](#Throttling)
- [手写一个JS深拷贝](#deepCopy)
+ [实现一个instanceOf](#instanceOf)


### **<span id='new'>1. 实现一个 <font color=#ff502c>new</font> 操作符</span>**
---

***new*** 操作符做了这些事：
- 它创建了一个全新的对象。
- 它会被执行 ***[[ Prototype ]]***（也就是 ***`__proto__`***）链接。
- 它使 ***this*** 指向新创建的对象。
- 通过 ***new*** 创建的每个对象将最终被 ***[[ Prototype ]]*** 链接到这个函数的 ***prototype*** 对象上。
- 如果函数没有返回对象类型 *** Object*** (包含 ***Functoin, Array, Date, RegExg, Error*** )，那么 ***new*** 表达式中的函数调用将返回该对象引用。

<!-- 代码块 -->
``` base
function New(func) {
    var res = {};
    if (func.prototype !== null) {
        res.__proto__ = func.prototype;
    }
    var ret = func.apply(res, Array.prototype.slice.call(arguments, 1));
    if ((typeof ret === "object" || typeof ret === "function") && ret !== null) {
        return ret;
    }
    return res;
}
var obj = New(A, 1, 2);
// equals to
var obj = new A(1, 2);
```


### **<span id='stringify'>2. 实现一个 <font color=#ff502c>JSON.stringify</font></span>**
---

<font color=#ff502c>JSON.stringify(value[, replacer [, space]])</font>

- ***Boolean | Number| String*** 类型会自动转换成对应的原始值。
- ***undefined***、任意函数以及 ***symbol***，会被忽略（出现在非数组对象的属性值中时），或者被转换成 ***null***（出现在数组中时）。
- 不可枚举的属性会被忽略
- 如果一个对象的属性值通过某种间接的方式指回该对象本身，即循环引用，属性也会被忽略。

``` base
function jsonStringify(obj) {
    let type = typeof obj;
    if (type !== "object") {
        if (/string|undefined|function/.test(type)) {
            obj = '"' + obj + '"';
        }
        return String(obj);
    } else {
        let json = []
        let arr = Array.isArray(obj)
        for (let k in obj) {
            let v = obj[k];
            let type = typeof v;
            if (/string|undefined|function/.test(type)) {
                v = '"' + v + '"';
            } else if (type === "object") {
                v = jsonStringify(v);
            }
            json.push((arr ? "" : '"' + k + '":') + String(v));
        }
        return (arr ? "[" : "{") + String(json) + (arr ? "]" : "}")
    }
}
jsonStringify({x : 5}) // "{"x":5}"
jsonStringify([1, "false", false]) // "[1,"false",false]"
jsonStringify({b: undefined}) // "{"b":"undefined"}"
```

### **<span id='parse'>3. 实现一个 <font color=#ff502c>JSON.parse</font></span>**
---

<font color=#ff502c>JSON.parse(text[, reviver])</font>

用来解析JSON字符串，构造由字符串描述的JavaScript值或对象。提供可选的reviver函数用以在返回之前对所得到的对象执行变换(操作)。

#### 3.1 第一种：直接调用 eval

``` base
function jsonParse(opt) {
    return eval('(' + opt + ')');
}
jsonParse(jsonStringify({x : 5}))
// Object { x: 5}
jsonParse(jsonStringify([1, "false", false]))
// [1, "false", falsr]
jsonParse(jsonStringify({b: undefined}))
// Object { b: "undefined"}
```
<font color=#666>避免在不必要的情况下使用 eval，eval() 是一个危险的函数， 他执行的代码拥有着执行者的权利。如果你用 eval()运行的字符串代码被恶意方（不怀好意的人）操控修改，您最终可能会在您的网页/扩展程序的权限下，在用户计算机上运行恶意代码。</font>

***它会执行JS代码，有XSS漏洞。***

如果你只想记这个方法，就得对参数json做校验。
```
var rx_one = /^[\],:{}\s]*$/;
var rx_two = /\\(?:["\\\/bfnrt]|u[0-9a-fA-F]{4})/g;
var rx_three = /"[^"\\\n\r]*"|true|false|null|-?\d+(?:\.\d*)?(?:[eE][+\-]?\d+)?/g;
var rx_four = /(?:^|:|,)(?:\s*\[)+/g;
if (
    rx_one.test(
        json
            .replace(rx_two, "@")
            .replace(rx_three, "]")
            .replace(rx_four, "")
    )
) {
    var obj = eval("(" +json + ")");
}
```

#### 3.2 第二种：Function

核心：***Function*** 与 ***eval*** 有相同的字符串参数特性。

```
var func = new Function(arg1, arg2, ..., functionBody);
```

在转换JSON的实际应用中，只需要这么做。

```
var jsonStr = '{ "age": 20, "name": "jack" }'
var json = (new Function('return ' + jsonStr))();
```

***eval*** 与 ***Function*** 都有着动态编译js代码的作用，但是在实际的编程中并不推荐使用。

### **<span id='apply'>4. 实现一个 <font color=#ff502c>call</font> 或 <font color=#ff502c>apply</font></span>**
---

***call*** 语法：

<font color=#ff502c>fun.call(thisArg, arg1, arg2, ...)，</font>调用一个函数, 其具有一个指定的this值和分别地提供的参数(参数的列表)。

***apply*** 语法：

<font color=#ff502c>func.apply(thisArg, [argsArray])，</font>调用一个函数，以及作为一个数组（或类似数组对象）提供的参数。

#### 4.1 <font color=#ff502c>Function.call</font> 按套路实现

***call***核心：
- 将函数设为对象的属性
- 执行&删除这个函数
- 指定this到函数并传入给定参数执行函数
- 如果不传入参数，默认指向为 window

##### 4.1.1 简单版
```
var foo = {
    value: 1,
    bar: function() {
        console.log(this.value)
    }
}
foo.bar() // 1
```

##### 4.1.2 完善版
```
Function.prototype.call2 = function(content = window) {
    content.fn = this;
    let args = [...arguments].slice(1);
    let result = content.fn(...args);
    delete content.fn;
    return result;
}
let foo = {
    value: 1
}
function bar(name, age) {
    console.log(name)
    console.log(age)
    console.log(this.value);
}
bar.call2(foo, 'black', '18') // black 18 1
```

#### 4.2 <font color=#ff502c>Function.apply</font> 的模拟实现

***apply()*** 的实现和 ***call()*** 类似，只是参数形式不同。
```
Function.prototype.apply2 = function(context = window) {
    context.fn = this
    let result;
    // 判断是否有第二个参数
    if(arguments[1]) {
        result = context.fn(...arguments[1])
    } else {
        result = context.fn()
    }
    delete context.fn
    return result
}
```

### **<span id='bind'>5. 实现一个 <font color=#ff502c>Function.bind()</font></span>**
---

***bind()*** 方法:

会创建一个新函数。当这个新函数被调用时，bind() 的第一个参数将作为它运行时的 this，之后的一序列参数将会在传递的实参前传入作为它的参数。(来自于 MDN )


此外，***bind*** 实现需要考虑实例化后对原型链的影响。

```
Function.prototype.bind2 = function(content) {
    if(typeof this != "function") {
        throw Error("not a function")
    }
    // 若没问参数类型则从这开始写
    let fn = this;
    let args = [...arguments].slice(1);
    
    let resFn = function() {
        return fn.apply(this instanceof resFn ? this : content,args.concat(...arguments) )
    }
    function tmp() {}
    tmp.prototype = this.prototype;
    resFn.prototype = new tmp();
    
    return resFn;
}
```

### **<span id='extend'>6. 实现一个继承</span>**
---

**寄生组合式继承**

一般只建议写这种，因为其它方式的继承会在一次实例中调用两次父类的构造函数或有其它缺点。

核心实现是：**用一个 ***F*** 空的构造函数去取代执行了 ***Parent*** 这个构造函数。**

```
function Parent(name) {
    this.name = name;
}
Parent.prototype.sayName = function() {
    console.log('parent name:', this.name);
}
function Child(name, parentName) {
    Parent.call(this, parentName);  
    this.name = name;    
}
function create(proto) {
    function F(){}
    F.prototype = proto;
    return new F();
}
Child.prototype = create(Parent.prototype);
Child.prototype.sayName = function() {
    console.log('child name:', this.name);
}
Child.prototype.constructor = Child;

var parent = new Parent('father');
parent.sayName();    // parent name: father

var child = new Child('son', 'father');
```

### **<span id='kelihua'>7. 实现一个JS函数柯里化</span>**
---

**什么是柯里化？**

在计算机科学中，柯里化（Currying）是把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数且返回结果的新函数的技术。

**函数柯里化的主要作用和特点就是参数复用、提前返回和延迟执行。**

#### 7.1 通用版
```
function curry(fn, args) {
    var length = fn.length;
    var args = args || [];
    return function(){
        newArgs = args.concat(Array.prototype.slice.call(arguments));
        if (newArgs.length < length) {
            return curry.call(this,fn,newArgs);
        }else{
            return fn.apply(this,newArgs);
        }
    }
}

function multiFn(a, b, c) {
    return a * b * c;
}

var multi = curry(multiFn);

multi(2)(3)(4);
multi(2,3,4);
multi(2)(3,4);
multi(2,3)(4);
```

#### 7.2 ES6写法
```
const curry = (fn, arr = []) => (...args) => (
  arg => arg.length === fn.length
    ? fn(...arg)
    : curry(fn, arg)
)([...arr, ...args])

let curryTest=curry((a,b,c,d)=>a+b+c+d)
curryTest(1,2,3)(4) //返回10
curryTest(1,2)(4)(3) //返回10
curryTest(1,2)(3,4) //返回10
```
