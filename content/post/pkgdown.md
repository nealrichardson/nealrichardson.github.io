---
title: "Getting Down with pkgdown"
description: "Building a beautiful website for your R package is a great way to improve its documentation, usability, and visibility. 'pkgdown' makes it easy to build your site, particularly if you follow these conventions."
date: "2017-11-21T16:49:57-07:00"
categories: ["code"]
tags: ["R", "packages", "website"]
draft: false
images: []
---

[`pkgdown`](http://pkgdown.r-lib.org/) is an incredibly powerful tool for building beautiful websites for R packages. With `pkgdown` and services like [GitHub Pages](https://pages.github.com/), deploying a package website is so simple and straightforward that I'm starting to see it as an essential part of writing a package. What's more, I've found that the act of preparing a package website has led me to improve the package itself, particularly the readability and usefulness of its documentation.

I say "incredibly" powerful in the most literal sense---I could not believe how much `pkgdown` does with so little required of its user. `pkgdown` draws its power by relying on [conventions](https://en.wikipedia.org/wiki/Convention_over_configuration) of how R packages are structured. Some of these conventions are inherent to R packages, particularly what CRAN deems as acceptable. Others, however, are less obvious. If you're closely tracking the [best practices for package development](http://r-pkgs.had.co.nz/), you're may be following most of these conventions already. But best practices evolve, and if you're adding `pkgdown` to a package that is a year or two old, you may find that you need to tweak some details to make it work smoothly.

This post distills my experiences in setting up a [few](https://enpiar.com/r/httptest/) [pkgdown](https://enpiar.com/r/httpcache/) [sites](https://crunch.io/r/crplyr/) recently and exposes some of the features of `pkgdown` that I discovered along the way. While `pkgdown` does have basic documentation, whenever I hit something that didn't work quite like I expected, or if I wanted to move beyond the default behavior, I found that the best way to figure out what was happening was to read the source. My intent here is to capture what I learned [RTFS'ing](http://catb.org/jargon/html/R/RTFS.html), including links to the relevant source code, and share with you (and with my future self, who surely will have forgotten all of this).

# 1. Install `pkgdown`

If you don't have `pkgdown` already, you'll need to install it. It's not currently on CRAN, so to get it, run `devtools::install_packages("hadley/pkgdown")`. Then go make a cup of coffee or something---there are lots of dependencies to install, so it may take several minutes. That's an observation, not a complaint: you'll have a beautiful website before your coffee gets cold.

If you need coffee-brewing inspiration, here's a suggestion:

{{< youtube AtD8u9oSG4A >}}

# 2. Configuration

At this point, you could run `pkgdown::build_site()` and probably get a functioning website. In fact, give it a try and see what you get.

Generally, you won't want to stop here. The first step in customizing your site is to create the YAML file that `pkgdown` uses for configuration: `_pkgdown.yml`. The package does provide some initial [template generating functions](http://pkgdown.r-lib.org/reference/templates.html), but they don't actually create the .yml file---they print to the screen. We can solve that by using `sink()` to send the output to the file.

```r
library(pkgdown)
sink("_pkgdown.yml")
template_navbar()
template_reference()
sink()
```

You could throw `template_articles()`, the third template function, in there for good measure, if you've got multiple vignettes in your package and want to customize the display of `articles/index.html`. I personally didn't bother because by default, there is no link to that page anywhere that I can find.

Now that you have a basic YAML file, you can start customizing it. The docs for [`build_site()`](http://pkgdown.r-lib.org/reference/build_site.html) give an overview of what is possible, and you can get pretty far with trial-and-error and by looking at other `pkgdown` sites. The [site for pkgdown itself](https://github.com/hadley/pkgdown/blob/master/_pkgdown.yml) is a good place to start; [dplyr](https://github.com/tidyverse/dplyr/blob/master/_pkgdown.yml) has a more complex, customized one; my [httptest](https://github.com/nealrichardson/httptest/blob/master/_pkgdown.yml) package is somewhere in the middle.

The big thing you'll want to do is organize the "reference" (the help pages), grouping by topic and ordering them sensibly. [By default](https://github.com/hadley/pkgdown/blob/081639735104a03c01527f568a99f0ef7351433d/R/build-reference-index.R#L86), all exported, non-internal (that's `@keywords internal` in [roxygen](https://github.com/klutometis/roxygen)-speak) functions are included in the index, but you don't have to list them all. Any function you omit from the index in the YAML just won't appear in the index page, but it will still exist as a page on your site, so you can cross-reference to it.

For aesthetics in your YAML, you can remove the heavy quoting of reference and article entries that appear in the default templates, like

```yaml
articles:
- title: All vignettes
  contents:
  - '`pkgdown`'
  - '`highlight`'
```

For sensibly named files, the quoting is unnecessary, and you won't see it in good examples in the wild.

In the navbar, you can play around with ordering of entries, positioning groups of menu items in "left" or "right", and nesting menus. Note that you're not limited to just links to things in your pkgdown site---your navbar menu can include whatever you want. So, for example, if you have a personal or company website you want to link to, you can [like this](https://github.com/nealrichardson/httptest/blob/948e200df62572557fe10c3b9930bb31c4ebee2a/_pkgdown.yml#L5-L27).

One other customization I like to do to the navbar is to remove "index.html" from links so that they're prettier in the browser. That is, you see `you.github.io/yourpkg/` instead of `you.github.io/yourpkg/index.html`. Doing this does mean that if you preview your site locally by opening it as a file in your web browser (which is what `pkgdown` does in an interactive session, including in RStudio), those links won't work directly, but they do work fine when served by a web server, including GitHub Pages. It's a slight inconvenience for development but worth it for the end user experience, in my opinion. If you find it annoying to work with, just do it last, after you're done customizing your site locally.

Note that the default navbar YAML relies on a few conventions. If you have a [NEWS.md](https://github.com/hadley/pkgdown/blob/ad1b1dfb12871919e06a2aa7e366ec0980af2714/R/navbar.R#L82-L87) file at the top level of your package, it will be parsed and a link to it added to the navbar. If you have a [vignette with the same name as the package](https://github.com/hadley/pkgdown/blob/ad1b1dfb12871919e06a2aa7e366ec0980af2714/R/navbar.R#L55-L64), you'll get a "Get started" link in the navbar. If you don't have those initially and you want to add them later, you can recreate the YAML template, or you can just edit your `_pkgdown.yml` to add links to them.

# 3. Tweak your package

Now that you have the site basically set up, browse through it and make sure the content looks the way it should. Most of the changes from here on are to the package itself---the help files, vignettes, and other documentation.

## Edit man pages

Look at the "reference" index and the various help pages and see how they're displayed on the web. For me, this was an area where I could spend lots of time. I tend to write the roxygen docstrings when I write functions but seldom review how they're rendered as man pages. Sure, I could look at the HTML help files within the R package itself without `pkgdown`, or even preview the PDF version of the package manual---I'm sure everyone does `R CMD Rd2pdf man/` all the time, right? But some things stand out more clearly as needing refinement when viewed in the context of your website.

In addition to general content, a few features of the help pages are worth special attention. First, check out the "title" and "description" of each man page. The title is the top line of your roxygen block. You'll see the titles displayed on the `reference/index.html` page of your pkgdown site, and if you're like me, you'll probably immediately notice inconsistencies in how you title the man pages: capitalization, length, verb tense, etc.

The description distinction in roxygen is subtle---the first paragraph of text after the title becomes the "description", and the rest becomes "details". Description and details aren't displayed together, so if your second paragraph naturally flows from the first in your inline documentation, that's nice, but it won't render that way on the reference page. This is true in general---this is how the .Rd man pages are rendered, not a feature of `pkgdown`---but for me at least, seeing the pkgdown site made it clear that I needed to rethink how I wrote documentation so that it displayed in a helpful way.

Finally, note the names of the help page files themselves. You'll see them as the entries in the "reference" section of your `_pkgdown.yml`. These are the .Rd file names, which will be translated into .html files; thus, they define URLs on your website. Review these carefully: URLs define an API. Once you put content on the internet at that location, others can bookmark or share links to it. If you change those file names, you're breaking the API contract and those links won't work anymore. Plus, no one likes ugly URLs. So, choose them wisely.

If you have file names you want to make prettier---and if you have some autogenerated ones like `as.environment-MyClass-method.html`, you should---specify an alternate file name in the roxygen docstrings using the `@rdname` tag. See the discussion of `@rdname` [here](http://r-pkgs.had.co.nz/man.html#multiple-man) for more.

## Facilitate autolinking

One of the delightful features of `pkgdown` is that it automatically links and cross-references code across your website, including to other packages. [This page](http://pkgdown.r-lib.org/articles/test/highlight.html) illustrates the autolinking in practice.

Functions noted like `fun()` or `?fun` get auto-linked; however, just writing `fun` doesn't generate a link. (No fun!) Prior to using `pkgdown`, I usually would just refer to a function by name, no `()`, so I found that I needed to go through my README.md and NEWS.md and massage them, adding the calling parentheses so that links would be generated.

## Check vignette metadata

I had to do some updating of my vignettes to make them work correctly with `pkgdown`, specifically in their "front matter": the metadata at the top of the file. If your package is new and you're following the current [best practices](http://r-pkgs.had.co.nz/vignettes.html) for writing vignettes, you may be most of the way there already.

I was using R Markdown and building the vignettes with [knitr](https://yihui.name/knitr/), but not _à la mode_, which uses the [rmarkdown](https://github.com/rstudio/rmarkdown) package on top of [pandoc](http://pandoc.org/). For example, in [httptest](https://enpiar.com/r/httptest), the main vignette previously started like this:

```latex
<!--
%\VignetteEngine{knitr::knitr}
%\VignetteIndexEntry{httptest: A Test Environment for HTTP Requests in R}
%\VignetteEncoding{UTF-8}
-->

# httptest: A Test Environment for HTTP Requests in R

...
```

The new way, which `pkgdown` prefers, [uses YAML front matter](http://r-pkgs.had.co.nz/vignettes.html#vignette-metadata) and indicates that it should render with the `rmarkdown` package.

```yaml
---
title: "httptest: A Test Environment for HTTP Requests in R"
description: "This vignette covers the core features of the httptest package, focusing on how to mock HTTP responses, how to make assertions about requests, and how to record real requests for future use as mocks."
output: rmarkdown::html_vignette
vignette: >
  %\VignetteIndexEntry{httptest: A Test Environment for HTTP Requests in R}
  %\VignetteEngine{knitr::rmarkdown}
  %\VignetteEncoding{UTF-8}
---

...
```

This YAML front matter allows for additional fields. The `rmarkdown::html_vignette` default template will make use of "author" and "date" fields, though in the context of a package vignette, those might not be relevant. `rmarkdown` will also take your `title:` and add it as a `<h1>` header in your HTML (built) vignette, so you don't need to duplicate it in the body of your vignette, as I had previously (though you do still need to duplicate it, or at least provide something, in the `\VignetteIndexEntry` field).

I also included a `description:` field, which may look familiar to `blogdown` (or pure Hugo) users. If/when [this pull request](https://github.com/hadley/pkgdown/pull/438) is accepted (or today, if you use [my fork](https://github.com/nealrichardson/pkgdown/tree/dev) of `pkgdown`), this field will be used to generate meta tags in the HTML pages of your pkgdown site, such that when you share a link to the vignette on a social media platform or in a Slack channel, a nice preview will be generated.

One caveat: for `rmarkdown` vignettes to build correctly, you'll need to make sure you have [pandoc installed](http://pandoc.org/installing.html). If pandoc is not present, `knitr` will fall back to using the older `markdown` library, which does not handle the YAML metadata the same way. Notably, the `title:` is not printed as a header in your vignette. Unfortunately, `pkgdown` suppresses the warnings that `knitr` makes about pandoc not being available, so this won't be obvious unless you build your vignettes directly. Better to make sure that you have a good version of pandoc installed before proceeding.

## Check syntax highlighting

A small note: if you're using R Markdown for your vignettes, you'll see that your code chunks get appropriate R syntax highlighting, which makes them easier to read (and prettier). If you have other files, such as README.md, that are raw Markdown, and you have R code blocks in them, you can still get the syntax highlighting if you mark them inside triple backticks with "r" specified like

<pre><code class="hljs">
  ```r
  code()
  ```
</code></pre>

This turns

```
OMG.R_isFun <- function () {
    data.frame(
        snake_case = c("A", "A_B", "ABC", NA),
        stringsAsFactors = TRUE
    ) -> data.frame
    data.frame[, , drop = FALSE]$snake[-1]
}
```

into something like

```r
OMG.R_isFun <- function () {
    data.frame(
        snake_case = c("A", "A_B", "ABC", NA),
        stringsAsFactors = TRUE
    ) -> data.frame
    data.frame[, , drop = FALSE]$snake[-1]
}
```

depending on the highlighting theme/CSS applied.

# 4. Final setup: provide your URL and (optional) logo

When `pkgdown` links to functions in other packages, by default it constructs a link to help pages on rdocumentation.com and vignettes on cran.rstudio.com, knowing that there are hosted versions of the documentation in those places. That itself is quite nice. However, a very slick (and undersold) feature is that `pkgdown` will try to link to *other pkgdown sites* instead. It feels like magic: `pkgdown` [somehow](https://github.com/hadley/pkgdown/blob/ad1b1dfb12871919e06a2aa7e366ec0980af2714/R/metadata.R#L24-L46) knows what other packages have pkgdown sites.

In order for this to work, you have to add your website URL in two places:

1. `_pkgdown.yml`, under the `url:` field, as [documented](http://pkgdown.r-lib.org/reference/build_site.html#yaml-config)
2. the package's `DESCRIPTION` file, under the `URL:` field; if you already have a URL there (say, to your package's GitHub repository), you can add a second URL to your pkgdown site, separated by a comma

Then, if your package is on CRAN, submit a new release so that others who install your package will have the URL field reference. (Be sure to deploy your pkgdown site before submitting---`R CMD check` will warn that the URL does not exist and CRAN will reject your submission.)

After this, whenever anyone (including yourself in another package) builds a pkgdown site that references your package, theirs will include links back to your website.

Finally, `pkgdown` does some fun things with your package's logo, if it has one. (Does an R package truly exist if it doesn't have hexagonal stickers?) For one, it will attempt to turn it into a favicon, the small image displayed next to the page title in your browser's tab. For another (again pending [this PR](https://github.com/hadley/pkgdown/pull/438) or available on [this fork/branch](https://github.com/nealrichardson/pkgdown/tree/dev)), it will use it as a preview image on Twitter/Slack/etc., as in:

{{< figure src="/img/crunch-pkgdown-open-graph.png" class="centered-image">}}

To include a logo, you can drop a `logo.png` file at the top level of your repository (alongside `DESCRIPTION` et al.), and don't forget to add it to your `.Rbuildignore`. `pkgdown` will [also](https://github.com/hadley/pkgdown/blob/ad1b1dfb12871919e06a2aa7e366ec0980af2714/R/build-logo.R#L28) look for a logo in the `man/figures` directory, but unless you have a compelling reason to put it there, I like it at the top level and excluded in `.Rbuildignore` so that the built package stays lighter---your R package users won't benefit from including the .png in the build.

# 5. Publish!

Publishing the pkgdown site is just as easy. By default, `pkgdown` will build your site in the `docs/` directory, and GitHub Pages will let you specify that directory as the source for your site. So the fastest way to publish your site is to build it (`build_site()`), commit, push, and go to your repository settings and turn on GitHub Pages.

There's lots more you can do to customize the display of your site, including adding CSS and integrating it with your personal (non-pkgdown) blog or website. I'll discuss that in a future post.
