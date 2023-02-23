---
title: FSharp-07-Collections-Mutable
date: 2023-02-22 21:25:02
tags: 
  - FSharp
categories: 
  - Language
swiper_index: 
---
# Array

## 创建操作
> 步长为1的数组

```fsharp
let a = [|0..10|]
printfn $"%A{a}"

[|0; 1; 2; 3; 4; 5; 6; 7; 8; 9; 10|]
```

> 步长为2的数组

```fsharp
let a = [|1..2..10|]
// [|1; 3; 5; 7; 9|]
printfn $"%A{a}"
a
```
| *index* | value |
| ------- | ----- |
| 0       | `1`   |
| 1       | `3`   |
| 2       | `5`   |
| 3       | `7`   |
| 4       | `9`   |

> 步长为-2的数组

```fsharp
let a=[|10.. -2 ..0|]
a
```
| *index* | value |
| ------- | ----- |
| 0       | `10`  |
| 1       | `8`   |
| 2       | `6`   |
| 3       | `4`   |
| 4       | `2`   |
| 5       | `0`   |

> 创建数组的其他方式 

```fsharp
let arr123 = [|
    1
    2
    3 |]
arr123
```
| *index* | value |
| ------- | ----- |
| 0       | `1`   |
| 1       | `2`   |
| 2       | `3`   |


```fsharp
let arrOfSquares = [| for i in 0 .. 10 -> i*i |]
arrOfSquares
```
| *index* | value |
| ------- | ----- |
| 0       | `0`   |
| 1       | `1`   |
| 2       | `4`   |
| 3       | `9`   |
| 4       | `16`  |
| 5       | `25`  |
| 6       | `36`  |
| 7       | `49`  |
| 8       | `64`  |
| 9       | `81`  |
| 10      | `100` |

## 切片操作

> a[1..5]

```fsharp
let a=[|0..10|]
// [|0; 1; 2; 3; 4; 5; 6; 7; 8; 9; 10|]
printfn $"%A{a}"
a[1..5]
```
| *index* | value |
| ------- | ----- |
| 0       | `1`   |
| 1       | `2`   |
| 2       | `3`   |
| 3       | `4`   |
| 4       | `5`   |

> a[..2]

```fsharp
let a=[|0..10|]
// [|0; 1; 2; 3; 4; 5; 6; 7; 8; 9; 10|]
printfn $"%A{a}"

a[..2]
```
| *index* | value |
| ------- | ----- |
| 0       | `0`   |
| 1       | `1`   |
| 2       | `2`   |

> a[5..]

```fsharp
let a=[|0..1..10|]
a[5..]
```
| *index* | value |
| ------- | ----- |
| 0       | `5`   |
| 1       | `6`   |
| 2       | `7`   |
| 3       | `8`   |
| 4       | `9`   |
| 5       | `10`  |

> a[5..2]

```fsharp
let a=[|0..1..10|]
a[5..2]

// (empty)
```
## 常规操作

> Lists use square brackets and `;` delimiter

```fsharp
// Lists use square brackets and `;` delimiter
let arr1 = [| "a"; "b" |]
printfn $"%A{arr1}" 
```
[|"a"; "b"|]

> 添加元素

```fsharp
// :: is prepending
//let arr2 = "c" :: arr1
let arr2 = Array.append [|"c"|] arr1
printfn $"%A{arr2}"
```
[|"c"; "a"; "b"|]

> 连接数组

```fsharp
// @ is concat    
//let arr3 = arr1 @ arr2   
let arr3 = Array.append arr1 arr2
printfn $"%A{arr3}"
Array.append [|"c"|] arr1
```
[|"a"; "b"; "c"; "a"; "b"|]

| *index* | value |
| ------- | ----- |
| 0       | c     |
| 1       | a     |
| 2       | b     |

```fsharp
let arr1 = [| 1; 2; 3 |]

> 常见API

printfn $"%A{arr1}"
// functions
printfn "Array.isEmpty is %b" (Array.isEmpty arr1)
printfn "Array.length is %d" (arr1 |>Array.length)
printfn "Array.head is %d" (arr1 |>Array.head)
printfn "Array.tail.head is %d" (arr1 |>Array.tail   |>Array.head)
printfn "Array.tail.tail.head is %d" (arr1 |>Array.tail |>Array.tail |>Array.head)
printfn "Array.item 1  is %d" (arr1 |>Array.item 1)
arr1
```
[|1; 2; 3|]
Array.isEmpty is false
Array.length is 3
Array.head is 1
Array.tail.head is 2
Array.tail.tail.head is 3
Array.item 1  is 2
| *index* | value |
| ------- | ----- |
| 0       | `1`   |
| 1       | `2`   |
| 2       | `3`   |

> zip与unzip

```fsharp
// in[]

let arr1 = [| 1; 2; 3 |]
let arr2 = [| 4; 5; 6 |]
let arr3 = Array.zip arr1 arr2
printfn $"{arr3}"
// list<'t*'u> -> list<'t> * list<'u>
let arr4,arr5 = Array.unzip arr3
printfn $"{arr4}"
printfn $"{arr5}"

// out[] 

System.Tuple`2[System.Int32,System.Int32][]
System.Int32[]
System.Int32[]
```
**arr3**
| *index* | Item1 | Item2 |
| ------- | ----- | ----- |
| 0       | `1`   | `4`   |
| 1       | `2`   | `5`   |
| 2       | `3`   | `6`   |

**arr4**
| *index* | value |
| ------- | ----- |
| 0       | `1`   |
| 1       | `2`   |
| 2       | `3`   |

**arr5**
| *index* | value |
| ------- | ----- |
| 0       | `4`   |
| 1       | `5`   |
| 2       | `6`   |

> Contains

```fsharp
let arr1 = [| 1; 2; 3 |]
// value:'t -> 't list -> bool
Array.contains 1 arr1
// True
```
> 赋值

```fsharp
let arr1 = [| 1; 2; 3 |]
arr1[0] <- 10
arr1
```
| *index* | value |
| ------- | ----- |
| 0       | `10`  |
| 1       | `2`   |
| 2       | `3`   |

## 注意区分

**注意区别：逗号：元组，分号：元素**
> 元组

```fsharp
[|"a","b","c"|]
```
| *index* | Item1 | Item2 | Item3 |
| ------- | ----- | ----- | ----- |
| 0       | a     | b     | c     |

> 元素

```fsharp
[|"a";"b";"c"|]
```
| *index* | value |
| ------- | ----- |
| 0       | a     |
| 1       | b     |
| 2       | c     |

# Dictionary

> 定义字典

```fsharp
let d=
    [
    (1,"cyan")
    (2,"chau")
    ]
    
    |> dict

d    
```
| *key* | value |
| ----- | ----- |
| `1`   | cyan  |
| `2`   | chau  |

> 遍历操作

```fsharp
d[2]
```
chau

> 查找操作

```fsharp
d.ContainsKey(1)

my_dict.ContainsKey(1)
```
True

> 注意

```fsharp
// d.Add(1, "X")
//Error: System.NotSupportedException: 无法转变此值
```
> 添加操作

```fsharp
open System.Collections.Generic
let my_dict = Dictionary<_,_>(d)
my_dict.Add(3,"X")
my_dict
```
| *key* | value |
| ----- | ----- |
| `1`   | cyan  |
| `2`   | chau  |
| `3`   | X     |

```fsharp
// TryAdd 尝试添加，添加失败，返回原先字典
my_dict.TryAdd(3,"Y")
my_dict.TryAdd(1,"Y")
my_dict
```
| *key* | value |
| ----- | ----- |
| `1`   | cyan  |
| `2`   | chau  |
| `3`   | X     |

> 计数操作

```fsharp
my_dict.Count
```
3

> 取键操作

```fsharp
my_dict.Keys
```
| *index* | value |
| ------- | ----- |
| 0       | `1`   |
| 1       | `2`   |
| 2       | `3`   |

> 取值操作

```fsharp
my_dict.Values
```
| *index* | value |
| ------- | ----- |
| 0       | cyan  |
| 1       | chau  |
| 2       | X     |

# HashSet

> 创建操作

```fsharp
open System.Collections.Generic
let h = HashSet<_>([1;2;3])
h
```
| *index* | value |
| ------- | ----- |
| 0       | `1`   |
| 1       | `2`   |
| 2       | `3`   |

> 添加操作

```fsharp
h.Add(4)
h.Add(5)
h.Add(6)
h
```
| *index* | value |
| ------- | ----- |
| 0       | `1`   |
| 1       | `2`   |
| 2       | `3`   |
| 3       | `4`   |
| 4       | `5`   |
| 5       | `6`   |

> example

```fsharp
let set_1 = [1;2;3;0;-1;1;1;] |> HashSet
let set_2 = [1;2;3;5;6;7] |> HashSet
printfn $"%A{set_1}"
printfn $"%A{set_2}"
```
seq [1; 2; 3; 0; ...]
seq [1; 2; 3; 5; ...]

```fsharp
// 是否为子集
set_1.IsSubsetOf(set_2)
```
False

```fsharp
// 并集
set_1.UnionWith(set_2)
set_1
```
| *index* | value |
| ------- | ----- |
| 0       | `1`   |
| 1       | `2`   |
| 2       | `3`   |
| 3       | `0`   |
| 4       | `-1`  |
| 5       | `5`   |
| 6       | `6`   |
| 7       | `7`   |

```fsharp
// 交集
set_1.IntersectWith(set_2)
set_1
```
| *index* | value |
| ------- | ----- |
| 0       | `1`   |
| 1       | `2`   |
| 2       | `3`   |

# System.Collections.Generic

包含用于定义泛型集合的接口和类，可允许用户创建强类型集合，以提供比非泛型强类型集合更好的类型安全性和性能。
[官方文档](https://learn.microsoft.com/zh-cn/dotnet/api/system.collections.generic?view=net-7.0)