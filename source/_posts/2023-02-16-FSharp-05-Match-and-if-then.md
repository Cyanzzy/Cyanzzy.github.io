---
title: FSharp-05-Match_and_if_then
date: 2023-02-16 14:28:13
tags: 
  - FSharp
categories: 
  - Language
---
# if then else

```fsharp
// define
let f x =
    if x % 2 = 0
    then 
        "Even"
    else 
        "Odd"

// invoke  
f 1 
Odd      
```
```fsharp
let g x =
    if x =1 then "One"
    elif x=2 then "Two"
    elif x=3 then "Three"
    else "Other"

// Other
g 11
```

> 练习：<br>
>   string -> int<br>
>   "one" -> 1<br>
>   "two" -> 2<br>
>   "three" -> 3<br>
>   _ -> 0

# match with

```fsharp
let f x =
    match (x % 2) with
    | 0 -> "Even"
    | 1 -> "Odd"

f 2
// Even    
```

```fsharp
let g x = 
    match x with
    | 1 -> "One"
    | 2 -> "Two"
    | 3 -> "Three"
    | _ -> "Other"

g 7
// Other
```

```fsharp
let h x = 
    match x with
    | 1 | 2 | 3-> "OneTwoThree"    
    | _ -> "Others"

h 2   
```

```fsharp
let compare x y = 
    match x with
    | _ when x<y -> "smaller"
    | _ when x>y -> "larger"
    | _ -> "equal"

compare 1 2
```
> 练习
> h: a: int -> b:  int -> unint 
> h: a: int * b: int -> unit 
> if a is odd then print "a = b + c" 
> if a is even then print "a = b - c"

```fsharp
let g x =
    match x with 
    | (var1, var2) when var1 % 2 = 0 -> printfn"%d = %d + (%d)" var1 var2 (var1 + var2)
    | (var1, var2) when var1 % 2 = 1 -> printfn"%d = %d - %d" var1 var2 (var1 - var2)
```

