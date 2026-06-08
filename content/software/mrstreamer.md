---
title: "MRStreamer"
date: 2026-06-08
draft: false
---

## Overview

MRStreamer is a MapReduce framework which provides a basic Apache
Hadoop-compatible API and advanced features:

- online and iterative processing via ability to output and use in
  mappers preliminary (reducer) results,
- efficient processing on shared-memory systems with capability to run
  in distributed environments without code changes.

MRStreamer is developed for GNU/Linux, but should run on MacOS X and
other UNIX-like operating systems. MRStreamer should run on Microsoft
Windows if you install [http://www.cygwin.com/](Cygwin).


## Documentation

There are several pages that describe certain parts of MRStreamer, which
can be found here:

1.  [Tutorial](tutorial/)
2.  [Streaming Capability](streaming-/capability/)
3.  [K-Means Example](k-means-/example/)

## Download

[Download](download/) the latest version of the MRStreamer.

## Installation and basic usage

Unpack the downloaded file on all the machines where you want to deploy
your MapReduce program.

When implementing your MapReduce program you can use either of the
Hadoop APIs: the old org.apache.hadoop.mapred or the new
org.apache.hadoop.mapreduce. However, you need to consider the
following:

- instead of the Hadoop's JobClient use MRSJobClient (part of
  MRStreamer)
- instead of Hadoop's Job use MRSJob (part of MRStreamer).

In general, replacing JobClient and Job by MRStreamer's equivalents are
the only changes necessary to port Hadoop code to MRStreamer (if new
features are not used). Examples can be found in the "examples"
subdirectory.

The machines need to have a shared filesystem such as NFS, since
MRStreamer does not implement HDFS. Run ./server on one node and
./worker on all the other nodes. To submit a MapReduce program run
./jobsubmitter providing a jar file containing your program.
