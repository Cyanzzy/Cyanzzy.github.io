---
title: FSharp-03-Collections
date: 2023-02-14 15:10:05
tags: 
  - FSharp
categories: 
  - Language
swiper_index: 
---
# List
> <font color = "red">**List元素默认不可变**</font>

## 定义List

* `let list = []`，空List
* `let list = [1..10]`，默认步长的List
* `let list = [1..2..10]`，步长为2的List
* `let list = [1;2;3]`，给定元素的List
* **注意`list = [1,2,3]`则是元组**

> exapmle

```fsharp
// 定义List
let list =[1..10]
list
```
| *index* | value |
| ------- | ----- |
| 0       | `1`   |
| 1       | `2`   |
| 2       | `3`   |
| 3       | `4`   |
| 4       | `5`   |
| 5       | `6`   |
| 6       | `7`   |
| 7       | `8`   |
| 8       | `9`   |
| 9       | `10`  |

```fsharp
// print
printfn $"%A{list}"
// answer
[1; 2; 3; 4; 5; 6; 7; 8; 9; 10]
```

```fsharp
// 步长-2
let list = [10..-2..0]
list
```
| *index* | value |
| ------- | ----- |
| 0       | `10`  |
| 1       | `8`   |
| 2       | `6`   |
| 3       | `4`   |
| 4       | `2`   |
| 5       | `0`   |

## 切片操作
```fsharp
let list = [0..10]
// [0; 1; 2; 3; 4; 5; 6; 7; 8; 9; 10]
printfn $"%A{list}"
list[1..2]
```
| *index* | value |
| ------- | ----- |
| 0       | `1`   |
| 1       | `2`   |

```fsharp
let list = [0..10]
// [0; 1; 2; 3; 4; 5; 6; 7; 8; 9; 10]
printfn $"%A{list}"
list[..2]
```
| *index* | value |
| ------- | ----- |
| 0       | `0`   |
| 1       | `1`   |
| 2       | `2`   |

```fsharp
let list = [0..10]
// [0; 1; 2; 3; 4; 5; 6; 7; 8; 9; 10]
printfn $"%A{list}"
list[5..]
```
| *index* | value |
| ------- | ----- |
| 0       | `5`   |
| 1       | `6`   |
| 2       | `7`   |
| 3       | `8`   |
| 4       | `9`   |
| 5       | `10`  |

```fsharp
let list = [0..10]
// [0; 1; 2; 3; 4; 5; 6; 7; 8; 9; 10]
printfn $"%A{list}"
list[5..2]
```
(empty)

```fsharp
let list = [for i in 0 .. 10 -> i*i]
list
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

## 遍历
> API 

* list[1]
* list.Item(1)
* list.IsEmpty
* list.Length
* list.Head
* list.Tail
* List.contains 1 list

> example

```fsharp
let list = [1; 2; 3]

// list.IsEmpty is false
printfn "list.IsEmpty is %b" (list.IsEmpty)

// list.Length is 3
printfn "list.Length is %d" (list.Length)

// list.Head is 1
printfn "list.Head is %d" (list.Head)

// list.Tail is [2; 3]
printfn "list.Tail is %A" (list.Tail)

// list.Tail.Head is 2
printfn "list.Tail.Head is %d" (list.Tail.Head)

// list.Tail.Tail.Head is 3
printfn "list.Tail.Tail.Head is %d" (list.Tail.Tail.Head)

// list.Item(1) is 2
printfn "list.Item(1) is %d" (list.Item(1)) 

// True
List.contains 1 list
```

## 拼接
> API 
* 用 `::`拼接元素
* 用 `@` 连接两个List
* List.zip list1 list2
* let list4, list5 = List.unzip list3

> example

```fsharp
// List使用;分隔，用'[]'包住
let list1 = ["a"; "b"]
// ["a"; "b"]
printfn $"%A{list1}"

// 用 '::'拼接元素
let list2 = "c" :: list1
// ["c"; "a"; "b"] 
printfn $"%A{list2}"

// 用'@'连接两个List
let list3 = list1 @ list2
// ["a"; "b"; "c"; "a"; "b"]
printfn $"%A{list3}"
```
```fsharp
// zip 
let list1 = [1; 2; 3]
let list2 = [4; 5; 6]
let list3 = List.zip list1 list2
// [(1, 4); (2, 5); (3, 6)]
printfn $"{list3}"
```
| *index* | Item1 | Item2 |
| ------- | ----- | ----- |
| 0       | `1`   | `4`   |
| 1       | `2`   | `5`   |
| 2       | `3`   | `6`   |

```fsharp
// unzip 
let list1 = [1; 2; 3]
let list2 = [4; 5; 6]
//zip
let list3 = List.zip list1 list2
// [(1, 4); (2, 5); (3, 6)]
printfn $"{list3}"
// unzip
let list4, list5 = List.unzip list3
// [1; 2; 3]
printfn $"{list4}"
// [4; 5; 6]
printfn $"{list5}"
```
> exercise

```fsharp
// 构造1到100的奇数List 
let oddList = [1..2..100]
printfn $"%A{oddList}"

// 构造1到100的偶数List
let evenList = [2..2..100]
printfn $"%A{evenList}"
```
# Seq
> <font color = "red">**Sequence是一个可枚举的序列，不支持切割操作**</font>

## 定义Seq
```fsharp
// Sequences can use yield and contain subsequences
let seq1 = 
    seq {
        // "yield" adds one element
        yield 0
        yield 1
        yield 2

        // "yield!" adds a whole subsequence
        yield! [3..10]
    }
seq1
```
## 遍历
* Seq.isEmpty
* Seq.length
* Seq.head
* Seq.tail
* Seq.item 1

```fsharp
let seq1 = seq{0..10}
// seq [0; 1; 2; 3; ...]
printfn $"%A{seq1}"

// functions
// Seq.isEmpty is false
printfn "Seq.isEmpty is %b" (Seq.isEmpty seq1)
// Seq.length is 11
printfn "Seq.length is %d" (seq1 |>Seq.length)
// Seq.head is 0
printfn "Seq.head is %d" (seq1 |>Seq.head)
// Seq.tail.head is 1
printfn "Seq.tail.head is %d" (seq1 |>Seq.tail   |>Seq.head)
// Seq.tail.tail.head is 2
printfn "Seq.tail.tail.head is %d" (seq1 |>Seq.tail |>Seq.tail |>Seq.head)
// Seq.item 1  is 1
printfn "Seq.item 1  is %d" (seq1 |>Seq.item 1)
```
```fsharp
seq1
```
| *index* | value |
| ------- | ----- |
| 0       | `0`   |
| 1       | `1`   |
| 2       | `2`   |
| 3       | `3`   |
| 4       | `4`   |
| 5       | `5`   |
| 6       | `6`   |
| 7       | `7`   |
| 8       | `8`   |
| 9       | `9`   |
| 10      | `10`  |

## List与Seq
```fsharp
let list1 = [ 1; 2; 3 ]
// functions
printfn "Seq."
printfn "List.isEmpty is %b" (Seq.isEmpty list1)
printfn "List.length is %d" (list1 |>Seq.length)
printfn "List.head is %d" (list1 |>Seq.head)
printfn "List.tail.head is %d" (list1 |>Seq.tail   |>Seq.head)
printfn "List.tail.tail.head is %d" (list1 |>Seq.tail |>Seq.tail |>Seq.head)
printfn "List.item 1  is %d" (list1 |>Seq.item 1)

Seq.
List.isEmpty is false
List.length is 3
List.head is 1
List.tail.head is 2
List.tail.tail.head is 3
List.item 1  is 2

------------------------------------------------------------
// functions
printfn "List."
printfn "List.isEmpty is %b" (List.isEmpty list1)
printfn "List.length is %d" (list1 |>List.length)
printfn "List.head is %d" (list1 |>List.head)
printfn "List.tail.head is %d" (list1 |>List.tail   |>List.head)
printfn "List.tail.tail.head is %d" (list1 |>List.tail |>List.tail |>List.head)
printfn "List.item 1  is %d" (list1 |>List.item 1)

List.
List.isEmpty is false
List.length is 3
List.head is 1
List.tail.head is 2
List.tail.tail.head is 3
List.item 1  is 2
```

> Seq与List进行转化

```fsharp
let list1= [0..10]
let seq1 = list1 |> Seq.ofList
let seq2 = list1 |> List.toSeq
```

# Set
> <font color = "red">**Set中元素不可重复**</font>

```fsharp
let set_1 = [1;2;3] |> set
let set_2 = [1;2;3] |> set
// set [1; 2; 3]
printfn $"{set_1}"
// set [1; 2; 3]
printfn $"{set_2}"
```

## API
* set_1.IsSubsetOf(set_2)
* set_1.IsSupersetOf(set_2)
* set_1.IsProperSubsetOf(set_2)
* set_1.IsProperSupersetOf(set_2)
* set0.Count
* set_1 = set_2 // 比较结构相等（值）基础类型
* let set_3 = set_1.Add(0)
* Set.union set_4 set_3
* Set.intersect set_4 set_3

# Map

```fsharp
let m=Map.ofList [(1,"cyan")]
m
```
| *key* | value |
| ----- | ----- |
| `1`   | cyan |

```fsharp
let m=
    [
    (1,"cyan")
    2,"gauss"
    ]
    
    |> Map.ofList  
m
```
| *key* | value  |
| ----- | ------ |
| `1`   | cyan  |
| `2`   | gauss |

```fsharp
m.Add(1,"X")
```
| *key* | value  |
| ----- | ------ |
| `1`   | X      |
| `2`   | gauss |

```fsharp
let m2=m.Add(3,"csharp")
m2
```
| *key* | value  |
| ----- | ------ |
| `1`   | cyan  |
| `2`   | gauss |
| `3`   | csharp  |

```fsharp
m2.Keys
m2.Values

// [[1, cyan]; [2, gauss]; [3, csharp]]
m2 |>List.ofSeq |> printfn "%A"
m2.TryFind(1)
m2.TryFind(4)
```

```fsharp
let m = [for i in 1..100 -> (string(i), i*i)] |> Map
```

| *key*        | value   |
| ------------ | ------- |
| 1            | `1`     |
| 10           | `100`   |
| 100          | `10000` |
| 11           | `121`   |
| 12           | `144`   |
| 13           | `169`   |
| 14           | `196`   |
| 15           | `225`   |
| 16           | `256`   |
| 17           | `289`   |
| 18           | `324`   |
| 19           | `361`   |
| 2            | `4`     |
| 20           | `400`   |
| 21           | `441`   |
| 22           | `484`   |
| 23           | `529`   |
| 24           | `576`   |
| 25           | `625`   |
| 26           | `676`   |
| *... (more)* |         |