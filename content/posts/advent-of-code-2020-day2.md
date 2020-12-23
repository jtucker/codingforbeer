---
title: "Advent of Code 2020 - Day2"
date: 2020-12-03T16:52:37-05:00
description: "My thoughts on solving Advent of Code 2020 - Day 2 with F#"
tags: ["adventofcode", "fsharp", "code"]
series: ["Advent of Code 2020"]
draft: false
---

### Problem
#### Part 1
> Each line gives the password policy and then the password. The password policy indicates the lowest and highest number of times a given letter must appear for the password to be valid. For example, 1-3 a means that the password must contain a at least 1 time and at most 3 times.

#### Part 2
> Each policy actually describes two positions in the password, where 1 means the first character, 2 means the second character, and so on. (Be careful; Toboggan Corporate Policies have no concept of "index zero"!) Exactly one of these positions must contain the given letter. Other occurrences of the letter are irrelevant for the purposes of policy enforcement.

### Solution
{{< highlight fsi >}}
open System
open System.Text.RegularExpressions

type PasswordLine = {
    Positions : (int * int)
    Letter    : Char
    Password  : String
}

module PasswordRule =
    let isInRange range (str: String) =
        match range with
        | (lower, higher) when str.Length >= lower && str.Length <= higher -> true
        | _ -> false
    
    let isInPositions positions letter (str: String) =
        match positions with 
        | (first, second) when str.[first - 1] = letter && str.[second - 1] = letter -> false
        | (first, second) when str.[first - 1] = letter || str.[second - 1] = letter -> true
        | _ -> false

    let parse line =
        let matches = Regex.Match(line, "(?<lower>\d+)-(?<higher>\d+) (?<letter>[a-z]): (?<password>[a-z]+)")
        {
            Positions = (int(matches.Groups.["lower"].Value), int(matches.Groups.["higher"].Value))
            Letter = char(matches.Groups.["letter"].Value)
            Password = matches.Groups.["password"].Value
        }

    let tobogganValidate rule =
        rule.Password |> isInPositions rule.Positions rule.Letter

    let validate rule =
        rule.Password |> String.filter(fun c -> c = rule.Letter) |> isInRange rule.Positions

let getValidPasswordCount passwordValidator content =
    content
    |> Seq.map (Password.parse >> passwordValidator)
    |> Seq.filter (id)
    |> Seq.length

printfn "The number of valid sled passwords is %i" (getValidPasswordCount PasswordRule.validate)
printfn "The number of valid toboggan passwords is %i" (getValidPasswordCount PasswordRule.tobogganValidate)
{{< /highlight >}}

### Thoughts on why I did what I did
This day was all about stretching my legs and trying to get fancy, first by generating the `PasswordLine` type and then the `PasswordRule` module.

Adding the Regex was something I tried to fight and I really wanted to use `FParsec` here but ultimately it felt like it wasn't really meant for this type of input. That maybe just because of my lack of knowledge of the library but I really want to try using it for one of these problems. 