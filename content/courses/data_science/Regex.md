---
title: Regex
linktitle: Regular Expresions
toc: true
type: docs
draft: false
date: "2019-10-01"
lastmod: "2019-10-01"
menu:
  data_science:
    parent: Data Science
    weight: 1

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 1
---

## A Short tutorial using the [Julia](https://julialang.org/) Programming Language

Regular expressions allows to define a search pattern in a corpus.

For example, we might want to find if a sequence of characters occurs in some text.

For instance, whether the sequence "dog" occurs in "I love dogs!". We can find this information in Julia through the `occursin` function,

````julia
occursin("dog", "I love dogs!")
````


````
true
````





Sometimes we want to match a pattern case insensitive. In the following example, the corpus might have the name _GitHub_ with the official spelling or as _github_, _Github_, or _GITHUB_.

````julia
pattern = r"(?i)github(?-i)";
````



````julia
all(text -> occursin(pattern, "The website I spend most of my time is $text"),
    ["GitHub", "Github", "GITHUB", "github"])
````


````
true
````





This pattern uses the ignore case indicator `(?i)`. It is equivalent to specifying all possible combinations that ignore case.

````julia
pattern = r"[Gg][Ii][Tt][Hh][Uu][Bb]"
all(text -> occursin(pattern, "The website I spend most of my time is $text"),
    ["GitHub", "Github", "GITHUB", "github"])
````


````
true
````





If instead we wish to only allow certain variations, we can specify these with the following pattern

````julia
pattern = r"(GitHub|github|GITHUB)"
all(text -> occursin(pattern, "The website I spend most of my time is $text"),
    ["GitHub", "Github", "GITHUB", "github"])
````


````
false
````





In this case, it is false since we are not interested in matching _Github_.

We can also perform look-aheads and look-behinds.

The following pattern will match anything after "My name is" and followed by a ".".

````julia
pattern = r"(?<=My name is ).*(?=\.)"
foreach(text -> println("\"$text\": $(occursin(pattern, text))"),
        ["Bayoán", "My name is Bayoán!", "My name is Bayoán."])
````


````
"Bayoán": false
"My name is Bayoán!": false
"My name is Bayoán.": true
````





We can also extract the matched pattern.

````julia
pattern = r"(?<=My name is ).*(?=\.)"
foreach(text -> println(match(pattern, text)),
        ["Bayoán", "My name is Bayoán!", "My name is Bayoán."])
````


````
nothing
nothing
RegexMatch("Bayoán")
````





To access the value

````julia
match(r"(?<=My name is ).*(?=\.)", "My name is Bayoán.").match
````


````
"Bayoán"
````





We can also find and replace through,

````julia
replace("My name is Bayoán.", r"(?<=My name is ).*(?=\.)" => "Nosferican")
````


````
"My name is Nosferican."
````





Certain patterns are readily a available such as,

````julia
pattern = r"^Starts with"
foreach(text -> println("\"$text\": $(occursin(pattern, text))"),
        ["Starts with this.", "Does not Starts with this.", " Starts with this."])
````


````
"Starts with this.": true
"Does not Starts with this.": false
" Starts with this.": false
````



````julia
pattern = r"Ends with$"
foreach(text -> println("\"$text\": $(occursin(pattern, text))"),
        ["Ends with this.", "This text Ends with this.", "This text Ends with"])
````


````
"Ends with this.": false
"This text Ends with this.": false
"This text Ends with": true
````





For more fun with regex, see the documentation at [here](https://www.regular-expressions.info/).