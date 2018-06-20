---
title: "Turbocharging 'readr'"
description: "The 'readr' package is a fast reader for text data files, and with a couple of tricks, we can make it even faster by only reading the parts of the files we care about."
date: "2018-04-25T21:49:57-07:00"
categories: ["code"]
tags: ["R", "performance", "benchmark", "readr"]
draft: false
images: ["http://enpiar.com/img/star-wars-light-speed.gif"]
---

<!-- The 'readr' package is fast. Can we make it faster? -->

Recently I analyzed some web traffic for my [team](http://crunch.io/team) at an all-hands meetup. I suspected there were some misconceptions about the frequency of certain kinds of requests, and that some basic descriptive statistics could help.

Whenever I start an analysis like this, I also think about how to make it easier to repeat it. If the question is interesting enough to merit analysis now, it will probably be interesting next month too---particularly if it reveals things we want to improve. In this particular exercise, not only did I get statistics for my presentation, I also got a [new R package](https://github.com/nealrichardson/elbr) and some insights into how to read in data stored in lots of text files in an efficient, fast way, using [`readr`](http://readr.tidyverse.org/).

This post shares some of the insights I learned about `readr` and some tactics for squeezing more performance out of it. The `readr` package is fast, and by incorporating knowledge about the data structure and the questions we want to answer, we can selectively read the parts of the files we care about to make it load even faster.

# The data source: AWS Elastic Load Balancer

Amazon Web Services provides the [Elastic Load Balancing](https://aws.amazon.com/elasticloadbalancing/) (ELB) service for distributing HTTP traffic across multiple servers. Load balancer logs have nice properties for understanding how our platform is performing. While they don't have the richness of detail that web server logs or Google Analytics might provide, they give a complete view of how our API is responding at the service boundary. We once had issues where clients reported timeouts, but the web server logs showed no problems. It turned out that the load balancer was timing out the requests; the server was successfully completing its work, only too late. The load balancer logs, though, confirmed the user reports and gave us evidence for how to resolve the issue.

The ELB log format isn't the same as some [other standard log file schemas](https://github.com/Ironholds/webreadr), but it is [documented](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/access-log-collection.html). Log files are written out in hour-long blocks, one per load balancing server---we usually have two at any given time---in a `.../year/month/day/` directory structure. There are 15 columns, space-delimited, with no header row. A log entry looks something like this:

```
2017-12-31T19:07:25.411521Z hpfv2 23.45.193.110:61424 10.20.0.2:443 0.000043 0.0181 0.000022 200 200 0 801 "GET https://example.com:443/api/ HTTP/1.1" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/47.0.2526.80 Safari/537.36" ECDHE-RSA-AES128-GCM-SHA256 TLSv1.2
```

Here's a simple function using `base::read.delim()` to read an ELB log file:

```r
# To avoid repetition below, define the column names globally here
.elb.column.names <- c(
    "timestamp", "elb", "client_port", "backend_port",
    "request_processing_time", "backend_processing_time",
    "response_processing_time", "elb_status_code",
    "backend_status_code", "received_bytes", "sent_bytes",
    "request", "user_agent", "ssl_cipher", "ssl_protocol"
)
read.elb <- function (file, ...) {
    read.delim(
        file,
        sep=" ",
        header=FALSE,
        stringsAsFactors=FALSE,
        col.names=.elb.column.names,
        ...
    )
}
```

# The `readr` package

{{< figure src="/img/millennium-falcon-awareness.gif" class="floating-left" attr="Yes, it's a fast ship." attrlink="https://meandtenforever1216.tumblr.com/post/126611741283/star-wars-episode-iv-a-new-hope-gifs-and-pics">}}

<!-- gif of millennium falcon is it a fast ship -->

I had heard that `readr` was fast, but I hadn't bothered to check it out before. In the three years since it was first released, I haven't had to read much text data in R, and when I have, I could stand to wait an extra second or two that the base R reader might take.

In this case, though, I was reading in lots of large files that took around 4-6 seconds each to read with `read.elb()`. With 2 files per hour, that's 48 per day, 336 per week. That's about a half hour just to _read_ a week's data, no analysis.

I tried the simplest change to `read.elb()`---replace `.` with `_` and otherwise make the argument names line up---and tested it out. Read time dropped to about a second for the same file (around 82,000 rows), a savings of about three seconds. Three seconds itself isn't a huge win, but compounded over a week of files, it added up to around 20 minutes.

```r
read_elb <- function (file, ...) {
    read_delim(
        file,
        col_names=.elb.column.names,
        delim=" ",
        ...
    )
}

library(microbenchmark)
microbenchmark(read.elb(f), read_elb(f), times=25)

## Unit: milliseconds
##         expr       min        lq     mean   median       uq     max neval
##  read.elb(f) 3181.3075 3697.9343 4164.783 3966.614 4943.768 5419.93    25
##  read_elb(f)  650.4931  853.3361 1198.328 1166.088 1478.491 2325.52    25
```

Being able to read the week's data in 7 minutes is much better, but it's still not interactive speed. It's long enough for me to go do something else while I wait for it to run, and then forget that I was running something on the log server until much later. Plus, 7 minutes still seemed inefficient to load what amounted to only about 15 million rows of data. There had to be some way to optimize further.

## Specify column schema

Since I had to read the `readr` docs to learn how to adapt my old function to use `read_delim()`, I saw that there was a convenient way to specify the types of the data in each column of the file. I'm generally too cynical to trust that any data file I've found is coded in an internally consistent way---to paraphrase Jean-Paul Sartre, hell is other people's data---so I generally let the reader sniff and then correct any miscoding in R. In this case, however, I trusted well enough that the ELB would write consistent data.

In the interest of comparing apples to apples, I opened the man page for `base::read.table()` to look for a similar option, and it turns out that it had been there all along too (albeit inconsistently named). So I wrote two versions of the function specifying column types: `base` and `readr`.

```r
read.elb2 <- function (file, ...) {
    read.delim(
        file,
        sep=" ",
        header=FALSE,
        stringsAsFactors=FALSE,
        col.names=.elb.column.names,
        colClasses=c("POSIXct", "character", "character", "character", "numeric", "numeric", "numeric", "integer", "integer", "integer", "integer", "character", "character", "character", "character"),
        ...
    )
}

read_elb2 <- function (file, ...) {
    read_delim(
        file,
        col_names=.elb.column.names,
        col_types="Tcccdddiiiicccc",
        delim=" ",
        ...
    )
}

microbenchmark(read.elb2(f), read_elb2(f), times=25)

## Unit: milliseconds
##          expr       min        lq     mean   median       uq      max neval
##  read.elb2(f) 5372.7229 5842.2494 6444.295 6404.604 6910.416 8239.447    25
##  read_elb2(f)  647.8243  740.1607 1110.859 1255.674 1332.852 1758.287    25
```

This second version using `readr` was not really any faster than the version without type specification. That suggests that the type guesser it uses when you don't declare column types is pretty efficient.

Interestingly, the base R version was significantly _slower_ than without the type declarations. My guess, though I didn't confirm, is that specifying `"POSIXct"` for the timestamp column made it do more expensive work than it did without the specification (and in fact, the timestamp column is returned as `character` by default, unlike the simple `readr` version that correctly guesses the type).

## Select columns

Specifying the schema didn't really give us much for this file, but while reading the docs about column types, I noticed that there was an option to skip columns. Given that I only cared about a couple of the columns in the dataset, I wondered whether skipping columns would save some time.

The approach I took was to add a `col_names` argument to my `read_elb()` that let you specify a subset of the column names. The default of `col_names` is all of the columns, so that makes it easy to document/show what the options are. This made for some interesting code to validate the column selections and construct the type mask appropriately:

```r
read_elb3 <- function (file, col_names=.elb.column.names, ...) {
    ## Allow specifying only a selection of cols. Fill in "col_types" with "-"
    col_types <- unlist(strsplit("Tcccdddiiiicccc", ""))
    keepcols <- .elb.column.names %in% match.arg(col_names, several.ok=TRUE)
    col_types[!keepcols] <- "-"

    read_delim(
        file,
        col_names=.elb.column.names[keepcols],
        col_types=paste(col_types, collapse=""),
        delim=" ",
        ...
    )
}
```

Once again, it turned out that this was also possible in `base::read.table()` too:

```r
read.elb3 <- function (file, col.names=.elb.column.names, ...) {
    colClasses <- list("POSIXct", "character", "character",
        "character", "numeric", "numeric", "numeric", "integer",
        "integer", "integer", "integer", "character",
        "character", "character", "character")
    keepcols <- .elb.column.names %in% match.arg(col.names, several.ok=TRUE)
    colClasses[!keepcols] <- list(NULL)

    read.delim(
        file,
        sep=" ",
        header=FALSE,
        stringsAsFactors=FALSE,
        col.names=all_cols,
        colClasses=colClasses,
        ...
    )
}
```

I benchmarked selecting just two columns that I generally was interested in analyzing: the "request", containing HTTP method and URL, and "elb_status_code", the HTTP status returned to the user.

```
Unit: milliseconds
                                                      expr       min        lq
 read.elb3(f, col.names = c("elb_status_code", "request")) 2346.7510 2877.7151
 read_elb3(f, col_names = c("elb_status_code", "request"))  415.2961  423.3368
      mean    median        uq       max neval
 3345.7744 3272.2818 3872.3245 4210.4662    25
  540.7898  451.9291  601.3482  968.7295    25
```

For the `readr` version, this change saved 800 milliseconds on the median read time, around a 60 percent improvement. That turns 7 minutes into 2:30 to read a week of data. The base version was also faster but still 2 seconds slower on average than the simplest `readr` version.

My hunch, again unconfirmed, is that we're saving time by not allocating memory for the other columns, some of which, particularly "user_agent", are large character vectors with lots of distinct values.

## Raw row filtering

Two and a half minutes is pretty good, but it still seemed excessive. For analyzing large log files, my first tool usually is `grep` at the command line, often combined with some `find` search for log files and piping that file list to it. For example, if I want to know how many 504 Gateway Timeout responses there were, I can search for this pattern in a directory of log files:

```
find . -name *.log | xargs -n 1 egrep " -1 -1 504 " | wc -l
```

and I get my answer almost instantly. So the machine is capable of scanning across large quantities of text data much faster than R seemed to be able to ingest it.

A lot of the things I was interested in learning from the logs were similar: relatively rare events. It seemed inefficient to read in 15 million records just to find the 200 that matched my query and drop the rest. I wondered if there was a way to use the system tools to filter the data _before it got to R_.

In the `readr::read_delim()` docs, I saw that the `file` argument accepted "literal data":

```r
## file: Either a path to a file, a connection, or literal data
##       (either a single string or a raw vector).
##
##       ...
##
##       Literal data is most useful for examples and tests. It must
##       contain at least one new line to be recognised as data
##       (instead of a path).
```

So I tried shelling out to `egrep`, capturing stdout in R, and passing that as literal data.

```r
read_elb4 <- function (file,
                       col_names=.elb.column.names,
                       line_filter=NULL,
                       ...) {

    col_types <- unlist(strsplit("Tcccdddiiiicccc", ""))
    keepcols <- .elb.column.names %in% match.arg(col_names, several.ok=TRUE)
    col_types[!keepcols] <- "-"

    if (!is.null(line_filter)) {
        ## Shell out to egrep
        ## If there are no matches, egrep returns status 1, which creates
        ## a warning in R (with system2 when stdout=TRUE)
        file <- suppressWarnings(
            system2("egrep", c(shQuote(line_filter), file), stdout=TRUE)
        )
        nmatches <- length(file)
        ## Concatenate
        file <- paste(file, collapse="\n")
        if (nmatches < 2) {
            ## Append a newline so that readr recognizes this as literal data
            file <- paste0(file, "\n")
        }
    }
    read_delim(
        file,
        col_names=.elb.column.names[keepcols],
        col_types=paste(col_types, collapse=""),
        delim=" ",
        ...
    )
}
```

For the base R die-hards, yes, there is a pure base version:

```r
read.elb4 <- function (file,         
                       col.names=.elb.column.names,
                       line_filter=NULL,
                       ...) {

    colClasses <- list("POSIXct", "character", "character",
        "character", "numeric", "numeric", "numeric", "integer",
        "integer", "integer", "integer", "character",
        "character", "character", "character")

    keepcols <- .elb.column.names %in% match.arg(col.names, several.ok=TRUE)
    colClasses[!keepcols] <- list(NULL)

    if (!is.null(line_filter)) {
        file <- textConnection(
            system2("egrep", c(shQuote(line_filter), file), stdout=TRUE)
        )
    }
    read.delim(
        file,
        sep=" ",
        header=FALSE,
        stringsAsFactors=FALSE,
        col.names=all_cols,
        colClasses=colClasses,
        ...
    )
}
```

I was curious whether `textConnection`, which it turned out `readr` wouldn't read from, gave a benefit relative to `paste(..., collapse="\n")`.

To test this, I first benchmarked a search for requests to a specific entity in the system, which contained the hash `"3aefb7"` in the request URL. It appeared in 37 of the 82,000 requests in this file: less than 0.05 percent of requests.

```r
microbenchmark(
    read.elb4(f, line_filter="3aefb7"),
    read_elb4(f, line_filter="3aefb7"),
    times=25
)

## Unit: milliseconds
##                                expr      min       lq      mean   median
##  read.elb4(f, line_filter="3aefb7")  52.7301  57.7062   85.8613  69.7250
##  read_elb4(f, line_filter="3aefb7")  49.5254  57.4182   83.8109  62.1328
##         uq       max neval
##   117.3559  133.6879    25
##    94.4999  356.3631    25
```

Whoa! We're down from over a second to under 70 milliseconds! For relatively rare events, preprocessing the log file by shelling out to `grep` is a big win.

Let's try another benchmark, this time with a less rare line filter: a search for `POST` requests, which made up around 35 percent of this file (28,516 lines).

```r
microbenchmark(
    read.elb4(f, line_filter="POST "),
    read_elb4(f, line_filter="POST "),
    times=25
)

## Unit: milliseconds
##                               expr        min        lq      mean    median
##  read.elb4(f, line_filter="POST ") 27355.1855 32679.313 34085.754 34171.807
##  read_elb4(f, line_filter="POST ")   868.1771  1392.776  1591.701  1674.178
##         uq       max neval
##  36018.332 38215.991    25
##   1776.117  2060.066    25
```

Alright, so that's worse than without the filtering. Clearly, this raw row filtering has its biggest benefits when the number of rows that match your search is small. I didn't attempt to nail down the specific threshold where one is faster than the other, but clearly it's somewhere less than this. In general, it's important to have an intuition about your data.

A few final notes. First, I tried a version using the [`processx`](https://github.com/r-lib/processx) package for managing external processes, but the overhead it added eroded the filtering benefits: the direct `system2` call was notably faster. Second, I suspect that this code isn't platform-independent; I knew `egrep` was on my log server, so that wasn't a concern for me, but if you're looking to use this pattern on Windows, you may need to adapt the code or install a tool.

## Update: _il y a une pipe!_

After this was first published, [Jim Hester pointed out](https://twitter.com/jimhester_/status/993172484458508289) that you could use the `pipe()` connection as an input and that would save having to read the standard output into an R object and `paste()`ing it together.

I made a `read_elb5()` version that replaced the `line_filter` block with this much simpler code:

```r
if (!is.null(line_filter)) {
    cmd <- paste("egrep", shQuote(line_filter), file)
    file <- pipe(cmd)
}
```

I found an even bigger log file (230 MB) than the one I'd been benchmarking with before, and I repeated the `"POST "` search. Even though the file was bigger, there were fewer `POST` requests, only 16 percent. Because there were fewer lines being read in, the `paste(system2(...))` version was a bit faster than in the benchmark with the log file used throughout this post. The `pipe()` version, though, was nearly twice as fast as that:

```r
##  Unit: milliseconds
##                              expr      min       lq     mean   median
## read_elb4(f, line_filter="POST ") 612.9062 623.6136 693.2473 641.7480
## read_elb5(f, line_filter="POST ") 327.5515 343.9537 380.7328 350.2193
##       uq       max neval
## 724.7104 1027.4293    25
## 440.5271  477.9381    25
```

Under 400 milliseconds to read in 16,000 rows of log data, already filtered to the set I want to consider. `pipe()` it is!

One other observation: pipe or no pipe, doing the filtering before the data hits R is much faster than filtering it in R. For comparison, here's benchmarking of calling `grep` to find the `POST` requests in the full `data.frame` (roughly 100,000 rows) read in without any filtering:

```r
##  Unit: seconds
##                       expr      min       lq     mean   median       uq
## grep("^POST ", df$request) 7.991233 9.218501 9.532355 9.644332 9.881512
##      max neval
## 10.83446    25
```

That is, even after spending several seconds to read in all of the data, it's another 9-10 seconds to identify which rows to keep. If you can skip that work, why not?

## Conclusion: Turbocharging readr

{{< figure src="/img/star-wars-light-speed.gif" class="floating-left" attr="Cropdusting this is not." attrlink="https://meandtenforever1216.tumblr.com/post/126611741283/star-wars-episode-iv-a-new-hope-gifs-and-pics">}}

`readr` is fast, and if we know up front a bit about what questions we want to answer with the data, we can select and filter out of the files intelligently and make it even faster.

One postscript: the release notes for R 3.5 contained this entry:

> * Reading from connections in text mode is buffered, significantly
   improving the performance of readLines(), as well as scan() and
   read.table(), at least when specifying colClasses.

These benchmarks were done on R 3.4.2. It would be interesting to replicate these benchmarks on the new R release and see if that core R improvement closes the performance gap between `base` and `readr` functions.

# The 'elbr' package

I've incorporated these insights into the new `elbr` package, available [on GitHub](https://github.com/nealrichardson/elbr) now. The package is more than a single file reader: it provides a `dplyr` interface to the entire directory of log files, allowing you to `select`, `filter`, and aggregate over multiple files as if you were querying a single database. It does so as efficiently as possible, only reading and returning the minimum necessary from each file and binding together the results into a single table. This allows you to work with log data much larger than can fit comfortably into memory, without needing to set up an external database or even thinking much about the file system.

You can specify a date range in which to search, limiting the set of files to be considered at all, and then the other operations are passed down to each file as it is read. When you `select()` columns, those selections are passed to the file reader and only those columns are read in, as we did above; `filter()` operates on each file separately, before any stacking of `data.frames` into a single one. Because it is functionally a bit different, the `egrep` system-call filtering is its own function, `line_filter()`. All of this is specified on the single log "database" object---you don't have to think about how many files there are or mapping over them. So

```r
df <- ELBLog("2018-04-01", "2018-04-07") %>%
    select(request, elb_status_code) %>%
    filter(elb_status_code == 504) %>%
    collect()
```

would give you a `tbl` of all of the requests that timed out during that week, across all 300+ files, and

```r
df <- ELBLog("2018-04-01", "2018-04-07") %>%
    select(request, elb_status_code) %>%
    line_filter(" -1 -1 504 ") %>%
    collect()
```

would exploit knowledge of the raw log file contents to do that reading and filtering at hyperspeed.

Having a large dataset effectively sharded across multiple text files seems like a common enough problem, but I was surprised that in my (admittedly cursory) search for existing R packages, I didn't find one that provided a `dplyr`-like interface over them. (Maybe a friendly reader will point me at the relevant packages.)

Perhaps if you have data like this, you just put it in a proper database and forget about it. But I didn't feel like installing another system dependency and setting up an ETL process just to get a little more insight from these logs, particularly when it seemed like base R (or even some hard-earned `bash` skills) got me most of the way to what I wanted. Plus, I enjoyed the opportunity to explore `readr` and play some more with tidy evaluation.
