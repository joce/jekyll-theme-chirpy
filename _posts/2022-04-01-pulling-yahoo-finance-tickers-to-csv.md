---
layout: post
title: Pulling Yahoo! Finance data to CSV
date: 2022-04-01 16:26 -0400
description: How to fetch market data from Yahoo! Finance in CSV form, from the command line.
image:
  path: /assets/img/posts/pulling-yahoo-finance-tickers-to-csv/header/lo-lo-CeVj8lPBJSc-unsplash.jpg
  alt: "Photo by lo lo on Unsplash"
category:  How-to
tags:
- howto
- dev
- command line
- stock market
- script
---

In recent months, I've started hacking a little on [mop](https://github.com/mop-tracker/mop), a command-line utility that displays stock market data pulled from Yahoo! Finance. It's written in [Go](https://go.dev), a language I had never used before, and it's been fun to learn it while playing around with a tool I use on a day-to-day basis to keep me up-to-date with the values of some stock I own.

While getting more familiar with mop, I was looking at opened issues on GitHub, I stumbled upon this one, asking that [mop could be used as a tool to pull market data on a punctual basis](https://github.com/mop-tracker/mop/issues/88). This would make mop, an interactive tool, behave in a non-interactive way.

I was initially not opposed (as you can see), and I thought there were many features that could be implemented to get the data in various interesting ways. But thinking about it some more and realized it would probably be a mistake.

What follows is a somewhat more elaborate take of the answer I finally served to this issue.

So as I just stated, mop itself is an interactive tool, that serves a given purpose. Repurposing it would change its nature, and what it would have become would have been incompatible with the [UNIX Philosophy](https://en.wikipedia.org/wiki/Unix_philosophy):

* Write programs that do one thing and do it well.
* Write programs to work together.
* Write programs to handle text streams, because that is a universal interface.

So I started digging, and I found that using a set of already existing command-line tools, available on a variety of systems, could accomplish all that the person who opened the issue in the first place wanted, and then some. These tools are `curl` (available out of the box on pretty much all systems), `jq` and `mlr` (aka Miller). The latter two can be installed using your favorite package manager (e.g. `apt` on Ubuntu/Debian, [Chocolatey](https://chocolatey.org) on Windows, etc).

So, first and foremost, we're going to use `curl` to pull the data from Yahoo!. Simply use the URL below, and provide a comma-separated list of the symbols you're interested in (no spaces in between the symbols). Let's pull the data for Apple, Alphabet and Ford (`AAPL`, `GOOG` and `F`):

```console
$ curl https://query1.finance.yahoo.com/v7/finance/quote?symbols=AAPL,GOOG,F 2>/dev/null
```

> Note that I'm redirecting the error stream to null so only the raw data that interests us is spewed to the console. In this post, I'll be using the Linux way to redirect the error stream to null in my examples: `2>/dev/null`. If you're on Window, use `2>nul` instead.
{: .prompt-info }

I will not be posting the whole resulting data here, but let's just say that we're served with a nice, dense wall of text.

Next up, we'll be pumping this wall of text inside [`jq`, jq, a command line JSON processor](https://stedolan.github.io/jq/). The syntax can be a bit terse and overwhelming, but it's amazingly versatile, and it allows you to manipulate JSON data in anyway you want.

I will not get into all the wonderful things `jq` can do here, but I invite you to [check the manual](https://stedolan.github.io/jq/manual/) if you want to know more.

For our purposes, I want not to have to deal with a JSON stream, but rather with a CSV one. Digging around, I found this [enlightening Medium post that explains quite well how to transform the incoming JSON stream into a CSV one](https://medium.com/free-code-camp/how-to-transform-json-to-csv-using-jq-in-the-command-line-4fa7939558bf). I will skip all the explanations and skip right the the interesting bit, the command-line invocation:

```console
$ jq -r '(map(keys) | add | unique) as $cols | map(. as $row | $cols | map($row[.])) as $rows | $cols, $rows[] | @csv'
```

Well. This is all well and dandy, but this work when you have an array as your topmost JSON element, which is not the case for us. I've tried a bit to find a way to get to the interesting parts from within this command, but I have to admit that the easiest way that I found was to do another invocation of `jq` before to get the array we're after as the top most element:

```console
$ jq -r .quoteResponse.result | jq -r '(map(keys) | add | unique) as $cols | map(. as $row | $cols | map($row[.])) as $rows | $cols, $rows[] | @csv'
```

Now I hear you: "Joce. This is interesting, but how am I supposed to remember this?" I guess you're not. A way to alleviate this issue would be to bring all this together in a shell script:

```bash
#!/bin/bash

if [[ $# -eq 0 ]] ; then
    >&2 echo 'You need to pass in one or more ticker symbols to get their quotes.'
    exit 0
fi

yhooFinArgs=""

for i in $@
do
    if [ "$yhooFinArgs" = "" ]; then
        yhooFinArgs=$i
    else
        yhooFinArgs=$yhooFinArgs","$i
    fi
done

curl https://query1.finance.yahoo.com/v7/finance/quote?symbols=$yhooFinArgs 2>/dev/null | \
jq -r .quoteResponse.result | \
jq -r '(map(keys) | add | unique) as $cols | map(. as $row | $cols | map($row[.])) as $rows | $cols, $rows[] | @csv'

```
{:file="yqt.sh"}

A similar script for Windows would be this:

```batch
@ECHO OFF

IF "%~1"=="" (
    ECHO You need to pass in one or more ticker symbols to get their quotes. 1>&2
    GOTO :eof
)

SET YHOO_FIN_ARGS=

FOR %%A IN (%*) DO call :MakeList %%A
goto YahooQuery

:MakeList
IF DEFINED YHOO_FIN_ARGS (
    SET YHOO_FIN_ARGS=%YHOO_FIN_ARGS%,%1
) ELSE (
    SET YHOO_FIN_ARGS=%1
)
goto :eof

:YahooQuery

curl https://query1.finance.yahoo.com/v7/finance/quote?symbols=%YHOO_FIN_ARGS% 2>nul | jq -r .quoteResponse.result | jq -r "(map(keys) | add | unique) as $cols | map(. as $row | $cols | map($row[.])) as $rows | $cols, $rows[] | @csv"

SET YHOO_FIN_ARGS=
```
{:file="yqt.cmd"}

These very simple scripts will return the data pulled from the list of items you pass in as parameters as a CSV stream.

```console
$ ./yqt.sh AAPL GOOG F > myquotes.csv
```

So. Now you can generate a CSV file, with all these columns. Maybe you want to bring that into a spreadsheet application and do some fancy stuff with this data, but maybe you're just interested in a subset of the data, and you don't care about most of the columns. Can we filter this? Can we do better? But of course we can! This is where [Miller](https://github.com/johnkerl/miller) comes into play. As `jq`, Miller is super versatile, and as for `jq`, it has a [very comprehensive documentation](https://miller.readthedocs.io/en/latest/).

For example, say you just want the symbol, current price and market cap for the companies you've just pulled the data for, you'd do:

```console
$ mlr --csv cut -o -f symbol,regularMarketPrice,marketCap myquotes.csv
symbol,regularMarketPrice,marketCap
AAPL,176.465,2879802834944
GOOG,2825.56,1862784385024
F,17.04,68232425472
```

Or say you want it pretty printed on screen, with boxes and with the market cap expressed as millions of dollars, you could do:

```console
$ ./yqt.sh AAPL GOOG F | mlr --icsv --opprint --barred --right cut -o -f symbol,regularMarketPrice,marketCap then put '$marketCap=fmtnum($marketCap/1000000,"%.2fM")'
+--------+--------------------+-------------+
| symbol | regularMarketPrice |   marketCap |
+--------+--------------------+-------------+
|   AAPL |             176.46 | 2879802.83M |
|   GOOG |            2825.56 | 1862784.38M |
|      F |              17.04 |   68232.42M |
+--------+--------------------+-------------+
```

As you can see, Miller allows you to play around with your CSV in ways similar to which `jq` allows you to play around with JSON.

So, there you have it. I think with the right tools in hand, I think one can go much further that anything we could have come up with in `mop`. The UNIX Philosophy wins another round!
