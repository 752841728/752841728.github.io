title: TypeScript学习笔记
author: Funny Boy
date: 2023-09-05 11:15:26
tags: ["JavaScript"]
categories: ["前端"]

---

# 前言

近期在学习TypeScript，这里记录一下

---
# 一、基础类型
boolean、number、string、null、undefined比较简单，不再阐述

array
```typescript
let arr: number[] = [1, 2, 3];
let arr: Array<number> = [1, 2, 3];
let arr: [number, string] = [1, '2']	// 元组tuple
```
enum
```typescript
// 异构枚举，既包含数字又包含字符串
enum Direction {
  NORTH = 0,
  SOUTH = 1,
  EAST = 2,
  WEST = 'A',
}

console.log(Direction.NORTH)    // 0
console.log(Direction[0])   // NORTH

// es5
var Direction;
(function (Direction) {
    Direction[Direction["NORTH"] = 0] = "NORTH";
    Direction[Direction["SOUTH"] = 1] = "SOUTH";
    Direction[Direction["EAST"] = 2] = "EAST";
    Direction["WEST"] = "A";
})(Direction || (Direction = {}));

console.log(Direction.NORTH); // 0
console.log(Direction[0]); // NORTH
```
any、unknown

```typescript
// 所有类型可以赋值给any，any可以赋值给所有类型
// 所有类型可以赋值给unknown，unknown不可以赋值给任何类型
let test: number = 123;

let test1: any = test;
let test2: number = test1;

let test3: unknown = test;
let test4: number = test3; // error

console.log(test1[0])
console.log(test3[0])   // error
```
void
```typescript
// 函数没有返回值时使用
function warnUser(): void {
  console.log("This is my warning message");
}
```
never
```typescript
// 确保类型绝对安全的代码，避免新增了联合类型，但没有对应的实现
type Foo = string | number;

function controlFlowAnalysisWithNever(foo: Foo) {
  if (typeof foo === "string") {
  } else if (typeof foo === "number") {
  } else {
    const check: never = foo;
  }
}

// 新增了Foo的类型，但没有对应的实现
type Foo = string | number | boolean;

function controlFlowAnalysisWithNever(foo: Foo) {
  if (typeof foo === "string") {
  } else if (typeof foo === "number") {
  } else {
    const check: never = foo;	// error
  }
}
```

# 二、类型断言

```typescript
// 两种写法
let someValue: any = "this is a string";
let strLength: number = (<string>someValue).length;
let strLength: number = (someValue as string).length;
```