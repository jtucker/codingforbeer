---
title: "Advent of Code 2020 - Day 1"
date: 2020-12-02T09:47:58-05:00
description: "My thoughts on solving Advent of Code 2020 - Day 1 with F#"
tags: ["adventofcode", "fsharp", "code"]
series: ["Advent of Code 2020"]
draft: false
---


### Problem
From the [website](https://adventofcode.com/2020/day/1): 

#### Part 1
> Before you leave, the Elves in accounting just need you to fix your expense report (your puzzle input); apparently, something isn't quite adding up.
> Specifically, they need you to find the two entries that sum to 2020 and then multiply those two numbers together.

#### Part 2
> The Elves in accounting are thankful for your help; one of them even offers you a starfish coin they had left over from a past vacation. They offer you a second one if you can find three numbers in your expense report that meet the same criteria.

### Solution
{{< highlight fsi >}}
let collectFind list collector finder =
    list 
    |> List.collect (collector) 
    |> List.find (finder)

let pairsCollector x = entries |> List.map (fun y -> (x, y))
let pairsFinder (x, y) = x + y = 2020

let triplesCollector x = entries |> List.collect (fun y -> entries |> List.map (fun z -> (x, y, z)))
let triplesFinder (x, y, z) = x + y + z = 2020

let x, y = collectFind entries pairsCollector pairsFinder
printfn "Answer Part 1: %i * %i = %i" x y (x * y)

let a, b, c = collectFind entries triplesCollector triplesFinder
printfn "Answer Part 2: %i * %i * %i = %i" a b c (a * b * c)

{{< /highlight >}}

### Thoughts on why I did what I did

This actually wasn't the initial solution I came up with. My first thought was something like a `find_map` :

```fsharp
let rec find_map f a l =
    match a with
    | [] -> None
    | x :: y ->
        match f x y with
        | Some _ as result -> result
        | None -> find_map f y l
```
Which when I then supplied with the following function worked quite well.
```fsharp
let processor item listToSearch =
    let diff = 2020 - item
    match List.contains diff listToSearch with
    | false -> None
    | true -> Some(item, diff)
```
It wasn't until the second part of Day 1 that I started to rethink my solution. I started down the road of recursion and trying something like that when it finally dawned on me that all I really want is a giant list of all possible combinations of the entries, either in pairs or triplicate, and then just find the one combo that equals 2020. 

