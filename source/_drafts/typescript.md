---
title: typescript
date: 2020-05-03 11:21:40
categories:
tags: typescript
---

typescript 语法入门

## 基础类型

### 布尔值
```typescript
  let flag : boolean = false;
```
### 数字
TypeScript里的所有数字都是浮点数。 这些浮点数的类型是 number。 除了支持十进制和十六进制字面量，TypeScript还支持ECMAScript 2015中引入的二进制和八进制字面量。
```typescript
let num : number = 123;
```

### 字符串
1 . 和JavaScript一样，可以使用双引号（ "）或单引号（'）表示字符串。
```typescript
let firstName : string  = 'huang';
```
2 . 使用模版字符串，它可以定义多行文本和内嵌表达式。 这种字符串是被反引号包围 ``，并且以${ expr }这种形式嵌入表达式
```typescript
	let age : number = 18;
	let firstName : string  = 'huang';
	let lineBreak : string = `Hello, my name is ${ firstName }.
	I'll be ${ age + 1 } years old next month.`;
```

## 数组类型
ts中数组会区分类型
```typescript
//  1 字符串类型的数组  两种写法
let pets1: string[] = ["旺财", "小白"]
let pets2: Array<string> = ["旺财", "小白"];
//  2 对象类型的数组
let pets3: Array<object> = [{ name: "旺财" }, { name: "小白" }];
let pets4: object[] = [{ name: "旺财" }, { name: "小白" }];
//  3 数组中，随意放string、number、boolean类型 这里的 | 相当于 或 的意思 
let arr1: Array<string | number | boolean> = ["hello swr", 28];
//  4 想在数组中放任意类型
let arr2: Array<any> = ["hello xxx", 208, true]
let arr4: any[] = ["hello xxx", 208, true]
```

## 元祖类型tuple
元组类型必须一一对应上，多了少了或者类型不对都会报错。
元组类型是一个不可变的数组，长度、类型是不可变的。
```typescript
let personT: [string, number] = ['某某某', 28]
// 有bug 增加不会报错，直接访问增加的元素会报错
personT.push('1245789')
console.log(personT)  //  ['某某某', 28,'1245789]
console.log(personT[2])  // 报错
```

## 枚举类型
可以定义一些值，定义完了后可以直接拿来用了，用的时候也不会赋错值。并且不能再给枚举类型修改值
```typescript
// 实际项目中可以用来枚举订单状态等 
enum orderStatus {
    WAIT_FOR_PAY = "待支付",
    UNDELIVERED = "完成支付，待发货",
    DELIVERED = "已发货",
    COMPLETED = "已确认收货"
}
```

默认的枚举值是从0开始，如上述代码，red=0，green=1依次类推;
```typescript
enum Color {
  green,
  red,
  blue
}
console.log(Color[0]) // green
console.log(Color[green]) // 1
```
也可以修改默认的初始值，此时red=3, blue=4依次类推，但是red前面的枚举值是不会修改的

```typescript
enum Color1 {
  green ,
  red= 3,
  blue,
  orange,
}
console.log(Color1[0]) // green
console.log(Color1[1]) // undefined
console.log(Color1[2]) // undefined
console.log(Color1[4]) // blue
```

## 接口类型
若未定义接口声明的字段（非可选），则检查会抛出错误，传参中有接口未定义项也会报错
接口索引的key类型只能number或者string
```typescript
/* 函数或者方法实现接口的情况 */
interface Person {
  firstName: string;
  lastName: string;
  hostName?: string;
}
function greeter(person: Person) {
  return "Hello, " + person.firstName + " " + person.lastName 
}
console.log(greeter({
  firstName: 'h',
  lastName: 'ry'
}))

/* 类实现接口的情况 */
interface Point {
  x: number;
  y: number;
  z: number;
}
class myPoint implements Point {
  x:number = 20
  y:number = 30
  z:number = 40
}
```

## any
any在TypeScript中是一个比较特殊的类型，声明为any类型的变量相当于放弃了类型检查 任何值都可以。对于any类型的变量，可以将其赋予任何类型的值：
```typescript
  var power: any;
  power = '123';
  power = 123;

  // btn 有可能获取的到也有可能获取不到是null类型
  let btn:any = document.getElementById('btn')
  btn.style.color = "blue"
  // 当然也有粗暴一些的方式，利用 ! 强制断言
  let btn1 = document.getElementById("btn")
  btn1!.style!.color = "blue"
```
any对于JS代码的迁移是十分友好的，在已经成型的TypeScript项目中，我们要慎用any类型，当你设置为any时，意味着告诉编辑器不要对它进行任何检查。

## null和undefined
null和undefined作为TypeScript的特殊类型，可以赋值给任意类型的变量：
```typescript
  var num: number;
  var str: string;

  // 赋值给任意类型的变量都是合法的
  num = null;
  str = undefined;
```

## void
void 类型一般是定义函数没有返回值。不 return  或者return undeifned ,其他类型的返回值会报错
```typescript
function say(name: string): void {
    console.log("hello1", name)  // 不return任何值没有问题
    return undefined       // return  undefined 也可以 
}
// say('xiaoming')  
```

## never
never类型 永远不会有返回值 （很少用，用于抛出异常）
申明never类型的变量只能被never类型赋值(包括null和undefined)
```typescript
  let x:null ;
  x = null
  let xx: never
  function error(message: string): never {
      throw new Error(message)
  }
  error("error")
```

也就是说，当一个函数执行的时候，被抛出异常打断了，导致没有返回值或者该函数是一个死循环，永远没有返回值，这样叫做永远不会有返回值。
实际开发中，是never和联合类型来一起用，比如
```typescript
  function say5(): (never | string) {
      return "ok"
  }
  say5()
```

## readonly
readonly是只读属性的修饰符，当我们的属性是只读时，可以用该修饰符加以约束，在类中，用readonly修饰的属性仅可以在构造函数中初始化：
```typescript
  class Foo {
    readonly bar = 1; // OK
    readonly baz: string;
    constructor() {
        this.baz = "hello"; // OK
    }
  }
```

一个实用场景是在react中，props和state都是只读的：
```typescript
interface Props {
    readonly foo: number;
}
interface State {
    readonly bar: number;
}
export class Something extends React.Component<Props,State> {
  someMethod() {
    this.props.foo = 123; // ERROR: (props are immutable)
    this.state.baz = 456; // ERROR: (one should use this.setState)  
  }
}
```
当然，React本身在类的声明时会对传入的props和state做一层ReadOnly的包裹，因此无论我们是否在外面显式声明，赋值给props和state的行为都是会报错的。

readonly和const区别：
1. readonly是修饰属性的
2. const是声明变量的


## 函数类型
函数本身有参数类型和返回值类型，都可进行声明。
当函数没有返回值时，可以用void来表示。
当一个函数永远不会返回时，我们可以声明返回值类型为never
1 . 函数声明法
```typescript
  function run():string{
      return 'run';
  }
```
2 . 变量赋值法/匿名函数
```typescript
  var fun2=function():number{
    return 123;
  } 
```
3 . 形参和实参要完全一样
```typescript
  function say7(name: string): void {
    console.log("hello", name)
  }
  say7("某某某")
```
4 . 可选参数
形参和实参要完全一样，如想不一样，则需要配置可选参数，可选参数用 ？处理，只能放在后面
```typescript
  function say8(name: string, age?: number): void {
      console.log("hello", name, age)
  }
  say8("某某某")
```
5 . 设置默认参数
```typescript
  function ajax(url: string, method: string = "GET") {
      console.log(url, method)
  }
```
6 . 设置剩余参数 利用扩展运算符
```typescript
  function sum(a: number, ...args: Array<number>): number {
      console.log(args)  // [2,3,4,5]
      return eval(args.join("+"))
  }
  let total:number = sum(1,2,3,4,5)
```
7 . 函数的重载 就是判断参数类型执行不同的逻辑   同名函数的问题，js中不存在函数的重载
```typescript
  function pickCard(x: any): any {
      if (typeof x == "object") {
          console.log(1)
      } else if (typeof x == 'number') {
          console.log(2)
      }
  }
```

## 类
类内的成员有：字段、存取器、构造函数、方法。
1 . 存取器
```typescript
  class Person {
    private _age: number;
    get age(): number {
        return this._age;
    }
    set age(newName: number) {
        if(newName > 150){
            this._age = 150;
        }
        this._age = newAge;
    }
}
// 这里的get/set方法，就是_age变量的存取器。首先我们为_age变量添加一个private修饰符。表示它是个私有变量，禁止外部对它的访问。
const person = new Person();
person._age // 这句会报错，TS不允许我们读取私有变量。
// 要想使用_age，需要利用存取器：
const person = new Person();
person.age = 15; // 使用set age方法
const age = pereson.age; // 使用get age方法
// 使用存取器的好处是对变量的访问和设置，有了控制。原则上类内部的变量，外部是不能访问的，这就是封装性。要想访问，只能通过存取器方法。
```
2 . 构造函数
constructor就是类的构造器，通过 new 命令创建对象实例时，自动调用该方法 返回实例对象 this ，
但是也可以指定 constructor 方法返回一个全新的对象，让返回的实例对象不是该类的实例
```typescript
  class Person {
      private _age: number;
      constructor(age: number) {
          this._age = age;
      }
  }
const person = new Person(15);
```
ES6 要求，子类的构造函数必须执行一次 super 函数，否则会报错。 super 这个关键字，既可以当做函数使用，也可以当做对象使用。
这两种情况下，它的用法完全不用。
```typescript
  class A {}
  class B extends A {
  constructor() {
      super();  // ES6 要求，子类的构造函数必须执行一次 super 函数，否则会报错。
  }
```
3 . 定义一个类
```typescript
class Person1 {
  name: string
  age: number
  constructor(name: string, age: number) {
    // this.name和this.age必须在前面先声明好类型
    // name:string   age:number
    this.name = name,
    this.age = age
  }
  say() {
      console.log('hello' + name)
  }
}
```
4 . 类的继承
```typescript
class PersonX {
  name: string
  age: number
  constructor(name: string, age: number) {
      this.name = name
      this.age = age
  }
  // 原型方法
  say(): string {
      return "hello"
  }
}
class Child1 extends PersonX {
  childName: string
  constructor(name: string, age: number, childName: string) {
    super(name, age)
    this.childName = childName
  }
  childSay(): string {
    return this.childName
  }
}
let child = new Child1('某某某1',28,'bb')
console.log(child.say()) //'hello'
console.log(child.childSay()) // '某某某1'
```
5 . 类的修饰符
public公开的，可以供自己、子类以及其它类访问
protected受保护的，可以供自己、子类访问，但是其他就访问不了
private私有的，只有自己访问，而子类、其他都访问不了 
6 . 抽象类 使用abstract 关键字
```typescript
  abstract class Animal {
    // 实际上是使用了public修饰符
    // 如果添加private修饰符则会报错
    abstract eat(): void;
  }
  // 需要注意的是，这个Animal类是不能实例化的
  let animal = new Animal() // 报错

  // 抽象类的抽象方法，意思就是，需要在继承这个抽象类的子类中实现这个抽象方法，不然会报错
  // Person6 会报错，因为在子类Person6中没有实现eat抽象方法
  class Person6 extends Animal {
      eat1() {
          console.log("吃米饭")
      }
  }
  // 不会报错,因为Dog类中实现了抽象方法eat
  class Dog extends Animal {
      eat() {
          console.log("吃骨头")
      }
  }
```

## 接口
接口主要是一种规范，规范对像，函数，数组，类必须遵守规范，
需要使用到关键字interface
1 . 接口规范对象 
可以让多个方法使用这个规范 
我们通过这样的方式，规范必须传name和age的值  多个接口都规范必须传name和age的值呢，每个方法都写就很重复
```typescript
  interface infoInterface {
    name: string,
    age: number,
    city?: string // 该参数为可选参数
  }
  function getUserInfo(user: infoInterface) {
    console.log(`${user.name} ${user.age}`)
  }
```
2 . 接口规范函数
对一个函数的参数和返回值进行规范
```typescript
  interface mytotal {
    // 左侧是函数的参数，右侧是函数的返回类型
    (a: number, b: number): number
  }
  let total: mytotal = function (a: number, b: number): number {
    return a + b
  }
  console.log(total(10,20))
```
3 . 接口规范数组
```typescript
  interface userInterface {
    // index为数组的索引，类型是number
    // 右边是数组里为字符串的数组成员
    [index: number]: string
  }
  let arr: userInterface = ['规范数组', 'hello'];
```
4 . 接口规范类
```typescript
// 首先实现一个接口
interface Animal1 {
    // 这个类必须有name
    name: string,
    // 这个类必须有eat方法
    // 规定eat方法的参数类型以及返回值类型
    eat(any: string): void
}
// 新增一个接口
interface Animal2 {
    sleep(): void
}
// 关键字 implements 实现
// 因为接口是抽象的，需要通过子类去实现它
// 可以在implements后面通过逗号添加，实现遵循多个接口，
// 一个类只能继承一个父类，但是却能遵循多个接口
class Person9 implements Animal1, Animal2 {
    name: string
    constructor(name: string) {
        this.name = name
    }
    eat(any: string): void {
        console.log(`吃${any}`)
    }
    sleep() {
        console.log('睡觉')
    }
}
```
5 . 接口可以继承接口 就和类继承类一样的
```typescript
interface Animal3 {
    name: string,
    eat(any: string): void
}
// 像类一样，通过extends继承
interface Animal4 extends Animal3 {
    sleep(): void
}
// 因为Animal4类继承了Animal3
// 所以这里遵循Animal4就相当于把Animal3也继承了
class Person10 implements Animal4 {
    name: string
    constructor(name: string) {
        this.name = name
    }
    eat(any: string): void {
        console.log(`吃${any}`)
    }
    sleep() {
        console.log('睡觉')
    }
}
```
**有了接口为什么还需要抽象类？**
  接口里面只能放定义，抽象类里面可以放普通类、普通类的方法、定义抽象的东西。
  比如抽象类中有10个方法, 其中九个是实现过的方法, 只有一个是抽象的方法, 那么子类继承过来

## 泛型
在一般化的场景，我们的类型可能并不固定已知，它和any有点像，只不过我们希望在any的基础上约束参数和返回值要保持一致性
1 . 类的泛型
使用 < > 跟在类名后面
```typescript
class MyMath<T>{
    // 定义一个私有属性
    private arr: T[] = []
    // 规定传参类型
    add(value: T) {
        this.arr.push(value)
    }
    // 规定返回值的类型
    max():T{
      return Math.max.apply(null,this.arr)
    }
}
const class123 = new MyMath<string>('hell')//实例化类，指定类的类型是string
```
2 . 函数的泛型
```typescript
function reverse<T>(items: T[]): T[] {
    var toreturn = [];
    for (let i = items.length - 1; i >= 0; i--) {
        toreturn.push(items[i]);
    }
    return toreturn;
}
var reversed = reverse<number>([1, 2, 3]);// 调用方法一
var reversed = reverse([1, 2, 3]);        //调用方法二
```
3 . 接口的泛型
**写法一：**
```typescript 
 // 规范函数的接口的泛型
interface fnInt {
    <T> (value:T) : any
}

let newFn : fnInt = <T>(value : T) : any => {
    return `打印value:${value}`
}
console.log(newFn<string>('119'))
console.log(newFn<number>(120))
```
**写法二：**
```typescript 
interface ConfigFnTwo<T>{
    (value:T):T;
}
function setDataTwo<T>(value:T):T{
    return value
}
var setDataTwoFn:ConfigFnTwo<string> = setDataTwo
setDataTwoFn('name');
```

## 联合类型
在JS中，一个变量的类型可能是几种类型之一，使用一个|分割符来分割多种类型，这种复合类型，称之为联合类型：
```typescript
function formatCommandline(command: string[]|string) {
    var line = '';
    if (typeof command === 'string') {
        line = command.trim();
    } else {
        line = command.join(' ').trim();
    }
}
```
当 TypeScript 不确定一个联合类型的变量到底是哪个类型的时候，我们只能访问此联合类型的所有类型里共有的属性或方法：
```typescript
function f11(name : string, age : string | number) {
     console.log(age.length)//报错
 }
f11('ljy', '21')
// 报错,因为length 不是 string 和 number 的共有属性，所以会报错。所以只能访问类型的共有的属性或方法
```
## 交叉类型
联合类型的语义类似于或者，交叉类型的语义类似于集合中的并集
```typescript
interface DogInterface {
  run(): void
}
interface CatInterface {
  jump(): void
}
let pet: DogInterface & CatInterface = {
  run(){
  },
  jump(){
  }
}    
```

## 类型断言
类型断言可以用来手动指定一个值的类型。
类型断言不是类型转换，断言成一个联合类型中不存在的类型是不允许的：
类型断言有两种写法：
```typescript
  function assert(type:number|string):void{
    if(type as number){  // 第一种写法
      console.log(type)
    }
  } 
```
```typescript
  function assert(type:number|string):void{
    if(<number>type){  // 第二种写法
      console.log(type)
    }
  } 
```

## 命名空间
```typescript
  namespace Utility {
    export function log(msg) {
      console.log(msg);
    }
    export function error(msg) {
      console.log(msg);
    }
  }
  // usage
  Utility.log('Call me');
  Utility.error('maybe');
```
待完成
ts声明文件 xxx.d.ts
global.d.ts 与 lib.d.ts 以及自定义*.d.ts
命名空间 声明空间
@types 环境声明
tsconfig.json 配置项
react 项目中配置ts（即支持jsx）
https://juejin.im/post/5d8efeace51d45782b0c1bd6#heading-7 ts场景，优缺点，vscode插件安利
ts社区 https://github.com/DefinitelyTyped/DefinitelyTyped


《TypeScript Deep Dive》 的中文翻译版  https://jkchao.github.io/typescript-book-chinese/typings/types.html#%E4%BD%BF%E7%94%A8-types
global.d.ts 是一种扩充 lib.d.ts 很好的方式，如果你需要。
当你从 JS 迁移到 TS 时，定义 declare module "some-library-you-dont-care-to-get-defs-for" 能让你快速开始。

**typescript.json配置**
```typescript
{
  "compilerOptions": {
    /* Basic Options *//* 基本选项 */
    // "incremental": true,                   /* Enable incremental compilation */
    "target": "es5",    // 指定 ECMAScript 目标版本: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017', or 'ESNEXT'
    "module": "commonjs",    // 指定使用模块: 'commonjs', 'amd', 'system', 'umd' or 'es2015'
    // "lib": [],                            // 指定要包含在编译中的库文件
    "lib": ["es6", "dom"],                            // 指定要包含在编译中的库文件
    // "allowJs": true,                      // 允许编译 javascript 文件
    // "checkJs": true,                      // 报告 javascript 文件中的错误
    // "jsx": "preserve",                    // 指定 jsx 代码的生成: 'preserve', 'react-native', or 'react'
    // "declaration": true,                  // 生成相应的 '.d.ts' 文件
    // "declarationMap": true,               /* Generates a sourcemap for each corresponding '.d.ts' file. */
    // "sourceMap": true,                    // 生成相应的 '.map' 文件
    // "outFile": "./",                      // 将输出文件合并为一个文件
    // "outDir": "./ts/dist/",               // 指定输出目录
    "outDir": "../tsCompile/", /* Redirect output structure to the directory. */
    // "rootDir": "./",                      // 用来控制输出目录结构 --outDir.
    // "composite": true,                     /* Enable project compilation */
    // "tsBuildInfoFile": "./",               /* Specify file to store incremental compilation information */
    "removeComments": true,                  // 删除编译后的所有的注释
    // "noEmit": true,                       // 不生成输出文件
    // "importHelpers": true,                // 从 tslib 导入辅助工具函数
    // "downlevelIteration": true,            /* Provide full support for iterables in 'for-of', spread, and destructuring when targeting 'ES5' or 'ES3'. */
    // "isolatedModules": true,              // 将每个文件做为单独的模块 （与 'ts.transpileModule' 类似）.

    /* Strict Type-Checking Options *//* 严格的类型检查选项 */
    "strict": true,                          // 启用所有严格类型检查选项
    // "noImplicitAny": true,                // 在表达式和声明上有隐含的 any类型时报错
    // "strictNullChecks": true,             // 启用严格的 null 检查
    // "strictFunctionTypes": true,           /* Enable strict checking of function types. */
    // "strictBindCallApply": true,           /* Enable strict 'bind', 'call', and 'apply' methods on functions. */
    // "strictPropertyInitialization": true,  /* Enable strict checking of property initialization in classes. */
    // "noImplicitThis": true,               // 当 this 表达式值为 any 类型的时候，生成一个错误
    "alwaysStrict": true,                    // 以严格模式检查每个模块，并在每个文件里加入 'use strict'

    /* Additional Checks *//* 额外的检查 */
    // "noUnusedLocals": true,                // 有未使用的变量时，抛出错误
    // "noUnusedParameters": true,            // 有未使用的参数时，抛出错误
    // "noImplicitReturns": true,             // 并不是所有函数里的代码都有返回值时，抛出错误
    // "noFallthroughCasesInSwitch": true,    // 报告 switch 语句的 fallthrough 错误。（即，不允许 switch 的 case 语句贯穿）

    /* Module Resolution Options *//* 模块解析选项 */
    // "moduleResolution": "node",            // 选择模块解析策略： 'node' (Node.js) or 'classic' (TypeScript pre-1.6)
    // 上面的"module": "commonjs",则moduleResolution 默认为node
    
    // "baseUrl": "./",                       // 用于解析非相对模块名称的基目录
    // "paths": {},                           // 模块名到基于 baseUrl 的路径映射的列表
    // "rootDirs": [],                        // 根文件夹列表，其组合内容表示项目运行时的结构内容
    // "typeRoots": [],                       // 包含类型声明的文件列表
    // "types": [],                           // 需要包含的类型声明文件名列表
    "allowSyntheticDefaultImports": true,     // 允许从没有设置默认导出的模块中默认导入。
    "esModuleInterop": true /* Enables emit interoperability between CommonJS and ES Modules via creation of namespace objects for all imports. Implies 'allowSyntheticDefaultImports'. */
    // "preserveSymlinks": true,              /* Do not resolve the real path of symlinks. */
    // "allowUmdGlobalAccess": true,          /* Allow accessing UMD globals from modules. */
    
    /* Source Map Options */
    // "sourceRoot": "./",                    // 指定调试器应该找到 TypeScript 文件而不是源文件的位置
    // "mapRoot": "./",                       // 指定调试器应该找到映射文件而不是生成文件的位置
    // "inlineSourceMap": true,               // 生成单个 soucemaps 文件，而不是将 sourcemaps 生成不同的文件
    // "inlineSources": true,                 // 将代码与 sourcemaps 生成到一个文件中，要求同时设置了 --inlineSourceMap 或 --sourceMap 属性
    
    /* Experimental Options */ /* 其他选项 */
    // "experimentalDecorators": true,        // 启用装饰器
    // "emitDecoratorMetadata": true          // 为装饰器提供元数据的支持
  },
  "include": [ /**要编译的文件*/
    // "src/**/*"
    // "./ts/*.ts"
    "./**/*"
  ]
  // "exclude": [   /** 忽略的文件*/
  //     "node_modules",
  //     "**/*.spec.ts"
  // ],
}
```