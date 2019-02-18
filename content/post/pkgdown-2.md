---
title: "Integrating 'pkgdown' with Your Personal Website"
description: "You don't have to be a web developer to have smooth flow and consistent style across your personal website or blog and your R package websites."
date: "2017-12-18T16:49:57-07:00"
categories: ["code"]
tags: ["R", "packages", "website", "hugo", "blogdown", "documentation"]
draft: false
images: ["https://enpiar.com/img/css_is_awesome_mug.jpg"]
---

Publishing a website for your R package is simple with [`pkgdown`](http://pkgdown.r-lib.org/). [As I recently discussed](https://enpiar.com/2017/11/21/getting-down-with-pkgdown/), you can build a basic site and host it on GitHub Pages with just a handful of commands and clicks.

That's a great start, but why stop there? If you have a personal website or blog, it would be nice to have your package page mesh with it---that is, it should fit with your "brand". Your personal site has a style (font, color, layout, etc.), and ideally your pkgdown site should match it. If you have a custom domain name for your website, it would be great to serve your pkgdown site under the same domain, rather than the default `username.github.io/pkgname` GitHub Pages convention. And if you have more than one R package with a website, they should all have a coherent presentation.

All of this is possible with `pkgdown`, and while it requires getting your hands a little dirtier in code, it's not terribly challenging. Below, I'll walk through my experience doing this with [my personal packages](https://enpiar.com/r/) and [at work](https://crunch.io/r/).

A caveat: IANAWD (I am not a web developer). I can't claim that this is the "right" way to do any of this. All I can say is that it worked for me, and the code is minimally invasive.

# The basics of styling pkgdown

`pkgdown` provides several ways for customizing the appearance. First, you can provide the name of a [bootswatch](https://bootswatch.com/) template in your `_pkgdown.yml`, [like this](https://github.com/hadley/pkgdown/blob/081639735104a03c01527f568a99f0ef7351433d/_pkgdown.yml#L3-L5), and that will reskin the site. This is a good place to start.

For finer-grained control, you can put any custom CSS you want in a `pkgdown/extra.css` file, and it will override any other styles on the page.

For even more custom rendering, you could supply alternate HTML templates, as described [here](https://github.com/hadley/pkgdown/blob/081639735104a03c01527f568a99f0ef7351433d/R/build.r#L119-L139). This may be individual template files, or even a template package, which is what the [tidyverse packages](https://github.com/tidyverse/tidytemplate) do. However, the [docs](https://github.com/r-lib/pkgdown/blob/081639735104a03c01527f568a99f0ef7351433d/R/build.r#L137) say "These settings are currently recommended for advanced users only." I interpret this to mean that if your name is not Hadley, beware. From my experience, you can get pretty far with just adding CSS over the existing templates, so for most, it's probably not worth wading into the HTML templates.

# Integrating with your site

The general approach I took with my sites was to (1) add custom CSS to the pkgdown site to make it appear like my personal site, and then (2) copy the built pkgdown site into the source for the personal site, so that when it is deployed, the pkgdown site goes with it, hosted on my domain.

## Consistent branding

To get the website styles to match, you need to bring in the CSS from the personal site to the pkgdown site, and then do some fine tuning to make sure that the styles you want match the elements on the page.

The first part is straightforward: CSS has an [`@import`](https://developer.mozilla.org/en-US/docs/Web/CSS/@import) statement that lets you dynamically load a CSS file from within another. No copy-and-paste needed---just the URL to point at. For my personal site, I saw that its `index.html` loaded several `.css` files, and two were particularly relevant: one had the font declarations, and the other all of the custom styles I had added on top of the base theme (the equivalent to `pkgdown/extra.css` for Hugo). I added imports for those two files into my `extra.css`:

```css
@import url("https://enpiar.com/css/font.css");
@import url("https://enpiar.com/css/custom.css");
```

That got the basics. To work out the rest of the styling, I had to do some exploration of the pkgdown site markup, figure out how to identify elements on the page there, and then add special CSS classes to `pkgdown/extra.css` (after the `@import` statements). These styles overrode the defaults with styles that aligned with my personal site. In a few cases (notably, the navbar), it was easier---and more visually appealing---to edit the style of my personal site to make it align more closely with the pkgdown site.

This styling involved a bit of spelunking and iteration. I like to open up the dev console in the browser, interactively tweak the CSS on the page to find the right appearance, then copy the resulting styles into `extra.css`. Here's some references on how to use the browser dev tools in [Chrome](https://developers.google.com/web/tools/chrome-devtools/inspect-styles/) and [Firefox](https://developer.mozilla.org/en-US/docs/Tools/Page_Inspector/How_to/Examine_and_edit_CSS).

{{< figure src="/img/css_is_awesome_mug.jpg" class="floating-left halfwidth" attr="Truth" attrlink="https://www.zazzle.com/css_is_awesome_mug-168113658720373401">}}

As for how to use CSS properly, Google around. I'm not the one to teach good CSS skills: remember, IANAWD. At work earlier this year, I had this exchange while trying to style a website template:

> &lt;npr&gt; now it’s just getting worse

> &lt;mike&gt; welcome to css!

That said, I didn't have to do any advanced CSS to make the sites line up. In fact, [I didn't have to add much CSS at all](https://github.com/nealrichardson/httptest/blob/39da46541306d79f14fa9bd4d6401b932f8bd7e2/pkgdown/extra.css), and most of what is there is basic font size/weight definitions, like this:

<p style="clear: left;"></p>

```css
body {
    margin-top: 16px;
    margin-bottom: 16px;
    font-size: 19px;
    font-weight: 200;
}
```

The trickiest parts were finding the right selector to specify to override some [Bootstrap](https://getbootstrap.com/) theme defaults, but that could usually just be copied straight from the browser inspector.

{{< figure src="/img/browser-inspector.png" class="centered-image bordered-image">}}

Speaking of Bootstrap: I suspect that if my personal site had been using a [Bootstrap theme](https://themes.gohugo.io/tags/bootstrap/), the CSS imports alone may have been enough to style the pkgdown site, and I wouldn't have had to mess with the inspector and custom CSS. Because `pkgdown` uses Bootstrap, if my site also used Bootstrap, the CSS class names would have been consistent between the two sites, and the right styles should have been applied naturally.

## Site header

[In my previous post](https://enpiar.com/2017/11/21/getting-down-with-pkgdown/), I discussed how you can add links to anything you want to your pkgdown site navbar by editing the `_pkgdown.yml`. You could add links to parts of your personal site, even arrange it to mirror how your personal site organizes header links.

I personally didn't do much to harmonize the contents of the navbar across the personal and pkgdown sites. I wanted to keep the standard pkgdown navbar arrangement (Get Started, Reference, Articles, News) and didn't want that on the personal site, so I just worked to make the style consistent even if the links themselves weren't.

The one thing I did do add was a link back to the personal site, using the "home" icon and placing it at top right, next to the usual GitHub link:

```yaml
right:
- icon: fa-github fa-lg
  href: https://github.com/nealrichardson/httptest
- icon: fa-home fa-lg
  href: https://enpiar.com
```

## Add to your personal site

Both of the websites with which I've integrated pkgdown sites are built by [Hugo](http://gohugo.io/): one via [blogdown](https://bookdown.org/yihui/blogdown/) and one not, so the process I'll describe is oriented to how Hugo works. I can't speak to how you'd do this with Jekyll or other static-site generators, but I would imagine that something similar would work there.

Hugo uses a [folder named "static"](https://gohugo.io/content-management/static-files/#readout) for containing extra CSS, JavaScript, and image files. A file put in `static/css/custom.css` in the source code, for example, will be included at `css/custom.css` in the built site. _We can exploit this feature and copy our built pkgdown site wholesale into the `static` directory_, so when our Hugo site is built and deployed ([automated](https://enpiar.com/2017/06/01/building-a-blogdown-site-with-travis-ci/), [of course](https://crunch.io/dev/blog/building-the-blog-on-travis/)), our pkgdown site is too.

Assume you have your Hugo website repository checked out in a directory called `mysite`, alongside your R package repository, and you want to host your pkgdown site at `yourdomain.com/r/mypkg`. This would build the site and insert it in the right location in the file system:

```bash
R -e 'pkgdown::build_site()'
rm -rf ../mysite/static/r/mypkg
cp -r docs/. ../mysite/static/r/mypkg
```

That is, you put the built pkgdown site in your Hugo project's `static/r/mypkg` directory, so that when Hugo builds your personal site, the pkgdown pages are placed at `/r/mypkg`.

You should `rm` the old version of the site before you `cp` in the new, just so you get a clean build every time (though of course you wouldn't need to `rm` the first time).

If you don't want to shell out to do this, you can, of course, accomplish the building and moving from within R:

```r
path <- "../mysite/static/r/mypkg"
unlink(path, recursive=TRUE)
pkgdown::build_site(path=path)
```

# Handling multiple packages

I have several R packages and I wanted them all to have consistent style. At first, I thought that that's a problem that not many people have and thus not worth discussing here: most R users don't write packages at all, let alone multiple packages. But, looking at the [CRAN package database](https://www.rdocumentation.org/link/CRAN_package_db?package=tools), at the time of writing there are over 2,000 package maintainers with more than one package:

```r
cran <- tools::CRAN_package_db()
m <- table(cran$Maintainer)
table(m > 1)

## FALSE  TRUE
##  5151  2008
```

If you were to include packages that are hosted elsewhere (Bioconductor, GitHub, etc.), that number would be higher. So, for those thousands of you out there with multiple packages to style and incorporate into your website, here's how I did it.

## CSS

The CSS part is trivial for every package after your first one: `@import url` from your first site's `extra.css`. You've worked out all of the imports, classes, and styles there, so bring it over to your new pkgdown site.

For example, the second pkgdown site I did was for [httpcache](https://enpiar.com/r/httpcache), and its `pkgdown/extra.css` is just

```css
@import url("https://enpiar.com/r/httptest/extra.css");
```

that is, it imports the `extra.css` file from `httptest`, the first one I built. Again, IANAWD, but this approach was uncomplicated for me---and dead simple.

## Site directory

As you get more code and packages on your website, it's good to provide links to them from the rest of the site. You can achieve this by customizing the navbar in all of your pkgdown sites and your main website, but it may also be nice to have index-like page listing your packages. In my case, because I had placed my pkgdown sites in an `r` directory (https://enpiar.com/r/httptest, https://enpiar.com/r/httpcache), the URL structure implied that https://enpiar.com/r/ would exist and that it might have some sort of listing or overview.

We can again exploit a feature of Hugo to create this index page. Hugo's [content management](http://gohugo.io/content-management/organization/) works by organizing pages within "sections", i.e. directories. Each section can have an `_index.md` file that provides content for the index page of the section, regardless of how many other pages are in the section.

So, I [created a `content/r/_index.md` page](https://raw.githubusercontent.com/nealrichardson/nealrichardson.github.io/a4d4b59caf1908df980413bd317bd7522efc4bbf/content/r/_index.md) with some Markdown content, containing links to the pkgdown sites and some other work. It doesn't matter that `/r/httptest` doesn't exist in the Hugo section (instead, it's in `static/`)---the links work.

In order for the Markdown content in `_index.md` to render, you need to make sure that the default `list.html` template---[the one that index pages use](http://gohugo.io/templates/lists/)---in your Hugo theme includes `{{.Content}}`, which is the Markdown body. Many themes don't: the standard use of an index page is to list the pages within the section, not show content. My theme was one of those that did not.

In Hugo, it is easy to provide custom templates by placing an alternate file in your `layouts` directory, which [takes precedence](http://gohugo.io/templates/section-templates/) over the theme and defaults---similar to how the CSS you provide overrides the defaults. In this case, to define a custom index page for `/r/`, put a new template in `layouts/r/list.html`. The easiest way to start is to copy the default from your theme (found at the path `themes/your-theme-name/layouts/_default/list.html`) and then tweak it. Delete the list of pages from the template and add in a `<p>{{.Content}}</p>` or something, adjusting as desired.

[Here's my `layouts/r/list.html`](https://raw.githubusercontent.com/nealrichardson/nealrichardson.github.io/a4d4b59caf1908df980413bd317bd7522efc4bbf/layouts/r/list.html#). I added one other fun element to it: a list of recent blog posts about R. This template block does that:

```html
<h3>Recent R blog posts:</h3>
<ul>
{{ range first 5 (where .Site.Pages ".Params.tags" "intersect" .Params.feed_tags) }}
    <li><a href="{{ .Permalink }}">{{ .Title | emojify }}</a>
{{ end }}
</ul>
```

In addition to not being a web developer, I am also not a Hugo expert, so I can't justify all of the specifics of why certain things are quoted or in parentheses; this was the fruit of trial and much error (and googling). Basically, it looks at `.Site.Pages`---that is, all pages and not those from the current "section", which is empty as far as Hugo knows---filters them based on their "tags", and displays links to the first five of them. The tag filtering compares the `tags: []` list from the front matter of each blog post with the `feed_tags` front matter field I added to `content/r/_index.md`:

```yaml
feed_tags = ["r", "R"]
```

For some reason (my Hugo ignorance, no doubt), I didn't figure out how to just inline that `["r", "R"]` into the `where` statement, but it worked when reading from the index page front matter.

The result looks like [this](https://enpiar.com/r/):

{{< figure src="/img/enpiar-r-index.png" class="centered-image bordered-image">}}

We have a page on the personal site with links to all of the pkgdown sites, plus a dynamically generated list of recent R blog posts from the personal site, seamlessly appearing together. Magic!

# Want more?

With these minor extensions, I've made it so every pkgdown site I make has an appearance that is consistent with this blog's style, and I've plugged them into my personal site. I've stopped there, but if you're a better web designer (or have more time to play around with CSS), there's certainly more that you can do to enhance your pkgdown site and integrate it with your other websites.
