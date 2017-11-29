---
title: "Integrating a pkgdown Site with Your Personal Site"
description: "Publishing a website for your R package is simple with 'pkgdown', "
date: "2017-12-10T16:49:57-07:00"
categories: ["code"]
tags: ["R", "packages", "website"]
draft: false
images: []
---


# Basics of customizing pkgdown

custom css in pkgdown/extra.css (and gitignore/rbuildignore it)

use chrome inspector to see what classes are on elements, trial and error to provide alternate styles in your extra.css

can override templates, even make a template package. but my guess is that if your name is not Hadley, you should tread lightly here. and although IANAWD, you can get pretty far with just pure CSS over the existing templates.

# Integrating with your site

## Consistent branding

css (import url, custom stuff to make the class names match (could put that a few places)

```css
@import url("http://enpiar.com/css/font.css");
@import url("http://enpiar.com/css/custom.css");
```

## Add to your site

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
