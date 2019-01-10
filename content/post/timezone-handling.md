+++
date = "2017-09-21T12:00:00+06:00"
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

mem_limit set is per node

ensure that table statistics are enabled (incremental if possible) 

Impala has a big issue with handling timestamp operations. It internally stores and processes timestamp assuming they are UTC based. This causes problems when data pulled from a database located in NY has timestamps '2006-09-30' & '2006-10-31' which is shown in impala as '2006-09-30 04:00:00' & '2006-10-31 05:00:00'. The 04 and 05 difference is because DST was active during 30th of Sep but ends when we are in 31st October. You would think that Impala would be smart enough to take care of the offets when we convert this UTC to 'America/New_York' timezone but not luck. 

The fix which has been suggested in open JIRA threads is to basically store timestamps as strings in the 'yyyy-MM-dd' format which are automatically converted to timestamp when performing time/date based operation on the column. This has the downside of "polluting" the stored parquet file with string based timestamps just to get around the glitch in the view / query layer (impala)!

Interesting links
* https://issues.apache.org/jira/browse/IMPALA-3316
* https://issues.apache.org/jira/browse/IMPALA-1773
* http://boristyukin.com/watch-out-for-timezones-with-sqoop-hive-impala-and-spark-2/
* https://issues.apache.org/jira/browse/IMPALA-5203


# HDFS

hdfs quorum journal manager https://www.cloudera.com/documentation/enterprise/5-10-x/topics/cdh_hag_hdfs_ha_intro.html

Journal mgr is used only in HA setup (a hot standby namenode) for writing edit logs. For a normal setup (active namenode plus secondary passive namenode), the edits are written out locally and merged periodically by the secondary namenode running on the same host (also called checkpointing node)


Sizing namenode heap memory https://www.cloudera.com/documentation/enterprise/5-10-x/topics/admin_nn_memory_config.html


http://www.waitingforcode.com/hdfs/append-and-truncate-in-hdfs/read

https://martin.atlassian.net/wiki/spaces/lestermartin/blog/2014/07/11/26148906/small+files+and+hadoop+s+hdfs+bonus+an+inode+formula

### Cryptography

Asymmetric encryption

* public key and private key
* Rarely used since slow
* Mainly used for key exchange

Life cycle of https request:

1. Client sends hello with list of supported cipher suites
2. Server sends back the most appropriate cipher suite to be used along with public key
3. client generates a random key for that particular session and uses the preferred key-exchange algorithm to send across the session key
4. Server receives the "session key" which is then used going forward for encrypting data used for that particular session.
5. These sort of hybrid approach makes a lot of sense given that symmetric encryption is usually a lot faster than asymmetric encryption

MAC v/s Digital Signature
1. Simply put MAC's can't prove authorship since the secret is shared. For e.g. Alice can claim that Bob sent her a message saying "he owes her $100" and there would be no way to verify it given that the secret is shared.
2. On the other hand, if Bob were to use Digital Signature, Alice can prove that the message was sent by Bob since he is the only way who has access to his private key

Handy Links
1. [Browser and SSL](https://crypto.stackexchange.com/questions/34235/how-does-ssl-work-on-the-browser)
2. [Cipher Suites supported by your browser](https://cc.dcsec.uni-hannover.de/)
3. [Symmetric v/s Asymmetric encryption](https://www.jscape.com/blog/bid/84422/Symmetric-vs-Asymmetric-Encryption)
4. [Why use MAC when Digital Signature can do the same and much more](https://crypto.stackexchange.com/questions/3251/is-mac-better-than-digital-signature)
   

### JVM info (8th jan)

1. jinfo can be used to dynamically alter the JVM flags for a running process (albeit a limited set of flags). More info [here](https://plumbr.io/blog/garbage-collection/turning-on-gc-logging-at-runtime)
2. You can print all possible flags (and the ones enabled by default) using the command `java -XX:+PrintFlagsFinal -version`