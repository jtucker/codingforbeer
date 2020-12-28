---
title: "Advent of Code 2020 - Day 5"
date: 2020-12-06T23:22:37-05:00
description: "My thoughts on solving Advent of Code 2020 - Day 5 with F#"
tags: ["adventofcode", "fsharp", "code"]
series: ["Advent of Code 2020"]
draft: false
---

### Problem
#### Part 1
> For example, consider just the last 3 characters of FBFBBFFRLR:

    - Start by considering the whole range, columns 0 through 7.
    - R means to take the upper half, keeping columns 4 through 7.
    - L means to take the lower half, keeping columns 4 through 5.
    - The final R keeps the upper of the two, column 5.

> So, decoding FBFBBFFRLR reveals that it is the seat at row 44, column 5.
> Every seat also has a unique seat ID: multiply the row by 8, then add the column. In this example, the seat has ID 44 * 8 + 5 = 357.

> What is the highest seat ID on a boarding pass?
#### Part 2
> It's a completely full flight, so your seat should be the only missing boarding pass in your list. However, there's a catch: some of the seats at the very front and back of the plane don't exist on this aircraft, so they'll be missing from your list as well.

> Your seat wasn't at the very front or back, though; the seats with IDs +1 and -1 from yours will be in your list.

> What is the ID of your seat?
### Solution
{{< highlight fsi >}}
module codingforbeer.AdventOfCode.Day5

let inputLines = fileLines @"2020\assets\day5.txt"

let getSeatList input = 
    let split x = Seq.toList x |> List.splitAt 7
    let positionFolder (range: int list) half =
        match half with
        | 'F' | 'L' ->  range.[.. range.Length / 2 - 1]
        | 'B' | 'R' -> range.[range.Length / 2 ..]
        | _ -> range

    let seatNumber (row, col) = 
        match row with
        | [x] -> match col with
                 | [y] -> x * 8 + y
                 | _ -> 0
        | _ -> 0

    let getPosition (rows, cols) =
        (rows |> List.fold positionFolder [0..127], cols |> List.fold positionFolder [0..7])
    
    input |> Seq.map (split >> getPosition >> seatNumber)
    
let seatList = getSeatList inputLines
let getHighestSeat =  seatList |> Seq.max
let getLowestSeat = seatList |> Seq.min
let seatRange = [getLowestSeat .. getHighestSeat]
let missing = seatRange |> List.except seatList

printfn "Highest Seat: %i" getHighestSeat
printfn "Missing Seat: %i" (List.head missing)
{{< /highlight >}}

### Thoughts on why I did what I did
For getting the input data, I read the data line via line from a text file into the symbol `inputLines`. This gave me a `string []`  of my puzzle data. 

From there I converted each string in the array into an array of chars and then split at the 7th position. I now had a list of tuples consisting of the first 7 chars representing the row position and one containing the column positions. I then worked on getting the seat position on the plane via the rules in the problem. While I was reading the problem statement I knew I would be leaning toward a `fold`  for doing the heavy lifting. Once I had my method for determining which half of the provided range I would use, this would then produce a new list of tuples containing the row and col positions that would then be converted into a list of seat positions. Once I had a list of all the seat numbers it was easy to get the highest seat ID via the `max`  function.

For part two, there is a call out about the `+1`  and the `-1` seats being in the list of ID's but I didn't really care about those. My idea consisted of finding the smallest seat ID (via the `min`  function) and using that with the highest seat ID to create a new list of seat IDs. Next I then used the `except`  function to create a new list of missing items, took the first value and it turned out to be the missing seat. 