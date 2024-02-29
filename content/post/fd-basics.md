---
title: "Seaching For Fun With fd"
date: 2024-02-28T21:15:02Z
lastmod: 2024-02-28T21:15:02Z
draft: false
keywords: []
description: "Exploring the CLI tool fd"
tags: ["cli-tool", "fd"]
categories: ["programming"]
author: ""

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

---
`fd` is a blazing fast (I know, I know 😉) replacement for the in-built `find` command. I've been having a lot of fun incorporating it as part of my workflow so figured out I should at least blog about its basics.

The [official readme](https://github.com/sharkdp/fd?tab=readme-ov-file) details out all the possible flags/options for using the tool but I've reiterated them here for my personal quick reference, to internalise the syntax and with the hope that someone else might find it useful.

# Overview

`fd` usage follows the below syntax (kind-of) with all arguments as optional:
```sh
fd one-or-more-flags 'search-pattern' target-directory
```
`fd` without any arguments recursively lists out all the contents of the given directory except for:
1. All patterns ignored by the `.gitignore` file (assuming it's a `git` repo and `.gitignore` exists)
2. All hidden entries

The above exclusion applies by default to all cases *but* it's pretty simple to get around it by using flags:
1. Want to include hidden files? Use `fd -H`
2. Want to ignore `.gitignore` exclusions? Use  `fd -I`
3. Want to enable both of these together? Just use `fd -u`

# Pattern

`fd pattern` recursively lists all filesystem entries matching the given `pattern`.  By default, the pattern is treated as a regex string and the syntax is well documented in the [regex](https://docs.rs/regex/latest/regex/index.html#syntax) crate docs. Attempting to use our usual glob patterns fails in `fd` with an error. Thankfully, we have the `-g` flag to enable glob search.
```sh
# recursively search for all files containing the word 'controller'
fd -g '*controller*' src/java
```
It's worth that the pattern always comes first so if we want to list out all the entries of a sub-directory, we'll have to use a catch-all `.` pattern and then the target directory:
```sh
fd . sub_dir
```
# Extension

The bread-and-butter of all find-like utilities is finding files with a given extension. This can be achieved using our trusted old `*.md` glob pattern for finding all markdown files. But a more "fd-like" approach would be to use the in-built `-e` flag, like, `fd -e md` and these can of course be chained (so `fd -e md -e rs` is totally valid). Plus I'm pretty sure there are some corner cases out there wherein our glob pattern for file extensions won't work.

# Path Search

In some scenarios, you might want to search across path names instead of file names starting with a given root directory. One such scenario would be a bulk image downloader utility which downloads all images for a given website per day to a certain `output_dir`. Assuming the directory names are of the format `out_yyyymmdd` we can use the path search capabilities of `fd` to find all `jpg` images for a given month and date (let's say 29th of Feb) across multiple years using the below command:
```sh
fd -p -e jpg -e jpeg '0229' output_dir
```
# Attribute Filters

`fd`, just like `find` allows us to filter on file attributes like `type`, `size` & `owner`. For e.g. if we want to list the details all files with size greater than `2kb` owned by `sanjayts` we can use:
```sh
# -tf implies type 'file'; read man page for other options
# --size +2k implies size greater than 2k
# --size -2k implies file size less than 2k
# --size 2k implies file exact file size of 2k
# -o sanjayts implies only files with 'sanjayts' as the owner
# -l provides a detailed listing with file-masks, owners, size etc. just like ls -l
fd -tf --size +2k -o sanjayts -l
```
Another advantage of `fd` over `find` is the support of user-friendly duration strings and dates as opposed to messing around with `-mmin`, `-mtime`, `-newermt` etc. For e.g. if we want to list out all files within the `src` directory modified on the 5th of Jan 2024, we can use:
```sh
fd --newer '2024-1-5' --older '2024-1-6' -l . src
```
# Case Sensitivity

You might have noticed that we haven't given any care to the case sensitivity of patterns till now. This is because `fd` is awesome and uses smart case matching by default. What this means is that matches are case insensitive by default but switches to a case sensitive match when it encounters any upper-case character as part of the pattern. You can of course use the `-i` and `-s` flags for forcing a case insensitive or case sensitive match respectively.

This feature is a positive data-point for user-friendly defaults in modern tools.
# Further reading

This blog post covers only a fraction of what `fd` is capable of. I have omitted the `-x` and `-X` flags which allow us to execute a command for each or all the matching filesystem entities. Plus I haven't covered file deletion since I don't have any use of that feature yet. I would also whole-heartedly recommend reading the detailed man page by invoking `fd --help` since it covers each flag in much more detail.
