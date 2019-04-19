# prototype
> 构造函数都有一个prototype属性指向其原型对象；同时实例也有一个__proto__属性指向构造函数的prototype，将公共方法封装到原型上
```js
function Person(name) {
    this.name = name
}
Person.prototype.say = function() {
    console.log(`person say ${this.name}`)
}
function Man(name) {
    Person.call(this)
}
Man.prototype = Object.create(Person.prototype)
```
> prototype，子类继承父类，先创建子类的this对象，然后在子类的构造函数中对子类的this进行扩展，同时更改子类的prototype的指向

# class 
```js
class Person {
    constructor(name) {
        this.name = name
        this.number = 3
    }
    say() {
        console.log(`person say ${this.name}`)
    }
    s() {
        console.log(this.number)
    }
}
class People extends Person {
    constructor(name) {
        console.log(this) // 报错
        super(name)
        this.number = 2
    }
    test() {
        super.s() // 2
    }
}
console.log(new People('rose'))
```
> 在class的继承中。子类自己的this对象必须通过父类的构造函数完成塑造，其this是继承父类来进行扩展的（所以在构造器中，必须要在super后才能用this）
> 在子类的继承中，虽然通过super调用的是父类的构造器，但是返回的是子类的实例，即super内部的this指向子类的实例
> super作为函数在构造器中调用时，实际上是Parent.prototype.constructor.call(this)
> super作为对象使用时，super表示父类的原型对象（Parent.prototype），可以通过super.xxx()来进行调用，调用的函数内部的this指向的是当前子类

# class中的__proto__
```js
class A {

}
class B extends A {
    constructor() {
        super()
    }
}
console.log(B.__proto__ === A) // true
console.log(B.prototype.__proto === A.prototype) // true
```
> 子类的__proto__属性指向父类，子类原型对象的__proto__属性指向父类的原型
```js
// 实现原理
class A {
}
class B {
}
Object.setPrototype(B.prototype, A.prototype)
Object.setPrototype(B, A)
Object.setPrototype = function(a, b) {
    a.__proto__ = b
    return a
}
```

# class实例中的__proto__
```js
class Person {
    constructor(name) {
        this.name = name
        this.number = 3
    }
}
class People extends Person {
    constructor(name) {
        super(name)
        this.number = 2
    }
}
console.log(People.__proto__ === Person)   // true
console.log(People.prototype.__proto__ === Person.prototype) // true
var p = new People('rose')
// 实例p的__proto__属性指向其类的原型
console.log(p.__proto__ === People.prototype) // true
console.log(p.__proto__.__proto__ === Person.prototype) // true
```