# 类、对象与对象的作用域

## 类
&nbsp;&nbsp;原生JS或者说ES6以下的JS语法是不支持直接声明一个类型的，而是通过创建一个对象的构造器来声明一个类的，而这个构造器可以简单地理解成返回一个对象的函数。因此，可以说JS是不支持声明一个类的，JS只能创建对象。
```
        function Animal(name) {
            return {
                name: name,
                live: function() {
                    console.log(name + " live in the world");
                }
            };
        }
        let a = Animal("low animal");
        console.log("a type is " + typeof(a));
        a.live();
```
上面代码创建了一个Animal的类，或者说是创建了一个创建Animal类型对象的构造器，这个类有name属性和live函数(在Java中对象的函数叫作方法)。接着使用Animal构造器创建了一a对象，给name赋值成"low animal"，打印出a对象的类型，最后调用对象的live函数。
执行结果为：
```
a type is object
low animal live in the world
```
## 对象
不论是Java和c/c++还是JS，对象并没有那么神秘，从内存的角度来看，对象其实就是一块内存而已，这块内存储存了变量的值以及对象的函数的指针，这些函数在执行时会被切换到当前对象的上下文。而所谓的上下文，其实就是在执行代码的过程中，访问的变量是属于这个对象的。JS中的对象可以表现为一个map，map的key是变量或者函数的名字，value就是变量的值或函数的指针，例如前面提到的创建一个Animal对象的时候，代码是
```
        {
                name: name,
                live: function() {
                    console.log(name + " live in the world");
                }
        }
```

在live函数中访问的name会先从当前对象的使用域中查找，如果查找不到才会一层一层地往外查找。

## 对象的使用域
&nbsp;&nbsp;前面我们直接声明一些变量或者函数时，没有指定它们的对象是谁，而且我们在访问这些变量或者调用这些函数的时候也没有指定对象，那么这些变量或函数是怎么找到的呢？其实这些直接声明的变量或函数被称为顶层作用域对象，它们所属的对象是window对象，JS环境默认就提供了一些全局对象，而访问window对象的成员，是不需要加window前缀的。
```
        var a = 5;
        console.log("a=" + a);
        console.log("window.a=" + window.a);
```
上面代码的执行结果是：
```
a=5
window.a=5
```
但当我们把声明a的var改为let的时候，window.a却变成了undefined，这说明let还可以改变顶层对象的所属对象。

对于函数，前面也提到了对象的函数是可以直接访问到其对象的属性的，也就是上下文的概念，那么要是我们在对象外部声明了一个函数，但有时我们却需要将这个函数变成对象的函数。
```
        var a = 5;
        function test() {
            console.log("a=" + this.a);
        }
        function TestObject(a) {
            return {
                a: a
            };
        }
        test();
        var obj = TestObject(6);
        console.log("obj.a=" + obj.a);
        obj.test = test;
        obj.test();
        var bindTest = test.bind(obj);
        bindTest();
```
上面的代码演示了创建了一个a=6的对象，并且将test函数赋值给对象的test变量。执行结果是:
```
a=5
obj.a=6
a=6
a=6
```
这表示直接将test函数赋值给对象的test是能够改变函数的上下文的，而使用函数的bind函数也可以，其原理是将函数test创建了另一个新函数，这个新函数的上下文就是我们传入的obj对象。

## 原型链
&nbsp;&nbsp;JS的原型链对于其他语言转过来的Coder显得有点玄乎，但其实了解过Go语言就很好理解原型链了。JS对象的继承方式跟Go很像，原理上都是基于代理模式，也就是一个类要去继承另一个类，则需要持有父类，虽然语法上没有显式地使用代理。<br/>
&nbsp;&nbsp;前面的类与对象是用一个构造器函数来构建的，而每个函数都有一个prototype属性