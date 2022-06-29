# TS 随记

## 对 TS 的理解

### TS 是什么呢？

-`TypeScript`是`JavaScript`的类型的超集，支持`ES6`语法，支持面向对象编程的概念，如类、接口、继承、泛型等

- 是一种静态类型检查的语言，提供了类型注解，在代码编译阶段就可以检查出数据类型的错误
- 扩展了`JavaScript`的语法，所以任何现有的`JavaScript`程序可以不加改变的在`TypeScript`下工作

### TS 的特性

- 类型批注和编译时类型检查 ：在编译时批注变量类型
- 类型推断：`TypeScript`中没有批注变量类型会自动推断变量的类型
- 类型擦除：在编译过程中批注的内容和接口会在运行时利用工具擦除
- 接口：`TypeScript`中用接口来**定义对象类型**
- 枚举：用于取值被限定在一定范围内的场景
- Mixin：可以接受任意类型的值
- 泛型编程：写代码时使用一些以后才指定的类型
- 名字空间：名字只在该区域内有效，其他区域可重复使用该名字而不冲突
- 元组：元组合并了不同类型的对象，相当于一个可以装不同类型数据的数组
- ...

### TS 与 JS 的区别

-`TypeScript`是`JavaScript`的超集，扩展了`JavaScript`的语法。`TypeScript`可处理已有的`JavaScript`代码，并只对其中的`TypeScript`代码进行编译 -`TypeScript`文件的后缀名 .ts （.ts，.tsx，.dts），JavaScript 文件是 .js

- 在编写`TypeScript`的文件的时候就会自动编译成 js 文件

## TS 数据类型

`TypeScript`拥有`JavaScript`一样的数据类型，同时还扩展了一些使用的类型。通常我们会在开发时为明确的一些变量定义类型，以配合`TypeScript`在编译阶段进行类型检查，帮助我们看到编写代码的提示。

### TS 数据类型一览

与 JS 相同的：

- boolean（布尔类型）
- number（数字类型）
- string（字符串类型）
- array（数组类型）
- null 和 undefined 类型
- object 对象类型

扩展类型：

- tuple（元组类型）：允许表示一个已知元素数量和类型的数组，各元素的类型不必相同
- enum（枚举类型）：一个被命名的整型常数的集合（例如，当一个变量有几种可能的取值时的场景，后端返回 0、1 状态或者 0~6 标记的场景）
- any（任意类型）：允许被赋值为任意类型
- void 类型：标识方法返回值的类型（表示无返回值）
- never 类型：一般用来指定那些总是会抛出异常、无限循环

## TS 接口

### 接口是什么？

接口是一系列抽象方法的声明，是一些方法特征的集合，这些方法都应该是抽象的，需要由具体的类去实现，然后第三方就可以通过这组抽象方法调用，让具体的类执行具体的方法

> 一个接口所描述的是一个对象相关的属性和方法，但并不提供具体创建此对象实例的方法

### 使用方式

**接口定义：**

```ts
interface huohuo_interface {}
```

例子：

```ts
interface huohuo {
  name: string;
  age: number;
}
```

**可选属性**：

```ts
interface huohuo {
  name: string;
  age: number;
  sex?: string;
}
```

**只读属性：**

```ts
interface huohuo {
  name: string;
  age: number;
  sex?: string;
  readonly isGirl: boolean;
}
```

**函数：**

```ts
interface huohuo {
  name: string;
  age: number;
  sex?: string;
  readonly isGirl: boolean;
  say: (words: string) => string;
}
```

**类型推断：**使用 as

**索引签名**

**继承：**extends

## TS 类

具有与`JavaScript`一样的能力，`TypeScript`的 class 支持面向对象的所有特性，比如 类、接口等

### TS 类修饰符

- 公共 public：可以自由的访问类程序里定义的成员
- 私有 private：只能够在该类的内部进行访问，实例对象并不能够访问（继承该类的子类也不能访问）
- 受保护 protect：除了在该类的内部可以访问，还可以在子类中仍然可以访问（同私有修饰符，但 protected 成员在子类中仍然可以访问）

- 只读修饰符 readonly
- 静态属性 static
- 抽象类 abstract

## TS 函数

先看与`JavaScript`的区别

- 从定义的方式而言，`typescript`声明函数需要定义参数类型或者声明返回值类型 -`typescript`在参数中，添加可选参数供使用者选择 -`typescript`增添函数重载功能，使用者只需要通过查看函数声明的方式，即可知道函数传递的参数个数以及类型

## TS 泛型

> 泛型允许我们在强类型程序设计语言中编写代码时使用一些以后才指定的类型，在实例化时作为参数指明这些类型 在`typescript`中，定义函数，接口或者类的时候，不预先定义好具体的类型，而在使用的时候在指定类型的一种特性

### 泛型声明方式

函数声明：

```ts
function Fn<T>(parms: T): T {
  return parms;
}
```

接口声明：

```ts
interface GetFn<T> {
  (parms: T): T;
}
```

类声明：

```ts
class Stack<T> {
  private arr: T[] = [];

  public push(item: T) {
    this.arr.push(item);
  }
}
```

## 装饰器

装饰器是一种特殊类型的声明，它能够被附加到类声明，方法， 访问符，属性或参数上。是一种在不改变原类和使用继承的情况下，动态地扩展对象功能。

## 命名空间

与 JS 一样，TS 中，任何包含顶级`import`或者`export`的文件都被当成一个模块。

与 TS 的区别：

- 命名空间是位于全局命名空间下的一个普通的带有名字的`JavaScript`对象，使用起来十分容易。但它很难去识别组件之间的依赖关系，尤其是在大型的应用中
- 像命名空间一样，模块可以包含代码和声明。 不同的是模块可以声明它的依赖
- 在正常的 TS 项目开发过程中并不建议用命名空间，但通常在通过`d.ts`文件标记 js 库类型的时候使用命名空间，主要作用是给编译器编写代码的时候参考使用
