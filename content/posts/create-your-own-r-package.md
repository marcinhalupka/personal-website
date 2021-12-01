---
title: "Create Your Own R Package"
date: 2021-11-29T21:33:55+01:00
draft: false
---

Packages are the fundamental units of reproducible R code. They include reusable R functions, the documentation that describes how
to use them and sample data. This posts will be the essence from Hadley Wickham's ([Hadley's website](http://hadley.nz/))
book `R packages` which teaches how to turn your code into packages that others can easily download and use. As Hadlay says:
"It doesn't matter if your first version isn't perfect as long as the next version is better".

# Introduction

Package is the fundamental unit of shareable code in R. It bundles together code, data, documentation and tests and is easy to shareable
with others. The huge variety of packages is one of the reasons that R is so popular - the chances are that someone has already solved a problem
that you're working on and you can benefit from their work by downloading their package.

Using packages is simple:

* You install them from CRAN (Comprehensive R Archive Network) with `install.packages("x")`
* You use them in R with `library("x")`
* You get help on them with `package?x` and `help(package = "x")`

Why would you ever want to write one by yourself?
1. To share your code with others - R users know how to download packages, install and use them
2. To make your life easier - organising code in a package forces you to follow some conventions, you don't need to think about the best way
to do this, just follow the template
3. To get many tools for free - standardized conventions lead to standardized tools

# Let's build a package

There's no better way to present a big picture and suggest the workflow than to develop a simple package!

## Load devtools and utilities

You can initiate your new package from any active R session.

The first step is to load the `devtools` package - a public face of a set of packages that support various aspects of package development.

```r
library(devtools)
```

## Toy package: regexcite

We will use verious functions from devtools to build a small package from scratch, with features commonly seen in released packages:

* Functions to address a specific need - in this case helpers for work with regular expressions
* Version control and an open development process - optional but highly recommended
* Access to established workflows for installation, getting help and checking quality:
	* Documentation for individual functions via `roxygen2`
	* Unit testing with `testthat`
	* Documentation for package as a whole via an executable `Readme.Rmd`

The package's name will be `regexcite` and it will have a couple functions that make common tasks with regular expressions easier.
Functions will be super simple and not the point - the goal is to demonstrate a typical workflow for package development with devtools.

## `create_package()`

Call `create_package()` to initialize a new package in a directory on your computer.

It's up to you where to create it. It should not be nested inside another RStudio Project, R Package or Git repo.

```r
create_package("~/regexcite")
```

The output may differ but you should see something like this:

```r
√ Creating 'C:/Users/halupkam/regexcite/'
√ Setting active project to 'C:/Users/halupkam/regexcite'
√ Creating 'R/'
√ Writing 'DESCRIPTION'
Package: regexcite
Title: What the Package Does (One Line, Title Case)
Version: 0.0.0.9000
Authors@R (parsed):
    * First Last <first.last@example.com> [aut, cre] (YOUR-ORCID-ID)
Description: What the package does (one paragraph).
License: `use_mit_license()`, `use_gpl3_license()` or friends to
    pick a license
Encoding: UTF-8
Roxygen: list(markdown = TRUE)
RoxygenNote: 7.1.2
√ Writing 'NAMESPACE'
√ Writing 'regexcite.Rproj'
√ Adding '^regexcite\\.Rproj$' to '.Rbuildignore'
√ Adding '.Rproj.user' to '.gitignore'
√ Adding '^\\.Rproj\\.user$' to '.Rbuildignore'
√ Opening 'C:/Users/halupkam/regexcite/' in new RStudio session
√ Setting active project to '<no active project>'
```

If you're working in RStudio, you'll find yourself in a new instance of R Studio, opened into your new regexcite package (and Project).

What files can you find in a new directory?

* `Rbuildignore` - lists files that we need but that should be ignored when building R package from source
* `.Rproj.user` - if you have it, it's a directory used internally by RStudio
* `.gitignore` - anticipates Git usage and ignores some standard files created behind the scenes, if you do not plan to use Git, this is harmless
* `DESCRIPTION` - provides metadata about your package
* `NAMESPACE` - declares the functions your package exports for external use and the external functions your package imports from other packages
* `R/` directory - a place for our `.R` files with function definitions
* `regexcite.Rproj` - a file that makes this directory an RStudio Project, if you do not use RStudio, this is harmless

## `use_git()`

The use of Git or another version control system is optional but a recommended practice. The regexcite directory is an R source package and
an RStudio Project. To make it also a Git repository, we can use `use_git()`

```r
> use_git()
√ Setting active project to 'C:/Users/halupkam/regexcite'
√ Initialising Git repo
√ Adding '.Rhistory', '.Rdata', '.httr-oauth', '.DS_Store' to '.gitignore'
```

You'll be asked if you want to commit some files and you should probably agree. If you receive an error `config value 'user.name' was not found`,
introduce yourself to Git:

```r
use_git_config(user.name = "Your name", user.email = "Your email")
```

What happened? Only the creation of a `.git` directory which is hidden in most contexts. You can relaunch your project now by quitting and
opening once again `regexcite.Rproj` file. Now, on the right hand side, you can see a basic Git client in the Git tab of the
`Environment/History/Build` pane. If you click on history (the clock icon), you will see an initial commit made via `use_git()`.

## Write the first function

A common task when dealing with strings is to split a single string into many parts. In base R there is a `strsplit()` function designed for that.

```r
> (x <- c("one,two,three,four"))
[1] "one,two,three,four"
> strsplit(x, split = ",")
[[1]]
[1] "one"   "two"   "three" "four" 
```

Let's look closely at the return value:

```r
> str(strsplit(x, split = ","))
List of 1
 $ : chr [1:4] "one" "two" "three" "four"
```

The shape of the return value can be a bit confusing. The input is a character vector length one and the output is a list of length one.
This makes total sense from R's tendency towards vectorization but may often catch people off guard.

We know that our input is a single string and we expect the output to be a character vector with its parts. That leads R users to employ various
methods of "unlist"-ing the result, like:

```r
> strsplit(x, split = ",")[[1]]
[1] "one"   "two"   "three" "four" 
```

That's a great candidate for our first function!

```r
strsplit1 <- function(x, split) {
  strsplit(x, split = split)[[1]]
}
```

## `use_r()`

Where should we define `strsplit1()`? Save it in a `.R` file in the `R/` subdirectory of your package. A reasonable starting point is to make
a new `.R` file for each user-facing function in your package and name the file after the function. As you add more functions, it's a good idea
to relax this and group related functions together. We'll save the definition of `strsplit1()` in the file `R/strsplit1.R`.

The helper function `use_r()` creates and/or opens a script below `R/`. It really shines in a more mature package when navigating between `.R` files and the associated test file.

```r
> use_r("strsplit1")
√ Setting active project to 'C:/Users/halupkam/regexcite'
* Modify 'R/strsplit1.R'
* Call `use_test()` to create a matching test file
```

Put the definition of `strsplit1()` **and only the definition of `strsplit()`** in `R/strsplit1.R` and save it.
That file should not contain any of the other top-level code we have recently executed, like the definition of our practice input `x` or `library(devtools)`.
That may be a bit confusing but packages and scripts use different mechanisms to declare their dependency on other packages.

## `load_all()`

How do we test drive our `strsplit1()`? If this were a regular R script we could send the function definition to the R console and define `strsplit1()` in the global environment or maybe we'd
call `source("R/strsplit1.R")`. For package development, devtools can offer us something different.

Call `load_all()` to make `strsplit1()` available for experimentation.

```r
> load_all()
i Loading regexcite
```

And call `strsplit1(x)` to see how it works.

```r
> (x <- c("one,two,three,four"))
[1] "one,two,three,four"
> strsplit1(x, split = ",")
[1] "one"   "two"   "three" "four" 
```

Important remark - `load_all()` makes the `strsplit1()` available for experimentation, although it does not exist in the global environment.

```r
> exists("strsplit1", where = globalenv(), inherits = FALSE)
[1] FALSE
```

`load_all()` function simulates the process of building, installing and attaching the regexcite package - it improves the speed of our iterations.

## Commit `strsplit1()`

If you're using Git, use your preferred method to commit the new `R/strsplit.R` file. From this point on, we'll commit after each step.

## `check()`

We have an empirical evidence that `strsplit1()` works but how can we be sure that all the moving parts of the regexcite package still work? It may seem silly to check after such a small addition, but
it's good to establish the habit of checking this often.

`R CMD check`, executed in the shell, is the gold standard for checking that an R package is in full working order. `check()` is a convenient way to run this without leaving your R session.

```r
check()
```

The output will be pretty big, in my case the last few lines looked like this:

```r
-- R CMD check results ----------------------------------------------------------------------------------------------------------------- regexcite 0.0.0.9000 ----
Duration: 24.1s

> checking DESCRIPTION meta-information ... WARNING
  Non-standard license specification:
    `use_mit_license()`, `use_gpl3_license()` or friends to pick a
    license
  Standardizable: FALSE

0 errors √ | 1 warning x | 0 notes √
```

**Reading the output of the check** can allow you to deal with problems early and often. The longer you go between full checks that everything works, the harder it becomes to pinpoint and solve your problems.

We received one warning and we'll solve it soon by doing exactly what it says.

## Edit `DESCRIPTION`

The `DESCRIPTION` file provides metadata about your package. You can see that it's populated with boilerplate content, which needs to be replaced.

Make few edits so it looks similar to this:

```
Package: regexcite
Title: Make Regular Expressions More Exciting
Version: 0.0.0.9000
Authors@R: 
    person("First", "Last", , "first.last@example.com", role = c("aut", "cre"))
Description: Convenience functions to make some common tasks with string
    manipulation and regular expressions a bit easier.
License: `use_mit_license()`, `use_gpl3_license()` or friends to pick a
    license
Encoding: UTF-8
Roxygen: list(markdown = TRUE)
RoxygenNote: 7.1.2

```

## `use_mit_license()`

There's already a placeholder in the `License` field of `DESCRIPTION`.

Let's call `use_mit_license()`.

```r
> use_mit_license()
√ Setting License field in DESCRIPTION to 'MIT + file LICENSE'
√ Writing 'LICENSE'
√ Writing 'LICENSE.md'
√ Adding '^LICENSE\\.md$' to '.Rbuildignore'
```

This configures the `License` field correctly for the MIT license which promises to name the copyright holders and year in a `LICENSE` file. When you open it you should see the following output:

```
YEAR: 2021
COPYRIGHT HOLDER: regexcite authors
```

Same as other license helpers, `use_mit_license()` also puts a copy of the full license in `LICENSE.md` and adds this file to `.Rbuildignore`. CRAN disallows the inclusion of this file in a package tarball
but it's considered a best practice to include a full license in your package's source, such as on GitHub.

## `document()`

It would be nice to get help on `strplit1()`, just like we do with other R functions. This requires that your package has a special R documentation file, `man/strsplit1.Rd`, written in an R-specific
markup language that is sort of like LaTeX.

Fortunately, we don't have to write that directly, we can write a specially formatted comment right above `strsplit1()` in its source file and then let a package called `roxygen2` handle the creation
of that file.

If you use RStudio, open `R/strsplit1.R` in the source editor and put the cursor somewhere in the `strsplit1()` function definition. Now do `Code > Insert roxygen skeleton`.

A very special comment should appear above your function, in which each line begins with `#'`. After few modifications it should look like something below:

```r
#' Split a string
#'
#' @param x A character vector with one element.
#' @param split What to split on.
#'
#' @return A character vector.
#' @export
#'
#' @examples
#' x <- "one,two,three,four"
#' strsplit1(x, split = ",")
strsplit1 <- function(x, split) {
  strsplit(x, split = split)[[1]]
}

```

We're almost done. Now we need to trigger the conversion of this new roxygen comment into `man/strsplit1.Rd` with `document()`:

```r
> document()
i Updating regexcite documentation
i Loading regexcite
Writing NAMESPACE
Writing strsplit1.Rd
```

Now you can preview your help file with `?strsplit1`!

## `NAMESPACE` changes

In addition to convertion `strsplit1()`'s special comment into `man/strsplit1.Rd`, the call to `document()` updates the `NAMESPACE` file, based on `@export` directives found in roxygen comment.
Open `NAMESPACE` for inspection, the contents should be:

```
# Generated by roxygen2: do not edit by hand

export(strsplit1)
```

The export directive in `NAMESPACE` is what makes `strsplit1()` available to user after attaching regexcit via `library(regexcite)`.

## `check()` again

regexcite should pass `R CMD check` cleanly now, the last few lines after execution:

```r
-- R CMD check results ----------------------------------------------------------------------------------------------------------------- regexcite 0.0.0.9000 ----
Duration: 17.7s

0 errors √ | 0 warnings √ | 0 notes √
```

## `install()`

Since the minimum viable version of the package is ready, we can install the regexcite into our library via `install()`:

```r
install()
```

Now the regexcite can be used like any other package. Let's come back to our example, but first restart R session to ensure we have a clean workspace.

```r
> library(regexcite)
> 
> x <- c("one,two,three,four")
> strsplit1(x, split = ",")
[1] "one"   "two"   "three" "four" 
```

It works!

## `use_testthat()`

We've tested `strsplit1()` informally, in a single example. We can formalize this as a unit test. This means we express a concrete expectation about the correct `strsplit1()` result for a specific
input.

First, we declare our intent to write unit tests and to use the testthat package for this via `use_testthat()`.

```r
> use_testthat()
√ Setting active project to 'C:/Users/halupkam/regexcite'
√ Adding 'testthat' to Suggests field in DESCRIPTION
√ Setting Config/testthat/edition field in DESCRIPTION to '3'
√ Creating 'tests/testthat/'
√ Writing 'tests/testthat.R'
* Call `use_test()` to initialize a basic test file and open it for editing.
```

This initialized the unit testing machinery for your package:
* Added `Suggests: testthat` to `DESCRIPTION`,
* Created the directory `tests/testthat/`
* Added the script `tests/testthat.R`

That's a good starting point it's up to us to write the actual tests!

The helper `use_test()` opens and/or creates a test file.

```r
> use_test("strsplit1")
√ Writing 'tests/testthat/test-strsplit1.R'
* Modify 'tests/testthat/test-strsplit1.R'
```

This created a new file `tests/testthat/test-strsplit1.R`. Let's put some content inside:

```r
test_that("strsplit1() splits a string", {
  expect_equal(strsplit1("a,b,c", split = ","), c("a", "b", "c"))
})
```

This tests that `strsplit1()` gives the expected result when splitting a string.

Let's run this test:

```r
> test()
i Loading regexcite
i Testing regexcite
√ | F W S  OK | Context
√ |         1 | strsplit1                                                     

== Results ===================================================================
[ FAIL 0 | WARN 0 | SKIP 0 | PASS 1 ]
```

Your tests are also run whenever you `check()` the package. In this way you basically augment the standard checks with some of your own that are specific to your package.
You can use the `covr` package to track what proportion of your package's source code is exercised by the tests.

## `use_package()`

You will inevitably want to use a function from another package in your own package. Just as we needed to **export** `strsplit1()`, we need to **import** functions from the namespace of other packages.
If you plan to submit package to CRAN, this even applies to functions in packages that you think of as "always available" such as `stats::median()` or `utils::head()`.

Let's imagine you'd rather build regexcite based on `stringr` than base R's regular expression functions. The `stringr` package "provides a cohesive set of functions designed to make working with strings
as easy as possible".

First, you need to declare your genetal intent to use some functions from there with `use_package()`

```r
> use_package("stringr")
√ Adding 'stringr' to Imports field in DESCRIPTION
* Refer to functions with `stringr::fun()`
```

`stringr` package was added to the "Imports" section of `DESCRIPTION`.

Let's revisit `strsplit1()` to make it more stringr-like:

```r
str_split_one <- function(string, pattern, n = Inf) {
  stopifnot(is.character(string), length(string) <= 1)
  if (length(string) == 1) {
    stringr::str_split(string = string, pattern = pattern, n = n)[[1]]
  } else {
    character()
  }
}
```

We:
* Renamed the function to `str_split_one()` to signal that it's a wrapper around `stringr::str_split()`.
* Adopted the argument names from `stringr::str_split()`.
* Introduced a bit of argument checking and edge case handling.
* Used the `package::function()` form when calling `stringr::str_split()`. This specifies that we want to call the `str_split()` function from the stringr namespace.

Where should we write this new function definition? For now, let's stick to convention where we name the `.R` file after the function it defines so we need to do some file shuffling.

Function `rename_files()` can help us with this - renaming a file in `R/` and associated one below `test/`.

```r
> rename_files("strsplit1", "str_split_one")
√ Moving 'R/strsplit1.R' to 'R/str_split_one.R'
√ Moving 'tests/testthat/test-strsplit1.R' to 'tests/testthat/test-str_split_one.R'
```

Remember that we still need to update the contents of these files!

Updated `R/str_split_one.R`:

```r
#' Split a string
#'
#' @param string A character vector with, at most, one element.
#' @inheritParams stringr::str_split
#'
#' @return A character vector.
#' @export
#'
#' @examples
#' x <- "alfa,bravo,charlie,delta"
#' str_split_one(x, pattern = ",")
#' str_split_one(x, pattern = ",", n = 2)
#'
#' y <- "192.168.0.1"
#' str_split_one(y, pattern = stringr::fixed("."))
str_split_one <- function(string, pattern, n = Inf) {
  stopifnot(is.character(string), length(string) <= 1)
  if (length(string) == 1) {
    stringr::str_split(string = string, pattern = pattern, n = n)[[1]]
  } else {
    character()
  }
}
```

Updated `tests/testthat/test-str_split_one.R` with a couple more tests:

```r
test_that("str_split_one() splits a string", {
  expect_equal(str_split_one("a,b,c", ","), c("a", "b", "c"))
})

test_that("str_split_one() errors if input length > 1", {
  expect_error(str_split_one(c("a,b","c,d"), ","))
})

test_that("str_split_one() exposes features of stringr::str_split()", {
  expect_equal(str_split_one("a,b,c", ",", n = 2), c("a", "b,c"))
  expect_equal(str_split_one("a.b", stringr::fixed(".")), c("a", "b"))
})
```

Before we take the `str_split_one()` for a test drive, we need to call `document()` as it does two main jobs:
1. Converts roxygen comments into proper R documentation
2. (Re)generates `NAMESPACE`

The second point is especially important here, since we will no longer export `strsplit1()` and we will newly export `str_split_one()`.

```r
> document()
i Updating regexcite documentation
i Loading regexcite
Writing NAMESPACE
Writing NAMESPACE
Writing str_split_one.Rd
Deleting strsplit1.Rd
Warning message:
In setup_ns_exports(path, export_all, export_imports) :
  Objects listed as exports, but not present in namespace: strsplit1
```

Warnings always occur when you remove something from the namespace.

Try out the new `str_split_one()` function by simulating package installation via `load_all()`:

```r
> load_all()
i Loading regexcite
> str_split_one("one, two, three", pattern = ", ")
[1] "one"   "two"   "three"
```

## `use_github()`

You've seen us making commits during the development process for regexcite. How would you connect your local regexcite package and Git repository to a companion repository on GitHub?

1. `use_github()` is a recommended helper which needs some credential setup on your end
2. Set up the GitHub repo first! a bit counter-intuitive but the easiest way to get your work onto GitHubis to initiate there, then use RStudio to start working in a synced local copy.
3. Command line Git can always be used to add a remote repository *post hoc*.

Any of these approaches will do fine.

## `use_readme_rmd()`

Now that your package is on GitHub, the `README.md` file matters. It's the package's home page and welcome mat.

The `use_readme_rmd()` function initializes a basic, executable `README.Rmd` ready for you to edit (`rmarkdown` package is required):

```r
use_readme_rmd()
#> ✔ Writing 'README.Rmd'
#> ✔ Adding '^README\\.Rmd$' to '.Rbuildignore'
#> • Update 'README.Rmd' to include installation instructions.
#> ✔ Writing '.git/hooks/pre-commit'
```

In addition to creating `README.Rmd` this adds some lines to `.Rbuildignore` and creates a Git pre-commit hook to help you keep `README.Rmd` and `README.md` in sync.

`README.Rmd` has already sections that prompt you tok:
* Describe the purpose of the package.
* Provide installation instructions.
* Show a bit of usage.

How to populate this skeleton? Anything is better than nothing, it's all about helping other people figure out how to use it.

Example:


	---
	output: github_document
	---

	<!-- README.md is generated from README.Rmd. Please edit that file -->

	```{r, include = FALSE}
	knitr::opts_chunk$set(
	  collapse = TRUE,
	  comment = "#>",
	  fig.path = "man/figures/README-",
	  out.width = "100%"
	)
	```

	**NOTE: This is a toy package created for expository purposes, for the second edition of [R Packages](https://r-pkgs.org). It is not meant to actually be useful. If you want a package for factor handling, please see [stringr](https://stringr.tidyverse.org), [stringi](https://stringi.gagolewski.com/),
	[rex](https://cran.r-project.org/package=rex), and
	[rematch2](https://cran.r-project.org/package=rematch2).**

	# regexcite

	<!-- badges: start -->
	<!-- badges: end -->

	The goal of regexcite is to make regular expressions more exciting!
	It provides convenience functions to make some common tasks with string manipulation and regular expressions a bit easier.

	## Installation

	You can install the development version of regexcite from [GitHub](https://github.com/) with:
		  
	``` r
	# install.packages("devtools")
	devtools::install_github("jennybc/regexcite")
	```

	## Usage

	A fairly common task when dealing with strings is the need to split a single string into many parts.
	This is what `base::strplit()` and `stringr::str_split()` do.

	```{r}
	(x <- "alfa,bravo,charlie,delta")
	strsplit(x, split = ",")
	stringr::str_split(x, pattern = ",")
	```

	Notice how the return value is a **list** of length one, where the first element holds the character vector of parts.
	Often the shape of this output is inconvenient, i.e. we want the un-listed version.

	That's exactly what `regexcite::str_split_one()` does.

	```{r}
	library(regexcite)

	str_split_one(x, pattern = ",")
	```

	Use `str_split_one()` when the input is known to be a single string.
	For safety, it will error if its input has length greater than one.

	`str_split_one()` is built on `stringr::str_split()`, so you can use its `n` argument and stringr's general interface for describing the `pattern` to be matched.

	```{r}
	str_split_one(x, pattern = ",", n = 2)

	y <- "192.168.0.1"
	str_split_one(y, pattern = stringr::fixed("."))
	```

Now you have to render it to make `README.md`!

The best way to render `README.Rmd` is with `build_readme()` because it takes care to render with the most current version of your package.

```r
> build_readme()
i Installing regexcite in temporary library
i Building C:/Users/halupkam/regexcite/README.Rmd
```

## The end: `check()` and `install()`

Let's run `check()` again to make sure all is still well.

```r
> check()
...
-- R CMD check results ----------------------------- regexcite 0.0.0.9000 ----
Duration: 21.4s

0 errors √ | 0 warnings √ | 0 notes √
```

There should be no errors, warning or notes. It's a good time now to re-build it, properly install...

```r
> install()
...
** testing if installed package can be loaded from temporary location
** testing if installed package can be loaded from final location
** testing if installed package keeps a record of temporary installation path
* DONE (regexcite)
```

...and celebrate!