---
title: "One-Hour Package"
description: "Here's the story of the time I wrote a R package for an undocumented web API in under an hour, with documentation and full test coverage. With the right tools, creating an API client can be quick and painless."
date: "2017-08-11T16:49:57-07:00"
categories: ["code"]
tags: ["httptest", "R", "tdd", "packages"]
draft: false
images: ["/img/fotomat.jpg"]
---

At [Crunch](http://crunch.io/), I occasionally pick up small [coding projects to help with project management](../../../07/04/better-management-through-code/). Recently, I wanted a way to get automated reporting on our open issues in [Usersnap](http://usersnap.com/), a service embedded in our web application that allows users to report bugs from within the app. My thought was to set up a cron job to report daily summaries in our Slack channel on things like the number of reports created, resolved, and so on, so that we could make sure that bug reports were being addressed quickly.

{{< figure src="/img/fotomat.jpg" class="floating-right halfwidth" attr="Wikimedia Commons" attrlink="https://commons.wikimedia.org/wiki/File:This_is_a_typical_drive-up_Fotomat_booth..jpg">}}

Given that I don't have much time to code, and that I've been working on [tools to make working with APIs easier](https://github.com/nealrichardson/httptest), I wanted to challenge myself: how quickly could I write an R package for a new API? In addition to getting the job done, I set a few additional requirements: the package must have full test coverage, be fully documented, and pass `R CMD check`—that is, it must be complete enough for [CRAN](https://cran.r-project.org/) submission.

Just to make it interesting, it turns out that the API I set out to tackle is neither documented nor even officially acknowledged. When I asked Usersnap about their API, they responded that they "currently do not offer API access." Clearly, though, there was an API behind their web application, so I was going to have to infer how the API works by examining how their app uses their unpublished API. It reminded me of my days in grad school collecting data by scraping Brazilian government websites, so with a touch of nostalgia, I embraced the challenge.

Despite the obstacles, from start to finish, it took me an hour between generating the package skeleton and getting passing builds on [Travis-CI](https://travis-ci.org/) and [Appveyor](https://www.appveyor.com/) with [100 percent line coverage](https://codecov.io/gh/nealrichardson/useRsnap). I've written a few public [API](https://github.com/Crunch-io/rcrunch) [client](https://nealrichardson/pivotaltrackR) [packages](https://github.com/nealrichardson/useRsnap) now, and in the process have worked out a good system that gets them started quickly and makes them easy to both test and extend. This approach worked well here too, and in the discussion below, I'll show you how. There are code snippets inline, but it may be easier to get a more complete picture of the project by [browsing it on GitHub](https://github.com/nealrichardson/useRsnap).

# Tools

We'll need a few R packages to move forward. A couple merit highlighting here; the rest I'll mention in the discussion below. First, when dealing with HTTP APIs in R, the [httr](https://github.com/r-pkg/httr) package is essential. It handles requests and responses and lets you focus on how to translate your (users') intuitions about the data you're accessing into sensible R functions and objects.

Next, we'll need some tools for writing tests. [Testing is especially important](https://t.co/IljRyeTIsj) when writing packages. The [testthat](https://github.com/hadley/testthat) package is the current standard for R package test suites, but it lacks some useful tools for working with remote APIs. To make it easy to test API packages, I wrote [httptest](https://github.com/nealrichardson/httptest), which extends `testthat`. As we'll see below, `httptest` allowed me to get test coverage for all of the lines of code I wrote and enabled me to run the tests quickly and without complication on public continuous-integration services (Travis-CI and Appveyor).

## Step 1: Set up the package skeleton

The first step is to set up some basic scaffolding for the R package. Base R has a `package.skeleton` function that I don't find particularly useful. There are some helpers in the `devtools` package as well, but for simplicity, I like the [skeletor](https://github.com/nealrichardson/skeletor) package skeleton generator. With just one line of code, it sets everything up for using GitHub, Travis-CI, Codecov.io, and Appveyor, gives you a basic test suite setup, and includes helpful instructions for things like how to write a package "Description" that will pass muster with CRAN.

So, make the skeleton and open up the files in your preferred editor or IDE:

    R -e 'skeletor::skeletor("useRsnap")'
    cd useRsnap
    atom .

## Step 2: Add general API wrappers

Our next step is to create [R/api.R](https://github.com/nealrichardson/useRsnap/blob/8c2662de5e726ad99e5985b92b97ef5b092c5e0a/R/api.R) and drop in a few boilerplate functions, which we'll extend as needed. It's brief: only 33 lines long including comments, with five functions, three of which are one-liners. The core functions, through which every request and response for every API endpoint will go, include

1. a function through which all API requests will pass;
2. a function that assembles necessary request configuration for this API, such as authentication headers or cookies;
3. a generic API response handler, which handles success and error responses according to the API's conventions;
4. a URL-constructing utility.

By centralizing this logic, it will be cheaper to implement each new API endpoint in our package. APIs follow conventions; good APIs have strict internal coherence of these conventions. Every request will require authentication to be expressed in the same way, and response status codes and error message formats will be standardized, so it makes sense that all requests and responses pass through the same core logic in your client code.

Since these functions build in the configuration needed to make requests of the API, I'll describe them in more detail in the next section.

## Step 3: Work out authentication and other configuration

Most APIs require authentication; many also need a project ID or similar to be specified in the request as well. We need to determine what those configuration parameters are, how package users will supply them, and how to incorporate them into requests.

If the API is well documented, it will be clear what parameters a user needs to supply and how you should supply them. Particularly friendly APIs also give you an easy way to create an API token with limited authorization scope, which you can pass in with your requests from R.

### Don't try this at home

In this case, with no documentation available and no official way to request an API token, I had to do some guessing. The project ID was easy to identify—it was even in the browser URL—but authentication was trickier. Watching the network panel in the inspector of my web browser when I used the Usersnap web app, I looked at the requests made to see what they were doing and how they included authentication. It was clear that authentication was being sent in a cookie, but there were many cookies being sent with each request, so it was unclear which was the operative one.

Ultimately, I watched the inspector closely as I logged in again and saw that my authentication request responded with a `Set-Cookie` header containing one key: "bugrep". That must be the auth cookie. So I copied it from the console and used that in my R session, as if it were a proper API access token. Some crude testing confirmed that I got a success response from the API when I included it in a request.

A better solution might have been to implement the same authentication requests that the browser made, so that you'd essentially prompt your R user for a username and password and authenticate the same way. But, I was using Google Account OAuth with Usersnap, and while that's possible to do with `httr`, it wasn't something I wanted to mess with—not with the limited time I had for writing the API client. Perhaps later.

### How should users supply their credentials to the R session?

Once we know what credentials are required to authenticate, we need to determine how package users will provide them. There are a few alternatives: set them as `options` and thereby be able to specify them in your `.Rprofile`; set them as environment variables; or some configuration file format. Security-wise, there's really not much difference. And, from the perspective of bootstrapping a new package, it's not an architecturally significant decision: it's easy to support multiple ways of specifying these parameters should you decide you want to do something different in the future. So, just pick one.

I opted for `options` purely out of convenience, both for me as a package developer and for ease of setting them as a user—no special setting and loading functions are required. We need two options, which I namespaced as "usersnap.project" and "usersnap.cookie", so

```r
options(usersnap.project="some-project-slug",
        usersnap.cookie="bugrep=4de1f6somehashstring912fe0c")
```

sets them, either in your current session, or for all sessions if you put it in your `.Rprofile`.

### Including configuration in requests

The next step is to integrate these parameters into our boilerplate functions. In this API, it appeared that the "project" was built into the request URLs, so the URL-constructing utility looks like:

```r
apiRoot <- "https://ec2.usersnap.com/angular/apikeys"

usURL <- function (segment, project=getOption("usersnap.project"), ...) {
    file.path(apiRoot, project, segment)
}
```

This means that `usURL("reports/42")` would return `"https://ec2.usersnap.com/angular/apikeys/some-project-slug/reports/42"`.

The auth cookie needs to be set as `httr` request "config" for all requests, so I defined a config function that pulls the cookie string from the options and included that result in all requests that go through the generic API function:

```r
#' @importFrom httr config
usConfig <- function () {
    return(config(cookie=getOption("usersnap.cookie")))
}

#' @importFrom httr GET
usAPI <- function (verb, url, config=list(), ...) {
    FUN <- get(verb, envir=asNamespace("httr"))
    x <- FUN(url, ..., config=c(usConfig(), config))
    return(handleUSResponse(x))
}
```

So `usAPI("GET", usURL("reports/42"))` would perform a `GET` request on the report URL mentioned above, including our Usersnap auth cookie in the request.

The other basic functions are the response handler

```r
#' @importFrom httr http_status content
handleUSResponse <- function (resp) {
    if (resp$status_code >= 400L)  {
        msg <- http_status(resp)$message
        ## The API tends not to respond with anything else useful, so there's
        ## nothing to add to that message
        stop(msg, call.=FALSE)
    } else {
        return(content(resp))
    }
}
```

which doesn't yet do much other than stop on error and otherwise return the response content, but it will grow as we learn more about the API and its standard response behaviors; and a convenience `GET` method that passes through the API function:

```r
usGET <- function (url, ...) {
    usAPI("GET", url, ...)
}
```

I like to namespace these core functions, primarily so that you can call, for example, `usGET("some/url")` and you know that it will include the right authentication credentials for that API. That's why these functions all have "us" in the name.

Note that this is mostly boilerplate code at this point. The only real customization to the current API is the specification of the auth cookie. The `usURL` is a convenience tool for building URLs that follow an apparent convention, but it's not essential.

While we don't strictly need a generic requesting function and a generic response handler to start working on our first methods, they will help us as we go forward. They give us a place to put our common API logic, so the cost of adding each additional API endpoint wrapper stays low. Plus, they help encapsulate our code and keep the complexities of the API and of HTTP separated from our R object space as much as possible.

## Step 4: Pick an endpoint

Now, with the core infrastructure in place, let's pick our first endpoint and write a wrapper for it. The one I was most interested in was the "reports" list so that I could get a catalog of recently opened issues. This is the first view you see when you open up a project in Usersnap's web interface.

While I usually like to write tests before I write the main code, for this initial case, it made sense to debug interactively, against the live API, just to make sure the end-to-end works. Then we can set up the unit tests, with and without API mocks.

Normally, you'd read the docs and set up something that matches their guidelines. Good documentation usually gives example requests and responses as well. In this case, my available documentation was "whatever the web app does". In some ways, not having API documentation helped to keep the initial implementation lean: because I have no idea what the API supports other than what I see the web app doing, there was no sense in trying to be too clever about designing the R function interface. I just set the web app's query as the default, allowed it to be overridden in the `...` arguments so that I could experiment and not be limited by those defaults, and wrapped it up neatly:

```r
getReports <- function (...) {
    query <- modifyList(list(
        includeapikey="false",
        offset=0,
        limit=10,
        search='[{"type":"state","id":"open"}]'
    ), list(...))
    return(usGET(usURL("reports"), query=query)$data)
}
```

So, drop that in, install the package, and try it out against the live API.

```r
library(useRsnap)
reps <- getReports()
length(reps) # Should be 10
reps[[1]]    # Inspect one
```

After tweaking to make sure that the response looked as expected (my first attempt didn't return only the `$data` element of the response object, so `length` was not 10), I repeated the `getReports` query but wrapped in `httptest`'s [capture_requests](https://github.com/nealrichardson/httptest/blob/master/inst/doc/httptest.md#recording-mocks-with-capture_requests) context, which records the responses from real API requests as fixtures that you can use in tests.

```r
reps <- capture_requests(getReports())
```

Now we can write tests around that fixture using the [with_mock_API](https://github.com/nealrichardson/httptest/blob/master/inst/doc/httptest.md#the-with_mock_api-context) context, and we can iterate on the R code and tests as needed, all without hitting the actual API or making any potentially expensive network requests. In [tests/testthat/test-get-reports.R](https://github.com/nealrichardson/useRsnap/blob/8c2662de5e726ad99e5985b92b97ef5b092c5e0a/tests/testthat/test-get-reports.R),

```r
context("Get reports")

with_mock_API({
    test_that("Get reports", {
        reps <- getReports()
        expect_length(reps, 3)
    })
})
```

Note that this test says there's only 3 records, not the 10 from my fixture. To save on file size, I deleted the other seven from the recorded mock. It's a plain text file, so I could massage it as needed.

We can add a few more tests from here, but we're basically done with code—we have a function that returns a list of report entries in our R session. That's enough of an API wrapper to start writing my daily cron job summarizing our support tickets.

## Step 5: Document and wrap up

At this point, we've set up the skeleton for the package, worked out authentication and configuration for the API, added the core functions through which all requests will go, and added the first method that we're going to expose to our package users. It's been about 40 minutes at this point. We'll spend the next 20 minutes on documentation and polishing.

First, we need to document the `getReports` function. The [roxygen2](https://github.com/klutometis/roxygen) system for inline documentation of functions is the way to go. Add a description of what the function does, what the arguments are (to the best of our understanding of the API), include a usage example, and finally add the `@export` note so that the function is exported from the package namespace so that it is public to our package users. Typically this would also include external links to the API documentation, but that wasn't available in this particular case. Finally, as in the examples above, our internal functions already have the `@importFrom` statements to bring into the package namespace the functions from `httr` that we need.

```
#' Query Your Usersnaps
#'
#' @param ... query parameters to include. There's no documentation, but known
#' parameters are (1) includeapikey, which should always be `"false"`; (2)
#' offset; (3) limit; and (4) search, which by default is
#' `'[{"type":"state","id":"open"}]'`, the default in the Usersnap GUI. You
#' can also specify a "project" to query against a project other than your
#' default.
#' @return A list of report entries.
#' @export
#' @importFrom utils modifyList
```

In this phase, we'll run `R CMD check` until it passes. The main things we'll be battling with here will be namespace (imports and exports) and documentation. Run it, fix whatever it says is wrong, and repeat until the docs and namespace are satisfactory.

Next, we need a useful README file that gives instructions on how to install and configure the package, particularly with regards to authentication. We want package users to be able to get started quickly.

Finally, there's some source control and continuous integration setup, which might take you 30 seconds if you know where to go and what to click and type. Fortunately, the `skeletor` package skeleton puts [instructions in your README](https://github.com/nealrichardson/skeletor/blob/master/inst/pkg/README.md#how-to-finish-setting-up-your-new-package) for how to `git init` and set up your repository on GitHub, Travis-CI, and Appveyor. Follow the instructions, `git add .`, `git commit`, and `git push`. Then we're done.

# Lean and extensible

So, we've made a basic package that provides an API client in R, even doing so without any API documentation for support. And it's fully tested.

Sure, it's bare: it only really supports a single endpoint. But I only needed the one endpoint to make my reporting job work. Any other endpoints or options to this one can be added when they're needed and not sooner, following the [YAGNI principle](https://martinfowler.com/bliki/Yagni.html).

But even though the package doesn't do much yet, the framework established for making requests and handling responses will make it easy to add more endpoint methods when we need to. And the testing setup makes it simple to make incremental improvements. Indeed, I found it simple to extend `getReports` to [query by "open" or "closed" state](https://github.com/nealrichardson/useRsnap/commit/bec1472d8f6396270bf675b7cb6fa0a26d9daf7a) when I needed to access the closed reports list. So, you can use the package and quickly extend it when you need.
