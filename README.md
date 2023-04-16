# TypesSript 快速上手指南

## 1 基础部分（一看就会）

### 1.1 环境

- 在线 PlayGround: https://www.typescriptlang.org/play

- 使用 ts-node / esno 直接执行 ts

```
npm install ts-node -g
ts-node fileName.ts

npm install esno -g
esno fileName.ts
```

- 使用 tsc / tsup 编译 ts

```
npm install typescript  -g
// 查看 tsc 版本
tsc -v
// 编译 ts 文件
tsc fileName.ts
```

> esno 和 tsup 都是基于 esbuild 开发的

### 1.2 类型基础

JavaScript 的原始数据类型：

- boolean / number / string / symbol / bigint
- undefined / null

> undefined 和 null 是所有类型的子类型，可赋值给其他类型；如果指定了 --strictNullChecks 标记，null 和 undefined 只能赋值给 void 和它们自己。

```ts
let isDone: boolean = false;
let age: number = 10;
let str: string = "test";

let u: undefined = undefined;
let n: null = null;

// 非--strictNullChecks下
let num: number = undefined;
```

TS 的特殊类型：

- **any**：回到了 JS，一般不会用
- **unknown**：不知道用什么类型声明变量时使用，但操作变量时需要收缩类型范围，否则会报错（any不会报错）。
- **void**：单独声明一个 void 类型的变量没有什么大用，因为你只能为它赋值 undefined / null，函数没有返回值时其返回的类型为 void。
- **never**：表示一个函数永远没有返回值(表现为抛出异常或无法执行到函数结束)，never 类型是任何类型的子类型

```ts
function test(x: unknown) {
  if (typeof x === "string") {
    return x.toUpperCase();
  }
}

function error(message: string): never {
    throw new Error(message);
}
```

- **数组**：两种定义方法：使用`[]`定义和使用Array泛型，推荐使用前者

```ts
let arr1: number[] = [1, 2, 3];

let arr2: Array<number> = [1, 2, 3];
```

- **元组**：元组类型是一种特殊的数组类型，它确切地知道元素数量及其特定位置的类型。

```ts
let x: [string, number];

x = ['hello', 10]; // OK
x = [10, 'hello']; // Error
```

- **枚举**：用于常量映射

```ts
// 默认情况下，从 0 开始为元素编号。 你也可以手动的指定成员的数
enum Direction {
  Up, // 相当于 = 0
  Down,
  Left,
  Right,
}
console.log(Direction.Up) // 0
// 反向映射
console.log(Direction[0])

// 如果枚举第一个元素赋有初始值，就会从初始值开始递增
enum Direction {
    Up = 6,
    Down,
    Left,
    Right
}

// 字符串枚举
enum Direction {
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT",
}
console.log(Direction.Up); // UP
```

### 1.3 interface 和 type

interface：接口，一般用于描述对象和类以及函数

```ts
interface Person {
  name: string;
  age?: number; // 可选
  readonly address: string; // 只读
}

// 函数
interface SearchFunc {
  (source: string): boolean;
}

interface Shape {
  color: string;
}

// 接口继承接口
interface Square extends Shape {
  length: number;
}
```

type：类型别名，type 可以声明基本类型别名，联合类型，交叉类型，元组等类型

```ts
type Name = string;
type NameResolver = () => string;
type NameOrResolver = Name | NameResolver;

type Shape = { color: string; };
type Square = Shape & { length: number; }; // 相当于interface的extends

// 函数
type SearchFunc = (source: string) => boolean;
```

区别和联系：

- interface 是一种数据结构的描述（shape），比如对象和类，interface 可以多次定义并能够合并，可以被 extends 和 implements
- type 是类型别名，是一种表达式；type 可以声明基本类型别名，联合类型，交叉类型、元组等类型
- 用 interface 描述数据结构，用 type 描述类型关系；能用 interface 实现，就用 interface , 如果不能就用 type
- 两者都可以用来描述对象或函数的类型

### 1.3 类型工具

- **类型推导**：初始化变量和成员，设置函数的默认值和决定返回值时，TS 会自带推导类型。

```ts
const hello = "hello typescipt!"; // 自动推导为 string 类型

// 自动推导 函数的参数为 string 类型和返回类型为 void
function sayHello(str = "test") {
  console.log(str);
}
```

- **联合类型**：表示一个值可以是几种类型之一，**交集**

```ts
let numberOrString: number | string 
// 注意只能使用 联合类型的所有类型里 共有的属性或方法：
numberOrString.length
numberOrString.toString()
```

- **交叉类型**：将多个类型合并为一个类型，**并集**
```ts
interface IName  {
  name: string
}
type IPerson = IName & { age: number }
let person: IPerson = { name: 'hello', age: 12}
```

- **字面量类型**：提供一个准确的变量值，目前支持 字符串、数字、布尔

```ts
type Direction = 'North' | 'East' | 'South' | 'West';
```

- **类型断言**：告诉编译器你比它更了解这个类型，**小心使用**

```ts
let value: unknown = "this is a string";

let length1: number = (<string>value).length; // jsx语法会有歧义

let length2: number = (value as string).length; // 推荐使用 as
```

- **非空断言**：告诉编译器变量一定不是 `null` 和 `undefined`，**小心使用**

```tsx
let flag: null | undefined | string;
flag!.toString(); // ok
flag.toString(); // error
```

- **类型守卫**：是可执行运行时检查的一种表达式，用于确保该类型在一定的范围内。

```ts
// 在条件语句中使用 typeof/instanceof/in 收缩类型范围，TS 会自动推导类型
function getLength2(input: string | number): number {
  if (typeof input === 'string') {
    return input.length
  } else {
    return input.toString().length
  }
}

// 将判断逻辑封装起来提取到函数外部进行复用, 使用 is
function isString(input: unknown): input is string {
  return typeof input === "string";
}
```

### 1.4 类与继承

TS 支持了修饰符：

- public 修饰的属性或方法是公有的，可以在任何地方被访问到，默认所有的属性和方法都是 public 的
- private 修饰的属性或方法是私有的，不能在声明它的类的外部访问
- protected 修饰的属性或方法是受保护的，它和 private 类似，区别是它在子类中也是允许被访问的

```ts
interface Radio {
  switchRadio(trigger: boolean): void;
}
// 类实现接口
class Car implements Radio {
  switchRadio(trigger) {
    return 123
  }
}

class Person {
  public name: string;
  protected age: number;
  private tel: number;

  constructor(name: string, age: number, tel: number) {
    this.name = name;
    this.age = age;
    this.tel = tel;
  }
}

// 继承
class Child extends Person {
  constructor(name: string, age: number, tel: number) {
    super(name, age, tel);
  }

  getName() {
    console.log(`我的名字叫${this.name},年龄是${this.age}`);
  }
}

```

- **抽象类**：只能被继承，但不能被实例化的类

```ts
abstract class Animal {
    constructor(name:string) {
        this.name = name
    }
    public name: string
    public abstract sayHi():void
}

class Dog extends Animal {
    constructor(name:string) {
        super(name)
    }
    public sayHi() {
        console.log('wang')
    }
}
```

### 1.5 泛型

泛型（Generics）是指在定义函数、接口或类的时候，不预先指定具体的类型，而在使用的时候再指定类型的一种特性，增强程序的扩展性

```ts
function echo<T>(arg: T): T {
  return arg
}

// 多个泛型
function swap<T, U>(tuple: [T, U]): [U, T] {
  return [tuple[1], tuple[0]]
}

// 继承类型
function echoWithLength<T extends IWithLength>(arg: T): T {
  console.log(arg.length)
  return arg
}

//泛型和 interface
interface KeyPair<T, U> {
  key: T;
  value: U;
}
//泛型和 interface
interface KeyPair<T, U> {
  key: T;
  value: U;
}
const kp1: KeyPair<number, string> = { key: 123, value: "str" };
const kp2: KeyPair<string, number> = { key: "str", value: 123 };
```

## 2 工程部分（不能不看）

### 1.1 tsconfig

通过 `tsc --init` 初始化，生成 tsconfig.json 文件；[配置文档](https://www.typescriptlang.org/zh/tsconfig#compilerOptions)

顶层选项：

- compilerOptions：编译配置
- files / extends / include /exclude / references
- watchOptions
- typeAcquisition

### 1.2 类型声明

- d.ts 类型声明文件： 一般来说 ts 会解析所有的`.ts`和`.d.ts`文件
- declare：手动声明

```ts
declare module "*.vue" {
  import Vue from "vue";
  export default Vue;
}
```

### 1.3 命令空间

命名空间是TS提供隔离手段，防止命名冲突；一般来说使用模块化足够了，

- 使用 export 向外暴露
- import 定义别名

```ts
namespace Vehicle {
  const name = "Toyota"
  // 暴露
  export function getName () {
      return `${name}`
  }
}

//允许嵌套    
namespace TransportMeans {  
  export namespace Vehicle {
    const name = "Toyota"
    export function getName () {
        return `${name}`
    }
  }
}
// 别名
import carName= TransporMeans.Vehicle;
```

### 1.3 三斜线指令

只能放在文件最顶端，告诉编译器在编译过程需要引入的文件。

```ts
/// <reference types="node" />   // 表示库的类型声明 如@types/node/index.d.ts 

/// <reference path="..." />  // 引入文件
```

## 3 类型进阶（够用就好）

### 3.1 索引类型

支持字符串和数字两种索引签名，数字索引的返回值必须是字符串索引返回值类型的子类型。JavaScript 中，数字索引访问会自动转换为字符串索引访问。

```ts
interface AllStringTypes {
  [key: string]: string;
}

// 将对象中的所有键转换为对应字面量类型
interface Foo {
  PropA: string;
  PropB: number;
}
type FooKeys = keyof Foo; // 字面量类型 "PropA"|"PropB"

type PropAType = Foo[PropA]; 

type PropTypeUnion = Foo[keyof Foo]; // 联合类型 string | number
```

- 类型映射

```ts
type Clone<T> = {
  [K in keyof T]: T[K];
};
```



### 3.2 内置工具

**Partial**

```ts
type Partial<T> = {
    [P in keyof T]?: T[P];
};


interface ApiKey {
  id: number;
  name: string;
}

const dataType1: ApiKey = {
  id: 1,
  name: 'static'
}

const dataType2:  Partial<ApiKey> = {
  name: 'json'
}
```

**Record**

```ts
type Record<K extends keyof any, T> = {
    [P in K]: T;
};

type Coord = Record<'x' | 'y', number>;

// 等同于
type Coord = {
	x: number;
	y: number;
}
```

**Readonly** 

```ts
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};

type Coord = Readonly<Record<'x' | 'y', number>>;

// 等同于
type Coord = {
    readonly x: number;
    readonly y: number;
}
```



**Pick**

```ts
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
};

type Coord = Record<'x' | 'y', number>;
type CoordX = Pick<Coord, 'x'>;

// 等用于
type CoordX = {
	x: number;
}
```

**Omit**

```ts
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;

interface I1 {
	a: number;
	b: string;
	c: boolean;
}

type AC = Omit<I1, 'b'>;     // { a:number; c:boolean } 
type C = Omit<I1, 'a' |'b'>  // { c: boolean }

```

**infer**

`infer`可以在`extends`的条件语句中推断待推断的类型

`infer R` 的位置代表了一个未知的类型，可以理解为在条件类型中给了它一个占位符，然后就可以在后面的三元运算符中使用它。

```ts
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;

type func = () => number;
type variable = string;
type funcReturnType = ReturnType<func>; // funcReturnType 类型为 number
type varReturnType = ReturnType<variable>; // varReturnType 类型为 any


type Unpack<T> = T extends Array<infer R> ? R : T

type NumArr = Array<number>

type U = Unpack<NumArr> // number
```



## 参考

- [TS 入门完全指南](https://juejin.cn/post/7215796935298596920)
