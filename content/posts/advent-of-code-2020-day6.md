---
title: "Advent of Code 2020 - Day 6"
date: 2020-12-08T00:05:37-05:00
description: "My thoughts on solving Advent of Code 2020 - Day 6 with F#"
tags: ["adventofcode", "fsharp", "code"]
series: ["Advent of Code 2020"]
draft: false
---

### Problem
#### Part 1
> The form asks a series of 26 yes-or-no questions marked a through z. All you need to do is identify the questions for which anyone in your group answers "yes". Since your group is just you, this doesn't take very long.

> However, the person sitting next to you seems to be experiencing a language barrier and asks if you can help. For each of the people in their group, you write down the questions for which they answer "yes", one per line. For example:

`abcx
abcy
abcz`

> In this group, there are 6 questions to which anyone answered "yes": a, b, c, x, y, and z. (Duplicate answers to the same question don't count extra; each question counts at most once.)

> For each group, count the number of questions to which anyone answered "yes". What is the sum of those counts?
#### Part 2
> As you finish the last group's customs declaration, you notice that you misread one word in the instructions:

> You don't need to identify the questions to which anyone answered "yes"; you need to identify the questions to which everyone answered "yes"!

> For each group, count the number of questions to which everyone answered "yes". What is the sum of those counts?
### Solution
{{< highlight fsi >}}
open System

let getAllYesAnswers items =
    items |> (Array.collect (Seq.toArray) >> Array.distinct >> Array.length)

let getAnswerCounts counter group = 
    let splitter (splitOn: string) (str: string) = str.Split(splitOn, StringSplitOptions.RemoveEmptyEntries);
    let singleSplitter = splitter Environment.NewLine
    let doubleSplitter = splitter (Environment.NewLine + Environment.NewLine)

    Array.map (singleSplitter >> counter) (group |> doubleSplitter)

printfn "Sum total of groups yes answers: %i" (getAnswerCounts getAllYesAnswers responses |> Array.sum)

let getEveryoneYesAnswers items =
    items |> (Array.map Set.ofSeq >> Array.reduce Set.intersect >> Set.count)

printfn "Sum total of groups same answers: %i" (getAnswerCounts getEveryoneYesAnswers responses |> Array.sum)
{{< /highlight >}}

### Thoughts on why I did what I did

This one I refactored a few times, even refactored it a bit for this post. For Part 1, I simply wanted to get all the groups and then find all the distinct answers and then count them all up. The sum of all those gave me the answer. 

For Part 2, I took basically the same approach except this time, I took the resulting array within the groups and created a unique set of answers via the `Set`  type. Then it was simply a matter of using `reduce`  and `intersect`  to thread the `Set`s each time finding all the answers that were the same and then counting them. Finally taking all those counts and then summing them all up. 