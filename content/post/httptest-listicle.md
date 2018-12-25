---
title: "7 Hard Testing Problems Made Easy By httptest"
description: "Testing code that communicates with remote APIs can be challenging, but it doesn't have to be. The 'httptest' package for R makes testing HTTP code simple. Here are some examples of scenarios that are easily testable using httptest."
date: "2017-06-21T21:49:57-07:00"
categories: ["code"]
tags: ["httptest", "R", "testing"]
draft: false
---

I'm a big advocate of testing and test automation, both on [the team I lead](https://crunch.io/dev/) and in [my own projects](https://github.com/nealrichardson). Tests provide valuable evidence that your code works. Without tests, you're engaging in [faith-based development](https://twitter.com/enpiar/status/748082354455969796): you believe your code works because you believe in your own infallibility as a coder. Theology aside, as a practical matter, tests are liberating because they allow you to modify and extend your code without fear of breaking existing functionality. Unfortunately, testing code that communicates with remote services can be challenging. Dealing with authentication, bootstrapping server state, cleaning up objects that may get created during the test run, network flakiness, and other complications can make testing seem too costly to bother with.

To solve these problems, I developed the [httptest](https://enpiar.com/r/httptest/) package for R. The latest [2.0.0 CRAN release](https://cran.r-project.org/package=httptest) adds a range of new features that allow testing practically any request and response behavior without actually connecting to the remote service. It also includes a [vignette](https://enpiar.com/r/httptest/articles/httptest.html) that illustrates how to get started using the package. Drawing from examples of several R packages that use `httptest` (and one that doesn't), this post outlines a set of testing challenges, both common and esoteric, that it makes easy.

# The basics

{{< figure src="https://charlespaolino.files.wordpress.com/2010/10/tin-can-1.jpg" class="floating-right halfwidth" attr="'Can you hear me now?'" attrlink="https://charlespaolino.com/tag/tin-can-and-string-telephone/">}}

APIs provide a contract: if you make this request, the service will return this response. So writing an API client is all about making the right requests and handling the responses correctly. Even though these questions are the essentials that you'd want to have covered, without a testing setup like `httptest`, they can be challenging to test. `httptest` is designed not only to make testing requests and responses simple but also to make your tests easily readable.

## 1. Am I making the right request?

`httptest` provides three test **contexts**---"with"-style functions that you wrap around other code you want to execute---that mock the network connection in different ways. One context, `without_internet`, simulates the situation when any network request will
fail, as in when you are without an internet connection. Any HTTP request will raise
an error with a message containing the request information is raised. Together with the package's custom **expectation** functions, you can make assertions about the HTTP requests made in those contexts. The verb-expectation functions, such as `expect_GET` and `expect_POST`, look for the formatted error message that `without_internet` raises.

For example, suppose we have a simple function that wraps the [httpbin](http://httpbin.org) API:

~~~r
httpbin <- function (method, path, ...) {
    VERB <- get(method, asNamespace("httr"))
    VERB(file.path("http://httpbin.org", path), ...)
}
~~~

We can test that that function makes a `GET` request at the correct URL using `expect_GET`:

~~~r
without_internet({
    test_that("httpbin() constructs requests", {
        expect_GET(httpbin("GET", "get"),
            "http://httpbin.org/get")
    })
})
~~~

## 2. Does my request have the right payload?

The `expect_VERB` expectations also allow you to assert that your request include the expected query parameters:

~~~r
without_internet({
    test_that("httpbin() accepts query parameters", {
        expect_POST(httpbin("POST", "post", query=list(a=1)),
            "http://httpbin.org/post?a=1")
    })
})
~~~

And you can also assert the shape of your request body:

~~~r
without_internet({
    test_that("httpbin() also takes a request body", {
        expect_PUT(httpbin("PUT", "put", body=jsonlite::toJSON(list(a=c(1, 2)))),
            'http://httpbin.org/put',
            '{"a":[1,2]}')
    })
})
~~~

The expected results themselves are strings, easily readable in your test code.

## 3. Can I handle the server's response?

{{< figure src="https://img.memecdn.com/what-is-this_o_2856875.jpg" class="floating-right halfwidth" attr="'What Is This?'" attrlink="https://www.memecenter.com/fun/2856875/what-is-this">}}

In a second context that the package provides, `with_mock_api`, HTTP requests are intercepted and mapped to local file paths, factoring in the request URL, query, and method. If the file exists, it is loaded and returned as the response; if it does not, an error with a message containing the request information is raised, just as in `without_internet`. By supplying mock files, we can test the behavior of our code that handles API responses.

Check out the [package vignette](https://enpiar.com/r/httptest/articles/httptest.html) for a longer discussion of how to use `with_mock_api` and how to put mock files in the right place. For an abridged version, suppose we want to add tests to the popular `twitteR` package, which lacks a test suite. We can start by writing a basic test of the `getUser` function, like:

```r
with_mock_api({
    test_that("We can get a user object", {
        user <- getUser("twitterdev")
    })
})
```

When we run the tests, it fails with

```
    Error:
    GET https://api.twitter.com/1.1/users/show.json?screen_name=twitterdev
    (api.twitter.com/1.1/users/show.json-84627b.json)
```

The last part of error message is a file name. That's the mock file that the test context was looking for and didn't find. If the file had existed, it would have been loaded and the code would have continued executing *as if the server had returned it*. We can grab an example JSON response from the [API documentation](https://dev.twitter.com/rest/reference/get/users/show), put it in that location (inside our test directory), and then test more about the results of `getUser`:

```r
test_that("We can get a user object", {
    user <- getUser("twitterdev")
    expect_is(user, "user")
    expect_identical(user$name, "TwitterDev")
    expect_output(print(user), "TwitterDev")
})
```

We can also record real server responses with the `capture_requests` context, which writes the responses as files in the correct location so that they can be used in future test runs. We'll give an example of that below.

# Harder problems

You can test this basic request and response handling without `httptest` if you point your tests at a real live server. However, running against a real server, while useful for integration testing, has some limitations. You may likely need to authenticate with the API, which means that in order to run your tests, one needs access to an API token, and perhaps even *your* API token. This complicates (though does not make impossible) running tests on a continuous integration platform. You may also have to worry about API rate limiting, which could cause spurious test failures. Another big concern is what is stored on the server: do you need some state to exist in order to run your tests, and can others alter that state and disrupt you? Lastly, it's slow to connect to a remote server, which makes you [less productive](http://www.draconianoverlord.com/pages/first-principles.html#the-speed-of-your-tdd-cycle-is-the-primary-driver-of-productivity) when you're writing code and running the tests as you go.

This next set of issues are either difficult, costly, or impossible to test even with a real API to connect to, but `httptest` lets you ensure that your code does the right thing even in these cases.

## 4. Error handling

{{< figure src="https://httpstatusdogs.com/img/400.jpg" class="floating-left halfwidth" attr="HTTP Status Dogs" attrlink="https://httpstatusdogs.com/400-bad-request">}}

When you make an invalid API request, the server may return useful information about why your request was bad. Different APIs have different conventions for returning that information, however, so your code will probably need special logic for handling different server responses. You can include fast, networkless tests for your error handling code by first using `capture_requests` to record the server's response to an invalid (real) request, and then writing tests against that fixture.

In the [pivotaltrackR](https://github.com/nealrichardson/pivotaltrackR/) package, a client for the Pivotal Tracker API, there's a mock test that makes an [invalid search query](https://github.com/nealrichardson/pivotaltrackR/blob/0852ae8ff5fd3dcd0bf036e011dd59d9211cf707/tests/testthat/test-stories.R#L66-L69):

~~~r
with_mock_api({
    test_that("Bad request error handling", {
        expect_error(getStories(created="-5days..now"),
            "The date you requested could not be parsed")
    })
})
~~~

According to the API documentation, "now" is not a valid date string---it should be "today". This request, in the `with_mock_api` context, hits the [captured response file](https://github.com/nealrichardson/pivotaltrackR/blob/0852ae8ff5fd3dcd0bf036e011dd59d9211cf707/tests/testthat/www.pivotaltracker.com/services/v5/projects/12345/stories-9daeb9.R) which contains a "response" object with a 400 Bad Request status, and the response content contains the error message "The date you requested could not be parsed". That response fixture was captured by doing

```r
capture_requests(getStories(created="-5days..now"))
```

in an R session against the real Pivotal Tracker API using my authentication token, then sanitizing the response to remove sensitive information. This lets us exercise the [lines of code in our API handler](https://github.com/nealrichardson/pivotaltrackR/blob/0852ae8ff5fd3dcd0bf036e011dd59d9211cf707/R/api.R#L24-L33) that deal with 400 Bad Request responses.

## 5. Rare or difficult-to-trigger server behavior

{{< figure src="https://httpstatusdogs.com/img/429.jpg" class="floating-right halfwidth" attr="HTTP Status Dogs" attrlink="https://httpstatusdogs.com/429-too-many-requests">}}

APIs may behave differently in extreme circumstances. When handling large requests or a high volume of requests, some APIs respond by advising users that they should back off. This rate-limiting behavior needs to be handled by your code that communicates with the API, yet it is difficult to test against a real server: you may not know what threshold triggers the rate limiting; if you did, it would likely take a lot of requesting (i.e. time) to trigger the limit; and then if you manage to hit the limit, then you can't run anymore tests against the API!

With `httptest`, you can create a fixture that has whatever HTTP response status code, headers, and content that you want, so you can make one that looks like what the API returns when it is pushing back. Using that, you can then test that your code handles that response as intended.

In the [Crunch.io API](http://docs.crunch.io), when an operation requires moving a lot of data around before it can return a result, the server responds with a 503 status and includes a "Retry-After" header indicating when the client may try again and expect a result ready. [This code](https://github.com/Crunch-io/rcrunch/blob/d1b5cade5e7b0608ddc1ce5de1a576e2d3614d84/R/api.R#L105-L116) in the [crunch](https://crunch.io/r/crunch/) package handles all API requests and handles the server's responses appropriately. When it hits a 503 response on a `GET` request, it messages to the user that it's going to retry, waits the amount of time that the header recommends, then does a fresh `GET` on the request URL.

The [test](https://github.com/Crunch-io/rcrunch/blob/d1b5cade5e7b0608ddc1ce5de1a576e2d3614d84/tests/testthat/test-api.R#L31-L35) relies on a [mock response](https://github.com/Crunch-io/rcrunch/blob/d1b5cade5e7b0608ddc1ce5de1a576e2d3614d84/tests/testthat/app.crunch.io/503.R) that is an `httr` "response" class object with a 503 `status_code`. So, a `GET` on that resource "returns" 503 status, which triggers the relevant API handler code, and does the retry:

```r
with_mock_api({
    test_that("503 on GET with Retry-After is handled", {
        expect_message(resp <- crGET("https://app.crunch.io/503/"),
            "This request is taking longer than expected. Please stand by...")
        expect_identical(resp, crGET("https://app.crunch.io/api/"))
    })
})
```

The 503 fixture object was created not from a real server response---it's hard to trigger---but rather by constructing the object that was needed in R. You can also dump out a regular, successful "response" object and then edit the .R fixture file to give it a different status code and headers---it's just a text file.

## 6. Pagination

Many APIs paginate the responses of queries that could yield potentially large responses. Your code may want to conceal that API detail from its users and collect the results from all "pages" and return them together. `httptest` easily lets you test functions that makes multiple requests because you can provide mock responses for each request, which allows the rest of your code to proceed evaluating using the mocks.

The `pivotaltrackR` package deals with pagination in this way. The API defines a convention for wrapping paginated responses in an ["envelope"](https://www.pivotaltracker.com/help/api#Paginating_List_Responses) object that returns information about the page size, the total number of responses, and where you are in the list. The [R code](https://github.com/nealrichardson/pivotaltrackR/blob/0852ae8ff5fd3dcd0bf036e011dd59d9211cf707/R/api.R#L44-L59) that wraps that API steps through the paginated responses as needed to collect them all. That way, the R user can just call `getStories` and will get all of the stories that match the query, without having to think about---or even be aware of---the API's pagination behavior.

And because the R code that the package user needs is simple, the [test](https://github.com/nealrichardson/pivotaltrackR/blob/0852ae8ff5fd3dcd0bf036e011dd59d9211cf707/tests/testthat/test-stories.R#L77-L80) for this looks simple:

```r
test_that("getStories when there is pagination", {
    s <- getStories(label="really common label")
    expect_length(s, 5)
})
```

Behind the scenes, however, two (mocked) requests are made. The first one hits [this](https://github.com/nealrichardson/pivotaltrackR/blob/0852ae8ff5fd3dcd0bf036e011dd59d9211cf707/tests/testthat/www.pivotaltracker.com/services/v5/projects/12345/stories-0ad23d.json) response, which has

```json
{
    "pagination": {
        "limit": 3,
        "offset": 0,
        "total": 5
    },
    "data": [...]
}
```

and the [second one](https://github.com/nealrichardson/pivotaltrackR/blob/0852ae8ff5fd3dcd0bf036e011dd59d9211cf707/tests/testthat/www.pivotaltracker.com/services/v5/projects/12345/stories-97adb9.json) has `"offset":3` and contains records 4 and 5. The mock file paths contain the hash of the query parameters in the requests made, which is how they are distinguished. I did create them from an actual API response, but because the fixtures are plain-text files I can see and edit, I could reduce the actual size down from the 100 results per page that the API paginates at to 3 per page so that the size of the test data is smaller. The code follows the correct logic, following what the server responds with.

## 7. Code that shouldn't make a request

{{< figure src="http://cf.chucklesnetwork.com/items/8/3/5/2/3/original/what-if-silence-is-just-another-type-of-sound.jpg" class="floating-right halfwidth" attr="Conspiracy Keanu" attrlink="http://creatememe.chucklesnetwork.com/memes/83523/what-if-silence-is-just-another-type-of-sound/">}}

Mocking API responses isn't the only thing you might want to do in order to test your code. Sometimes, the request that matters is the one you don't make. Here's a example of how `without_internet` can be used to assert that code that should not make network requests in fact does not. This is a simplified version of a test from the [httpcache](https://enpiar.com/r/httpcache/) package, a library that implements a query cache for HTTP requests in R. The point of the query cache is that only the first time you make a certain GET request should it hit the remote API; subsequent requests should read from the cache and not make a request. The test first makes a request (artificially, using `with_fake_http`, the third test context the package provides) to prime the cache.

```r
with_fake_http({
    test_that("Cache gets set on GET", {
        expect_length(cacheKeys(), 0)
        expect_GET(a <- GET("https://app.crunch.io/api/"),
            "https://app.crunch.io/api/")
        expect_length(cacheKeys(), 1)
        expect_identical(a, getCache("https://app.crunch.io/api/"))
    })
})
```
Then, using `without_internet`, the test checks two things: first, that doing the same GET succeeds because it reads from cache; and second, that if you bypass the query cache, you get an error because you tried to make a network request.

```r
without_internet({
    test_that("When the cache is set, can read from it even with no connection", {
        expect_identical(GET("https://app.crunch.io/api/")$url,
            "https://app.crunch.io/api/")
    })
    test_that("But uncached() prevents reading from the cache", {
        expect_error(uncached(GET("https://app.crunch.io/api/")),
            "GET https://app.crunch.io/api/")
    })
})
```

This tells us that our cache is working as expected: we can get results from cache and we don't make a (potentially expensive) network request more than once.

# What about when...

There's more that `httptest` can do, and even more that it can grow to support. Check it out, and if you encounter features of an API that could be better supported or mocked by `httptest`, please [make an issue on GitHub](https://github.com/nealrichardson/httptest/issues)!
