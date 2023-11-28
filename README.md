# Hadoop Docker

### Deploy a Hadoop cluster

Run this:
```
  $ docker-compose up -d
```
This command makes multiple containers of Hadoop (1 namenode, 3 Datanodes and etc. as specified in "docker-compose.yml") at the same time.

### Checking the Hadoop cluster status

```
  $ docker ps
```

Also, going to http://DockerIP:9870/ to see the namenode status.

## Test the Hadoop cluster

Let's go to the namenode container.
```
  $ docker exec -it namenode bash
```
now we are in the namenode. (you can exit from the name node by typing "exit").

In the namenode, we will make a directory and 2 files in it (f1.txt and f2.txt). We fill them up with some simple content. type this:

```
  # mkdir input
  # echo "a b c" >input/f1.txt
  # echo "a d e" >input/f2.txt
```

now we will make a directory in HDFS with the same name and copy all the content in the "input" directory we just made to it. So they are now in the HDFS and distributed over our 3 datanodes:
```
  # hadoop fs -mkdir -p input
  # hdfs dfs -put ./input/* input
```

now type
```
  # exit
```
to finish namenode terminal.

Let's run a wordcount example over our 2 files in the HDFS and read the output. First, let's put the wordcount file into the HDFS and find the namenode Container ID.
```
  $ docker container ls
```

Now run the following to copy the jar file to the namenode docker container:
```
  $ docker cp ./hadoop-mapreduce-examples-2.7.1-sources.jar NAMENODE_CONTAINER_ID:hadoop-mapreduce-examples-2.7.1-sources.jar
```

So let go back to the namenode Terminal:
```
  $ docker exec -it namenode bash
```

now lets run the wordcount file that we just copied to the namenode over 2 files in the HDFS cluster. (Reading from the "input" directory and saving the results in the "output" directory.)
```
  # hadoop jar hadoop-mapreduce-examples-2.7.1-sources.jar org.apache.hadoop.examples.WordCount input output
  # hdfs dfs -cat output/part-r-00000
```
You will see the result.

To shut down the cluster and remove containers, use this command:
```
  $ docker-compose down
```


## Another Test 

To deploy an example HDFS cluster, run:
```
  docker-compose up
```

Run example wordcount job:
```
  make wordcount
```

Or deploy in a swarm:
```
docker stack deploy -c docker-compose-v3.yml hadoop
```

## Network of Hadoop Cluster

`docker-compose` creates a docker network that can be found by running `docker network list`.

Run `docker network inspect` on the network (e.g. `dockerhadoop_default`) to find the IP the Hadoop interfaces are published on. Access these interfaces with the following URLs:

* Namenode: http://<dockerhadoop_IP_address>:9870/dfshealth.html#tab-overview
* History server: http://<dockerhadoop_IP_address>:8188/applicationhistory
* Datanode: http://<dockerhadoop_IP_address>:9864/
* Nodemanager: http://<dockerhadoop_IP_address>:8042/node
* Resource manager: http://<dockerhadoop_IP_address>:8088/

## Configure Environment Variables

The configuration parameters can be specified in the hadoop.env file or as environmental variables for specific services (e.g. namenode, datanode, etc.):
```
  CORE_CONF_fs_defaultFS=hdfs://namenode:8020
```

CORE_CONF corresponds to core-site.xml. fs_defaultFS=hdfs://namenode:8020 will be transformed into:
```
  <property><name>fs.defaultFS</name><value>hdfs://namenode:8020</value></property>
```
To define dash inside a configuration parameter, use triple underscore, such as YARN_CONF_yarn_log___aggregation___enable=true (yarn-site.xml):
```
  <property><name>yarn.log-aggregation-enable</name><value>true</value></property>
```

The available configurations are:
* /etc/hadoop/core-site.xml CORE_CONF
* /etc/hadoop/hdfs-site.xml HDFS_CONF
* /etc/hadoop/yarn-site.xml YARN_CONF
* /etc/hadoop/httpfs-site.xml HTTPFS_CONF
* /etc/hadoop/kms-site.xml KMS_CONF
* /etc/hadoop/mapred-site.xml  MAPRED_CONF

If you need to extend some other configuration file, refer to base/entrypoint.sh bash script.
