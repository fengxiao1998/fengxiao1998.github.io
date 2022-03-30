### #什么是typeScript



```
TypeScript 是 JavaScript 的类型的超集，主要提供了类型系统和对 ES6 的支持，它可以编译成纯 JavaScript。编译出来的 JavaScript 可以运行在任何浏览器上。TypeScript 编译工具可以运行在任何服务器和任何系统上。
```



优点



```
（1）TypeScript 增加了代码的可读性和可维护性 

    类型系统实际上是最好的文档，大部分的函数看看类型的定义就可以知道如何使用了

    可以在编译阶段就发现大部分错误，这总比在运行时候出错好

    增强了编辑器和 IDE 的功能，包括代码补全、接口提示、跳转到定义、重构等

（2）TypeScript 非常包容

         TypeScript 是 JavaScript 的超集，`.js` 文件可以直接重命名为 `.ts` 即可

        即使不显式的定义类型，也能够自动做出类型推论

    可以定义从简单到复杂的几乎一切类型

    即使 TypeScript 编译报错，也可以生成 JavaScript 文件

    兼容第三方库，即使第三方库不是用 TypeScript 写的，也可以编写单独的类型文件供 TypeScript 读取

（3）社区活跃
```



缺点



```
（1）需要一定学习成本（理解接口（Interfaces）、泛型（Generics）、类（Classes）、枚举类型（Enums））等概念
 (2) 短期增加开发成本
 (3) 与一些库结合不完美
```



### #安装



TypeScript 的命令行工具安装方法如下：



```
   npm install -g typescript
```



 编译



```
   tsc hello.ts
```



### #基础



- boolean为布尔值类型，如let isDone: Boolean = false

- number为数值类型，如let decimal: number = 6; let decimal: number = 'string' error: Type '"string"' is not assignable to type 'number'.

- string为字符串类型，如let color: string = 'blue'

- null和undefined

- 多种类型可以用|隔开，比如number | string表示可以是number或string类型

- any为任意类型，如let notSure: any = 4; notSure = "maybe a string instead"

- void为空类型，如let unusable: void = undefined

- array为数组，如let arr: number[] = [1,2,3]



## #类



### public private 和 protected



TypeScript 可以使用三种访问修饰符（Access Modifiers），分别是 public、private 和 protected。



- public 修饰的属性或方法是公有的，可以在任何地方被访问到，默认所有的属性和方法都是 public 的

- private 修饰的属性或方法是私有的，不能在声明它的类的外部访问

- protected 修饰的属性或方法是受保护的，它和 private 类似，区别是它在子类中也是允许被访问的



下面举一些例子：



```
class Animal {
    public name;
    public constructor(name) {
        this.name = name;
    }
}

let a = new Animal('Jack');
console.log(a.name); // Jack
a.name = 'Tom';
console.log(a.name); // Tom
```



上面的例子中，name 被设置为了 public，所以直接访问实例的 name 属性是允许的。



很多时候，我们希望有的属性是无法直接存取的，这时候就可以用 private 了：



```
class Animal {
    private name;
    public constructor(name) {
        this.name = name;
    }
}

let a = new Animal('Jack');
console.log(a.name); // Jack
a.name = 'Tom';

// index.ts(9,13): error TS2341: Property 'name' is private and only accessible within class 'Animal'.
// index.ts(10,1): error TS2341: Property 'name' is private and only accessible within class 'Animal'.
```



需要注意的是，TypeScript 编译之后的代码中，并没有限制 private 属性在外部的可访问性。



使用 private 修饰的属性或方法，在子类中也是不允许访问的：



```
class Animal {
    private name;
    public constructor(name) {
        this.name = name;
    }
}

class Cat extends Animal {
    constructor(name) {
        super(name);
        console.log(this.name);
    }
}

// index.ts(11,17): error TS2341: Property 'name' is private and only accessible within class 'Animal'.
```



而如果是用 protected 修饰，则允许在子类中访问：



```
class Animal {
    protected name;
    public constructor(name) {
        this.name = name;
    }
}

class Cat extends Animal {
    constructor(name) {
        super(name);
        console.log(this.name);
    }
}
```



### 抽象类



abstract 用于定义抽象类和其中的抽象方法。



什么是抽象类？



首先，抽象类是不允许被实例化的：



```
abstract class Animal {
    public name;
    public constructor(name) {
        this.name = name;
    }
    public abstract sayHi();
}

let a = new Animal('Jack');

// index.ts(9,11): error TS2511: Cannot create an instance of the abstract class 'Animal'.
```



上面的例子中，我们定义了一个抽象类 Animal，并且定义了一个抽象方法 sayHi。在实例化抽象类的时候报错了。



其次，抽象类中的抽象方法必须被子类实现：



```
abstract class Animal {
    public name;
    public constructor(name) {
        this.name = name;
    }
    public abstract sayHi();
}

class Cat extends Animal {
    public eat() {
        console.log(`${this.name} is eating.`);
    }
}

let cat = new Cat('Tom');

// index.ts(9,7): error TS2515: Non-abstract class 'Cat' does not implement inherited abstract member 'sayHi' from class 'Animal'.
```



上面的例子中，我们定义了一个类 Cat 继承了抽象类 Animal，但是没有实现抽象方法 sayHi，所以编译报错了。



下面是一个正确使用抽象类的例子：



```
abstract class Animal {
    public name;
    public constructor(name) {
        this.name = name;
    }
    public abstract sayHi();
}

class Cat extends Animal {
    public sayHi() {
        console.log(`Meow, My name is ${this.name}`);
    }
}

let cat = new Cat('Tom');
```



上面的例子中，我们实现了抽象方法 sayHi，编译通过了。



### 类的类型



给类加上 TypeScript 的类型很简单，与接口类似：



```
class Animal {
    name: string;
    constructor(name: string) {
        this.name = name;
    }
    sayHi(): string {
      return `My name is ${this.name}`;
    }
}

let a: Animal = new Animal('Jack');
console.log(a.sayHi()); // My name is Jack
```



# #泛型



泛型（Generics）是指在定义函数、接口或类的时候，不预先指定具体的类型，而在使用的时候再指定类型的一种特性。



## 简单的例子



首先，我们来实现一个函数 createArray，它可以创建一个指定长度的数组，同时将每一项都填充一个默认值：



```
function createArray(length: number, value: any): Array<any> {
    let result = [];
    for (let i = 0; i < length; i++) {
        result[i] = value;
    }
    return result;
}

createArray(3, 'x'); // ['x', 'x', 'x']
```



上例中，我们使用了[之前提到过的数组泛型](https://ts.xcatliu.com/basics/type-of-array.html#数组泛型)来定义返回值的类型。



这段代码编译不会报错，但是一个显而易见的缺陷是，它并没有准确的定义返回值的类型：



Array<any> 允许数组的每一项都为任意类型。但是我们预期的是，数组中每一项都应该是输入的 value的类型。



这时候，泛型就派上用场了：



```
function createArray<T>(length: number, value: T): Array<T> {
    let result: T[] = [];
    for (let i = 0; i < length; i++) {
        result[i] = value;
    }
    return result;
}

createArray<string>(3, 'x'); // ['x', 'x', 'x']
```



上例中，我们在函数名后添加了 <T>，其中 T 用来指代任意输入的类型，在后面的输入 value: T 和输出 Array<T> 中即可使用了。



接着在调用的时候，可以指定它具体的类型为 string。当然，也可以不手动指定，而让类型推论自动推算出来：



```
function createArray<T>(length: number, value: T): Array<T> {
    let result: T[] = [];
    for (let i = 0; i < length; i++) {
        result[i] = value;
    }
    return result;
}

createArray(3, 'x'); // ['x', 'x', 'x']
```



## 多个类型参数



定义泛型的时候，可以一次定义多个类型参数：



```
function swap<T, U>(tuple: [T, U]): [U, T] {
    return [tuple[1], tuple[0]];
}

swap([7, 'seven']); // ['seven', 7]
```



上例中，我们定义了一个 swap 函数，用来交换输入的元组。



## 泛型约束



在函数内部使用泛型变量的时候，由于事先不知道它是哪种类型，所以不能随意的操作它的属性或方法：



```
function loggingIdentity<T>(arg: T): T {
    console.log(arg.length);
    return arg;
}

// index.ts(2,19): error TS2339: Property 'length' does not exist on type 'T'.
```



上例中，泛型 T 不一定包含属性 length，所以编译的时候报错了。



这时，我们可以对泛型进行约束，只允许这个函数传入那些包含 length 属性的变量。这就是泛型约束：



```
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);
    return arg;
}
```



上例中，我们使用了 extends 约束了泛型 T 必须符合接口 Lengthwise 的形状，也就是必须包含 length 属性。



此时如果调用 loggingIdentity 的时候，传入的 arg 不包含 length，那么在编译阶段就会报错了：



```
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);
    return arg;
}

loggingIdentity(7);

// index.ts(10,17): error TS2345: Argument of type '7' is not assignable to parameter of type 'Lengthwise'.
```



多个类型参数之间也可以互相约束：



```
function copyFields<T extends U, U>(target: T, source: U): T {
    for (let id in source) {
        target[id] = (<T>source)[id];
    }
    return target;
}

let x = { a: 1, b: 2, c: 3, d: 4 };

copyFields(x, { b: 10, d: 20 });
```



上例中，我们使用了两个类型参数，其中要求 T 继承 U，这样就保证了 U 上不会出现 T 中不存在的字段。



## 泛型接口



[之前学习过](https://ts.xcatliu.com/basics/type-of-function.html#接口中函数的定义)，可以使用接口的方式来定义一个函数需要符合的形状：



```
interface SearchFunc {
  (source: string, subString: string): boolean;
}

let mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
    return source.search(subString) !== -1;
}
```



当然也可以使用含有泛型的接口来定义函数的形状：



```
interface CreateArrayFunc {
    <T>(length: number, value: T): Array<T>;
}

let createArray: CreateArrayFunc;
createArray = function<T>(length: number, value: T): Array<T> {
    let result: T[] = [];
    for (let i = 0; i < length; i++) {
        result[i] = value;
    }
    return result;
}

createArray(3, 'x'); // ['x', 'x', 'x']
```



进一步，我们可以把泛型参数提前到接口名上：



```
interface CreateArrayFunc<T> {
    (length: number, value: T): Array<T>;
}

let createArray: CreateArrayFunc<any>;
createArray = function<T>(length: number, value: T): Array<T> {
    let result: T[] = [];
    for (let i = 0; i < length; i++) {
        result[i] = value;
    }
    return result;
}

createArray(3, 'x'); // ['x', 'x', 'x']
```



注意，此时在使用泛型接口的时候，需要定义泛型的类型。



## 泛型类



与泛型接口类似，泛型也可以用于类的类型定义中：



```
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };
```



## 泛型参数的默认类型



在 TypeScript 2.3 以后，我们可以为泛型中的类型参数指定默认类型。当使用泛型时没有在代码中直接指定类型参数，从实际值参数中也无法推测出时，这个默认类型就会起作用。



```
function createArray<T = string>(length: number, value: T): Array<T> {
    let result: T[] = [];
    for (let i = 0; i < length; i++) {
        result[i] = value;
    }
    return result;
}
```



# #枚举



枚举（Enum）类型用于取值被限定在一定范围内的场景，比如一周只能有七天，颜色限定为红绿蓝等。



## 简单的例子



枚举使用 enum 关键字来定义：



```
enum Days {Sun, Mon, Tue, Wed, Thu, Fri, Sat};
```



枚举成员会被赋值为从 0 开始递增的数字，同时也会对枚举值到枚举名进行反向映射：



```
enum Days {Sun, Mon, Tue, Wed, Thu, Fri, Sat};

console.log(Days["Sun"] === 0); // true
console.log(Days["Mon"] === 1); // true
console.log(Days["Tue"] === 2); // true
console.log(Days["Sat"] === 6); // true

console.log(Days[0] === "Sun"); // true
console.log(Days[1] === "Mon"); // true
console.log(Days[2] === "Tue"); // true
console.log(Days[6] === "Sat"); // true
```



事实上，上面的例子会被编译为：



```
var Days;
(function (Days) {
    Days[Days["Sun"] = 0] = "Sun";
    Days[Days["Mon"] = 1] = "Mon";
    Days[Days["Tue"] = 2] = "Tue";
    Days[Days["Wed"] = 3] = "Wed";
    Days[Days["Thu"] = 4] = "Thu";
    Days[Days["Fri"] = 5] = "Fri";
    Days[Days["Sat"] = 6] = "Sat";
})(Days || (Days = {}));
```



## 手动赋值



我们也可以给枚举项手动赋值：



```
enum Days {Sun = 7, Mon = 1, Tue, Wed, Thu, Fri, Sat};

console.log(Days["Sun"] === 7); // true
console.log(Days["Mon"] === 1); // true
console.log(Days["Tue"] === 2); // true
console.log(Days["Sat"] === 6); // true
```



上面的例子中，未手动赋值的枚举项会接着上一个枚举项递增。



如果未手动赋值的枚举项与手动赋值的重复了，TypeScript 是不会察觉到这一点的：



```
enum Days {Sun = 3, Mon = 1, Tue, Wed, Thu, Fri, Sat};

console.log(Days["Sun"] === 3); // true
console.log(Days["Wed"] === 3); // true
console.log(Days[3] === "Sun"); // false
console.log(Days[3] === "Wed"); // true
```



上面的例子中，递增到 3 的时候与前面的 Sun 的取值重复了，但是 TypeScript 并没有报错，导致 Days[3] 的值先是 "Sun"，而后又被 "Wed" 覆盖了。编译的结果是：



```
var Days;
(function (Days) {
    Days[Days["Sun"] = 3] = "Sun";
    Days[Days["Mon"] = 1] = "Mon";
    Days[Days["Tue"] = 2] = "Tue";
    Days[Days["Wed"] = 3] = "Wed";
    Days[Days["Thu"] = 4] = "Thu";
    Days[Days["Fri"] = 5] = "Fri";
    Days[Days["Sat"] = 6] = "Sat";
})(Days || (Days = {}));
```



所以使用的时候需要注意，最好不要出现这种覆盖的情况。



手动赋值的枚举项可以不是数字，此时需要使用类型断言来让tsc无视类型检查 (编译出的js仍然是可用的)：



```
enum Days {Sun = 7, Mon, Tue, Wed, Thu, Fri, Sat = <any>"S"};
var Days;
(function (Days) {
    Days[Days["Sun"] = 7] = "Sun";
    Days[Days["Mon"] = 8] = "Mon";
    Days[Days["Tue"] = 9] = "Tue";
    Days[Days["Wed"] = 10] = "Wed";
    Days[Days["Thu"] = 11] = "Thu";
    Days[Days["Fri"] = 12] = "Fri";
    Days[Days["Sat"] = "S"] = "Sat";
})(Days || (Days = {}));
```



当然，手动赋值的枚举项也可以为小数或负数，此时后续未手动赋值的项的递增步长仍为 1：



```
enum Days {Sun = 7, Mon = 1.5, Tue, Wed, Thu, Fri, Sat};

console.log(Days["Sun"] === 7); // true
console.log(Days["Mon"] === 1.5); // true
console.log(Days["Tue"] === 2.5); // true
console.log(Days["Sat"] === 6.5); // true
```



## 常数项和计算所得项



枚举项有两种类型：常数项（constant member）和计算所得项（computed member）。



前面我们所举的例子都是常数项，一个典型的计算所得项的例子：



```
enum Color {Red, Green, Blue = "blue".length};
```



上面的例子中，"blue".length 就是一个计算所得项。



上面的例子不会报错，但是**如果紧接在计算所得项后面的是未手动赋值的项，那么它就会因为无法获得初始值而报错**：



```
enum Color {Red = "red".length, Green, Blue};

// index.ts(1,33): error TS1061: Enum member must have initializer.
// index.ts(1,40): error TS1061: Enum member must have initializer.
```



下面是常数项和计算所得项的完整定义，部分引用自[中文手册 - 枚举](https://zhongsp.gitbooks.io/typescript-handbook/content/doc/handbook/Enums.html)：



当满足以下条件时，枚举成员被当作是常数：



- 不具有初始化函数并且之前的枚举成员是常数。在这种情况下，当前枚举成员的值为上一个枚举成员的值加 1。但第一个枚举元素是个例外。如果它没有初始化方法，那么它的初始值为 0。

- 枚举成员使用常数枚举表达式初始化。常数枚举表达式是 TypeScript 表达式的子集，它可以在编译阶段求值。当一个表达式满足下面条件之一时，它就是一个常数枚举表达式：

- 数字字面量

- 引用之前定义的常数枚举成员（可以是在不同的枚举类型中定义的）如果这个成员是在同一个枚举类型中定义的，可以使用非限定名来引用

- 带括号的常数枚举表达式

- +, -, ~ 一元运算符应用于常数枚举表达式

- +, -, *, /, %, <<, >>, >>>, &, |, ^ 二元运算符，常数枚举表达式做为其一个操作对象。若常数枚举表达式求值后为NaN或Infinity，则会在编译阶段报错



所有其它情况的枚举成员被当作是需要计算得出的值。



## 常数枚举



常数枚举是使用 const enum 定义的枚举类型：



```
const enum Directions {
    Up,
    Down,
    Left,
    Right
}

let directions = [Directions.Up, Directions.Down, Directions.Left, Directions.Right];
```



常数枚举与普通枚举的区别是，它会在编译阶段被删除，并且不能包含计算成员。



上例的编译结果是：



```
var directions = [0 /* Up */, 1 /* Down */, 2 /* Left */, 3 /* Right */];
```



假如包含了计算成员，则会在编译阶段报错：



```
const enum Color {Red, Green, Blue = "blue".length};

// index.ts(1,38): error TS2474: In 'const' enum declarations member initializer must be constant expression.
```



## 外部枚举



外部枚举（Ambient Enums）是使用 declare enum 定义的枚举类型：



```
declare enum Directions {
    Up,
    Down,
    Left,
    Right
}

let directions = [Directions.Up, Directions.Down, Directions.Left, Directions.Right];
```



之前提到过，declare 定义的类型只会用于编译时的检查，编译结果中会被删除。



上例的编译结果是：



```
var directions = [Directions.Up, Directions.Down, Directions.Left, Directions.Right];
```



外部枚举与声明语句一样，常出现在声明文件中。



同时使用 declare 和 const 也是可以的：



```
declare const enum Directions {
    Up,
    Down,
    Left,
    Right
}

let directions = [Directions.Up, Directions.Down, Directions.Left, Directions.Right];
```



编译结果：



```
var directions = [0 /* Up */, 1 /* Down */, 2 /* Left */, 3 /* Right */];
```

TypeScript 的枚举类型的概念[来源于 C#](https://msdn.microsoft.com/zh-cn/library/sbbt4032.aspx)。



# #对象的类型——接口



在 TypeScript 中，我们使用接口（Interfaces）来定义对象的类型。



## 什么是接口



在面向对象语言中，接口（Interfaces）是一个很重要的概念，它是对行为的抽象，而具体如何行动需要由类（classes）去实现（implements）。



TypeScript 中的接口是一个非常灵活的概念，除了可用于[对类的一部分行为进行抽象](https://ts.xcatliu.com/advanced/class-and-interfaces.html#类实现接口)以外，也常用于对「对象的形状（Shape）」进行描述。



## 简单的例子



```
interface Person {
    name: string;
    age: number;
}

let tom: Person = {
    name: 'Tom',
    age: 25
};
```



上面的例子中，我们定义了一个接口 Person，接着定义了一个变量 tom，它的类型是 Person。这样，我们就约束了 tom 的形状必须和接口 Person 一致。



接口一般首字母大写。[有的编程语言中会建议接口的名称加上 I 前缀](https://msdn.microsoft.com/en-us/library/8bc1fexb(v=vs.71).aspx)。



定义的变量比接口少了一些属性是不允许的：



```
interface Person {
    name: string;
    age: number;
}

let tom: Person = {
    name: 'Tom'
};

// index.ts(6,5): error TS2322: Type '{ name: string; }' is not assignable to type 'Person'.
//   Property 'age' is missing in type '{ name: string; }'.
```



多一些属性也是不允许的：



```
interface Person {
    name: string;
    age: number;
}

let tom: Person = {
    name: 'Tom',
    age: 25,
    gender: 'male'
};

// index.ts(9,5): error TS2322: Type '{ name: string; age: number; gender: string; }' is not assignable to type 'Person'.
//   Object literal may only specify known properties, and 'gender' does not exist in type 'Person'.
```



可见，**赋值的时候，变量的形状必须和接口的形状保持一致**。



## 可选属性



有时我们希望不要完全匹配一个形状，那么可以用可选属性：



```
interface Person {
    name: string;
    age?: number;
}

let tom: Person = {
    name: 'Tom'
};
interface Person {
    name: string;
    age?: number;
}

let tom: Person = {
    name: 'Tom',
    age: 25
};
```



可选属性的含义是该属性可以不存在。



这时**仍然不允许添加未定义的属性**：



```
interface Person {
    name: string;
    age?: number;
}

let tom: Person = {
    name: 'Tom',
    age: 25,
    gender: 'male'
};

// examples/playground/index.ts(9,5): error TS2322: Type '{ name: string; age: number; gender: string; }' is not assignable to type 'Person'.
//   Object literal may only specify known properties, and 'gender' does not exist in type 'Person'.
```



## 任意属性



有时候我们希望一个接口允许有任意的属性，可以使用如下方式：



```
interface Person {
    name: string;
    age?: number;
    [propName: string]: any;
}

let tom: Person = {
    name: 'Tom',
    gender: 'male'
};
```



使用 [propName: string] 定义了任意属性取 string 类型的值。



需要注意的是，**一旦定义了任意属性，那么确定属性和可选属性都必须是它的子属性**：



```
interface Person {
    name: string;
    age?: number;
    [propName: string]: string;
}

let tom: Person = {
    name: 'Tom',
    age: 25,
    gender: 'male'
};

// index.ts(3,5): error TS2411: Property 'age' of type 'number' is not assignable to string index type 'string'.
// index.ts(7,5): error TS2322: Type '{ [x: string]: string | number; name: string; age: number; gender: string; }' is not assignable to type 'Person'.
//   Index signatures are incompatible.
//     Type 'string | number' is not assignable to type 'string'.
//       Type 'number' is not assignable to type 'string'.
```



上例中，任意属性的值允许是 string，但是可选属性 age 的值却是 number，number 不是 string 的子属性，所以报错了。



另外，在报错信息中可以看出，此时 { name: 'Tom', age: 25, gender: 'male' } 的类型被推断成了 { [x: string]: string | number; name: string; age: number; gender: string; }，这是联合类型和接口的结合。



## 只读属性



有时候我们希望对象中的一些字段只能在创建的时候被赋值，那么可以用 readonly 定义只读属性：



```
interface Person {
    readonly id: number;
    name: string;
    age?: number;
    [propName: string]: any;
}

let tom: Person = {
    id: 89757,
    name: 'Tom',
    gender: 'male'
};

tom.id = 9527;

// index.ts(14,5): error TS2540: Cannot assign to 'id' because it is a constant or a read-only property.
```



上例中，使用 readonly 定义的属性 id 初始化后，又被赋值了，所以报错了。



**注意，只读的约束存在于第一次给对象赋值的时候，而不是第一次给只读属性赋值的时候**：



```
interface Person {
    readonly id: number;
    name: string;
    age?: number;
    [propName: string]: any;
}

let tom: Person = {
    name: 'Tom',
    gender: 'male'
};

tom.id = 89757;

// index.ts(8,5): error TS2322: Type '{ name: string; gender: string; }' is not assignable to type 'Person'.
//   Property 'id' is missing in type '{ name: string; gender: string; }'.
// index.ts(13,5): error TS2540: Cannot assign to 'id' because it is a constant or a read-only property.
```



上例中，报错信息有两处，第一处是在对 tom 进行赋值的时候，没有给 id 赋值。



第二处是在给 tom.id 赋值的时候，由于它是只读属性，所以报错了。



# 类与接口



[之前学习过](https://ts.xcatliu.com/basics/type-of-object-interfaces.html)，接口（Interfaces）可以用于对「对象的形状（Shape）」进行描述。



这一章主要介绍接口的另一个用途，对类的一部分行为进行抽象。



## 类实现接口



实现（implements）是面向对象中的一个重要概念。一般来讲，一个类只能继承自另一个类，有时候不同类之间可以有一些共有的特性，这时候就可以把特性提取成接口（interfaces），用 implements 关键字来实现。这个特性大大提高了面向对象的灵活性。



举例来说，门是一个类，防盗门是门的子类。如果防盗门有一个报警器的功能，我们可以简单的给防盗门添加一个报警方法。这时候如果有另一个类，车，也有报警器的功能，就可以考虑把报警器提取出来，作为一个接口，防盗门和车都去实现它：



```
interface Alarm {
    alert();
}

class Door {
}

class SecurityDoor extends Door implements Alarm {
    alert() {
        console.log('SecurityDoor alert');
    }
}

class Car implements Alarm {
    alert() {
        console.log('Car alert');
    }
}
```



一个类可以实现多个接口：



```
interface Alarm {
    alert();
}

interface Light {
    lightOn();
    lightOff();
}

class Car implements Alarm, Light {
    alert() {
        console.log('Car alert');
    }
    lightOn() {
        console.log('Car light on');
    }
    lightOff() {
        console.log('Car light off');
    }
}
```



上例中，Car 实现了 Alarm 和 Light 接口，既能报警，也能开关车灯。



## 接口继承接口



接口与接口之间可以是继承关系：



```
interface Alarm {
    alert();
}

interface LightableAlarm extends Alarm {
    lightOn();
    lightOff();
}
```



上例中，我们使用 extends 使 LightableAlarm 继承 Alarm。



## 接口继承类



接口也可以继承类：



```
class Point {
    x: number;
    y: number;
}

interface Point3d extends Point {
    z: number;
}

let point3d: Point3d = {x: 1, y: 2, z: 3};
```



## 混合类型



[之前学习过](https://ts.xcatliu.com/basics/type-of-function.html#接口中函数的定义)，可以使用接口的方式来定义一个函数需要符合的形状：



```
interface SearchFunc {
    (source: string, subString: string): boolean;
}

let mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
    return source.search(subString) !== -1;
}
```



有时候，一个函数还可以有自己的属性和方法：



```
interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
}

function getCounter(): Counter {
    let counter = <Counter>function (start: number) { };
    counter.interval = 123;
    counter.reset = function () { };
    return counter;
}

let c = getCounter();
c(10);
c.reset();
c.interval = 5.0;
```



### #在vue项目中使用typescript



[https://segmentfault.com/a/1190000011744210](https://segmentfault.com/a/1190000011744210#articleHeader13)