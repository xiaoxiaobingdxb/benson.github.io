# 变量、函数、作用域

## 变量
&nbsp;&nbsp;JS的原生变量类型也具有string、number、boolean、null等，但JS跟Python更类似是具有动态语言特性的(相较于Java和c/c++)，比如JS的一个类型可重复赋值成不同类型的值，而Java和c/c++如果重复赋值不同类型的值就会报类型错误的编译时异常。

JS原生使用var来定义变量
```
        var a = 5;
        console.log(typeof(a));
        var a = "string";
        console.log(typeof(a));
```
输出number和string
这里a被声明并重复赋值成了两种不同类型的值，并且正确输出了其值的类型，为了防止变量的重复声明，JS推荐使用let来声明变量，更进一步为了防止重复赋值，JS推荐使用const来声明一个常量
var：声明一个变量可重复声明重复赋值
let：声明一个变量，不可重复声明，可重复赋值
const：声明一个常量，不可重复声明，不可重复赋值

而如果使用let来声明一个变量如下代码
```
        let b = 3;
        b = "5";
        let b = 7;
```
则console会报异常：Uncaught SyntaxError: Identifier 'b' has already been declared
而如果对一个const的变量重新赋值，则会报异常：Uncaught TypeError: Assignment to constant variable.
    at varFun.html:19
```
        const c = 10;
        c = "16";
```


## 函数
&nbsp;&nbsp;JS的函数以function开头，并且JS的函数是一等成员，也就是说JS的函数是可以当作一个变量的，只不过这个变量的类型是函数
下面的代码演示了如何将一个函数赋值给另一个变量，执行两个函数变量，并且打印两个函数的类型
```
        function test() {
            console.log("function test");
        }

        let copyTest = test;

        test();
        copyTest();
        console.log(typeof(test));
        console.log(typeof(copyTest));
```

打印结果是：
```
function test
function test
function
function
```

既然函数可以被当作一个变量，那么一个函数也可以被当作返回值给return了。
```
function superFun() {
            console.log("before create a inner function");
            let f = function() {
                console.log("inner function");
            }
            console.log("after create a inner function");
            return f;
        }
        superFun()();
```
上面的代码执行结果是
```
before create a inner function
after create a inner function
inner function
```
表示其实是先创建了内部函数f的，调用方式是superFun()()，这里使用了两个()，可以拆分为
```
let returnF = superFun();
returnF();
```
也就是superFun()调用返回的也是一个函数，再调用返回的函数。


## 作用域
&nbsp;&nbsp;其他语言例如Java和c/c++或者Go的作用域都是类似的，基本就是可以称作为块级作用域，而对于对象，也是有对象作用域的。JS的块级作用域有点奇葩，一个变量(包含函数类型的变量)可以会从它所在的块中逃逸出来，这可能是JS的词法解析词的设计导致的。
例如如下代码
```
        for (var a = 5; a < 10; a++) {
            console.log("in a=" + a);
        }
        console.log("out a=" + a);
```
如果按照其他语言的正确执行，会报一个a变量找不到的编译时异常，但在JS环境下这段代码是能够正常执行的，执行结果是：
```
in a=5
in a=6
in a=7
in a=8
in a=9
out a=10
```
这里的变量a所在的块作用域是for循环，但在JS环境下，a却从块中逃逸出来了。前面变量声明中所说到的let可以解决变量从块作用域中逃逸的问题，这也许是JS对其他语言的编程习惯的一种妥协吧。


