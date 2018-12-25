---
title: "Back to the Future"
description: "Maintaining R packages on CRAN sometimes means you have to find creative ways to ensure that your code runs on different platforms and on multiple versions of R---even ones that haven't been released yet."
date: "2018-02-01T16:49:57-07:00"
categories: ["code"]
tags: ["R", "packages", "CRAN"]
draft: false
images: ["https://enpiar.com/img/bttf-cover.png"]
---

When you submit a package update to CRAN, they ask you to check some boxes confirming that you've done your due diligence on your submission: that it passes its checks. One confirmation is that you've addressed any failures on the CRAN package check page, which looks [like this](https://cran.r-project.org/web/checks/check_results_httptest.html).

{{< figure src="/img/cran-check-httptest.png" attr="Hey, all passing today!">}}

This page is a window into one of the key features of CRAN: it runs continuous-integration tests of your package against multiple versions of R, on multiple operating systems, with the current version of your dependencies. You can't (without a special dispensation) submit a package to CRAN that doesn't work on all platforms that R supports, and you always have to pass against the bleeding-edge development version.

A benefit of this system is that, as a user, you can trust that a new R release isn't going to break your packages, and if a package update makes it to CRAN, it has at least not broken other packages in the CRAN ecosystem. The cost of supporting the system gets passed along to R developers, both core and package maintainers, who have to maintain backwards compatibility---and forwards compatibility to future R releases.

Every few months, it seems that there is a change in the development version of R that causes test failures in one of my packages, and I need to find a way to work around it, whether to submit an update to CRAN or just to make my Travis-CI builds stop failing.

While no one likes to have a package release held up by some funky compatibility issue, I've found that I've oddly enjoyed solving these problems when they arise. It's a logic puzzle: how do I most efficiently get my code (package and test suite) to pass in different worlds that have different functions, options, and rules? Although it is frustrating in the moment having to spend time and energy on it, there's satisfaction in the clever solution: it feels good to solve the CRAN-sphinx's riddle.

Here's a few examples, set to the story of the Marty McFly trilogy.

# I. Back to the recent past

{{< figure src="/img/bttf.gif" class="floating-right halfwidth" attr="Gotta get back in time" attrlink="https://media.giphy.com/media/JIzGoJKPAHO2Q/giphy.gif">}}

Python has this fun [feature](https://docs.python.org/2/library/__future__.html) in which some functions that are in upcoming releases can be pulled into current or older versions. It facilitates writing code that follows the new style but still runs on old Python, so upgrading to new tools and deprecating old ways can be done gradually. And it sounds cool:

```python
from __future__ import print_function
```

R doesn't have a similar mechanism (though it has natively had a `print()` function for forever, just sayin').

I had a couple of instances where my package code worked on the current version of R, but it didn't in the previous version (aka "oldrel"), which CRAN requires. So, I needed to add some shims to make it work and pass all checks.

First, the handy `startsWith` function was [added in R 3.3](https://github.com/wch/r-source/blob/trunk/doc/NEWS.Rd#L2108-L2110). It checks whether the first characters of a string match a given value. I started using it then, but still had to have code pass on R 3.2. So I added this code to my package to backport it:

```r
alt.startsWith <- function (x, prefix) {
    substr(x, 1, nchar(prefix)) == prefix
}
if (!("startsWith" %in% ls(envir=asNamespace("base")))) {
    startsWith <- alt.startsWith
}
```

Note that the "alt" function is only used if `base::startsWith()`, the official version, doesn't exist. Defining the "alt" function outside of the `if` statement has the added benefit of allowing me to run tests on it regardless of what version of R I'm running. This increases confidence that my code will work correctly on an old version of R, even if I don't use that version anymore.

Similarly, in the [skeletor](https://github.com/nealrichardson/skeletor/blob/5213ee23796dfde52cef498453086b415aec1539/tests/testthat/helper.R#L16-L28) package tests, I conditionally define the `Rcmd()` function, which was also added in R 3.3:

```r
if ("Rcmd" %in% ls(envir=asNamespace("tools"))) {
    Rcmd <- tools::Rcmd
} else {
    # R < 3.3
    Rcmd <- function (args, ...) {
        if (.Platform$OS.type == "windows") {
            system2(file.path(R.home("bin"), "Rcmd.exe"), args, ...)
        } else {
            system2(file.path(R.home("bin"), "R"), c("CMD", args), ...)
        }
    }
}
```

Of course, this conditional definition of the functions isn't really necessary: I could always use my version of the function, even if the official one exists. I included it here because it illustrates a trick for identifying the state of the version of R the code is currently running in and changes behavior accordingly. By checking for the existence of a function in a namespace, we don't have to be so diligent in identifying exactly which version of R the function first appeared.

{{< figure src="/img/Erasedfromexistence.jpg" class="floating-right halfwidth" attr="If you go back in time, some functions may be erased from existence." attrlink="http://backtothefuture.wikia.com/wiki/File:Erasedfromexistance.jpg">}}

And while this is knowable for past releases if you know how to search the release notes, it's not so simple with the R-devel "unstable" builds, which every day report the same "R version". And on CRAN, the R-devel builds for different operating systems and architectures aren't in lock-step: you may be running against last week's R-devel build on Windows and yesterday's on Linux. Rather than trying to identify and parse exactly which version or development source-code revision added the behavior in question, it's more foolproof to check for the behavior itself.

# II. Back to the (actual) future

{{< figure src="/img/hoverboard.gif" class="floating-left halfwidth" attr="The future has new toys" attrlink="https://giphy.com/gifs/michael-j-fox-back-to-the-future-part-two-S2wDawnEOEV68">}}

Here are two examples of changes creeping into the development version of R meant that I had to alter package code to anticipate the future.

In R 3.4, a [change](https://github.com/wch/r-source/blob/trunk/doc/NEWS.Rd#L1074-L1075) was made to the signature of the `median()` function, adding `...`. This made `median` consistent with similar functions (e.g. `mean`) and allowed methods to be defined that take additional arguments.

That's a good change, but unfortunately it made it difficult to define those methods because you had to find a way to support both versions, with and without `...`. Using S4 methods, when you call `setMethod`, the function you assign as the method has to match the signature of the generic. So in the old (at the time, current) version of R, I needed something like

```r
setMethod("median", "MyClass", function (x, na.rm) do_stuff(x, na.rm))
```

But starting in 3.4 (and in R-devel builds prior to the release, which is when it bit me), that would fail because the signature didn't match. Instead, I needed

```r
setMethod("median", "MyClass", function (x, na.rm, ...) do_stuff(x, na.rm))
```

The first part of the solution is knowing which signature you need to match. Rather than checking version strings, the approach I took was to inspect the `median` function's arguments using the `formals` function

```r
formals(median)

## $x
##
##
## $na.rm
## [1] FALSE
##
## $...
##
```

which returns a list, and then seeing if "..." was in the names of that list:

```r
is.R.3.4 <- "..." %in% names(formals(median))
```

(Re-reading the man page for [`formals`](https://www.rdocumentation.org/link/formals?package=base) now, I see that there is a [`methods::formalArgs()`](https://www.rdocumentation.org/link/formalArgs?package=base) function that is a shortcut for `names(formals(x))`. It also turns out that there is even a `formals<-` assignment function that lets you redefine function arguments and default values!)

Then, I wrote a function that returned the right method (function) based on a logical value.

```r
median_func <- function (v) {
    if (v) {
        return(function (x, na.rm=FALSE, ...) do_stuff(x, na.rm))
    } else {
        return(function (x, na.rm=FALSE) do_stuff(x, na.rm))
    }
}
```

Finally, I called `setMethod` and evaluated `median_func` with the logical value of `is.R.3.4` to return the function with the signature that will match the generic.

```r
setMethod("median", "NumericVariable", median_func(is.R.3.4))
```

One last challenge to circumvent: while I could conditionally set the method like this, I couldn't conditionally specify the man page: the .Rd file is created at build time, not install time. This meant that it was not possible to document this function/method in a forward-compatible way. And `R CMD check` also fails if you "export" undocumented functions, so I had to remove it from the explicit namespace export too. Fortunately (I guess?), method dispatch still worked, so it was fine.

Check out the [real code](https://github.com/Crunch-io/rcrunch/blob/1e9de0de2c57d518532e49afdacb7fad09ecdaa3/R/univariate.R#L49-L65) in the [crunch](https://crunch.io/r/crunch/) package.

More recently, another change in R-devel, slated for R 3.5, broke [httptest](https://enpiar.com/r/httptest/). The `deparse` function is getting some attention, initially just internal improvements ([which caused breakage for me along the way](http://r.789695.n4.nabble.com/Bug-dput-deparse-with-named-character-vector-inside-list-td4745298.html)) but now also a change to the arguments. Up to R 3.4, `deparse` (and by association `dput`) printed named lists and vectors in a verbose way. `list(a=1)` got deparsed as `structure(list(a = 1), .Names = "a")` by default. But, you could specify "control" arguments to change that behavior, and `control = NULL` yielded `list(a = 1)` without the `structure` business. So I used that control option in `httptest`.

The [change](https://github.com/wch/r-source/commit/62fced00949b9a261034d24789175b205f7fa866) adds a "niceNames" deparse option, which is now required to get named lists printed with names---they no longer are named with `control = NULL`. So the new (future) default behavior of `deparse(list(a=1))` is what we want, and `deparse(list(a=1), control = NULL)` would now return `list(1)`, which is not what we want.

So, we want `control = NULL` on current R, and `control = "niceNames"` in future R, in any version of R where the "niceNames" argument is valid. To solve this, let's assume we have a function, let's call it `deparseNamedList`, that either returns `NULL` or `"niceNames"`, depending on the version of R. We can then call `deparse` like

```r
deparse(x, control=deparseNamedList())
```

Now we just need that function. As it turns out, if you try to specify "niceNames" on older versions of R that don't support it, you get an error. That's too bad; if `deparse` just ignored options it didn't recognize, we could just always specify "niceNames". Instead, though, we can use that validation error to determine which control parameters to provide. Something like this:

```r
deparseNamedList <- function () {
    if (inherits(try(.deparseOpts("niceNames"), silent=TRUE), "try-error")) {
        # niceNames isn't valid, so we're in old R
        return(NULL)
    }
    return("niceNames")
}
```

(Aside: that's not actually how I wrote it, though it is the more straightforward way. Instead, I did:

```r
deparseNamedList <- function () {
    past <- inherits(try(.deparseOpts("niceNames"), silent=TRUE), "try-error")
    past <- ifelse(past, "old", "new")
    return(list(new="niceNames")[[past]])
}
```

This allowed me technically to keep 100 percent line coverage in the test suite. In the more natural function, it's not possible to call every line in the function in a single version of R, but in this function, you can. Basically, test coverage became a game within a game: how to get a function that would execute every line in any version of R. Of course, it still doesn't mean that both logical cases are tested in a single version---one example among many of why 100 percent line coverage is nice to have but doesn't guarantee quality.)

To reiterate, for both `median` and `deparse`, we could have used the `R.Version()` function to figure out which version we were on. It returns even the `"svn rev"`, the revision number, for R-devel, so we could have pinpointed the change in deparse by revision and switched that way. But I like the _de facto_ way of determining which version you're on: all that matters is the presence or absence of some feature, not what the name or revision number of the code is. Particularly when dealing with changes as they come in R-devel, trying to reason about version numbers may not be the most effective solution anyway.

# III. Way back to the distant past

{{< figure src="/img/bttfiii.gif" class="floating-right halfwidth" attr="A train wreck" attrlink="http://33.media.tumblr.com/d99ae1c753b568dcb87139cf099cedb9/tumblr_nnica9WQb31qfr6udo1_500.gif">}}

Just like the movies, the one going back farthest in time was the least enjoyable. I had a CRAN submission for the [crunch](https://crunch.io/r/crunch/) package rejected because the tests failed to run on the Solaris operating system, which R still supports and CRAN includes in their continuous integration checks. The problem was an encoding issue: specifically, I had non-ASCII UTF-8 in a test file, and the test file _failed to parse_. That's right: it wasn't that the tests ran and failed; the file couldn't be read at all. Naturally, `R CMD check` passed locally for me, and it passed on Travis and Appveyor.

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

doesn't resolve the issue because it's the file itself that can't parse: it wasn't even getting to the point of executing the test code.

To work around it, I moved the `test_that` code block that contained UTF-8 to another file, "utftesting.R", and then `source()`d that file:

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
    for (f in dir(pattern="^test.*R$")) {
        expect_warning(scan(f, what=character(), fileEncoding="ascii", quiet=TRUE),
            NA, info=i)
    }
})
```

Actual code change [here](https://github.com/Crunch-io/rcrunch/commit/7f749717f6b796046671981edaeabf41edf49947), and current file status [here](https://github.com/Crunch-io/rcrunch/blob/master/tests/testthat/test-encoding.R). Maybe that's overzealous, but I'd rather not worry about a CRAN submission failing due to Solaris again.

# Make like a tree

{{< figure src="/img/arent_ready_yet_bttf.gif" class="floating-left halfwidth" attr="Coding for the future" attrlink="http://www.reactiongifs.us/wp-content/uploads/2014/07/arent_ready_yet_bttf.gif">}}

Because the language is continually evolving (slowly but surely), those who maintain packages on CRAN will periodically have to figure out how to adjust their code to work across a range of versions. I've found that adapting code based on the features and behavior of the R version in which it is running, rather than reading a version number or string, is a reliable way to keep things working, now, in the past, and into the future.

<!-- Some sort of conclusion -->
