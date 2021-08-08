---
title: javascript 原型链
date: 2021-08-02 06:51:46
tags:
---

## javascript 原型链与继承

### es5 构造函数

#### 继承（三种）

公有属性 私有属性 静态方法（静态属性）的继承

```javascript
    function Parent(){
        // 构造函数中的this，通过new调用则this指代的是实例
        this.name = 'parent'  // 私有属性
    }
    // 公有属性
    Parent.prototype.eat = function(){
        console.log('eat')
    }

    function Child(){
        this.age = 9 
        // 1. 仅继承私有属性 
        Parent.call(this) // 继承Parent类上的私有属性
    }

    Child.prototype.smoking = function(){
        console.log('吸烟')
    }
    //2. 仅继承公有属性
    // 1) Child.prototype.__proto__ = Parent.prototype 
    // Object.setPrototype(Child.prototype,Parent.prototype) 1的es6写法
    // 2) Child.prototype = Object.create(Parent.prototype，{constructor:{value:Child}})只继承公有属性

    // Object.create的实现
    function create(parentPrototype,props){
        function Fn(){}
        Fn.prototype = parentPrototype
        let fn = new Fn()
        for(let key in props){
            Object.defineProperty(fn,key,{
                ...props[key],
                enumerable:true
            })    
        }
        // 
        return fn
    }

    Child.prototype = Object.create(Parent.prototype)
    let child = new Child()
    child.eat() // 可以继承到

    // 错误方法1：实质是将子类原型替换为父类原型 再添加原型属性相当于在父类上添加原型属性
    // Child.prototype = Parent.prototype 
    // Child.prototype.a = 100 
    // let parent = new Parent()
    // console.log(parent.a) // 100


// 3. 继承公有属性和私有属性
Child.prototype = new Parent()  // 不会使用这种方式，既继承了父类的公有又有私有属性
    
```

<!-- http://www.javascriptpeixun.cn/course/3158/task/196220/show -->

### Class

```javascript
class Parent {
    constructor() {
        this.name = 'Parent';
        // return { a: 1 }  // console.log(child)  { a: 1, age: 9 }   并且执行child.eat()时会报错 child.eat is not a function
    }
    static b(){
        console.log('parent b')
    }
    eat(){
        console.log('parent eat')
    }
}
class Child extends Parent{  // 继承私有和公有 
    constructor(){
        super() // Parent.call(this)
        this.age = 9
    }
    static a(){
        console.log('child a')
    }
    smoking(){
        console.log()
    }

}

let child = new Child()
child.eat()  // 子类实例继承了父类的公有方法
console.log(child.name)  //  子类实例继承了父类实例的私有属性
Child.b()  // 子类继承了父类的静态方法
```

1. 类只能new 
2. 类可以继承公有私有和静态方法
3. 父类的构造函数中返回了一个引用类型会把这个引用类型作为子类的this 

、、、javascript
<!-- 类的编译 -->
```