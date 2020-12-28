---
title: "Advent of Code 2020 - Day 4"
date: 2020-12-05T16:06:37-05:00
description: "My thoughts on solving Advent of Code 2020 - Day 4 with F#"
tags: ["adventofcode", "fsharp", "code"]
series: ["Advent of Code 2020"]
draft: false
---

### Problem
#### Part 1
>The automatic passport scanners are slow because they're having trouble detecting which passports have all required fields. The expected fields are as follows:

    - byr (Birth Year)
    - iyr (Issue Year)
    - eyr (Expiration Year)
    - hgt (Height)
    - hcl (Hair Color)
    - ecl (Eye Color)
    - pid (Passport ID)
    - cid (Country ID)

>Passport data is validated in batch files (your puzzle input). Each passport is represented as a sequence of key:value pairs separated by spaces or newlines. Passports are separated by blank lines.
#### Part 2
> You can continue to ignore the cid field, but each other field has strict rules about what values are valid for automatic validation:

    - byr (Birth Year) - four digits; at least 1920 and at most 2002.
    - iyr (Issue Year) - four digits; at least 2010 and at most 2020.
    - eyr (Expiration Year) - four digits; at least 2020 and at most 2030.
    - hgt (Height) - a number followed by either cm or in:
    - If cm, the number must be at least 150 and at most 193.
    - If in, the number must be at least 59 and at most 76.
    - hcl (Hair Color) - a # followed by exactly six characters 0-9 or a-f.
    - ecl (Eye Color) - exactly one of: amb blu brn gry grn hzl oth.
    - pid (Passport ID) - a nine-digit number, including leading zeroes.
    - cid (Country ID) - ignored, missing or not.

### Solution
{{< highlight fsi >}}
open System.Text.RegularExpressions
open System.Collections.Generic

type Height =
    | Centimeters of int
    | Inches of int
    | NoHeight

type Passport =
    { PassportId: string
      EyeColor: string
      Height: Height
      HairColor: string
      ExpirationYear: int
      IssueYear: int
      BirthYear: int }

let processPassports passportList mapper filter =
    let regexMatch str =
        let getMatchValue (groupName: string) (mtch: Match) = mtch.Groups.[groupName].Value
        
        Regex.Matches(str, "(?<Field>\w+):(?<Value>\S+)")
        |> Seq.cast<Match>
        |> Seq.map (fun m -> ((getMatchValue "Field" m), (getMatchValue "Value" m)))

    passportList
    |> Array.map (regexMatch)
    |> Seq.map mapper
    |> Seq.filter filter
    |> Seq.length

let validPassport p =
    [ "byr"
      "iyr"
      "eyr"
      "hgt"
      "hcl"
      "ecl"
      "pid" ]
    |> Seq.forall (fun x -> Set.contains x p)

let requiredFieldsMapper p = Set.ofSeq p |> Set.map fst
let validPassportsRequiredFields = processPassports passports requiredFieldsMapper validPassport
printfn "Answer Part 1: %i passports with all required fields" validPassportsRequiredFields

let validRulesMapper keyValues =
    let items = keyValues |> Seq.distinctBy fst |> dict
    let getValueOrDefault key defaultValue (items: IDictionary<string, string>) =
        match items.TryGetValue(key) with
        | (true, x) -> x
        | (false, _) -> defaultValue
    let height s =
        let mtch = Regex.Match(s, "(?<value>\d+)(?<measurement>(in|cm))")
        match mtch.Success with
        | true ->
            match mtch.Groups.["measurement"].Value with
            | "in" -> Inches(mtch.Groups.["value"].Value |> int)
            | "cm" -> Centimeters(mtch.Groups.["value"].Value |> int)
            | _ -> NoHeight
        | false -> NoHeight

    { PassportId = getValueOrDefault "pid" "" items
      EyeColor = getValueOrDefault "ecl" "" items
      Height = getValueOrDefault "hgt" "" items |> height
      HairColor = getValueOrDefault "hcl" "" items
      ExpirationYear = getValueOrDefault "eyr" "0" items |> int
      IssueYear = getValueOrDefault "iyr" "0" items |> int
      BirthYear = getValueOrDefault "byr" "0" items |> int }

let validRulesFilter (p: Passport) = 
    let str r s = Regex.IsMatch(s, r)
    let isBetween f l v =
        v >= f && v <= l

    let validHeight = function
        | Inches i when i |> isBetween 59 76 -> true
        | Centimeters c when c |> isBetween 150 193 -> true
        | _ -> false
        
    let eye  = function
        | "amb" | "blu" | "brn" | "gry" | "grn" | "hzl" | "oth" -> true
        | _ -> false

    (p.PassportId |> str "^\d{9}$") &&
    (p.EyeColor |> eye) &&
    (p.HairColor |> str "^#([0-9a-fA-F]){6}$") &&
    (p.ExpirationYear |> isBetween 2020 2030) &&
    (p.IssueYear |> isBetween 2010 2020) &&
    (p.BirthYear |> isBetween 1920 2002) &&
    (p.Height |> validHeight)

let validPassportsByRules = processPassports passports validRulesMapper validRulesFilter
printfn "Answer Part 2: %i passports with based on rules" validPassportsByRules

{{< /highlight >}}

### Thoughts on why I did what I did
It's all about the Regex and learning about [Set type](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-collections-fsharpset-1.html) in the language. I again initially would have liked to have tried using `FParsec` but was a little apprehensive on it, so I opt'd against it.

For Part 1 using the `Set` type made making sure there weren't any duplicate Field/Value pairs easy and operates just like you would expect a collection to. So once I knew I had a list of pairs that were unique it was a simple call to make sure that all the required fields were present in the passport line and then filtering out all the ones that didn't. 

For Part 2, the bulk of the work was really constructing the `Passport`  type I defined. This felt very much in the imperative style I normally would use in my day to day but I couldn't really think of another more functional way of doing it. Once all the lines were made into `Passport`  types it came down to just filtering out all the invalid `Passport`s based on the rules. 