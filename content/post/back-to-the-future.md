---
title: "Back to the Future"
description: "Maintaining R packages on CRAN sometimes means you have to find creative ways to ensure that your code runs on different platforms and on multiple versions of R---even ones that haven't been released yet."
date: "2017-11-21T16:49:57-07:00"
categories: ["code"]
tags: ["R", "packages", "CRAN"]
draft: false
images: []
---

When you submit a package update to CRAN, they ask you to check some boxes confirming that you've done your due diligence on your submission: that it passes its checks. One confirmation is that you've addressed any failures on the CRAN package check page, which looks [like this](https://cran.r-project.org/web/checks/check_results_httptest.html).

<iframe src="https://cran.r-project.org/web/checks/check_results_httptest.html" width=600 height=600/>

This page is a window into one of the key features of CRAN: it runs continuous-integration tests of your package against multiple versions of R, on multiple operating systems, with the current version of your dependencies. You can't (without a special dispensation) submit a package to CRAN that doesn't work on all platforms that R supports, and you always have to pass against the bleeding edge development version.

A benefit of this system is that, as a user, you can trust that a new R release isn't going to break your packages, and if a package update makes it to CRAN, it has at least not broken other packages in the CRAN ecosystem. The cost of supporting the system gets passed along to R developers, both core and package maintainers, who have to maintain backwards compatibility---and forwards compatibility to future R releases.

While no one likes to have a package release held up by some funky compatibility issue, I've found that I've oddly enjoyed solving these problems when they arise. It's a logic puzzle: how do I most efficiently get my code (package tests) to pass in different worlds that have different functions, options, and rules? Although it is frustrating in the moment having to spend time and energy on it, there's satisfaction in the clever solution: it feels good to solve the CRAN-sphinx's riddle.

Here's four examples.

0. UTF-8 nuisance

Like the movies, the one going back farthest in time was the least enjoyable. I had a CRAN submission rejected because the tests failed to run on Solaris, which R still supports and CRAN includes in their continuous integration checks. The problem was an encoding issue: specifically, I had non-ASCII UTF-8 in a test file, and the test file _failed to parse_. That's right: the tests didn't run and fail, the file couldn't be read at all. Naturally, `R CMD check` passed locally for me, and it passed on Travis and Appveyor.

Suppose the offending test file contained this.

```r
test_that("Something about UTF-8", {
    expect_identical(Encoding("Budějovický Budvar"), "UTF-8")
})
```

Adding `skip_on_cran()` like this

```r
test_that("Something about UTF-8", {
    skip_on_cran()
    expect_identical(Encoding("Budějovický Budvar"), "UTF-8")
})
```

doesn't fix the issue because it's the file itself that can't parse: it wasn't even getting to the point of executing the test code.

To work around the issue, I moved the `test_that` code block that contained UTF-8 to another file, "utftesting.R", and then `source()`d that file:

```r
test_that("Something about UTF-8", {
    skip_on_cran()
    source("utftesting.R", local=TRUE)
})
```

That way, the tests can still exist and be run locally and on Travis, but they're appropriately skipped on CRAN. This works because `testthat` only runs the `test-*.R` files, not any other .R or other kinds of files you have in your test directory. "utftesting.R" only gets touched after `skip_on_cran()` is called.

To prevent a future relapse, I added this check that my test- files are all ASCII:

```r
test_that("All test- files are ASCII (for CHECK)", {
    testfiles <- dir(pattern="^test.*R$")
    for (i in testfiles) {
        expect_warning(scan(i, what=character(), fileEncoding="ascii", quiet=TRUE),
            NA, info=i)
    }
})
```

1. startsWith

```r
## from future import ...
basefuns <- ls(envir=asNamespace("base"))
alt.startsWith <- function (x, prefix) {
    substr(x, 1, nchar(prefix)) == prefix
}
if (!("startsWith" %in% basefuns)) {
    startsWith <- alt.startsWith
}

alt.endsWith <- function (x, suffix) {
    z <- nchar(x)
    substr(x, z - nchar(suffix) + 1, z) == suffix
}
if (!("endsWith" %in% basefuns)) {
    endsWith <- alt.endsWith
}
```

2. median signature

```r
## Future-proofing for change to median signature in R >= 3.4
is.R.3.4 <- "..." %in% names(formals(median))

median_func <- function (v) {
    if (v) return(function (x, na.rm=FALSE, ...) .summary.stat(x, "median", na.rm=na.rm))
    return(function (x, na.rm=FALSE) .summary.stat(x, "median", na.rm=na.rm))
}

setMethod("median", "NumericVariable", median_func(is.R.3.4))
```

3. deparse

`deparse(x, control=deparseNamedList())`

```r
deparseNamedList <- function () {
    ## r73699 2017-11-09
    ## (https://github.com/wch/r-source/commit/62fced00949b9a261034d24789175b205f7fa866)
    ## adds a "niceNames" deparse option, which is now required to get named
    ## lists printed with names (they no longer are named with `control=NULL`).
    ## As it turns out, you can't specify "niceNames" prophalactically---it
    ## errors on older versions of R that don't support it. So this function
    ## standardizes the behavior across R versions.
    ##
    ## R 3.4.2 has the old behavior. The new behavior may appear in R 3.4.3, or
    ## perhaps not until R 3.5.
    past <- inherits(try(.deparseOpts("niceNames"), silent=TRUE), "try-error")
    past <- ifelse(past, "old", "new")
    return(list(new="niceNames")[[past]])
}
```
