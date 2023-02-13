---
title: Fsharp-01-Values
date: 2023-02-12 10:02:00
update: 2023-02-12 10:02:00
tags: 
  - FSharp
categories: 
  - Language
swiper_index:  
---

> 大纲：int、string、float、float32、decimal、tuple

# 数据类型

> 补充：
>
> * F#一般使用`let`声明变量和定义函数
>
> * F#自动推断类型、无需显式声明变量类型
>
> * 使用`typeof<xxx>`查看类型
>
> * 数据类型转换
>
>   * `float(1.4m)`
>   * `int(1.4m)`
>   * `int(1.4)`
>   * `int "1"`
>
> * 关于格式字符补充
>
>   * `A` ：
>
>  ```fsharp
>  // %A 的使用 All
>  // 假设var与var2已经定义
>  let temp = $"First lecture, var = %A{var}, var1 = %A{var1}"
>  ```
>
>   * `@`：
>
>  ```fsharp
>  // @ 的使用 
>  let temp = @"D:\test\test\test"
>  
>  D:\test\test\test
>  
>  // 不使用`@`，则变为转义字符`\t`
>  let temp = "D:\test\test\test"
>  
>  D:	est	est	est
>  ```
>
>   * `"""..."""`:
>
>  ```fsharp
>  let temp =
>      """
>      test is good
>      \test\test\test
>      go go good@@@@
>      ""
>      ''
>      """
>      
>  test is good
>  \test\test\test
>  go go good@@@@
>  ""
>  ''    
>  ```
>
>  

## 整型 int

* 隐式定义 `let var = 10`

* 显示定义 `let var:int = 10`

## 浮点型 float

* 双精度浮点类型数据定义`let var = 10.90`

* 单精度浮点类型数据定义`let var = 4.5f`

## 定点型 decimal

* 浮点数小数点会变化

* 定点数小数点是不变的

* 定义decimal类型数据`let var = 1.0m`

> 浮点数与定点数区别演示

```fsharp
let dividend = 1.0m
dividend
let divisor = 3
(dividend/decimal(divisor) * decimal(divisor))

0.9999999999999999999999999999
```

```fsharp
let dividend = 1.0
let divisor = 3
(dividend/float(divisor) * float(divisor))

1
```

## 字符串型 string

* 定义字符串数据类型数据`let var = "Cyan Chau"`

## 元组型 tuple

* 元组数据类型数据定义`let var = ("Cyan Chau", 20, 1000M)`

**定义元组**

```fsharp
let var = ("Cyan Chau", 20, 1000M)
```

| Item1       | Item2 | Item3  |
| ----------- | ----- | ------ |
| `Cyan Chau` | `20`  | `1000` |

**访问元组**

```fsharp
// 访问元组
let name, age, money = var
name

Cyan Chau
```

