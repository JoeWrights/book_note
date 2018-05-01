---
title: 深入学习JavaScript函数
tags: js
date: 
categories: JavaScript语言精粹
---


前言：
>函数对于任何一门语言来说都是核心的概念，通过函数可以封装任意多条语句，而且可以在任何地方、任何时候调用执行。而JavaScript中最好的特性就是它对函数的实现。它几乎无所不能。但是，函数在JavaScript中并非万能的，就像《JavaScript语言精粹》中所说：函数在JavaScript里也并非万能药。

## **一、函数对象**
在`JavaScript`中函数就是对象，对象是“名值”对的集合并拥有一个连到原型对象的隐藏连接。
我们都知道，**对象字面量产生的对象**连接到`Object.prototype`，例如：
```JavaScript
var obj={
    name:"JoeWright"
};
console.log(Object.prototype.isPrototypeOf(obj)); //true
```
而**函数对象**连接到`Function.prototype`（该原型对象本身连接到`Object.prototype`）。例如：
```JavaScript
var foo=new Function("a","b","return a+b");
console.log(foo(2,3)); //5
console.log(obj.constructor); //ƒ Object()
console.log(Function.prototype.isPrototypeOf(foo)); //true
//Function对象本身连接到Object.prototype
console.log(Object.prototype.isPrototypeOf(Function)); //true
```
一些属性：

- constructor：返回创建该对象的构造函数
- length：返回函数定义的参数个数
- caller(ECMA标准之外的属性)：返回调用当前函数的函数
- argument：返回该函数执行时内置的arguments对象

例如：
```JavaScript
function foo(a,b,c){
    return foo2.caller;
}
function bar(){
    return foo2();
}
console.log(foo.length); //3
console.log(foo.name); //foo
console.log(bar()); //f bar()
```
注意：`Function`构造函数虽然使你的脚本在运行时重新定义函数的情况下具有更大的灵活性，但它也会减慢代码的执行速度。为了避免减慢脚本速度，应该尽可能少使用`Function`构造函数，多使用`function`关键字的形式声明函数。

## **二、函数字面量**
函数字面量包括四部分：

- 第一部分是保留字`function`。
- 第二部分是函数名，它可以被省略（匿名函数）。函数可以用它的名字来递归调用自己。
- 第三部分是包围在圆括号中的一组参数（形参）。其中每个参数用逗号分隔。
- 第四部分是包围在花括号中的一组语句

格式如下：
```JavaScript
function functionName(parameters) {
  执行的代码
}
```
例如：
```JavaScript
function add(a,b) {
    return a+b;
}
```

## **三、函数的调用**
- 调用一个函数将**暂停**当前函数的执行，传递控制权和参数给新函数。
- 函数除了声明时定义的**形式参数**，每个函数接收两个附加的参数:`this`和`arguments`。参数this在面向对象编程中非常重要，**它的值取决于调用的模式**。
- 在`JavaScrip`t中一共有**四种**调用模式：<font style="color:#C69C52">**方法调用模式**、**函数调用模式**、**构造器调用模式**和**apply调用模式**。</font>这些调用模式在如何初始化关键参数this上存在差异。

上面讲到在JavaScript中一共有四种调用模式，在这里我们分别对这几种模式一一详解。
### **方法调用模式**
- 当一个函数被保存为对象的一个属性时，我们称它为一个**方法**
- <font style="color:#C69C52">当一个方法被调用时，this被绑定到该对象</font>（<font style="color:#00f">这句话对于我们理解this关键字非常重要！！！！</font>）
- 如果一个调用表达式包含一个**属性存取表达式**（即.表达式或者是[subscript]下标表达式，注意:**如果属性不确定，要用[]**）

下面是方法调用模式的一个例子：
```JavaScript
var myObj={
    name:"JoWright",
    value:0,
    increment:function(inc){
        this.value+=typeof inc ==="number"?inc:1;
    }
};
myObj.increment(); 
console.log(myObj.value); //1

myObj.increment(3)
console.log(myObj.value); //4

var key="name"; //将name属性赋给比变量key，此时属性名不确定
console.log(myObj.key); //undefined
console.log(myObj[key]); //JoeWright
```

`this`到对象的绑定 **发生在调用的时候**，这个绑定又称为 **“超级”迟绑定**（very late binding）。这样的绑定使得函数可以对`this` **高度复用**。通过this可取得它们所属对象的上下文的方法称为**公共方法**

### **函数调用模式**
- 当一个函数并非一个对象的属性时，那么它被当作一个函数调用：
```JavaScript
function add(a,b){
    return a+b;
}
console.log(add(1,2)); //3
```

- 当函数以此模式调用时，**this被绑定到全局对象**（浏览器下为Window，node下为 global）。

例如：
```JavaScript
var myObj={
    value:1,
    multiply:function(){
        var add=function(){
            return this.value*2;
        };
        console.log(add());
    }
};
myObj.multiply(); //NaN
```
为什么是NaN呢？因为add方法是一个匿名函数，此时add函数中的this绑定的是全局对象，不是myObj（实际上add方法是被Window对象调用的，所以它没有对myObj对象的访问权）

解决方法：利用 **闭包** 的特性（先用that变量来存储this）
```JavaScript
var myObj={
    value:1,
    multiply:function(){
        var that=this;
        var add=function(){
            return that.value*2;
        };
        console.log(add());
    }
};
myObj.multiply(); //2
```

### **构造器调用模式**
- `JavaScript`<font style="color:#C69C52">是一门基于原型继承的语言。这意味着对象可以直接从其他对象继承属性。</font>
- 如果在函数前面带上new关键字来调用（当作构造函数调用），那么将创建一个隐藏连接到改函数的prototype成员的新对象，**同时this 将会绑定到那个新对象上**。
例如：
```JavaScript
function Person(name){
    this.name=name;
}
//为Person的所有实例提供一个名为getName的公共方法
Person.prototype.getName=function(){
    return this.name;
}
var p=new Person("JoeWright");
console.log(p);//Person {name: "JoeWright"}
```
- 按照约定，构造器函数保存在以大写格式命名的变量里。如果调用构造器函数时没有在前面加上new，可能会发生非常糟糕的事情（即没有编译时的警告，也没有运行时的警告），所以 **大写约定非常重要**

### **Apply调用模式（类似的还有call、bind）**
格式如下：
```JavaScript
functionName.apply(thisArg[, [arg1, arg2, ...]])
```
- thisArg：将被绑定给this的值
- [arg1,arg2,..]：参数数组（可选）

例如：
```JavaScript
function add(a,b){
    return a+b;
}
var sum=add.apply(null,[3,4]);
console.log(sum); //7
```
```JavaScript
function Person(name){
    this.name=name;
} 
//为Person的所有实例提供一个名为getName的公共方法
Person.prototype.getName=function(){
    return this.name;
}
var Pet={
    name:"dogg"
};
//Pet并没有继承自Person.prototype,但我们可以在Pet上调用getName方法，尽管Pet并没有一个名为getName的方法。
var petName=Person.prototype.getName.apply(Pet);
console.log(petName); //dogg
```

## **四、参数**
当函数被调用时，会得到一个“免费”奉送的参数，这个参数就是`arguments`数组（`arguments`对象）。通过它函数可以访问所有它被调用时传递给它的参数列表，这使得编写一个无须指定参数个数的函数成为可能。

例如：
```JavaScript
//用arguments对象求最大值
function max(){
    var max=0;
    for(var i=0,len=arguments.length;i<len;i++){
        if(arguments[i]>max){
            max=arguments[i];
        }
    }
    return max;
}
console.log(max(2,1,4,3,7)); //7
```
### **arguments对象的callee属性:引用当前被调用的函数对象**
```JavaScript
function bar(){
    return arguments.callee;
 }
console.log(bar()); //ƒ bar()
```
典型应用：函数的递归调用
```JavaScript
//匿名函数的递归调用
(
    function(count){
        if(count<=3){
            console.log(count);
            arguments.callee(++count);
        }
    }
)(0);//0 1 2 3
```
### **默认参数**
实现方法一：
```JavaScript
function add(x,y){   
    x=x||0;
    y=y||1;
    return x+y;
}
console.log(add()); //1
console.log(add(3,4)); //7
```
实现方法二：
```JavaScript
function add(x,y){  
    x=x===undefined?0:x;
    y=y===undefined?1:y;
    return x+y;
}
console.log(add()); //1
console.log(add(3,4)); //7
```

## **五、返回**
- return语句可以使函数提前返回。当return被执行时，函数立即返回而不再执行余下的语句。
- 一个函数总是会返回一个值。如果没有指定值，就返回undefined
```JavaScript
function f(a,b){
    return a+b;
    return "hello JoeWright"; //永远不会被执行
}
console.log(f(1,2)); //3
```

## **六、异常**
JavaScript提供了一套异常处理机制。异常是干扰程序正常流程的非正常的事故。
例如：
```JavaScript
var add=function(a,b){
    if(typeof a!=="number"||typeof b!=="number"){
        throw{
            name:"TypeError",
            message:"add needs numbers"
        };
    }
    return a+b;
};
var try_it=function(){
 try{
    add("hello");
 }catch(e){
     console.log(e.name+":"+e.message);
    }
}
try_it(); //TypeError add needs numbers
```

## **七、给类型添加方法**
JavaScript允许给语言的基本类型添加方法。这样的方式对**函数（Function）**、**数组（Array）**、**字符串（String）**、**正则表达式（RegExp）** 和 **布尔值（Boolean）** 同样适用。
例如：`String`对象原来没有`reverse`方法，我们可以从它的原型上扩展该方法。
```JavaScript
String.prototype.reverse=function(){
    return Array.prototype.reverse.apply(this.split("")).join("");
}
console.log("hello".reverse()); //olleh
```

## **八、递归**
递归函数是直接或间接地调用自身的一种函数。
例如：
```JavaScript
function sum1(n){
    return n==1?1:f3(n-1)+n;
}
console.log(sum1(100)); //5050
//等价于
var sum2=function(n){ //匿名函数的递归
    return n==1?1:arguments.callee(n-1)+n;
}
console.log(sum2(100)); //5050
```

## **九、作用域**
在编程语言中，作用域控制着变量与参数的**可见性**及**生命周期**。这对开发者来说是一个重要的帮助，因为它**减少了名称冲突**，并且提供了自动内存管理
```JavaScript
var fun1=function(){
    var a=3,b=5;
    var fun2=function(){
        var b=7,c=11;
        console.log(a+" "+b+" "+c); //3 7 11
        a+=b+c;
        console.log(a+" "+b+" "+c); //21 7 11
    };
    console.log(a+" "+b); //3 5
    fun2();
    console.log(a+" "+b);// 21 5
};
fun1();
```
作用域的知识就写这么多，下次的文章深入学习:)

## **十、闭包**

各种专业文献上的"闭包"（closure）定义非常抽象，很难看懂。所以这里引用阮一峰老师的理解：
>闭包就是能够读取其他函数内部变量的函数，闭包让这些变量的值始终保持在内存中，不会在调用结束后，被垃圾回收机制（garbage collection）回收。

闭包的例子：
```JavaScript
function func1(){
    var n=5;
    function func2(){
        console.log(n);
    }
    return func2;
}
var res=func1();
res(); //5
```
上面的例子中，func2函数就是**闭包**。
由于在Javascript语言中，只有函数内部的子函数才能读取局部变量，因此可以把闭包简单理解成 **"定义在一个函数内部的函数"**。
所以，在本质上，**闭包就是将函数内部和函数外部连接起来的一座桥梁**。

## **十一、回调**
回调函数就是一个参数，将这个函数作为参数传到另一个函数里面，当那个函数执行完之后，再执行传进去的这个函数。这个过程就叫做回调。

看下面的例子：
```JavaScript
function addOne(a){
    return a+1;
}
function test(a,b,c,callback){
    var array=[];
    for(var i=0;i<3;i++){
        array[i]=callback(arguments[i]*2);
    }
    return array;
}
console.log(test(10,20,30,addOne)); //21,41,61
//匿名函数的写法（推荐）
console.log(test(10,20,30,function(a){return a+1}));//21,41,61
```

## **十二、模块**
模块就是**实现特定功能的一组方法**。
原始写法：
```JavaScript
function f1(){
    //...
}
function f2(){
    //...
}
```
上述函数f1、f2组成了一个模块，要使用时直接调用就可以了
缺点：**容易造成变量污染**

对象写法：
```JavaScript
var obj={
    name:"JoeWright",
    f1:function(){
       //...
    },
    f2:function(){
        //...
    }
};
```
上面的函数f1()和f2(），都封装在obj对象里。使用的时候，就是调用这个对象的属性obj.f1()。
缺点：暴露所有模块成员，内部状态可以被外部改写，例如：`obj.name="Tom"`;

完善：
```JavaScript
var obj={};
Object.defineProperties(obj,{
    name:{
        value:"JoeWright",
        writable:false //设置可写为false（默认为 false）
    },
    f1:function(){
        //...
    },
    f2:function(){
        //...
    }
});
obj.name="Joe";
console.log(obj.name); //JoeWright
```
立即执行函数写法：
```JavaScript
var obj=(function(){
    var value="JoeWright";
    var f1=function(){
        //...
    };
    var f2=function(){
        //...
    }
    return {
        f1:f1,
        f2:f2
    }; 
});
console.info(obj.value); //undefined
```
使用上面的写法，外部代码无法读取内部的value变量。

## **十三、套用**
函数也是值，从而我们可以用有趣的方式去操作函数值。**套用**允许我们将函数与传递给它的参数相结合去产生一个新函数。
例如：
```JavaScript
function sum1(n){
    return n==1?1:f3(n-1)+n;
}
console.log(sum1(100)); //5050
var sum2=sum1; //把sum1函数赋给变量sum2，实现套用
console.log(sum2(100)); //5050
```

#### **完结**
以上就是我对js函数的一些总结，不足的地方希望小伙伴们能够提出来，大家一起学习，一起进步:)