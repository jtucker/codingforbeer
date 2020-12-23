---
title: "Advent of Code 2020 - Day 3"
date: 2020-12-04T11:06:37-05:00
description: "My thoughts on solving Advent of Code 2020 - Day 3 with F#"
tags: ["adventofcode", "fsharp", "code"]
series: ["Advent of Code 2020"]
draft: true
---

### Problem
#### Part 1
> From your starting position at the top-left, check the position that is right 3 and down 1. Then, check the position that is right 3 and down 1 from there, and so on until you go past the bottom of the map.
> Starting at the top-left corner of your map and following a slope of right 3 and down 1, how many trees would you encounter?

#### Part 2
> Determine the number of trees you would encounter if, for each of the following slopes, you start at the top-left corner and traverse the map all the way to the bottom:

    - Right 1, down 1.
    - Right 3, down 1. (This is the slope you already checked.)
    - Right 5, down 1.
    - Right 7, down 1.
    - Right 1, down 2.

### Solution
{{< highlight fsi >}}
type Position = {
    First: int
    Second: int
}

let calcTrees arrContent slopes =
    let startState = ({ First = 0; Second = 0 }, 0)

    let movePosition slope position maxLength =
        let (right, down) = slope
        match position with
        | p when p.First + down > maxLength -> None
        | _ -> Some ({ position with First = position.First + down; Second = position.Second + right })

    let isTree =
        function | '#' -> 1
                | _ -> 0

    let rec arrFold slope state (array: char [,]) =
        let (position: Position), (trees:int) = state
        match movePosition slope position (Array2D.length1 array) with
        | None -> state
        | Some (p) -> arrFold slope (p, trees + (isTree array.[position.First, position.Second % 31])) array

    let treeMapper = (fun s -> arrFold s startState arrContent) >> (fun (p, trees) -> int64 trees)

    slopes 
    |> List.map (treeMapper) 
    |> List.reduce ((*))

let singleSlope = [(3, 1)]
let singleSlopeTreeCount = calcTrees arrContent singleSlope
printfn "Number of trees: %i" singleSlopeTreeCount

let slopes = [(1, 1); (3, 1); (5, 1); (7, 1); (1, 2)]
let slopesTreeCount = caclTrees arrContent slopes
printfn "Number of trees in all slopes: %i" slopesTreeCount

{{< /highlight >}}

### Thoughts on why I did what I did
This problem really broke down to moving through a grid at a defined `x` and `y` coordinate all the while incrementing a counter based on if the positional character was a tree or not. 

Sounds like a `fold` to me. 

The process of moving from Part 1 to Part 2 didn't really require much tweaking, I only ended up pulling out the slopes into an array of Tuples. The folding function starts with a state value tuple defined as `(Postion * int)`  and threads it through each time moving the `Position` type based on current slope. For each execution the position is tested to see if the `down` movement will exceed the length of the file and if not, is moved. The new position is then passed to the fold function while also determining if the new position contains a tree. 

The incoming data is read line by line from the file system and then piped into `array2d` function turning it into a 2 dimensional array of `char`. In hindsight, I should probably go back and rename the `Position` values (`First` and `Second`) as they are actually flipped when moving through the grid.