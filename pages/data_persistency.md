# Data Persistency
[back](../README.md)

The data persistent layer is perhaps the most complex one, and its key points are: 
* How it is distributed.
* What is its geographical scope. 
* The Amount of data.

**SQL Datasource**
For the SQL data source either MySQL or Posgtgres are good choices since both of them provide:

- active/active cluster 
- federation
- async replication
- data partioniting

Our setup has a 3 nodes cluster active/active, if mysql, we use xtradb multi master setup if on Postgres we can use Bi Direction Replication, the master cluster would be inside eu-dc1 and as close as poossible to the processig area since this would be the one inserting huge chunks of data, other dc's would have a slave node of the cluster so that read's are local and only writes are sent back to the master cluster.

Some considerations:

For postgresSQL with large datasets:

Some OS Optmization points:
- kernel.shmmax: This value must to be higher than the value that is configurated in the shared_buffers parameter
- kernel.shmall: This parameter must to have a SHMMAX/PAGE_SIZE as minimun.
- vm.nr_hugepages: This parameter set the number of huge pages to configure, it is a value dependent on the page size and configuration parameters of the instances related to shared memory
- Optimum performance is achieved in values of 25% tol 40% of the RAM.
- another key point is the calculation for shared_buffers / wal_buffer / wal_memory.
```
Example for 400 connections:
400 concurrent connections, we will be using 400 connections*32MB/connection + 8GB (shared buffers)+ aimed memory to autocaavuum=
( 400x32MB=12.8GB=8GB=20GB=10GB=29/30GB )
```

For MySQL/xtraDB with large datasets:


**noSQL datasource:**

Cassandra: cluster with 3 nodes.

Considerations:
-single ring for the 3nodes
-



Elastic Search: cluster with 3 nodes.

Considerations for the elastic cluster:
* Calculate Shard Sizing - Choosing the right number of shards is complicated because we never know how many documents we will get before we start, having lots of shards can be both good and terrible for a cluster, indices and shards management can overload the master node, which might become unresponsive, leading to strange and nasty behavior, as a preemptove measure we should allocate the master nodes with enough resources to cope with the cluster size.

* Replication: The replication formula used by Elasticsearch for consistency is: 
```
primary + number_of_replicas) / 2 + 1
```

* Optimising allocation - based data requirements, we could classify data into hot and cold and indices that are accessed more frequently than others, can have allocated more data nodes while indices that are less frequently accessed indices can have less resources allocated.

Resources: we should take in consideration that too much heap can subject to long garbage collection pauses, the standard recommendation is to give 50% of the available memory to elasticsearch heap, while leaving the other 50% free.

* Availability from the elstic service to the webservices area: Exposing the elastic search api allows us expose elastic data on services running outside the data persistency layer, communications are over https and authentication is enabled.