---
title: "Using Travis-CI to Build Blogdown"
description: "Automate the building of your Rmarkdown static site with Travis-CI."
date: "2017-05-19T21:49:57-07:00"
categories: ["general"]
tags: ["travis-ci", "automation"]
draft: false
---

create repo on github
git init
git remote add origin git@github.com:nealrichardson/nealrichardson.github.io.git
https://travis-ci.org/profile, enable

git checkout -b src
(because it's a user page, not repo page)

.gitignore
.travis.yml
DESCRIPTION
generate key
# gem install travis
travis encrypt --add
(can't do until you've enabled Travis on your repository and have `git remote add origin`)


https://github.com/rstudio/blogdown/issues/26
https://bookdown.org/yihui/bookdown/github.html
https://github.com/curso-r/verao2017/blob/master/.travis.yml
