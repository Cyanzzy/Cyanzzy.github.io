---
title: FSharp-09-DataStructure
date: 2023-02-27 15:35:36
tags: 
  - FSharp
categories: 
  - Language
---

# Record

> example1

```fsharp
// Labels are separated by semicolons when defined on the same line.
type Point = { X: float; Y: float; Z: float; }
```
```fsharp
let mypoint = { X = 1.0; Y = 1.0; Z = -1.0; }
mypoint
```
> example2

```fsharp
// You can define labels on their own line with or without a semicolon.
type Customer = 
    { First: string
      Last: string;
      SSN: uint32
      AccountNumber: uint32; }
```
> example3

```fsharp
type Point = { X: float; Y: float; Z: float; }
type Point3D = { X: float; Y: float; Z: float }
// Ambiguity: Point or Point3D?
let mypoint3D = { X = 1.0; Y = 1.0; Z = 0.0; }
```
```fsharp
let myPoint1 = { Point.X = 1.0; Y = 1.0; Z = 0.0; }
```
> example4

```fsharp
type MyRecord = 
    { X: int
      Y: int
      Z: int }

let myRecord1 = { X = 1; Y = 2; Z = 3; }
myRecord1
```
```fsharp
let myRecord2 = { MyRecord.X = 1; MyRecord.Y = 2; MyRecord.Z = 3 }
myRecord2
```
```fsharp
let myRecord3 = { myRecord2 with Y = 100; Z = 2 }
myRecord3
```
> example5

```fsharp
type Car = 
    { Make : string
      Model : string
      mutable Odometer : int }

let myCar = { Make = "Fabrikam"; Model = "Coupe"; Odometer = 108112 }
myCar.Odometer <- myCar.Odometer + 21
myCar
```
> example6

```fsharp
// Rather than use [<DefaultValue>], define a default record.
type MyRecord =
    { Field1 : int
      Field2 : int }

let defaultRecord1 = { Field1 = 0; Field2 = 0 }
let defaultRecord2 = { Field1 = 1; Field2 = 25 }

// Use the with keyword to populate only a few chosen fields
// and leave the rest with default values.
let rr3 = { defaultRecord1 with Field2 = 42 }
rr3
```
> example7

```fsharp
// Create a Person type and use the Address type that is not defined
type Person =
  { Name: string
    Age: int
    Address: Address }
// Define the Address type which is used in the Person record
and Address =
  { Line1: string
    Line2: string
    PostCode: string
    Occupant: Person }
```
> example8

```fsharp
// Create a Person type and use the Address type that is not defined
let rec person =
  {
      Name = "Person name"
      Age = 12
      Address =
          {
              Line1 = "line 1"
              Line2 = "line 2"
              PostCode = "abc123"
              Occupant = person
          }
  }
person
```

> example9

```fsharp
type Person =
  { Name: string
    Age: int
    Address: string }

    static member Default =
        { Name = "Phillip"
          Age = 12
          Address = "123 happy fun street" }

let defaultPerson = Person.Default
defaultPerson
```
> example10

```fsharp
type Person =
  { Name: string
    Age: int
    Address: string }

let Default(name) =
    {     Name = $"{name}"
          Age = 12
          Address = "123 happy fun street" }

let defaultPerson = Default "Cyan"
defaultPerson
```
> example11

```fsharp
type Person =
  { Name: string
    Age: int
    Address: string }

    member this.WeirdToString() =
        this.Name + this.Address + string this.Age

let p = { Name = "a"; Age = 12; Address = "abc123" }
let weirdString = p.WeirdToString()
weirdString
```

**Differences Between Records and Classes**

Record fields differ from class fields in that they are automatically exposed as properties, and they are used in the creation and copying of records. Record construction also differs from class construction. In a record type, you cannot define a constructor. Instead, the construction syntax described in this topic applies. Classes have no direct relationship between constructor parameters, fields, and properties.

Like union and structure types, records have structural equality semantics. Classes have reference equality semantics. The following code example demonstrates this.

```fsharp
type RecordTest = { X: int; Y: int }

let record1 = { X = 1; Y = 2 }
let record2 = { X = 1; Y = 2 }

if (record1 = record2) then
    printfn "The records are equal."
else
    printfn "The records are unequal."
```
# Anonymous Records

```fsharp
open System

let getCircleStats radius =
    let d = radius * 2.0
    let a = Math.PI * (radius ** 2.0)
    let c = 2.0 * Math.PI * radius

    {| Diameter = d; Area = a; Circumference = c |}

let r = 2.0
let stats = getCircleStats r
printfn "Circle with radius: %f has diameter %f, area %f, and circumference %f"
    r stats.Diameter stats.Area stats.Circumference
```

# Discriminated Unions

> example

```fsharp
type Shape =
    | Rectangle of width : float * length : float
    | Circle of radius : float
    | Prism of width : float * float * height : float
```
```fsharp
let rect = Rectangle(length = 1.3, width = 10.0)
let circ = Circle (1.0)
let prism = Prism(5., 2.0, height = 3.0)
```

> option

```fsharp
// The option type is a discriminated union.
type Option<'a> =
    | Some of 'a
    | None

let div a b = 
    match b with 
    | 0.0 -> None
    | _ -> Some(a/b)  

let r1 = div 10 1
if r1.IsSome then r1.Value else 0.0
// r1      

let printValue opt =
    match opt with
    | Some x -> printfn "%A" x
    | None -> printfn "No value."

printValue r1    
```


```fsharp
let f1 x = x + 1.0
let f2 x = x + 2.0
let f3 x = x + 3.0

r1 
|> Option.map f1
|> Option.map f2
|> Option.map f3
|> printValue
```

```fsharp
let getShapeWidth shape =
    match shape with
    | Rectangle(width = w) -> w
    | Circle(radius = r) -> 2. * r
    | Prism(width = w) -> w
```

> 练习：<br>
定义一个用户登录数据Record<br>
定义一个男女DU<br>
定义一个学历DU（高中 of string，本科 of float，硕士）

```fsharp
type Sex = 
    | Male
    | Female

type Education =
    | Senior  of string
    | Undergraduate  of float
    | Master    

type User = {
    Name: string
    Password: string
    Sex: Sex
    Education: Education
}    

let user = {
    Name = "CyanChau";
    Password = "root";
    Sex = Male
}
user

let student = {
    Name = "CyanChau";
    Password = "root";
    Sex = Male;
    Education =  Master
}
student
```

# interface

```fsharp
type IPrintable =
   abstract member Print : unit -> unit
```
Implementing Interfaces by Using Object Expressions

```fsharp
let makePrintable(x: int, y: float) =
    { new IPrintable with
              member this.Print() = printfn "%d %f" x y }
let x3 = makePrintable(1, 2.0)
x3.Print()
```
```fsharp
type Interface1 =
    abstract member Method1 : int -> int

type Interface2 =
    abstract member Method2 : int -> int

type Interface3 =
    inherit Interface1
    inherit Interface2
    abstract member Method3 : int -> int
```
> 练习： <br>写一个函数实现接口Interface3

```fsharp
let makeI3(x:int, y: float) =
    {
        new Interface3 with
            member this.Method3(i) = i + 3 + x
            member this.Method2(i) = i + 2 + x
            member this.Method1(i) = i + 1 + x
    }
let i = makeI3 (10, 1.0)
i.Method1(5)   
```