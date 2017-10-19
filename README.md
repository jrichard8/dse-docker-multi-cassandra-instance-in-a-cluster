# dse-docker-multi-cluster
Docker conf to start multi cluster instance

## 1 - clone this project

## 2 - Download DataStax enterprise from https://academy.datastax.com/quick-downloads

## 3 - When downloaded, please remove the version number part of the filename (or create a symlink)
* so the resulting file is named dse-bin.tar.gz (that way the docker file itself remains version independent).

## 4 - Build image
* sudo docker build . -t <image_name>

## 5 - create a docker network
to force static IP
* sudo docker network create --subnet=172.18.0.0/16 dockernet

## 6 - Run clusters:

* Run analytics instance (spark)
`sudo docker run --net dockernet --ip 172.18.0.2 -e CLUSTER_NAME="Cluster_Spark" -e SPARK_LOCAL_IP="172.18.0.2" -e RPC_ADDRESS="172.18.0.2" -e LISTEN_ADDRESS="172.18.0.2" -e SEEDS="172.18.0.2,172.18.0.3" <image_name>  -k`

* Run Search instance (solr)
`sudo docker run --net dockernet --ip 172.18.0.3 -e CLUSTER_NAME="Cluster_Solr" -e RPC_ADDRESS="172.18.0.3" -e LISTEN_ADDRESS="172.18.0.3" -e SEEDS="172.18.0.2,172.18.0.3" <image_name>  -s`

## Create KEYSPACE with replication:

* `cqlsh 172..18.0.2`
* `CREATE KEYSPACE <yourekeyspace> WITH REPLICATION = { 'class' :'NetworkTopologyStrategy'[, 'Analytics' : 0, 'Solr' : 0};`

If all is ok a keyspace should be created on other docker

### Have fun


**Remember that docker doesn't persist data.**

if you want persistance:

## Create folder for persistant data
`mkdir -p /var/lib/cluster_datas/spark_cluster/data
mkdir -p /var/lib/cluster_datas/spark_cluster/conf
mkdir -p /var/lib/cluster_datas/spark_cluster/logs`

`mkdir -p /var/lib/cluster_datas/solr_cluster/data
mkdir -p /var/lib/cluster_datas/solr_cluster/conf
mkdir -p /var/lib/cluster_datas/solr_cluster/logs`

## Run docker on specific adress
* Run analytics instance (spark) with specify volume

`sudo docker run --net dockernet --ip 172.18.0.2 -e CLUSTER_NAME="Cluster_Spark" \
 -e SPARK_LOCAL_IP="172.18.0.2" \
 -e RPC_ADDRESS="172.18.0.2" \
 -e LISTEN_ADDRESS="172.18.0.2" \
 -e SEEDS="172.18.0.2,172.18.0.3" \
 -v /var/lib/cluster_datas/spark_cluster/data:/data \
 -v /var/lib/cluster_datas/spark_cluster/conf:/conf \
 -v /var/lib/cluster_datas/spark_cluster/logs:/logs \
 trax-dse-image  -k`
 
 * Run Search instance (solr) with specify volume
 
 `sudo docker run --net dockernet --ip 172.18.0.3 -e CLUSTER_NAME="Cluster_Solr" \
 -e RPC_ADDRESS="172.18.0.3" \
 -e LISTEN_ADDRESS="172.18.0.3" \
 -e SEEDS="172.18.0.2,172.18.0.3" \
 -v /var/lib/cluster_datas/solr_cluster/data:/data \
 -v /var/lib/cluster_datas/solr_cluster/conf:/conf \
 -v /var/lib/cluster_datas/solr_cluster/logs:/logs \
  trax-dse-image  -s`


