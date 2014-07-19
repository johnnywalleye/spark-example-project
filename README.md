# Spark Example Project [![Build Status](https://travis-ci.org/johnnywalleye/spark-example-project.png)](https://travis-ci.org/johnnywalleye/spark-example-project)

## Introduction

This is a simple binary classification job written in Scala for the [Spark] [spark] cluster computing platform, with instructions for running on [Amazon Elastic MapReduce] [emr] in non-interactive mode.  It is based very closely on (and would not exist without!) the [Spark Word Count Example] [spark-word-count-project] from [Snowplow Analytics] [snowplow].

Modifications to the original Snowplow project were made by the Data team at [Intent Media] [intent].

_See also (from Snowplow Analytics):_ [Spark Word Count Example] [spark-word-count-project] | [Scalding Example Project] [scalding-example-project] | [Cascalog Example Project] [cascalog-example-project]

## Building

Assuming you already have [SBT] [sbt] installed:

    $ git clone git://github.com/johnnywalleye/spark-example-project.git
    $ cd spark-example-project
    $ sbt assembly

The 'fat jar' is now available as:

    target/spark-example-project-0.2.0.jar

## Unit testing

The `assembly` command above runs the test suite - but you can also test the word count job manually with:

    $ sbt test
    <snip>
    [info] + A WordCount job should
    [info]   + count words correctly
    [info] Passed: : Total 1, Failed 0, Errors 0, Passed 1, Skipped 0

## Running the Logistic Regression on Amazon EMR

### Prepare

Assuming you have already assembled the jarfile (see above), now upload the jar to an Amazon S3 bucket and make the file publicly accessible.

### Run

Finally, you are ready to run the logistic regression job using the [Amazon Ruby EMR client] [emr-client]:

```
$ elastic-mapreduce --create --name "BinaryClassificationJob" --instance-type m3.xlarge --instance-count 3 \
  --bootstrap-action s3://intentmedia-spark/install-spark-shark.sh --bootstrap-name "Install Spark/Shark" \
  --jar s3://elasticmapreduce/libs/script-runner/script-runner.jar --step-name "binary classification run" \
  --step-action TERMINATE_JOB_FLOW \
  --arg s3://snowplow-hosted-assets/common/spark/run-spark-job-0.1.0.sh \
  --arg s3://{{JAR_BUCKET}}/spark-example-project-0.2.0.jar \
  --arg com.intentmedia.spark.BinaryClassificationJob \
  --arg s3://intentmedia-spark/sample_binary_classification_data.txt \
  --arg s3n://{{OUT_BUCKET}}/results \
  --arg --algorithm --arg LR \
  --arg --regType --arg L2 \
  --arg --regParam --arg 0.1 \
  --arg --executorMemory --arg 512m
```

Replace `{{JAR_BUCKET}}` and `{{OUT_BUCKET}}` with the appropriate paths.

### Inspect

Once the output has completed, you should see a folder structure like this in your output bucket:

     results
     |-- model-stats
     |    |-- _SUCCESS
     |    |-- part-00000
     |    +-- part-00001
     +-- predictions-and-labels
          |-- _SUCCESS
          |-- part-00000
          +-- part-00001
     
After downloading and catting together the part files (cat part-0* > part-all) under predictions-and-labels, you should see something like this:

 (0.0,0.0)
 (0.0,0.0)
 (1.0,1.0)
 (0.0,0.0)
 (1.0,1.0)
 (1.0,1.0)
 (1.0,1.0)
 (0.0,0.0)
 (0.0,0.0)
 (0.0,0.0)
 (0.0,0.0)
 (0.0,0.0)
 (0.0,0.0)
 (1.0,1.0)


## Further reading

* [Run Spark and Shark on Amazon Elastic MapReduce] [aws-spark-tutorial]
* [Running Spark job on EMR as a jar in non-interactive mode] [spark-emr-howto]

## Copyright and license

Copyright 2013-2014 Snowplow Analytics Ltd, Intent Media Inc.

Licensed under the [Apache License, Version 2.0] [license] (the "License");
you may not use this software except in compliance with the License.

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

[spark]: http://spark-project.org/
[wordcount]: https://github.com/twitter/scalding/blob/master/README.md
[intent]: http://intentmedia.com
[snowplow]: http://snowplowanalytics.com
[data-pipelines-algos]: http://snowplowanalytics.com/services/pipelines.html

[spark-word-count-project]: https://github.com/snowplow/spark-example-project
[scalding-example-project]: https://github.com/snowplow/scalding-example-project
[cascalog-example-project]: https://github.com/snowplow/cascalog-example-project

[issue-1]: https://github.com/snowplow/spark-example-project/issues/1
[issue-2]: https://github.com/snowplow/spark-example-project/issues/2
[aws-spark-tutorial]: http://aws.amazon.com/articles/4926593393724923
[spark-emr-howto]: https://forums.aws.amazon.com/thread.jspa?messageID=458398

[sbt]: http://www.scala-sbt.org/release/docs/Getting-Started/Setup.html

[emr]: http://aws.amazon.com/elasticmapreduce/
[hello-txt]: https://github.com/snowplow/spark-example-project/raw/master/data/hello.txt
[emr-client]: http://aws.amazon.com/developertools/2264

[elasticity]: https://github.com/rslifka/elasticity
[spark-plug]: https://github.com/ogrodnek/spark-plug
[lemur]: https://github.com/TheClimateCorporation/lemur
[boto]: http://boto.readthedocs.org/en/latest/ref/emr.html

[license]: http://www.apache.org/licenses/LICENSE-2.0
