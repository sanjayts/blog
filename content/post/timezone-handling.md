+++
date = "2017-09-22T12:00:00+06:00"
title = "Date & Time Gotchas"
categories = ["programming"]
tags = ["scala", "java", "date and time"]
draft = true
+++
----------------------------------

I personally believe that using a [terminal multiplexer](https://en.wikipedia.org/wiki/Terminal_multiplexer) is a must for anyone who spends a non-trivial amount working on remote *nix servers. This post is a gentle introduction to [GNU screen](https://en.wikipedia.org/wiki/GNU_Screen) but I would like to take a short detour and explain why they are useful.

### New York Time

I was using EST timezone for getting new york time; worked fine when UK was in daylight saving mode. After that started giving issues, turns out taht EST is not the correct timezone to use, it should be US/Eastern! 

also mention about how spark doesn't validate forcing schema on a dataframe of completely different type

- avro schema requires logical decimal type to be mentioend in a specific way
- spark says can't derive parquet schema either when the file is corrrupt or when the dir you are pointing to is empty
- if you get lease expired for data paths for spark in logs, check if there is a concurrent process writing data to that path

# Spark

Actions result in "jobs"
Jobs are composed of "stages" in case there are one or more wide transformations required as part of a job, we get stages
Each stage in turn is composed of one or more "tasks" which are independently executed by the executor
Spark scheduler builds a DAG of stages for each job

The default spark scheduler is a FIFO scheduler which executes tasks for jobs in order. Another variant is the fair policy scheduler wherein tasks from multiple jobs are executed incrementally as opposed to all tasks for the first job.

# Impala

Impala has a big issue with handling timestamp operations. It internally stores and processes timestamp assuming they are UTC based. This causes problems when data pulled from a database located in NY has timestamps '2006-09-30' & '2006-10-31' which is shown in impala as '2006-09-30 04:00:00' & '2006-10-31 05:00:00'. The 04 and 05 difference is because DST was active during 30th of Sep but ends when we are in 31st October. You would think that Impala would be smart enough to take care of the offets when we convert this UTC to 'America/New_York' timezone but not luck. 

The fix which has been suggested in open JIRA threads is to basically store timestamps as strings in the 'yyyy-MM-dd' format which are automatically converted to timestamp when performing time/date based operation on the column. This has the downside of "polluting" the stored parquet file with string based timestamps just to get around the glitch in the view / query layer (impala)!

Interesting links
* https://issues.apache.org/jira/browse/IMPALA-3316
* https://issues.apache.org/jira/browse/IMPALA-1773
* http://boristyukin.com/watch-out-for-timezones-with-sqoop-hive-impala-and-spark-2/