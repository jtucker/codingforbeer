---
title: "Addding my Untappd Checkins to my GitHub Profile README"
date: 2021-03-11T14:36:45-05:00
draft: false
tags: ['github', 'beer']
---

Recently, I wanted to give my [GitHub Profile README](https://github.com/jtucker) a little sprucing up. In what probably took longer than it should, I decided I wanted to include my [Untappd](https://untappd.com/user/jtucker) checkins at the bottom of the page.

I'm actually quite embarrassed in how long it took me to decide that, seeing how this blog is called "Coding for Beer".

I ended up using a combo of [GitHub Actions](https://github.com/features/actions) and a simple F# script to get the work done. 

### Goal

The idea was generating a simple table of my 3 most recent checkins on Untappd. I wanted that table to contain:
1. A picture if I took one.
1. The name and brewery of the beer.
1. The rating I gave it.

### Initial ideas

Here I would like to list out the first couple iterations of this solution before finally selecting the one I'm currently using.

1. An all GitHub workflow version. 

   This actually worked fairly well but the action I used for generating the table was very limited.
1. Generate an image via Azure Function (or similar). 
 
   This one I didn't even start because I wanted to keep it all in GitHub

1. Use some sort of templating engine to output the AsciiDoc/Markdown.

   I found the current state of cli based template engines to be lacking. Also, I didn't put much time into researching this past a few hours of google-bing.

1. AsciiDoc generation with includes. 

   This would have been great and my choosen solution. [Unfortunately GitHub](https://github.com/github/markup/issues/1095) currently doesn't support the `include::`  statement at the moment.

### Solution
> **tl;dr** I ended up using GitHub actions to download and parse my Untappd activity. From there I used an F# script to read in the data, generate the table HTML and output the updated file. 

### Create an Untappd App
First I needed to create an app registration over on Untappd. [That is done on this site](https://untappd.com/api/dashboard). 

Once that was approved I was given a `Client ID` and a `Client Secret` to use later to make authenticated calls to pull my activity from the site.

I then created two repository secrets in my profile repo lat look like this:

![Secrets Example](/posts/adding-untappd-checkins-secrets.png)

### Setting up the workflow

I then created a new workflow called `untappd.yaml` in the `.github/workflows` folder in my repo. 
{{< gist jtucker 6662b6eba4361e803747407e05d2f0e5 >}}

So what happens here?

1. Install [dotnet](https://dot.net) 5 
1. Install the [jq](https://stedolan.github.io/jq/) cli
1. Get my 3 most recent checkins via `curl`. The url is tokenized for GitHub to inject my secrets into. 
1. Use jq to parse the JSON and extract the pieces I want
1. Run my F# script to generate the updated `README.asciidoc`
1. Commit the updated file if necessary. The secret is to make sure that `skip_dirty_check` is set to false, otherwise it will always commit and mess up my history. 

### The F# Script
{{< gist jtucker ce81eea00e79dc176c16ea015df0d262 >}}

1. Use the JSONProvider from F# Data package to generate types based on the parsed JSON
1. Read in the entire `README.asciidoc` file
1. Perform a Regex match to pull out all the text between `// untappd beer` and ` //untappd end` markers
1. Generate the HTML table from the checkins
1. Finally, update the `README.asciidoc` with the (possibly) new content

### Some thoughts

All in all the process was pretty smooth. I especially loved that the [stefanzweifel/git-auto-commit-action](https://github.com/stefanzweifel/git-auto-commit-action) had built into functionality to not commmit a file that hasn't been updated. Another thing that helped during the process of putting this together was the [act](https://github.com/nektos/act) testing tool that allowed me to debug my workflow locally. I really wish there was one of these for Azure Pipelines.
