---
title: "httptest 2.3.2 Released"
description: "The latest 'httptest' :package: release provides tools for automatically redacting sensitive information from your test fixtures so that you never accidentally publish your auth tokens. It also includes a number of smaller fixes and enhancements based on user suggestions."
date: "2017-10-20T09:00:57-07:00"
categories: ["code"]
tags: ["httptest", "R", "testing"]
draft: false
---

The 2.3.2 release of [httptest](https://enpiar.com/r/httptest/) has just been published on [CRAN](https://cran.r-project.org/package=httptest). Since the 2.0.0 release in June, several key additions have been made. See the [NEWS](https://enpiar.com/r/httptest/news/) for the full list. If you follow those links, you'll see the first new "feature": `httptest` now has a proper website, using the [pkgdown](http://pkgdown.r-lib.org/) site generator!

In terms of functionality, the biggest change has been the addition of a framework for "redacting", or removing sensitive information like tokens and ids from test fixtures. While most API responses recorded by `capture_requests` wouldn't contain your authentication credentials---the default "simplified" behavior writes out only the JSON response bodies---I more than once had to edit files by hand to remove cookies and tokens. I even had to sanitize code pushed to GitHub that contained secret tokens and needed to reset API tokens elsewhere. After watching others struggle with the same issue, it was time to implement a solution.

By default, `capture_requests` will now purge any credentials contained in cookies, standard HTTP request headers, basic HTTP authentication methods, and OAuth token management objects. If you're working with an API that uses these standards for authentication and authorization, you can just use `capture_requests` freely and never worry about accidentally publishing your personal access tokens.

However, not all APIs use these standard auth methods. For these APIs, the new redacting framework is extensible: it allows you to provide a custom function that sanitizes the recorded requests and responses to your needs.

In fact, the redacting framework lets you go even further and alter any attribute of the requests and responses recorded as mocks, including editing the request URL and response body content itself. This ability is particularly useful for cases where personal ids are included in response content. For a detailed discussion on how to use these new tools, see the new [redacting vignette](https://enpiar.com/r/httptest/articles/redacting.html).

Another noteworthy addition to `httptest` since the 2.0.0 release is that `with_mock_api` and `without_internet` now handle multipart and url-encoded form data in mocked HTTP requests. The form data is now factored into the query parameters that determine the mock file path, and when no mock is found (or when using `without_internet`), it is included in the error message that is raised, allowing you to make assertions about the request data with the `expect_VERB` expectation functions.

In addition to these new features, several other fixes and enhancements were added that generally make recording requests and using them as test fixtures more robust and easier to work with. Perhaps equally valuable to the code changes was the growth of an [FAQ](https://enpiar.com/r/httptest/index.html#faq) to help solve common setup challenges. Many of these enhancements grew out of discussions during the [rOpenSci peer review](https://ropensci.org/blog/2016/03/28/software-review) for the [googleLanguageR](https://github.com/ropensci/googleLanguageR) package. I am grateful to [Mark Edmondson](https://twitter.com/HoloMarkeD) for the [exchanges](https://github.com/ropensci/onboarding/issues/127) we had during the process---while the goal of the peer review was to improve his package, this one benefitted as well.
