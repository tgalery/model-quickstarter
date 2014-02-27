Quickstarter for DBpedia Spotlight models
===================================================

You can use this tool for creating models of DBpedia Spotlight in your language.


    #Prepare the system:
    ./prepare.sh
    
    #Create a Dutch model on Hadoop:
    ./index_db.sh -o nl wdir nl_NL nl/stopwords.list Dutch models/nl
    
    #Create a Dutch model on the current machine (without Hadoop):
    ./index_db.sh -l -o nl wdir nl_NL nl/stopwords.list Dutch models/nl
    

## Requirements

This script requires Apache Hadoop and Apache Pig, for more information see [this guide](https://github.com/dbpedia-spotlight/dbpedia-spotlight/wiki/Internationalization-%28DB-backed-core%29). Tested with **Apache Pig version 0.10.0** and **Hadoop 1.1.0**, as well as the default Pig and Hadoop on AWS.

Alternatively, indexing can be run in local mode.

## Usage

### Locales and language codes

The locales used in `run.sh` are the locales supported by the `java.text` package. See the list of supported locales [here](http://www.oracle.com/technetwork/java/javase/locales-137662.html).

### Running the model creation

If you have OpenNLP models, add them to your language folder and add the following line to run.sh (example for Dutch):

    ./index_db.sh -o nl wdir nl_NL nl/stopwords.list Dutch models/nl

This will create a DBpedia Spotlight model (for the DB-backend) in `models/nl`. If you do not have OpenNLP models, only add a stopwords file to your 
language folder and add the following to run.sh:

    ./index_db.sh wdir fr_FR fr/stopwords.list French models/fr


### Running on a Hadoop cluster

Smaller Wikipedias can be run in local-mode. However, for the English Wikipedia, a cluster or a big machine are recommended.

The current version for English runs runs for about 24h (including data download) on three nodes (250GB disk, 4 CPUs, 32GB RAM) with the following `mapred-site.xml`:

```xml

<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
  <property>
    <name>mapred.job.tracker</name>
    <value>spotlight:54311</value>
    <description>The host and port that the MapReduce job tracker runs
    at.  If "local", then jobs are run in-process as a single map
    and reduce task.
    </description>
  </property>
  <property>
    <name>mapred.tasktracker.map.tasks.maximum</name>
    <value>4</value>
  </property>
  <property>
    <name>mapred.tasktracker.reduce.tasks.maximum</name>
    <value>3</value>
  </property>
  <property>
    <name>mapred.local.dir</name>
    <value>${hadoop.tmp.dir}/mapred/local</value>
  </property>
  <property>
    <name>mapred.map.child.java.opts</name>
    <value>-Xmx7600M</value>
  </property>
  <property>
    <name>mapred.reduce.child.java.opts</name>
    <value>-Xmx7600M</value>
  </property>
</configuration>
```
### Running on Amazon EC2

Some of the pig jobs require a significant amount of memory, so we recommend to run with nodes with at least 8GB of memory and the bootstrap action for memory-intensive job flows. The master node performs the post-processing of the data, so it must have enough memory (for indexing English, we recommend 24GB-32GB).
We also added a setup_aws.sh script that we use to automate our setup when we are running this on EC2.
Running setup_aws.sh will install other useful things that do not come out of the box in those instances such as Maven3, and other useful tools like screen and elinks.

To run the indexing, start an Elastic MapReduce job with an interactive pig session, login to the master node and run:

    $ git clone https://github.com/$YOU/model-quickstarter.git
    $ cd model-quickstarter
    $ ./scripts/setup_aws.sh
    $ ./run.sh
