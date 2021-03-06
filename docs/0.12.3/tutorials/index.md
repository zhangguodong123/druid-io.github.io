---
layout: doc_page
---

# Druid Quickstart

In this quickstart, we will download Druid, set it up on a single machine, load some data, and query the data.

## Prerequisites

You will need:

  * Java 8 or higher
  * Linux, Mac OS X, or other Unix-like OS (Windows is not supported)
  * 8G of RAM
  * 2 vCPUs

On Mac OS X, you can use [Oracle's JDK
8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) to install
Java.

On Linux, your OS package manager should be able to help for Java. If your Ubuntu-
based OS does not have a recent enough version of Java, WebUpd8 offers [packages for those
OSes](http://www.webupd8.org/2012/09/install-oracle-java-8-in-ubuntu-via-ppa.html).

## Getting started

To install Druid, issue the following commands in your terminal:

```bash
curl -O http://static.druid.io/artifacts/releases/druid-0.12.3-bin.tar.gz
tar -xzf druid-0.12.3-bin.tar.gz
cd druid-0.12.3
```

In the package, you should find:

* `LICENSE` - the license files.
* `bin/` - scripts useful for this quickstart.
* `conf/*` - template configurations for a clustered setup.
* `conf-quickstart/*` - configurations for this quickstart.
* `extensions/*` - all Druid extensions.
* `hadoop-dependencies/*` - Druid Hadoop dependencies.
* `lib/*` - all included software packages for core Druid.
* `quickstart/*` - files useful for this quickstart.

## Download tutorial example files

Before proceeding, please download the [tutorial examples package](../tutorials/tutorial-examples.tar.gz). 

This tarball contains sample data and ingestion specs that will be used in the tutorials. 

```bash
curl -O http://druid.io/docs/0.12.3/tutorials/tutorial-examples.tar.gz
tar zxvf tutorial-examples.tar.gz
```

## Start up Zookeeper

Druid currently has a dependency on [Apache ZooKeeper](http://zookeeper.apache.org/) for distributed coordination. You'll
need to download and run Zookeeper.

```bash
curl http://www.gtlib.gatech.edu/pub/apache/zookeeper/zookeeper-3.4.10/zookeeper-3.4.10.tar.gz -o zookeeper-3.4.10.tar.gz
tar -xzf zookeeper-3.4.10.tar.gz
cd zookeeper-3.4.10
cp conf/zoo_sample.cfg conf/zoo.cfg
./bin/zkServer.sh start
```

## Start up Druid services

With Zookeeper running, return to the druid-0.12.3 directory. In that directory, issue the command:

```bash
bin/init
```

This will setup up some directories for you. Next, you can start up the Druid processes in different terminal windows.
This tutorial runs every Druid process on the same system. In a large distributed production cluster,
many of these Druid processes can still be co-located together.

```bash
java `cat examples/conf/druid/coordinator/jvm.config | xargs` -cp "examples/conf/druid/_common:examples/conf/druid/_common/hadoop-xml:examples/conf/druid/coordinator:lib/*" io.druid.cli.Main server coordinator
java `cat examples/conf/druid/overlord/jvm.config | xargs` -cp "examples/conf/druid/_common:examples/conf/druid/_common/hadoop-xml:examples/conf/druid/overlord:lib/*" io.druid.cli.Main server overlord
java `cat examples/conf/druid/historical/jvm.config | xargs` -cp "examples/conf/druid/_common:examples/conf/druid/_common/hadoop-xml:examples/conf/druid/historical:lib/*" io.druid.cli.Main server historical
java `cat examples/conf/druid/middleManager/jvm.config | xargs` -cp "examples/conf/druid/_common:examples/conf/druid/_common/hadoop-xml:examples/conf/druid/middleManager:lib/*" io.druid.cli.Main server middleManager
java `cat examples/conf/druid/broker/jvm.config | xargs` -cp "examples/conf/druid/_common:examples/conf/druid/_common/hadoop-xml:examples/conf/druid/broker:lib/*" io.druid.cli.Main server broker
```

Once every service has started, you are now ready to load data.

### Resetting cluster state

All persistent state such as the cluster metadata store and segments for the services will be kept in the `var` directory under the druid-0.12.3 package root. 

Later on, if you'd like to stop the services, CTRL-C to exit from the running java processes. If you
want a clean start after stopping the services, delete the `log` and `var` directory and run the `init` script again, then shut down Zookeper and delete the Zookeeper dataDir under `/tmp/zookeeper`.

From the druid-0.12.3 directory:

```bash
rm -rf log
rm -rf var
bin/init
```

If you ran the [Tutorial: Loading stream data from Kafka](./tutorial-kafka.html), you should shut down Kafka before shutting down Zookeeper, and then delete the Kafka log directory, which is at `/tmp/kafka-logs`.

Ctrl-C to shut down the Kafka broker, and then delete the log directory:

```bash
rm -rf /tmp/kafka-logs
```

Now stop Zookeeper and clear its state. From the zookeeper-3.4.10 directory:

```bash
./bin/zkServer.sh stop
rm -rf /tmp/zookeeper
```

After clearing out Druid and Zookeeper state, restart Zookeeper and then restart the Druid services.

## Loading Data

### Tutorial Dataset

For the following data loading tutorials, we have included a sample data file containing Wikipedia page edit events that occurred on 2015-09-12.

This sample data is located at `quickstart/wikiticker-2015-09-12-sampled.json.gz` from the Druid package root. The page edit events are stored as JSON objects in a text file.

The sample data has the following columns, and an example event is shown below:

  * added
  * channel
  * cityName
  * comment
  * countryIsoCode
  * countryName
  * deleted
  * delta
  * isAnonymous
  * isMinor
  * isNew
  * isRobot
  * isUnpatrolled
  * metroCode
  * namespace
  * page
  * regionIsoCode
  * regionName
  * user
 
```json
{
  "timestamp":"2015-09-12T20:03:45.018Z",
  "channel":"#en.wikipedia",
  "namespace":"Main"
  "page":"Spider-Man's powers and equipment",
  "user":"foobar",
  "comment":"/* Artificial web-shooters */",
  "cityName":"New York",
  "regionName":"New York",
  "regionIsoCode":"NY",
  "countryName":"United States",
  "countryIsoCode":"US",
  "isAnonymous":false,
  "isNew":false,
  "isMinor":false,
  "isRobot":false,
  "isUnpatrolled":false,
  "added":99,
  "delta":99,
  "deleted":0,
}
```

The following tutorials demonstrate various methods of loading data into Druid, including both batch and streaming use cases.

### [Tutorial: Loading a file](./tutorial-batch.html)

This tutorial demonstrates how to perform a batch file load, using Druid's native batch ingestion.

### [Tutorial: Loading stream data from Kafka](./tutorial-kafka.html)

This tutorial demonstrates how to load streaming data from a Kafka topic.

### [Tutorial: Loading a file using Hadoop](./tutorial-batch-hadoop.html)

This tutorial demonstrates how to perform a batch file load, using a remote Hadoop cluster.

### [Tutorial: Loading data using Tranquility](./tutorial-tranquility.html)

This tutorial demonstrates how to load streaming data by pushing events to Druid using the Tranquility service.

### [Tutorial: Writing your own ingestion spec](./tutorial-ingestion-spec.html)

This tutorial demonstrates how to write a new ingestion spec and use it to load data.