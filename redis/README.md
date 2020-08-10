# **Redis Cheatsheet**

### **Installation on Linux**
---
* Using `apt` package manger

Update repository: `sudo apt update`

Install redis-server: `sudo apt install redis-server`

* Download and `make`
```
wget http://download.redis.io/releases/redis-5.0.7.tar.gz
tar xzf redis-5.0.7.tar.gz
cd redis-5.0.7
make
```

### **Start**
---
* If installed using package manager

To start Redis server: `sudo service redis-server start`

Open redis-cli: `redis-cli`
* If downloaded and `make`

To start Redis server: `./redis-5.0.7/src/redis-server`

Open redis-cli: `./redis-5.0.7/src/redis-cli`

Type: `PING`, this should return a reply of `PONG`

To check whether Redis is able to persist data even after it’s been stopped or restarted. First
set a key: `SET test "ABCD"`, then exit redis-cli: `EXIT` 

Now, Stop redis-server and start it again. Open redis-cli, get key value: `GET test`

This should return `"ABCD"`

*To stop Redis server: `sudo service redis-server stop`*

*Redis server runs on default port 6379*

*In order to start Redis with a configuration file use the full path of the configuration file as first argument, like in the following example: redis-server /etc/redis.conf*

### **Keys**
---
Redis keys are binary safe, this means that you can use any binary sequence as a key, from a string like "foo" to the content of a JPEG file. The empty string is also a valid key.

*The maximum allowed key size is 512 MB.*

### **SET & GET**
---
SET is the command to set a value to a key & GET will retrieve the value.
```
    > SET foo bar
    OK
    > GET foo
    "bar"
```
SET will replace any existing value already stored into the key.

You can pass additional argument to SET, to fail if the key already exists, or the opposite, that it only succeed if the key already exists:
```
    > SET foo new nx
    (nil)
    > SET foo new xx
    OK
    > GET foo
    "OK"
    > SET bar new xx
    (nil)
```

### **Data Tyepes**
---
[Redis Datatypes](https://redis.io/topics/data-types-intro) documentation has details about data types with examples.

## For a quick revision of strings, list, hash and set:
---

* **Redis Strings**

Values can be strings (including binary data) of every kind, for instance you can store a jpeg image inside a value. *A value can't be bigger than 512 MB.*

The `INCR` command parses the string value as an integer, increments it by one, and finally sets the obtained value as the new value.There are other similar commands like `INCRBY`, `DECR` and `DECRBY`.

`INCR` is atomic which basically means even if multiple clients issue `INCR` against the same key it'll never enter into a race condition.

`GETSET` command sets a key to a new value, returning the old value as the result.

`MSET` and `MGET` is used to set or retrieve the value of multiple keys in a single command.
*`MGET` returns an array of values*

`EXISTS` returns 1 if key exists otherwise 0. `DEL` deletes a key and associated value.
`TYPES` returns data type.

Redis `EXPIRE` set a timeout for a key:
```
    > SET key some-value
    OK
    > EXPIRE key 5
    (integer) 1
    > GET key (immediately)
    "some-value"
    > GET key (after some time)
    (nil)
    > SET key 100 ex 10
    OK
    > TTL key
    (integer) 9
```
* **Redis Lists**
    * `LPUSH` adds element on the left. `LPUSH list-name element1 element2`
    * RPUSH add element on the right. `RPUSH list-name element1 element2`
    
    *Both `LPUSH` & `RPUSH` are variadic commands*
    * `LRANGE` extracts ranges of elements from lists. It takes two arguments as range. 0 is the first & -1 is the last element. `LRANGE list-name 0 -1`
    * `RPOP list-name`
    * `LPOP list-name`
    * `LLEN list-name' returns the length of the list
    * The `LTRIM` command sets its range as the new list value. All the elements outside the given range are removed. `LTRIM list-name 0 2`

***Redis has automatic creation and removal of keys***

* **Redis Hashes**
```
    > HMSET user:1000 username antirez birthyear 1977 verified 1
    OK
    > HGET user:1000 username
    "antirez"
    > HGET user:1000 birthyear
    "1977"
    > HGETALL user:1000
    1) "username"
    2) "antirez"
    3) "birthyear"
    4) "1977"
    5) "verified"
    6) "1"
```
* **Redis Sets**
    * `SADD set-name element1, element2`
    * `SMEMBERS set-name`
    * `SISMEMBER myset 3` returns 1 if 3 exist in the set


### **Redis Cluster**
---
Redis cluster provides a way for automatically [data shardding](https://en.wikipedia.org/wiki/Shard_(database_architecture)) across multiple Redis nodes. In Redis cluster, every key is coneceptually part of what is called an hash slot.

*There are 16384 hash slots in Redis Cluster.*
Every node in a Redis Cluster is responsible for a subset of these hash slots. 

For example, if we have 3 nodes (A, B, C), then:

Node A contains hash slots from 0 to 5500.
Node B contains hash slots from 5501 to 11000.
Node C contains hash slots from 11001 to 16383.

If we add a new Node D, some hash slot will move from A, B, C to D. Similarly, if we want to remove A, slots served by A will move to B and C.

***Master-slave model***

Suppose, our node A fails to connect to majority of nodes, then what should happen? Here comes the master-slave model. 

Every master node will have replicas which are called slave. If a master fails, then among the slave, one node will be elected as master. If the previos master comes back, it'll act as a slave. Data write request goes to master node.
*If master and slave fails at the same time, redis cluster will not be able to continue to opearate*

***Redis Cluster is not able to guarantee strong consistency***

Suppose, master node A gets write request. It writes the data and send OK to client. Then A propagates the write to its slave. But, if A crashes before sending the data to slaves, the data will be lost. One of its slave will be act as master without the data.

**Cluster configuration parameters**

Redis cluster introduces cluster configuration parameters in `redis.conf` file.
* **cluster-enabled** <yes/no>: If yes, enables Redis Cluster support in a specific Redis instance. Otherwise the instance starts as a stand alone instance as usual.
* **cluster-config-file** <filename>: This is not a user editable configuration file. Basically, this is the file where a cluster node persists it's state so that, it can re-read from the file when needed.
* **cluster-node-timeout** <milliseconds>: The maximum amount of time a Redis Cluster node can be unavailable before a slave node takesover.

Enable clustering in redis.conf file:
```
    cluster-enabled yes 
    cluster-config0file nodes-6379.conf 
    cluster-node-timeout 15000
```

To create a cluster we need a few empty Redis instances running in cluster mode. As a special mode needs to be configured, so that the Redis instance will enable the Cluster specific features and commands.

Following is a minimal Redis cluster configuration:
```
    port 7000
    cluster-enabled yes
    cluster-config-file node.conf
    cluster-node-timeout 5000
    appendonly yes
```
`cluster-enabled` directive enables the cluster. Every instance will have a `nodes.conf` file that stores the configuration for that node. It is automatically generated at startup. We don't need to touch it.

*Note that the minimal cluster that works as expected requires to contain at least three master nodes. For our test we'll start a six nodes cluster with three masters and three slaves.*

Let's have the following the configuration for our nodes:
```
    port 7000
    cluster-enabled yes
    cluster-config-file node-0.conf
    cluster-node-timeout 5000
    appendonly yes
    appendfilename node-0.aof
    dbfilename dump-0.rdb
```
Create 6 files `7000.conf` `7001.conf` `7002.conf` `7003.conf` `7004.conf` `7005.conf` and paste the configuration. Make sure to change the port number, .conf filename, .aof filename and .rdb filename according to the port.

*You can type `touch 700{0..5}.conf` in terminal to create 6 files.*

Now, we need to start 6 server with their `.conf` files. 

*Depending on your installation it may vary how you start a server.*

If you used a package manager to install redis-server, then: `redis-server 7000.conf`

If you downloaded redis and did `make` then : `/path-of-redis-server/redis-5.0.7/src/redis-server 7000.conf`

Finally, the time has come to create the cluster. 
```
    /redis-5.0.7/src/redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 --cluster-replicas 1
```
*If you're on Redis < 5*
```
    ./redis-trib.rb create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005
```
We used `create` command. `--cluster-replicas 1` means we want a slave for every master created.

`redis-cli` will propose a configuration, type yes to accept.

voilà! 👏 We have our cluster with 3 master and 3 slave nodes.
