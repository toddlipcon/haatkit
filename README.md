haatkit
=======

haatkit is a toolkit of simple scripts that are useful when working with a
Hadoop cluster, or probably any system primarily made up of a lot of different
Java processes.

It was inspired by (but shares no code with) the maatkit suite of tools for
working with MySQL.

To be included in haatkit, a tool should be:
- no more than a few hundred lines of python or bash
- require no dependencies that aren't installed on a stock RHEL5 system
- follow the "Unix philosophy":
-- easy to connect with other tools via pipes
-- return reasonable exit codes on failure
-- use command line flags rather than configuration


Tools currently included
=========================


### hk-colorize

Colorizes certain regular expressions inside a log. For example:

    $ hk-colorize "{r'WARN|FATAL|Exception': 'red', r'blk_\d+': 'yellow,bold'}"

If you would like to page through the colorized output, 'less -R' is recommended.

### hk-jgrep, hk-jkill

Equivalents of pgrep/pkill for Java processes. For example:

    $ hk-jkill -9 HRegionServer

or:

    $ cat /proc/$(hk-jgrep NameNode)/limits

### hk-jstack

The JDK jstack utility can be very finicky. In particular, if the executing
user is not the same as the user who ran the JVM, or if the version of jstack
comes from a slightly different version of JVM than is running the process,
it will return incorrect results, and encourage the user to run 'jstack -F'.
This rarely produces a useful stack trace.

hk-jstack determines the correct version of jstack and uses sudo to become
the correct user to acquire the dump.

Usage:

    $ hk-jstack $(hk-jgrep NameNode)

### hk-interesting-threads

Filters a jstack of a Hadoop daemon to cut out "uninteresting" threads.
Uninteresting threads are things like threads in threadpools that are currently
just waiting for something to do, or IPC handlers that have no call to
process.

Usage:

    $ hk-jstack $(hk-jgrep NameNode) | hk-interesting-threads

### hk-lock-contention

Takes in a jstack and outputs a summary of which threads are waiting on which
locks, and where those locks are held.

Usage:

    $ hk-jstack $(hk-jgrep HRegionServer) | hk-lock-contention
    1 waiting on 0x000000060bdbe6f0 (java.util.HashMap)
      held at org.apache.hadoop.hbase.regionserver.HRegionServer.metrics(HRegionServer.java:1164)
