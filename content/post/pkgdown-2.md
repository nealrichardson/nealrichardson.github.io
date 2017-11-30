---
title: "Integrating a pkgdown Site with Your Personal Site"
description: ""
date: "2017-12-10T16:49:57-07:00"
categories: ["code"]
tags: ["R", "packages", "website"]
draft: true
images: []
---

Publishing a website for your R package is simple with 'pkgdown'. [As I recently discussed](http://enpiar.com/2017/11/21/getting-down-with-pkgdown/), you can build a basic site and host it on GitHub Pages with just a handful of commands and clicks.

That's a great start, but why stop there? If you have a personal website or blog, it would be nice to have your package page mesh with it---that is, it should fit with your "brand". Your personal site has a style (font, color, layout, etc.), and ideally your pkgdown site should match it. If you have a custom domain name for your website, it would be great to serve your pkgdown site under the same domain, rather than the default `username.github.io/pkgname` GitHub Pages convention. And if you have more than one R package with a website, they should all have a coherent presentation.

All of this is possible with `pkgdown`, and while it requires getting your hands a little dirtier in code, it's not terribly challenging. Below, I'll walk through my experience doing this with [my personal packages](http://enpiar.com/r/) and [at work](http://crunch.io/r/). The general approach is that we will (1) add custom CSS to the pkgdown site to make it appear like our personal site, and then (2) copy the built pkgdown site into the source for our personal site.

A caveat: IANAWD (I am not a web developer). I can't claim that this is the "right" way to do any of this. All I can say is that it worked for me, and the code is minimally invasive.

# Basics of customizing pkgdown

`pkgdown` provides several ways for customizing the appearance of the built site. First, you can provide the name of a [bootswatch](https://bootswatch.com/) template in your `_pkgdown.yml`, [like this](https://github.com/hadley/pkgdown/blob/081639735104a03c01527f568a99f0ef7351433d/_pkgdown.yml#L3-L5) and that will reskin the site. This is a good place to start.

{{< figure src="/img/css_is_awesome_mug.jpg" class="floating-right halfwidth" attr="Truth" attrlink="https://www.zazzle.com/css_is_awesome_mug-168113658720373401">}}

For finer-grained control, you can put any custom CSS you want in a `pkgdown/extra.css` file, and it will override any other styles on the page (provided that you have an appropriately specific selector---more below). To explore and interactively tweak the CSS on the page, I like to open up the dev console in the browser and use trial and error to find the right appearance, then copy into `extra.css`. Here's some references on how to use the browser dev tools in [Chrome](https://developers.google.com/web/tools/chrome-devtools/inspect-styles/) and [Firefox](https://developer.mozilla.org/en-US/docs/Tools/Page_Inspector/How_to/Examine_and_edit_CSS).

For even more custom rendering, you could supply alternate HTML templates, as described [here](https://github.com/hadley/pkgdown/blob/081639735104a03c01527f568a99f0ef7351433d/R/build.r#L119-L139). This may be individual template files, or even a template package, which is what the [tidyverse packages](https://github.com/tidyverse/tidytemplate) do. However, the docs say "These settings are currently recommended for advanced users only," which I interpret to mean that if your name is not Hadley, beware.

From my experience, you can get pretty far with just adding CSS over the existing templates, so it's probably not worth wading into the HTML templates.

# Integrating with your site



## Consistent branding

css (import url, custom stuff to make the class names match (could put that a few places)

```css
@import url("http://enpiar.com/css/font.css");
@import url("http://enpiar.com/css/custom.css");
```

specific selectors

## Site header

can add to your pkgdown header common nav

(http://enpiar.com/2017/11/21/getting-down-with-pkgdown/)

## Add to your site

Both of the websites I've integrated pkgdown sites with are built by [Hugo](http://gohugo.io/): one via [blogdown](https://bookdown.org/yihui/blogdown/) and one not, so the process I'll describe is oriented to how Hugo works. I can't speak to how you'd do this with Jekyll or other static-site generators, but I would imagine that something similar would work there.

Hugo uses a [folder named "static"](https://gohugo.io/content-management/static-files/#readout) for containing extra CSS, JavaScript, and image files. A file put in e.g. `static/css/custom.css` in the source code will be included at `css/custom.css` in the built site. We can exploit this feature and copy our built pkgdown site wholesale into the `static` directory, so when our Hugo site is built ([automated](http://enpiar.com/2017/06/01/building-a-blogdown-site-with-travis-ci/), [of course](http://crunch.io/dev/blog/building-the-blog-on-travis/)).

```bash
R -e 'pkgdown::build_site()'
rm -rf ../mysite/static/r/mypkg
cp -r docs/. ../mysite/static/r/mypkg
```

build and put in static/ dir; add link or page that links to it in the hugo site

# Handling multiple packages

```r
cran <- tools::CRAN_package_db()
m <- table(cran$Maintainer)
table(m > 1)

## FALSE  TRUE
##  5151  2008

head(sort(m, decreasing=TRUE))

## Scott Chamberlain <myrmecocystus@gmail.com>
##                                          65
##          Dirk Eddelbuettel <edd@debian.org>
##                                          52
##           Jeroen Ooms <jeroen@berkeley.edu>
##                                          33
##         Hadley Wickham <hadley@rstudio.com>
##                                          32
##    Thomas J. Leeper <thosjleeper@gmail.com>
##                                          31
##       Gábor Csárdi <csardi.gabor@gmail.com>
##                                          30
```

## CSS

The CSS part is trivial for every package after your first one: import url from your first site.

Again, IANAWD, and this approach

## Site directory

# Using Travis-CI

(make its own blog post?)
