---
title: FSharp-06-Function_Advance
date: 2023-02-20 11:40:29
tags: 
  - FSharp
categories: 
  - Language
password: zzy   
message: 亲，能不能输入密码啊？
---
# Recursive Function

> Fibonacci

recursion
```fsharp
// Fibonacci 1 1 2 3 5 ...
let rec f x =
    match x with
    | 1 -> 1 
    | 2 -> 1
    | _ -> f (x-1) + f (x-2)
```

> sum

conventional

```fsharp
let mutable x =0
for i in [1..100] do x <- x+ i
x

// 5050
```
recursion

```fsharp
let rec sum all result = 
    match all with 
    | head::tail -> 
        sum tail (result+head)
    | [] -> result

sum [1..100] 0
// 5050    
```

> exercise

```fsharp
// 求1..100的乘积
// example1 
let mutable x = 1
for i in [1..10] do x <- x * i
x

// example2
let prod all =
    let rec prod all ans = 
        match all with
        | head::tail -> prod tail (ans * head)
        | [] -> ans
    prod all 1  
// invoke          
prod [1..10]
```

# high order functions

## map


```fsharp
List.map (fun x -> x* x) [0..20]
```

| *index*      | value |
| ------------ | ----- |
| 0            | `0`   |
| 1            | `1`   |
| 2            | `4`   |
| 3            | `9`   |
| 4            | `16`  |
| 5            | `25`  |
| 6            | `36`  |
| 7            | `49`  |
| 8            | `64`  |
| 9            | `81`  |
| 10           | `100` |
| 11           | `121` |
| 12           | `144` |
| 13           | `169` |
| 14           | `196` |
| 15           | `225` |
| 16           | `256` |
| 17           | `289` |
| 18           | `324` |
| 19           | `361` |
| *... (more)* |       |

```fsharp
[0..20]
|>List.map (fun x -> x * x) 
|>printfn "%A"

// result
[0; 1; 4; 9; 16; 25; 36; 49; 64; 81; 100; 121; 144; 169; 196; 225; 256; 289; 324;
 361; 400]
```

## mapi

```fsharp
[0..20]
|>List.mapi (fun index x ->     
    printfn "%A\t %A" index x
    index + x
    ) 
```

## filter

```fsharp
List.filter (fun x -> x%2=0) [0..20]
```
| *index* | value |
| ------- | ----- |
| 0       | `0`   |
| 1       | `2`   |
| 2       | `4`   |
| 3       | `6`   |
| 4       | `8`   |
| 5       | `10`  |
| 6       | `12`  |
| 7       | `14`  |
| 8       | `16`  |
| 9       | `18`  |
| 10      | `20`  |

```fsharp
[0..20]
|>List.filter (fun x -> x%2=0) 
|>printfn "%A"

// result
[0; 2; 4; 6; 8; 10; 12; 14; 16; 18; 20]
```

```fsharp
[0..20]
|>List.filter (fun x -> x%2=0) 
|>List.map (fun x -> x * x) 
|>printfn "%A"

// result
[0; 4; 16; 36; 64; 100; 144; 196; 256; 324; 400]
```

## partition

```fsharp
List.partition (fun x -> x%2=0) [0..20]
```

| Item1                                       | Item2                                   |
| ------------------------------------------- | --------------------------------------- |
| `[ 0, 2, 4, 6, 8, 10, 12, 14, 16, 18, 20 ]` | `[ 1, 3, 5, 7, 9, 11, 13, 15, 17, 19 ]` |

## iter

```fsharp
List.iter (fun x -> printfn "%A" x) [0..20]

// result
    0
    1
    2
    3
    4
    5
    6
    7
    8
    9
    10
    11
    12
    13
    14
    15
    16
    17
    18
    19
    20
```
## iteri

```fsharp
[0..20]
|>List.iteri (fun index x -> printfn "%A\t %A" index x) 

// result
    0	 0
    1	 1
    2	 2
    3	 3
    4	 4
    5	 5
    6	 6
    7	 7
    8	 8
    9	 9
    10	 10
    11	 11
    12	 12
    13	 13
    14	 14
    15	 15
    16	 16
    17	 17
    18	 18
    19	 19
    20	 20
```

## allPairs

```fsharp
// 配对
List.allPairs [1..5] [1..5]
```
| *index*      | Item1 | Item2 |
| ------------ | ----- | ----- |
| 0            | `1`   | `1`   |
| 1            | `1`   | `2`   |
| 2            | `1`   | `3`   |
| 3            | `1`   | `4`   |
| 4            | `1`   | `5`   |
| 5            | `2`   | `1`   |
| 6            | `2`   | `2`   |
| 7            | `2`   | `3`   |
| 8            | `2`   | `4`   |
| 9            | `2`   | `5`   |
| 10           | `3`   | `1`   |
| 11           | `3`   | `2`   |
| 12           | `3`   | `3`   |
| 13           | `3`   | `4`   |
| 14           | `3`   | `5`   |
| 15           | `4`   | `1`   |
| 16           | `4`   | `2`   |
| 17           | `4`   | `3`   |
| 18           | `4`   | `4`   |
| 19           | `4`   | `5`   |
| *... (more)* |       |       |

## groupBy

```fsharp
[1;2;3;1;2;3;1;1;1;1;2;2;2;2;3;3;3]
|>List.groupBy (fun i -> i)
```
| *index* | Item1 | Item2                  |
| ------- | ----- | ---------------------- |
| 0       | `1`   | `[ 1, 1, 1, 1, 1, 1 ]` |
| 1       | `2`   | `[ 2, 2, 2, 2, 2, 2 ]` |
| 2       | `3`   | `[ 3, 3, 3, 3, 3 ]`    |

```fsharp
[1;2;3;1;2;3;1;1;1;1;2;2;2;2;3;3;3]
|>List.groupBy (fun i -> i)
|>List.map (fun (k,data)->k,data.Length)
```
| *index* | Item1 | Item2 |
| ------- | ----- | ----- |
| 0       | `1`   | `6`   |
| 1       | `2`   | `6`   |
| 2       | `3`   | `5`   |

```fsharp
[1;2;3;1;2;3;1;1;1;1;2;2;2;2;3;3;3]
|>List.groupBy (fun i -> i%2)
```

| *index* | Item1 | Item2                                 |
| ------- | ----- | ------------------------------------- |
| 0       | `1`   | `[ 1, 3, 1, 3, 1, 1, 1, 1, 3, 3, 3 ]` |
| 1       | `0`   | `[ 2, 2, 2, 2, 2, 2 ]`                |

## collect

```fsharp
[0..10]
|>List.collect (fun x -> [x*10 .. x*10+10])
```
| *index*      | value |
| ------------ | ----- |
| 0            | `0`   |
| 1            | `1`   |
| 2            | `2`   |
| 3            | `3`   |
| 4            | `4`   |
| 5            | `5`   |
| 6            | `6`   |
| 7            | `7`   |
| 8            | `8`   |
| 9            | `9`   |
| 10           | `10`  |
| 11           | `10`  |
| 12           | `11`  |
| 13           | `12`  |
| 14           | `13`  |
| 15           | `14`  |
| 16           | `15`  |
| 17           | `16`  |
| 18           | `17`  |
| 19           | `18`  |
| *... (more)* |       |

>Notice:
    Seq has all functions like List
    99%