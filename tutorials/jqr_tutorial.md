---
title: jqr tutorial
layout: tutorial
packge_version: 0.2.0
---



[jq](https://stedolan.github.io/jq/) is a _lightweight and flexible command-line JSON processor_, written in C. It's super fast, and very flexible. `jq` gives you the ability to index into, parse, and do calculations on JSON data. You can cut up and filter JSON data. You can change JSON key names and values. `jq` lets you do conditionals and comparisons, and write your own custom functions to operate on JSON data.

You can convert JSON into an R list or other R data structure, and proceed with data parsing, but why not do your JSON parsing on the actual JSON if it's easy enough?  That's where `jq` comes in. Doing your data manipulations on the actual JSON makes it easy to pass data to downstream processes that expect JSON.

If you already familiar with `jq` by using it on the command line you can use the exact same commands with `jqr`. If you've never used `jq`, `jqr` makes `jq` easy to learn with a domain specific language - and you can learn the actual `jq` syntax as you go and apply it on the command line outside of R.

## NSE vs. SE

Many functions in `jqr` have NSE (non-standard evaluation) as well as SE (standard evaluation) versions, where the NSE version for sorting an array is `sortj()` whereas the SE version is `sortj_()`. Some functions only have one version, and behave under SE rules.

When you pass JSON into a function as the first parameter (like `ad('["a","b","c"]')`) rather than piping it in (like `'["a","b","c"]' %>% ad`), `jq()` is not executed. Rather you get back an object of class `jqr` that holds the data you passed in and the query. To execute the query on the data, run `jq()`, e.g., like `jq(ad('["a","b","c"]'))` or `ad('["a","b","c"]') %>% jq()`.

When piping JSON to DSL functions `jq()` is executed on the last DSL function used.

## jqr API

There's low and high level (or DSL [domain specific language]) interfaces in `jqr`.

### jqr low level interface

The low level and high level interfaces are unified via the function `jq()`. You can access the low leve interface by using `jq()` directly, passing a JSON string as the first parameter, the program (query) as the second, and [the flags](https://stedolan.github.io/jq/manual/#Invokingjq) as the third (by default no flags are passed).

For example, a JSON string could be `'{"a": 7, "b": 4}'`, and the program could be `.`, resulting in

```
{
    "a": 7,
    "b": 4
}
```

The program passed is exactly the same as you'd pass on the command line. Because this is a  simple replication of the command line in R, there is a higher level interface, or DSL, to make it easier to use `jq`. Nonetheless, the low level interface is important as some `jq` veterans may not want to deal with a DSL, and you may need to drop down to the low level interface if the DSL doesn't work for some reason.

### jqr DSL

The `jqr` DSL uses a suite of functions to construct queries that are executed internally with `jq()` after the last piped command. We use some logic to determine whether the function call is the last in a series of pipes, and if so, we run `jq()` on the JSON string and program/query passed.

You don't have to use pipes - they are optional. Though they do make things easier in that you can build up queries easily, just as you would with `jq`, or any other tools, on the command line.

* Execute jq
  * `jq` - execute jq
* Utility functions
  * `peek` - peek at query, without running it
  * `string` - give back character string
  * `combine` - combine pieces into proper JSON
* Identity functions
  * `dot` - takes its input and produces it unchanged as output.
  * `dotstr` - produces value at the key 'foo'
  * `index` - index to all elements, or elements by name or number
  * `indexif` - same as above, but shouldn't fail when not found
* Operations on keys, or by keys
  * `keys` - takes no input, and retrieves keys
  * `haskey` - checks if a json string has a key or keys
  * `del` - deletes provided keys
* Maths
  * `do` - arbitrary math operations
  * `lengthj` - length
  * `sqrtj` - square root
  * `floorj` - returns the floor of its numeric input
  * `minj` - minimum element of input
  * `maxj` - maximum element of input
  * `add` - adds strings or numbers together
  * `map` - for any filter X, run X for each element of input array
* Manipulation operations
  * `join` - join strings on given separator
  * `splitj` - split string on separator argument
  * `ltrimstr` - remove given prefix string, if it starts with it
  * `rtrimstr` - remove given suffix string, if it starts with it
  * `startswith` - logical output, test if input start with foo
  * `endswith` - logical output, test if input ends with foo
  * `indices` - array with numeric indices where foo occurs in inputs
  * `tojson` - dump values to JSON
  * `fromjson` - parse JSON into values
  * `tostring` - convert to string
  * `tonumber` - convert to number
  * `contains` - logical output, determine if foo is in the input
  * `uniquej` - output unique set
  * `group` - groups the elements having the same .foo field into separate arrays
* Sort
  * `sortj` - sort an array
  * `reverse` - reverse sort an array
* Types
  * `type` - select elements by type
  * `types` - get type for each element
* Functions
  * `funs` - Define and use functions
* Variables
  * `vars` - Define variables to use later
* Recursion
  * `recurse` - Search through a recursive structure - extract data from all levels
* Paths
  * `paths` - Outputs paths to all the elements in its input
* Range
  * `rangej` - Produce range of numbers
* Format strings
  * `at` - Format strings and escaping

<section id="installation">

## Installation

stable version


```r
install.packages("jqr")
```

dev version


```r
devtools::install_github("ropensci/jqr")
```

Load `jqr`


```r
library("jqr")
```

<section id="usage">

## Usage

### Utility functions

Peek


```r
'{"a": 7}' %>% do(.a + 1) %>% peek
#> <jq query>
#>   query: .a + 1
'[8,3,null,6]' %>% sortj %>% peek
#> <jq query>
#>   query: sort
```

String


```r
'{"a": 7}' %>% do(.a + 1) %>% string
#> [1] "{\"a\": 7}"
'[8,3,null,6]' %>% sortj %>% string
#> [1] "[8,3,null,6]"
```

Combine


```r
x <- '{"foo": 5, "bar": 7}' %>% select(a = .foo)
combine(x)
#> {
#>     "a": 5
#> }
```

### index


```r
x <- '[{"message": "hello", "name": "jenn"}, {"message": "world", "name": "beth"}]'
x %>% index()
#> [
#>     {
#>         "message": "hello",
#>         "name": "jenn"
#>     },
#>     {
#>         "message": "world",
#>         "name": "beth"
#>     }
#> ]
```

### sort

Note the function name is `sortj` to avoid collision with `base::sort`. In addition, a
number of other functions in this package that conflict with base R functions have a
`j` on the end.


```r
'[8,3,null,6]' %>% sortj
#> [
#>     null,
#>     3,
#>     6,
#>     8
#> ]
```

sort in reverse order


```r
'[1,2,3,4]' %>% reverse
#> [
#>     4,
#>     3,
#>     2,
#>     1
#> ]
```

### join


```r
'["a","b,c,d","e"]' %>% join
#> "a, b,c,d, e"
'["a","b,c,d","e"]' %>% join(`;`)
#> "a; b,c,d; e"
```

### starts- and ends-with


```r
'["fo", "foo", "barfoo", "foobar", "barfoob"]' %>% index %>% endswith(foo)
#> [
#>     false,
#>     true,
#>     true,
#>     false,
#>     false
#> ]
```


```r
'["fo", "foo", "barfoo", "foobar", "barfoob"]' %>% index %>% startswith(foo)
#> [
#>     false,
#>     true,
#>     false,
#>     true,
#>     false
#> ]
```

### contains


```r
'"foobar"' %>% contains("bar")
#> true
```

### unique


```r
'[1,2,5,3,5,3,1,3]' %>% uniquej
#> [
#>     1,
#>     2,
#>     3,
#>     5
#> ]
```

### data types

Get type information for each element


```r
'[0, false, [], {}, null, "hello"]' %>% types
#> [
#>     "number",
#>     "boolean",
#>     "array",
#>     "object",
#>     "null",
#>     "string"
#> ]
'[0, false, [], {}, null, "hello", true, [1,2,3]]' %>% types
#> [
#>     "number",
#>     "boolean",
#>     "array",
#>     "object",
#>     "null",
#>     "string",
#>     "boolean",
#>     "array"
#> ]
```

Select elements by type


```r
'[0, false, [], {}, null, "hello"]' %>% index() %>% type(booleans)
#> false
```

### keys

Get keys


```r
str <- '{"foo": 5, "bar": 7}'
str %>% keys()
#> [
#>     "bar",
#>     "foo"
#> ]
```

Delete by key name


```r
str %>% del(bar)
#> {
#>     "foo": 5
#> }
str %>% del(foo)
#> {
#>     "bar": 7
#> }
```

Check for key existence


```r
str3 <- '[[0,1], ["a","b","c"]]'
str3 %>% haskey(2)
#> [
#>     false,
#>     true
#> ]
str3 %>% haskey(1,2)
#> [
#>     true,
#>     false,
#>     true,
#>     true
#> ]
```

### select

Select variables by name, and rename


```r
'{"foo": 5, "bar": 7}' %>% select(a = .foo)
#> {
#>     "a": 5
#> }
```

More complicated `select()`, using the included dataset `githubcommits`


```r
githubcommits %>%
  index() %>%
  select(sha = .sha, name = .commit.committer.name)
#> [
#>     {
#>         "sha": [
#>             "110e009996e1359d25b8e99e71f83b96e5870790"
#>         ],
#>         "name": [
#>             "Nicolas Williams"
#>         ]
#>     },
#>     {
#>         "sha": [
#>             "7b6a018dff623a4f13f6bcd52c7c56d9b4a4165f"
#>         ],
#>         "name": [
#>             "Nicolas Williams"
#>         ]
#>     },
#>     {
#>         "sha": [
#>             "a50e548cc5313c187483bc8fb1b95e1798e8ef65"
#>         ],
#>         "name": [
#>             "Nicolas Williams"
#>         ]
#>     },
#>     {
#>         "sha": [
#>             "4b258f7d31b34ff5d45fba431169e7fd4c995283"
#>         ],
#>         "name": [
#>             "Nicolas Williams"
#>         ]
#>     },
#>     {
#>         "sha": [
#>             "d1cb8ee0ad3ddf03a37394bfa899cfd3ddd007c5"
#>         ],
#>         "name": [
#>             "Nicolas Williams"
#>         ]
#>     }
#> ]
```

### maths

Maths comparisons


```r
'[5,4,2,7]' %>% index() %>% do(. < 4)
#> [
#>     false,
#>     false,
#>     true,
#>     false
#> ]
'[5,4,2,7]' %>% index() %>% do(. > 4)
#> [
#>     true,
#>     false,
#>     false,
#>     true
#> ]
'[5,4,2,7]' %>% index() %>% do(. <= 4)
#> [
#>     false,
#>     true,
#>     true,
#>     false
#> ]
'[5,4,2,7]' %>% index() %>% do(. >= 4)
#> [
#>     true,
#>     true,
#>     false,
#>     true
#> ]
'[5,4,2,7]' %>% index() %>% do(. == 4)
#> [
#>     false,
#>     true,
#>     false,
#>     false
#> ]
'[5,4,2,7]' %>% index() %>% do(. != 4)
#> [
#>     true,
#>     false,
#>     true,
#>     true
#> ]
```

### sqrt


```r
'9' %>% sqrtj
#> 3
```

### floor


```r
'3.14159' %>% floorj
#> 3
```

### find minimum


```r
'[5,4,2,7]' %>% minj
#> 2
'[{"foo":1, "bar":14}, {"foo":2, "bar":3}]' %>% minj
#> {
#>     "foo": 2,
#>     "bar": 3
#> }
'[{"foo":1, "bar":14}, {"foo":2, "bar":3}]' %>% minj(foo)
#> {
#>     "foo": 1,
#>     "bar": 14
#> }
'[{"foo":1, "bar":14}, {"foo":2, "bar":3}]' %>% minj(bar)
#> {
#>     "foo": 2,
#>     "bar": 3
#> }
```

### find maximum


```r
'[5,4,2,7]' %>% maxj
#> 7
'[{"foo":1, "bar":14}, {"foo":2, "bar":3}]' %>% maxj
#> {
#>     "foo": 1,
#>     "bar": 14
#> }
'[{"foo":1, "bar":14}, {"foo":2, "bar":3}]' %>% maxj(foo)
#> {
#>     "foo": 2,
#>     "bar": 3
#> }
'[{"foo":1, "bar":14}, {"foo":2, "bar":3}]' %>% maxj(bar)
#> {
#>     "foo": 1,
#>     "bar": 14
#> }
```


<section id="citing">

## Citing

To cite `jqr` in publications use:

<br>

> Rich FitzJohn, Scott Chamberlain, Stefan Milton Bache and Stephen Dolan. jqr: Client for 'jq', a
  JSON Processor. R package version 0.2.0. https://github.com/ropensci/jqr

<section id="license_bugs">

## License and bugs

* License: [MIT](http://opensource.org/licenses/MIT)
* Report bugs at [our Github repo for jqr](https://github.com/ropensci/jqr/issues?state=open)

[Back to top](#top)
