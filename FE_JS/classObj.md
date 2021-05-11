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
&nbsp;&nbsp;前面的类与对象是用一个构造器函数来构建的，而每个函数都有一个prototype属性,而这个prototype就是这个类对其原型的引用，可以对原型添加属性和函数，则用这个创建器创建出来的所有对象都会带有添加的属性和函数。
```
            function Person() {

            }
            Person.prototype.name = "xiaoming";
            Person.prototype.say = function() {
                console.log("my name is " + this.name);
            };

            var p = new Person();
            console.log("name=" + p.name);
            p.say();    
```
上面的代码创建了一个Person的创建器函数，并且对类Person的原型添加了一个name属性和一个say函数，在say函数中打印name，最后创建了一个Person的对象p，打印p的name并且调用p的say函数
代码执行结果是：
```
name=xiaoming
my name is xiaobing
```
&nbsp;&nbsp;既然类可以访问到其原型，那么对象如果访问到其原型呢？每个对象有一个__proto__属性，这个__proto__跟其类的prototype的指向是同一个对象。
```
            function Person() {

            }
            var p = new Person();
            console.log(p.__proto__ === Person.prototype)        
```
还是一个Person类，创建一个p对象，打印p的__proto__属性和Person的prototype是否为同一个内存地址，运行结果是true
&nbsp;&nbsp;既然一个类的原型是一个对象，而对象也是有原型的，那么就就会有原型原型了，也就是一个类有父类，父类又会有其父类，也是祖先类，因此原型就形成了一条链，这条原型链就实现了类的祖先继承关系

# 继承的实现方式
## 基于原型链的实现
&nbsp;&nbsp;基于原型链的实现就是将子类函数函数的prototype指向父类的一个对象，代码如下：
```
function Parent(name, age) {
    this.name = name;
    this.age = age;
}
Parent.prototype.say = function() {
    console.log(this.name + " say: my age is " + this.age);
}

function Child() {

}

Child.prototype = new Parent("xiaoming", 20);

let c = new Child();
c.say();
```
执行的打印结果是：xiaoming say: my age is 20
&nbsp;&nbsp;这种继承方式的缺陷很明显，我们无法使用Child的函数器向Parent传参

## 经典继承实现
&nbsp;&nbsp;既然使用原型链无法在Child的构造器中无法向Parent传参，那么我们就在Child的构造器中调用Parent的构造器，就可以实现把Child的构造器参数传给Parent的构造器了。代码如下：
```
function Parent(name, age) {
    this.name = name;
    this.age = age;
}
Parent.prototype.say = function() {
    console.log(this.name + " say: my age is " + this.age);
}
function Child(name, age) {
    Parent.call(this, name, age);
}

let c = new Child("xiaohong", 1);
c.say();
```
执行的打印结果是：xiaohong say: my age is 1
&nbsp;&nbsp;上面的代码在Child的构造器中调用了Parent.call(this, name ,age)，call函数是每个函数共用的一个函数，作用是把函数调用的this指向call的第一个参数，因此Parent函数中的this就变成了Child构造器中的this，也就是Child对象了，因此可以实现把Child构造器的参数传给Parent的构造器。但这样实现的缺陷也比较明显，就是我们构造子类对象时，把父类的函数都重新构造了一遍，这样构造的每个子类的say函数都不一样

## 组合原型链继承和经典继承
&nbsp;&nbsp;组合模式的实现原理是通过改变子类的原型并且利用经典继承的在子类构造器中调用父类构造器的方式，将子类构造器的参数传给父类构造器。代码如下：
```
function Parent(name, age) {
    this.name = name;
    this.age = age;
}
Parent.prototype.say = function() {
    console.log(this.name + " say: my age is " + this.age);
}
function Child(name, age) {
    Parent.call(this, name, age);
}
Child.prototype = new Parent();
console.log(Child.prototype.constructor);
Child.prototype.constructor = Child;
let c = new Child("xiaohong", 1);
c.say();
```
执行的打印结果是
```
[Function: Parent]
xiaohong say: my age is 1
```
这里的两个核心操作是
Child.prototype = new Parent();
Child.prototype.constructor = Child;
组合方式之所以解决了经典继承的缺陷，是因为Child的prototype已经指向了Parent对象，不会重复为Child创建Parent对象，因此Parent的say函数也只会被创建一个，这样所有的Child对象都会共享同一个Parent原型对象，因此也都会共享同一个say函数。
&nbsp;&nbsp;但,看上面的代码会发现Parent的构造器函数被调用了两次，一次是在设置Child的原型的时候，Child的原型是一个Parent对象，这个Parent对象在创建的时候没有传参数，但也调用了一次Parent的构造器，第二次是在Child的构造器中，也就是说如果要创建n个Child对象，就会调用n+1次Parent的构造器

## 寄生组合继承
&nbsp;&nbsp;所谓寄生，就是把一个对象的创建的过程封装到一个函数中，这个函数传入结果对象的原型，这个函数的操作就是在原型的基础上进行增强，增强的结果就是目标对象，跟装饰模式差不多的思路。寄生组合继承代码如下：
```
function Child(name, age) {
    Parent.call(this, name, age);
}

function object(o) {
    let F = function() {};
    F.prototype = o;
    return new F();
}

function prototype(parent, child) {
    let prototype = object(parent.prototype);
    prototype.constructor = child;
    child.prototype = prototype;
}

prototype(Parent, Child);
let c = new Child("xiaohong", 1);
c.say();
```
prototype函数中是组合的使用，而object就是寄生的使用，使用了一个F函数来做原型链的中转，避免了直接调用Parent的构造器，而是调用F函数，这样既能把Child和Parent的原型连起来，又避免了在建立原型链时直接创建Parent的对象，其实从根本上来看继承关系已经变成了Child继承F，F继承Parent，通过多创建一个F对象来避免多创建一个Parent。
