---
title: FSharp-02-Functions
date: 2023-02-13 15:54:36
tags: 
  - FSharp
categories: 
  - Language
---
> function basics, signature

# 函数的基本定义

> 不声明类型的函数

```fsharp
// example 1 
let f x  = x + 1

// example 2
let cyliderVolume radius length = 
	let pi = 3.1415
	length * pi * radius * radius
```


> 显式声明类型的函数

```fsharp
// example 1 
let f (x:int)  = x + 1

// example 2
let cyliderVolume (radius:float) (length:float) " float = 
	let pi = 3.1415
	length * pi * radius * radius
```

>  单参数函数

```fsharp
let f x = x * x

// invoke
f 10.0

// result
100
```
> 多参数函数

```fsharp
let g a b = a + b

// invoke
g 1 2

// result
3

let h (a, b, c) =
    let v = a + b + c
    let u = a * b * c
    u / v

// invoke
h(10, 9, 8)  

// result
26  
```

# 函数签名

```fsharp
// int -> int 
let f x = x * x

// int -> int -> int
let g a b = a + b

// int * int * int -> int
let h (a, b, c) = 
    let v = a + b + c
    let u = a * b * c
    u/v

// float * float * float -> float
let h (a:float, b, c) =
    let v = a + b + c
    let u = a * b * c
    u/v   
```
> 注意区别

```fsharp
// float * float * float -> int
let h (a:float, b, c) :int =
    let v = a + b + c
    let u = a * b * c
    let r = u/v
    int(r)   
    // float -> float -> float -> int

// float -> float -> float -> int    
let h (a:float) (b:float) (c:float) :int =
    let v = a + b + c
    let u = a * b * c
    let r = u/v
    int(r)
```

# 部分函数

> Curry （函数可理化） 

```fsharp
// val sumFunction:
//    a: int ->
//    b: int ->
//    c: int
//    -> int
// Full name: sumFunction
let sumFunction a b c = a + b + c + 1
let sumF1 = sumFunction 1
let sumF2 = sumF1 2
let sumF3 = sumF2 3
sumF3

// result
7
```

```fsharp
// float -> float -> float -> float
let h0 a b c = a + b + c
// float -> float -> float
let h1 = h0 10.0
// float -> float 
let h2 = h1 5.0
// float 
let h3 = h2 1.0
h3
```

# 函数作为参数

> 定义`apply1`与`apply2`：

```fsharp
// apply1
let apply1 (transform : int -> int) y = transform y

// apply2
let apply2 = (f:int -> int -> int) x y = f x y 

let mul = x y = x * y

let result2 = apply2 mul 10 20
```

> example

```fsharp
// val fun_i:
//    fun_j: (int -> int) ->
//    data : int
//        -> int
// Full name: fun_i
let fun_i (fun_j :int -> int) (data :int) :int =
    fun_j data

// invoke
let f_i x = x + x
fun_i f_i 10

// result 
10
```

# lambda 函数

``` fsharp
// 函数作为参数
let apply1 (transform : int -> int) y = transform y
let apply2 = (f:int->int->int) x y = f x y 

let mul = x y = x * y

let result2 = apply2 mul 10 20

// lambda 函数
let result3 = apply1(fun x -> x + 1) 100
let result4 = apply2(fun x y -> x * y) 10 20
```
> example

```fsharp
let f_1 = fun x -> x* x

// invoke
f_1 10

// result
100
```

```fsharp
let fun_i (fun_j :int -> int) (data :int) :int =
    fun_j data

fun_i (fun x -> x * 100) 10

// result 
1000
```

# 输出函数

```fsharp
// 1
printf "1"

// 1
printfn "1"

// 123	10M	"test"
printfn "%A\t%A\t%A\t" 123 10m "test"

// 123	10M	"Cyan Chau"
let str = "Cyan Chau"
printfn $"%A{123}\t%A{10m}\t%A{str}"
```