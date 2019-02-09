---
title: "Analyzing Logs with AWS Athena"
description: "With Athena, I got some answers out of several years of log data in S3 relatively quickly. I also had an opportunity to learn some new tricks."
date: "2019-02-08T16:49:57-07:00"
categories: ["code"]
tags: ["athena", "serverless", "sql", "logs"]
draft: false
images: ["https://enpiar.com/img/aquarius.gif"]
---

This week, I wanted to analyze some web traffic patterns back over several years. For contemporary analysis, we sync the last couple of months of load balancer (AWS ELB) log files from S3 to a EC2 instance, and I've [written some tools in R](https://enpiar.com/2018/04/25/turbocharging-readr/) to query them using `dplyr` methods. The [`elbr`](https://github.com/nealrichardson/elbr) R package I wrote does some clever things to take advantage of the directory structure for fast filtering by date, and it supports parallelized reading and querying of the files, among other optimizations, so it's reasonably fast. But it assumes that the source files are local.

To get access to the full history of 1.6 billion requests, I'd need a different way to query the data in S3. I'd read some interesting things about AWS's [Athena](https://aws.amazon.com/athena/) query service, including examples of how to use it to read ELB logs in S3, my exact need, and thought I'd give it a try. And rather than check out the fun-looking [R bindings](https://rud.is/b/2018/04/20/painless-odbc-dplyr-connections-to-amazon-athena-and-apache-drill-with-r-odbc/), I decided to do the SQL the old-fashioned way---by googling for help and trying things I found on Stack Overflow.

{{< figure src="/img/respect.gif" class="floating-right halfwidth" attr="source" attrlink="https://gifer.com/en/41AV">}}

I partly chose the old-fashioned way because of a dark secret: I had never actually written a SQL query before. I'm not sure how I've made it this far without having done it. I've been analyzing data for many years using all kinds of tools, and I've scraped, coded, cajoled, and assembled by any means necessary lots of kinds of data in both academia and industry. I've run jobs on big compute clusters, I've written lots of code to manage working with datasets too big to fit into memory, and I've gotten the results I needed. But not with a direct SQL query.

I've worked with plenty of data that ultimately rested in some database, but there's always been some other way to get the data I needed out: an ORM, a web API, or some other interface. R has some nice tools for doing SQL-like queries on large datasets, some that even compile to SQL, so I've always managed to get what I needed without shouting `SELECT` myself.

Here's how I learned by doing and got some quick results.

# Getting started with Athena

Athena is a service that lets you query data in S3 using SQL without having to provision servers and move data around---that is, it is "serverless". It uses a variant of Hive for defining tables and schemas (with [certain restrictions](https://docs.aws.amazon.com/athena/latest/ug/unsupported-ddl.html)) and Presto for querying the data (also with [some limitations](https://docs.aws.amazon.com/athena/latest/ug/other-notable-limitations.html)).

One of Athena's canonical examples is analyzing load balancer logs in S3. The [simple version](https://docs.aws.amazon.com/athena/latest/ug/elasticloadbalancer-classic-logs.html) seemed straightforward enough, and an [AWS blog post](https://aws.amazon.com/blogs/big-data/analyzing-data-in-s3-using-amazon-athena/) took an example even further. Since I wasn't planning on going too far off that happy path, it seemed easy enough for a novice to try out. Plus, since Athena is serverless, the barrier to entry was low. I didn't have to do a bunch of work before I could try it out; I only needed to learn how to construct the right flavor of SQL query.

From the beginning, Athena was quick to learn. When I first opened Athena in the AWS web console, it started me in a tutorial that used sample ELB logs. Rather than use that sample, I swapped in my S3 bucket for the `LOCATION` and followed the tutorial with my own data. The next step in the tutorial was

```sql
SELECT * FROM elb_logs
  LIMIT 10;
```

Ta-da! Data on the screen. It was really quick to get set up and get a "hello world"; all of the fussy details of setting up the table pointing to my S3 bucket and parsing the files I was able to lift from a recipe.

I noticed it was kinda slow for just grabbing 10 rows, and the timestamps suggested that it had just randomly grabbed some log file from the middle of the time window. And when I repeated the query, the rows it showed were different. So Athena didn't see the data as being ordered: good to know.

Reading the [AWS blog post](https://aws.amazon.com/blogs/big-data/analyzing-data-in-s3-using-amazon-athena/), it seemed like partitioning, taking advantage of the directory layout, was the next thing to do. It would be faster and cheaper to query. So I copied the `CREATE TABLE` command that the tutorial had generated, stuck `PARTITIONED BY(year string, month string, day string)`, the line from the blog post, at the end... and got my first error.

Googling around suggested that the syntax that `SHOW CREATE TABLE` returns from the GUI isn't great. Plus, I'd put the `PARTITIONED BY` line in the wrong place. Fixed that, and now we were ready to go. The next task is to specify the partitions. Unfortunately, the automatic "load all partitions" link that Athena provides doesn't work for ELB logs because the file paths aren't `key=value` explicitly, they're just `value` (i.e. to work automatically, the paths would have to look like `.../year=2019/month=02/day=05/` rather than `.../2019/02/05/`), so to get partitions, you have to specify them each individually.

Just to test it out, I added one, cribbing off the `ALTER TABLE ADD PARTITION` example from the blog post. I could now

```sql
SELECT * FROM elb_logs
  WHERE year='2019'
    AND month='02'
    AND day='05'
  LIMIT 10;
```

and get quick results just from that day. Nice.

I now had enough of my bearings that I could see the path to the end. First, I'd figure out how translate my R code that does some segmentation of traffic into SQL and get that working with just this single day partition. Then, I'd add one more day partition and work out how to do the aggregations by segment and date. Once that checked out, I could write some code to produce the `PARTITION` statements for every day of data in the logs going back to late 2015, paste those into the console, and then run the query on the whole time series. That way, I could learn safely and quickly (and cheaply) on a small subset of data, and only once that was solid would I throw all of the data at it.

My traffic segments were defined by pattern matching in the request URL and User-Agent string. This meant I became good friends with `CASE WHEN`, which is natural and reads nicely, and `REGEXP_EXTRACT`, which took a little experimentation to get to do what I wanted.

Here's what I ended up with, with the specific URL regexps redacted:

```sql
SELECT
    CASE
        WHEN REGEXP_EXTRACT(url, '.*/path1/$', 0) != '' THEN 'Segment 1'
        WHEN REGEXP_EXTRACT(url, '.*/path2/.*', 0) != '' THEN 'Segment 2'
        WHEN REGEXP_EXTRACT(user_agent, 'WebKit|Firefox|Trident', 0) != '' THEN 'Other Web App'
        ELSE 'Other'
        END
        as segment,
    CASE
        WHEN backend_processing_time < 0 THEN 120
        ELSE backend_processing_time + request_processing_time + client_response_time
        END
        as time,
    day,
    month,
    year,
    CAST(elb_response_code as INTEGER) as elb_response_code
    FROM elb_logs
    WHERE year='2019'
      AND month='02'
      AND day='05'
    LIMIT 10;
```

{{< figure src="/img/use-it.gif" class="floating-left halfwidth" attr="It's important to ask the right questions" attrlink="https://uproxx.files.wordpress.com/2012/06/40yearoldvirgin-useit.gif?w=650">}}

In hindsight, there's probably a `REGEXP_MATCH` or something that does more naturally what I wanted (turns out it's spelled [`REGEXP_LIKE`](https://prestodb.github.io/docs/0.172/functions/regexp.html) in Presto), but one of my challenges as an inexperienced SQL writer was learning how to search for help. Sorting out all of the flavors of SQL and knowing which ones are valid in this environment took some practice.

There is an additional `CASE WHEN` for the `time` variable, filling in what are effectively missing values for response time when the ELB returns 504 Gateway Timeout. This status is when the load balancer gives up waiting for the server to return, so `backend_processing_time` is unknown. ELB logs code this as `-1`, and if you're aggregating server response time, it's important not to treat that as a literal negative value.

One fun feature: the bits of the file path that defined the partitions, which happen to be year, month, and day, are now effectively part of the table, columns we can select and group by. That felt like getting something for free: no need to extract them from the timestamp column in the data.

## Aggregating

Next was to take that and compute some daily aggregates. This was mostly straightforward. Here's what I wrapped around that initial `SELECT` statement to do the aggregation:

```sql
SELECT
    COUNT() as n,
    SUM(time) as total_time,
    SUM(CASE
        WHEN elb_response_code > 499 THEN 1
        ELSE 0
        END) as n_5xx,
    SUM(CASE
        WHEN elb_response_code = 504 THEN 1
        ELSE 0
        END) as n_504,
    SUM(CASE
        WHEN time < .2 THEN 1
        ELSE 0
        END) as n_under200ms,
    segment, day, month, year
    FROM (SELECT
        ...
        FROM elb_logs
    )
    GROUP BY segment, day, month, year
```

The way I computed the counts of subgroups (for example, requests that returned 504 Gateway Timeout status) with `CASE WHEN` seemed verbose and like there should be an easier way, but that was how the blog post did it, and it worked so I didn't try to polish. One sticky point was the ELB status code, which needed to be cast from string to an integer (and that's `INTEGER` not `INT`) in the inner `SELECT` statement, but that was easy enough to figure out from the error messages and some persistent googling.

Before running against all of the data, I checked that the counts I was getting from Athena matched the results from my R code that runs against local copies of the log files. It looked sensible... except the counts of error responses were too low. Doing some exploration and querying counts grouped by `elb_response_code`, it appeared that all of the rows that corresponded to 504 Gateway Timeouts were all missing. (Once again, the first rule of working with data is _look at your data_!)

{{< figure src="/img/wax.gif" class="floating-left halfwidth" attr="Surprise!" attrlink="https://media3.giphy.com/media/8JKuFdUW7PVcs/giphy.gif">}}

Seeing how `REGEXP_EXTRACT` behaved and knowing that 504 responses look a little different in the ELB logs, my first guess was that the `CREATE TABLE`'s `'input.regex'` was to blame. I compared what I had in my table definition with the blog post and found that the blog post one was more complex. Specifically, the regular expression didn't account for a `-1` response time. Apparently starting with the one from the tutorial was a bad idea---the sample data probably didn't have this request behavior and didn't need the more complex regular expression to parse the fields correctly.

As a side note: as someone who has worked with lots of data sources, including these exact log files, it seemed strange to use a regular expression to parse a delimited text file. ELB logs are space-delimited, albeit with a little funny business where the `request` field contains the request method, URL, and protocol in a single quoted string. Maybe that was the reason for the special parsing, but it struck me as unnatural, like it was ignoring an important feature of the data and adding unnecessary complexity and room for error.

## Running over all the data

With that resolved, I was getting numbers for the three days of data that I had defined partitions for, and these matched the figures I got from my standard R script on the local files, so I was ready to try the whole series.

To do that, I needed to generate `PARTITION` statements for every date for which there was data. When I need something quick and dirty, R is my go-to tool. This was indeed quick and dirty, not code that I find aesthetically pleasing, but it got it done.

```r
# PARTITION (year='2019',month='02',day='05') location 's3://my/elb/logs/2019/02/05'

partition_statement <- function (date) {
    date <- format(date, "%Y/%m/%d")
    parts <- unlist(strsplit(date, "/"))
    part1 <- paste0("(year='", parts[1], "',month='", parts[2], "',day='", parts[3], "')")
    paste0(
        "  PARTITION ", part1,
        " location 's3://my/elb/logs/", date, "'"
    )
}

dates <- seq(as.Date("2015-10-22"), as.Date("2019-02-05"), 1)
cat(sapply(dates, partition_statement), file="athena-partitions.sql")
```

Then, I just needed to paste those contents into the `ALTER TABLE ADD PARTITION` query, and we're ready to run over all the data.

A query for a single day's worth of data took somewhere between two and four seconds to compute. Split the difference, and project over 3+ years of data, and if it scaled linearly, the query would take something on the order of an hour to finish; let's be generous and cut that in half because we had less traffic three years ago so the files are smaller back then. Surely, though, it would do better than 30 minutes---the whole point of the system is that it scales automatically---but how much better?

{{< figure src="/img/aquarius.gif" class="floating-right" attr="Aquarius" attrlink="https://media2.giphy.com/media/OEDhjYlskC3O8/giphy.gif">}}

I hit "submit" on the query, waited to see the progress bar indicating that it hadn't errored and was running, and went to the kitchen to get some coffee. When I came back... it was done! Half a terabyte of data, and results in 105 seconds. And I didn't have to configure a server or do anything beyond learn how to type a query. That was awesome.

Now to hit "download results to CSV" and finish off my analysis in R from the comfort of my laptop.

# Conclusion

Athena is slick. It scales well and is easy to set up: no servers to provision, just straight to the queries. It was the perfect tool for the problem I had, and I can definitely see using it again if I have a data split across lots of files in an S3 bucket. In fact, since my initial experiment with it, I've already come back to query the logs for different things. It's not instant but it's fast enough that I could use it to learn and iterate quickly, which was an important feature for me in this case.

For its part, I found SQL to be certainly effective, though a bit barbaric and retrograde. Maybe it's all the uppercase statements---although as a novice searching around, it wasn't clear what, if anything, was case sensitive and how that varied by SQL variant. Empirically, all lowercase `select * from elb_logs where year='2019' and month='02' limit 10` works just fine, at least in Athena.

And that's the other challenge, as I experienced it: SQL's utility seems to be as the lowest-common denominator language for accessing databases, yet the range of dialects is vast, so as a learner it takes some work to know what is idiomatic for the system you're querying. The experience was something like learning Spanish by looking up words in Google Translate: you get five alternatives back, the ones used in Spain, Mexico, Cuba, Peru, and Argentina, but you don't know which to use. Pick the wrong one, and the listener (database) will giggle because that _totally_ doesn't mean what you think. So while it may be quick to get started and communicate the basics, it takes some time to learn the specifics of a given dialect.  

Nevertheless, SQL is also quite expressive and human readable. Somehow what felt like a bunch of grunting and pointing resulted in a beautiful sonnet, and I got results back quickly that I could understand and work with further. And beyond the results, I build a good foundation of knowledge for next time I use Athena or bare SQL. 
