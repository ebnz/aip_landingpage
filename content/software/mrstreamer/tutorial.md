---
title: "Tutorial"
date: 2026-06-08
draft: false
---

## Tutorial

This tutorial assumes that you are familiar with the basic concepts of
the MapReduce programming paradigm and the Java programming language. If
not, read the original
[Hadoop Tutorial](http://hadoop.apache.org/docs/r1.0.4//mapred_tutorial/) **first**.

## WordCount

In MapReduce land WordCount takes a similar role as "Hello World" in
programming languages. Here is a slightly modified version of the
WordCount found in [Apache Hadoop's tutorial](http://hadoop.apache.org/common/docs/current//mapred_tutorial/).

```java

package examples;

import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.MRSJob;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;

public class NewWordCount {

    public static class Map extends
            Mapper<LongWritable, Text, Text, IntWritable> {
        private final static IntWritable one = new IntWritable(1);
        private Text word = new Text();

        @Override
        public void map(LongWritable key, Text value, Context context)
                throws IOException, InterruptedException {
            String line = value.toString();
            StringTokenizer tokenizer = new StringTokenizer(line);
            while (tokenizer.hasMoreTokens()) {
                word.set(tokenizer.nextToken());
                context.write(word, one);
            }
        }
    }

    public static class Reduce extends
            Reducer<Text, IntWritable, Text, IntWritable> {

        @Override
        public void reduce(
                Text key,
                Iterable<IntWritable> values,
                org.apache.hadoop.mapreduce.Reducer<Text, IntWritable, Text, IntWritable>.Context context)
                throws IOException, InterruptedException {
            int sum = 0;
            while (values.iterator().hasNext()) {
                sum += values.iterator().next().get();
            }
            context.write(key, new IntWritable(sum));
        }

    }

    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();

        conf.setBoolean("mrstreamer.distributed", false);

        Job job = new MRSJob(conf, "wordcount");

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        job.setMapperClass(Map.class);
        job.setReducerClass(Reduce.class);

        job.setInputFormatClass(TextInputFormat.class);
        job.setOutputFormatClass(TextOutputFormat.class);

        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        job.waitForCompletion(true);
    }

}
```

This MapReduce program consists of a map phase and a reduce phase.

The map function is repeatedly invoked for each line in the input files
and tokenizes the input line into words. In line 34 the function emits
for each word a key-value pair of the word and the integer 1,
representing one occurrence of the key word.

The reduce function is repeatedly invoked for each key word emitted by
the map function with an iterable containing all the values associated
with this key. In line 52 it reemits the key word with the sum of all
occurrence counts emitted in the map function.


The whole program thus transforms a input text into a key value file
with its keys being the words occurring in the input and the values the
number of occurrences of the key word in the input.


## Differences to the Apache Hadoop version

There are two lines which differ from the version of WordCount for
Apache Hadoop. In line 60 we set a job configuration parameter which is
only supported by MRStreamer: the boolean value of
mrstreamer.distributed controls if the program should be executed in a
distributed fashion or if should be run in a single JVM process. False
is the default value but we will be changing it later on in the
tutorial.

In order to run the program in MRStreamer we need to instantiate its
version of Job in line 62. If you were to run this program in Apache
Hadoop's execution environment you would have to change it to Hadoop's
Job implementation.

## Compilation

Save the program to examples/NewWordCount.java.

Compile it with the library contained in the download on the classpath
and create a JAR file using the jar command. Run the following commands
if you are not using an IDE:


```bash
javac -cp MRStreamer-assembly-0.9-SNAPSHOT.jar examples/NewWordCount.java
jar cvf mrstest.jar examples/*.class
```

## Running the program

### Shared-memory mode

In order to run the program we need to provide a directory containing
the input files and an output directory as parameters to our main
method. The downloaded file contains a testdata directory with a few
text files. Lets count them:

```bash
java -cp MRStreamer-assembly-0.9-SNAPSHOT.jar:mrstest.jar examples.NewWordCount testdata out
```

The program should create a subdirectory tree below the out directory
with its leaf containing a file called part-r-00000 which contains the
key value pairs of the result.


### Distributed mode

While running MapReduce programs in shared memory mode can be useful for
development purposes or when using a powerful multi-core machine you can
also run the program in distributed mode:

Change line 60 to true and recompile and repackage the program.

Start a server using the “server” script. Either provide no command line
options or specify --host and --port. The server should wait for workers
to connect:

```
$ ./server
...
[main] [INFO] [...] d.u.i.p.m.Server$: MRStreamer server started at 0.0.0.0:2553
[main] [INFO] [...] d.u.i.p.m.Server$: Waiting for workers to connect
```

Start a worker using the “worker” script. Provide --serverHost --serverPort and --workerPort.


```
$ ./worker --serverHost localhost --serverPort 2553 --workerPort 2554
...
[main] [INFO] [...] a.r.n.ActiveRemoteClient: Starting remote client connection to [localhost/127.0.0.1:2553]
```

Submit your JAR file to the server using the “jobsubmitter” script.
Provide --serverHost --serverPort and --jarFile as well as --mainClass.

```
$ ./jobsubmitter --serverHost localhost --serverPort 2553 --jarFile mrstest.jar --mainClass examples.NewWordCount testdata out
```

## What's next?

If you want to continue learning about writing Hadoop compatible
MapReduce programs check out the original
[Apache Hadoop MapReduce documentation](http://hadoop.apache.org/mapreduce/docs/r0.21.0/)
but keep in mind that not all features of Apache
Hadoop MapReduce are available, and there is no Apache Hadoop HDFS.
