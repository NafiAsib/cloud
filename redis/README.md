# **Redis Cheatsheet**

### **Installation**
---
Update repository: `sudo apt update`

Install redis-server: `sudo apt install redis-server`

### **Start**
---
To start Redis server: `sudo service redis-server start`

Open redis-cli: `redis-cli`

Type: `PING`, this should return a reply of `PONG`

To check whether Redis is able to persist data even after it’s been stopped or restarted. First
set a key: `SET test "ABCD"`, then exit redis-cli: `EXIT` 

Now, Stop redis-server and start it again. Open redis-clie, get key value: `GET tests`

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