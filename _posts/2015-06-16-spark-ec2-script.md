---
layout: post
title: Launch a Cluster with Spark EC2 Script
comments: true
categories: Spark
---
Following up with my [previous post](/2015/06/15/aws-quick-start/) on preparing AWS for Spark Cluster, I will go over how to use Spark EC2 Script to launch a Spark Cluster in EC2

<!-- more -->

For a birdeye view, the Spark EC2 Script is python script that

* launch the master and slave instances for your cluster.
 * This is an AWS operation so the Spark EC2 script will need to authenticate with AWS programmatically by `Access Key ID` and `Secrete Access Key` of an AWS user.

* Set up Hadoop and Spark in the instances.
 * This is an operation inside the instances so the Spark EC2 script will need to `ssh` into the instance with `ssh` private key.

Before we provide, let's assume you have done the following:

* Set up a user in IAM
* Obtain the `Access Key ID` and `Secrete Access Key` for that user
* Apply access policy to the user
* Obtain the `ssh` keypair name and private key

## Obtain Spark EC2 Script
Download a Spark binary from [https://spark.apache.org/downloads.html] (https://spark.apache.org/downloads.html). I recommend you download the Pre-build for Hadoop 2.6 and later, which saves you the build time and you can test run Spark without worrying Hadoop installation.

Untar the downloaded file and put it in a convenient location.

## Setup Bash Environment
The Spark EC2 script will capture the `Access Key ID` and `Secrete Access Key` from the Bash environment variables. This saves you from typing ID and key repeated if you need to run several AWS admin script in a row.

In the Bash, type the following command and replace `???` with your ID and key.

``` bash
export AWS_ACCESS_KEY_ID=???
export AWS_SECRET_ACCESS_KEY=???
```

## Launch the Cluster
In the Bash prompt that you have set the environment variables, `cd` to the `ec2` folder of the Spark package. Then typing the following command:

``` bash
./spark-ec2 --key-pair=sparkkeypair --identity-file=sparkkeypair.pem --instance-type="t2.micro" --slaves 2 launch spark-cluster
```

Let's go over the options one by one.

#### key-pair
The name of the `ssh` key pair.

#### identity-file
The file holding the private key of your `ssh` key. If the file is not in the current working directory, recommend to provide full path.

#### instance-type
The type of machines. Please see my previous post regarding the type of machines available in AWS. By default, the Spark EC2 script will use `m1.large`. You can specify `t2.micro` here to take advantage of the AWS free-tiered service.

#### slaves
The number of worker nodes.

#### launch
The action of the script is to launch a cluster. Please provide the name of the cluster following `launch`.

## Stuck in 'enter ssh-ready state' ?!
After submitting the `launch` command, it is very likely that your screen will show the following without much activities.

![Waiting for cluster to enter 'ssh-ready' state](/images/aws/ssh-ready.png)
*Waiting for cluster to enter 'ssh-ready' state ...*

What is going on here? If you go to the EC2 dashboard ([https://console.aws.amazon.com/ec2/](https://console.aws.amazon.com/ec2/)) and click `instances`, you will notice that the instances are currently being initializing and as a result, not responding to the `ssh` request.

![Instances being initialized](/images/aws/initializing.png)
*Instances being initialized*

Please be patients and wait for a few minutes. Eventually the instances will complete initialization and pass the checks:

![Instances complete initialization and pass the checks]( /images/aws/checks_passed.png)
*Instances complete initialization and pass the checks*

Then your cluster will be in 'ssh-ready' state and the Spark EC2 script will be setting up Hadoop and Spark on the master and slaves.

![Cluster in ssh-ready state]( /images/aws/script_working.png)
*cluster in 'ssh-ready' state*

If in any chances, you are really stuck, you can press `CTRL+C` to terminate the Spark EC2 script. Then you can add the `--resume` option to the launch script and try again.

``` bash
./spark-ec2 --key-pair=sparkkeypair --identity-file=sparkkeypair.pem --instance-type="t2.micro" --slaves 2 --resume launch spark-cluster
```

## Cluster is Ready
The Spark EC2 script will notify you when it has completed the setup.

![Cluster Ready](/images/aws/cluster_ready.png)

Now open the web browser at [http://ec2-52-6-188-205.compute-1.amazonaws.com:8080] and you can check the status of cluster. All the slave nodes are online as expected.

![Cluster Web Interface](/images/aws/web_gui.png)

Please keep a note of the spark URL [spark://ec2-52-6-188-205.compute-1.amazonaws.com:7077], which is needed for launching the Spark Shell.

## Use the Cluster with Spark-Shell
After all the hassles, we have arrived at the point that we could actually use the cluster. It is tempting to just fire up your local spark-shell and supply it with the Spark URL of the cluster. However, this will not work. The reason is that the Spark EC2 script has implemented the default security policy for the cluster: The master will listen to Internet traffic at the port 22 (for `ssh`), 8080 (for the monitoring interface) but not port 7077 (for the Spark URL). Such decision is to prevent anyone on Internet without any credential from submitting jobs to your cluster (which Amazon charges you for). To actually reach the cluster, you have two options:

* Set up an VPN
* `ssh` into the master node and run spark-shell from it

Amazon conveniently offers the [VPN service]( http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_VPN.html) at a cost. Being a cheap person, I will go with the *free* `ssh` approach.

First `ssh` into the master node using the login action of the Spark EC2 script.

``` bash
./spark-ec2 --key-pair=sparkkeypair --identity-file=sparkkeypair.pem login spark-cluster
```
Once you are in, you can browse through the directory and see the folders created by the script.

![logged into the master] (/images/aws/logged_into_master.png)

Next, you fire up the spark-shell on the master node supply it with the cluster URL

``` bash
spark/bin/spark-shell --master spark://ec2-52-6-188-205.compute-1.amazonaws.com:7077
```

In a few seconds, you are finally at the Scala prompt of the spark-shell. Hooray!

![Scala prompt of the Spark Shell] (/images/aws/spark_shell_success.png)

Let's test the Spark-shell is indeed connected to the cluster:

![Checking Spark-Shell is connected to the cluster] (/images/aws/spark_shell_test.png)

## Destroy the Cluster
When you have completed your work with the Spark cluster, you can exit the Spark-shell and log out of the master node. Please be aware that the entire cluster is still running after you log out. Please remember to issue the following command to formally terminate the cluster.

``` Scala
./spark-ec2 --key-pair=sparkkeypair --identity-file=sparkkeypair.pem destroy spark-cluster
```


Cool, we have covered the essential to launch a Spark cluster in EC2 and start the Spark-shell. In the next post, I will talk about how to move data to the AWS Simple Storage Service (S3) so that you can really analyze big data with the cluster. Stay tuned.
