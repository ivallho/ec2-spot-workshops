---
title: "Launch a cluster - Step 1"
weight: 60
draft: true
---

In this step we'll launch our first cluster. This will be a transient cluster that will be shut down after it finishes the job we submit to it, and will run solely on Spot Instances. The job is a simple word count application that will run against a public data set of Amazon product reviews.

To launch the cluster, follow these steps:\

1. [Open the EMR console] (https://console.aws.amazon.com/elasticmapreduce/home) in the region where you are looking to launch your cluster.\
1. Click "**Create Cluster**"\
1. Click "**Go to advanced options**"\
1. Select the latest EMR release, and in the list of components, only leave **Hadoop** checked and also check **Spark**.\
1. Under "**Add steps (Optional)**" -> Step type drop down menu select "**Spark application**" and click **Configure**, then add the following details in the Add step dialog window:\
**Spark-submit options**: here we will configure the memory and core count for each executor, as described in the previous section. Enter: *--executor-memory 18G --executor-cores 4*\
**Application location**: here we will configure the location of our Spark application. Save the following python code to a file and upload it to your S3 bucket using the AWS console or AWS CLI: *aws s3 cp \<filename\> s3://\<your-bucket-name\>*

```python
    import sys
    from pyspark.sql import SparkSession
    spark = SparkSession.builder.appName('Amazon reviews word count').getOrCreate()
    df = spark.read.parquet("s3://amazon-reviews-pds/parquet/")
    df.selectExpr("explode(split(lower(review_body), ' ')) as words").groupBy("words").count().write.mode("overwrite").parquet(sys.argv[1])
    exit()
```
Then add the location of the file under the **Application location** field, i.e: s3://\<your-bucket-name\>/\<filename\>\
**Arguments**: Here we will configure the location of where Spark will write the results of the job. Enter: s3://\<your-bucket-name\>/results/\
**Action on failure**: Leave this on *Continue* and click **Add** to save the step.

![sparksubmit](/images/running-emr-spark-apps-on-spot/sparksubmit.png)

Check the **Auto-terminate cluster after the last step is completed** option. Since we are looking to run a transient cluster just for running our Spark application, this will terminate the cluster once our submitted step (Spark Application) has completed.

Click **Next** to continue setting up the EMR cluster and move from "**Step 1: Software and steps**"" to "**Step 2: Hardware**".
