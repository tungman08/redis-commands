# Redis

### Table of Contents:

1. [Connection](#connection)
1. [Key](#key)
1. [Transaction](#transaction)
1. [Data Type](#data-type)
    - [String](#string)
    - [Lists](#lists)
    - [Sets](#sets)
    - [Sorted Sets](#sorted-sets)
    - [Hashes](#hashes)

# Connection

- AUTH
- ECHO
- PING
- QUIT
- SELECT

### Connection Commands:

- **AUTH** > Redis AUTH command is used to authenticate a password-protected server with a given password. If provided password matches the password in the configuration file, the server replies with the OK status code and starts accepting commands. Otherwise, an error is returned and the clients need to try a new password.

    **Example**

    ```code
    redis 127.0.0.1:6379> AUTH PASSWORD
    (error) ERR Client sent AUTH, but no password is set
    redis 127.0.0.1:6379> CONFIG SET requirepass "mypass"
    OK
    redis 127.0.0.1:6379> AUTH mypass
    Ok
    ```

- **ECHO** > Redis ECHO command is used to print the given string.

    **Example**

    ```code
    redis 127.0.0.1:6379> ECHO "Hello World"
    "Hello World"
    ```

- **PING** > Redis PING command is used to check whether a server connection is still alive, or to measure latency.

    **Example**

    ```code
    redis 127.0.0.1:6379> PING
    PONG
    ```

- **QUIT** > Redis QUIT command ask the server to close the connection. The connection is closed as soon as all pending replies have been written to the client.

    **Example**

    ```code
    redis 127.0.0.1:6379> QUIT
    OK
    ```

- **SELECT** > Redis SELECT command is used to select the DB with having the specified zero-based numeric index. New connections always use DB 0.

    **Example**

    ```code
    redis 127.0.0.1:6379> SELECT 1
    OK
    redis 127.0.0.1:6379[1]>
    ```

# Key:

- The maximum allowed key size is 512 MB.
- Very long keys are not a good idea.
- Very short keys are often not a good idea. While short keys will obviously consume a bit less memory, your job is to find the right balance.
- Try to stick with a schema. For instance "object-type:id" is a good idea, as in "user:1000". Dots or dashes are often used for multi-word fields, as in "comment:1234:reply.to" or "comment:1234:reply-to".

> So, I guess, in your case, your could store your key in the following format "user:{userid}:{username}. It can be searched for by using Redis SCAN with MATCH option. [http://redis.io/commands/scan](http://redis.io/commands/scan)

### Key Commands:

- **DEL** > The Redis DEL command is used to remove the specified keys. A key is ignored if it does not exist.

    **Example**

    ```code
    127.0.0.1:6379> SET key "Good"
    OK
    127.0.0.1:6379> SET key1 "Morning"
    OK
    127.0.0.1:6379> DEL key key1 key2
    (integer) 2
    ```

- **DUMP** > Redis DUMP command is used to serialize the value stored at key in a Redis-specific format and return it to the user. The returned value can be synthesized back into a Redis key using the RESTORE command.

    **Example**

    ```code
    127.0.0.1:6379> SET key1 Good
    OK
    127.0.0.1:6379> DUMP key1
    "\x00\x04Good\x06\x00\x94O\x0e\x8d\xb3\x0b\x00?"
    ```

- **EXISTS** > The Redis EXISTS command is used to check whether key exists in redis or not.

    **Example**

    ```code
    127.0.0.1:6379> SET key "Good"
    OK
    127.0.0.1:6379> EXISTS key
    (integer) 1
    127.0.0.1:6379> EXISTS KEY
    (integer) 0
    ```

- **EXPIRE** > Redis Expire command is used to set a timeout on key. After the timeout has expired, the key will automatically be deleted. A key with an associated timeout is often said to be volatile in Redis terminology.

    **Example**

    ```code
    127.0.0.1:6379> SET key "Apple"
    OK
    127.0.0.1:6379> EXPIRE key 5
    (integer) 1
    127.0.0.1:6379> TTL key
    (integer) -2
    127.0.0.1:6379> SET key "Banana"
    OK
    127.0.0.1:6379> TTL key
    (integer) -1
    ```

- **EXPIREAT** > The Redis EXPIREAT command is used to convert relative timeouts to absolute timeouts for the AOF persistence mode. It can be used directly to specify that a given key should expire at a given time in the future.

    **Example**

    ```code
    127.0.0.1:6379> SET key "Apple"
    OK
    127.0.0.1:6379> EXISTS key
    (integer) 1
    127.0.0.1:6379> EXPIREAT key 1039840000
    (integer) 1
    127.0.0.1:6379> EXISTS key
    (integer) 0
    ```

- **KEYS** > Redis KEYS command is used to return all keys matching a pattern.
    - h?llo matches hello, hallo and hxllo
    - h*llo matches hllo and heeeello
    - h[ae]llo matches hello and hallo, but not hillo
    - h[^e]llo matches hallo, hbllo, ... but not hello
    - h[a-b]llo matches hallo and hbllo

    **Example**

    ```code
    127.0.0.1:6379> MSET key 1 key1 2 key2 3 key3 4
    OK
    127.0.0.1:6379> KEYS *e*
    1) "key"
    2) "key3"
    3) "key1"
    4) "key2"
    127.0.0.1:6379> KEYS s??
    (empty list or set)
    ```

- **PERSIST** > Redis PERSIST command is used to remove the existing timeout on key, turning the key from volatile (a key with an expire set) to persistent.

    **Example**

    ```code
    127.0.0.1:6379> SET key "JavaScript"
    OK
    127.0.0.1:6379> EXPIRE key 20
    (integer) 1
    127.0.0.1:6379> TTL key
    (integer) 16
    127.0.0.1:6379> PERSIST key
    (integer) 1
    127.0.0.1:6379> TTL key
    (integer) -1
    127.0.0.1:6379> Get key
    "JavaScript"
    ```

- **PEXPIRE** > The Redis Pexpire command is used to set the time of living of a key in milliseconds instead of seconds. After the expiry time, key will not be available in redis.

    **Example**

    ```code
    127.0.0.1:6379> SET key "Apple"
    OK
    127.0.0.1:6379> PEXPIRE key 1800
    (integer) 1
    127.0.0.1:6379> TTL key
    (integer) -2
    127.0.0.1:6379> PTTL key
    (integer) -2
    ```

- **PEXPIREAT** > Redis Pexpireat command is used to set the expiry of key in unix timestamp at which the key will expire is specified in milliseconds instead of seconds. After the expiry time, key will not be available in redis.

    **Example**

    ```code
    127.0.0.1:6379> SET key "Apple"
    OK
    127.0.0.1:6379> PEXPIREAT key 1555555555005
    (integer) 1
    127.0.0.1:6379> TTL key
    (integer) 113072338
    127.0.0.1:6379> PTTL key
    (integer) 113072329357
    ```

- **PTTL** > Redis PTTL command is used to get remaining time to live of a key that has an expire set in milliseconds instead of seconds that TTL returns the amount of remaining time.

    **Example**

    ```code
    127.0.0.1:6379> SET key "Good"
    OK
    127.0.0.1:6379> PEXPIRE key 3000
    (integer) 1
    127.0.0.1:6379> PTTL key
    (integer) -2
    127.0.0.1:6379> GET key
    (nil)
    ```

- **RANDOMKEY** > Redis RANDOMKEY command is used to get a random key from currently selected redis database.

    **Example**

    ```code
    127.0.0.1:6379> RANDOMKEY
    "KEY"
    127.0.0.1:6379> RANDOMKEY
    "key1"
    ```

- **RENAME** > The Redis RENAME command is used to change the name of a key to newkey.

    **Example**

    ```code
    127.0.0.1:6379> SET key "PHP"
    OK
    127.0.0.1:6379> RENAME key JavaScript
    OK
    127.0.0.1:6379> GET JavaScript
    "PHP"
    ```

- **RENAMENX** > Redis RENAMENX command is used to change the name of a key to newkey if the new key does not exist. It returns an error under the same conditions as RENAME.

    **Example**

    ```code
    127.0.0.1:6379> SET key "Apple"
    OK
    127.0.0.1:6379> SET key1 "Banana"
    OK
    127.0.0.1:6379> RENAMENX key key1
    (integer) 0
    127.0.0.1:6379> GET key1
    "Banana"
    ```

- **RESTORE** > The Redis RESTORE command is used to create a key associated with a value that is obtained by deserializing the provided serialized value (obtained via DUMP).

    **Example**

    ```code
    127.0.0.1:6379> SET key "Apple"
    OK
    127.0.0.1:6379> Dump key
    "\x00\x05Apple\x06\x00\r\x88\x03\xb7\x96\x16\xf6p"
    127.0.0.1:6379> DEL key
    (integer) 1
    127.0.0.1:6379> restore key 0 "\x00\x05Apple\x06\x00\r\x88\x03\xb7\x96\x16\xf6p"
    OK
    127.0.0.1:6379> GET key
    "Apple"
    ```

- **SCAN** > The Redis SCAN command  is used in order to incrementally iterate over a collection of elements.

    **Example**

    ```code
    127.0.0.1:6379> SCAN 0
    1) "1"
    2)  1) "numbers"
        2) "hash key"
        3) "myhash"
        4) "myset"
        5) "w3rkey"
    ```

- **SORT** > Returns or stores the elements contained in the list, set or sorted set at the key. By default, sorting is numeric and elements are compared by their value interpreted as double precision floating point number.

    **Example**

    ```code
    127.0.0.1:6379> SORT xurl alpha desc
    1) "Youtube.com"
    2) "Yahoo.com"
    3) "Facebook.com"
    4) "Example.com"
    5) "Buddy.com"
    ```

- **TTL** > Redis TTL command is used to get the remaining time of key expiry in seconds.

    **Example**

    ```code
    127.0.0.1:6379> SET key "Apple"
    OK
    127.0.0.1:6379> EXPIRE key 20
    (integer) 1
    127.0.0.1:6379> TTL key
    (integer) 16
    ```

- **TYPE** > Redis TYPE command is used to get the data type of value stored in the key.

    **Example**

    ```code
    127.0.0.1:6379> SET STR "JavaScript"
    OK
    127.0.0.1:6379> TYPE STR
    string
    127.0.0.1:6379> lpush list JAVA
    (integer) 1
    127.0.0.1:6379> TYPE list
    list
    127.0.0.1:6379> SADD set Python
    (integer) 1
    127.0.0.1:6379> TYPE set
    set
    127.0.0.1:6379> HSET hash 0 MySQL
    (integer) 1
    127.0.0.1:6379> TYPE hash
    hash
    ```

- **WAIT** > The Redis WAIT command blocks the current client until all the previous write commands are successfully transferred and acknowledged by at least the specified number of slaves. If the timeout, specified in milliseconds, is reached, the command returns even if the specified number of slaves were not yet reached.

    **Example**

    ```code
    127.0.0.1:6379> SET key1 Apple
    OK
    127.0.0.1:6379> WAIT 2 1
    (integer) 0
    127.0.0.1:6379> WAIT 2 1000
    (integer) 0
    (1.01s)
    ```

# Transaction:

- DISCARD
- EXEC
- MULTI
- UNWATCH
- WATCH

### Transactions Commands:

- **DISCARD** > command is used to flush the all previously queued commands in a transaction and restores the connection state to normal.

    If WATCH was used, DISCARD unwatches all keys watched by the connection.

    **Example**

    ```code
    127.0.0.1:6379> MULTI
    OK
    127.0.0.1:6379> INCR test
    QUEUED
    127.0.0.1:6379> INCR test1
    QUEUED
    127.0.0.1:6379> DISCARD
    OK
    ```

- **EXEC** > Redis EXEC command is used to execute all previously queued commands in a transaction and restores the connection state to normal.

    **Example**

    ```code
    127.0.0.1:6379> MULTI
    OK
    127.0.0.1:6379> INCR test
    QUEUED
    127.0.0.1:6379> INCR test1
    QUEUED
    127.0.0.1:6379> EXEC
    1) (integer) 3
    2) (integer) 3
    ```

- **MULTI** > Redis MULTI command is used to mark the start of a transaction block.

    **Example**

    ```code
    127.0.0.1:6379> MULTI
    OK
    127.0.0.1:6379> INCR test
    QUEUED
    127.0.0.1:6379> INCR test1
    QUEUED
    ```

- **UNWATCH** > Redis UNWATCH command is used to flush all the previously watched keys for a transaction.

- **WATCH** > Redis WATCH command is used to mark the given keys to be watched for conditional execution of a transaction.

    **Example**

    ```code
    127.0.0.1:6379> WATCH zset
    OK
    127.0.0.1:6379> MULTI
    OK
    127.0.0.1:6379> ZREM zset element
    QUEUED
    127.0.0.1:6379> EXEC
    1) (integer) 0
    ```

# Data Type:

Redis is not a plain key-value store, actually, it is a data structures server, supporting a different kind of values. In traditional key-value stores, you associated string keys to string values, in Redis the value is not limited to a simple string, but can also hold more complex data structures. The following is the list of all the data structures supported by Redis: 

- String
- Lists
- Sets
- Sorted Sets
- Hashes

# String

Strings are Redis’ most basic data type. It is the only data type in Memcached, so it is also very natural for newcomers to use it in Redis. Since Redis keys are strings, when we use the string type as a value too, we are mapping a string to another string. The string data type is useful for a number of use cases, like caching HTML fragments or pages. Here are some common commands associated with strings:

- SET: sets a value to a key
- GET: gets a value from a key
- DEL: deletes a key and its value
- INCR: atomically increments a key
- INCRBY: increments a key by a designated values
- EXPIRE: the length of time that a key should exist (denoted in seconds)

### String Commands:

- **SET** > Redis SET command is used to set some string value in redis key. If the key already holds a value, it is overwritten, regardless of its type. Any previous time to live associated with the key is discarded on a successful SET operation.

    **Example**

    ```code
    redis 127.0.0.1:6379> SET key1 "redis"
    OK
    redis 127.0.0.1:6379> SET key2 "redis" EX 60 NX 
    OK
    ```

- **GET** > Redis GET command is used to get the value stored in specified key. If the key does not exist, then nil is returned. If returned value is not a string, then an error is returned.

    **Example**

    ```code
    redis 127.0.0.1:6379> SET example "redis"
    OK
    redis 127.0.0.1:6379> GET example
    "redis"
    ```

- **GETRANGE** > Redis GETRANGE command is used to get the substring of the string value stored at key, determined by the offsets start and end (both are inclusive). Negative offsets can be used in order to provide an offset starting from the end of the string.

    The function handles out of range requests by limiting the resulting range to the actual length of the string.

    **Example**

    ```code
    redis 127.0.0.1:6379> SET mykey "This is my test key"
    OK
    redis 127.0.0.1:6379> GETRANGE mykey 0 3
    "This"
    redis 127.0.0.1:6379> GETRANGE mykey 0 -1
    "This is my test key"
    ```

- **GETSET** > Redis GETSET command automatically set specified string value in redis key and returns its old value. Returns an error when key exists but does not hold a string value.

    **Example**

    ```code
    redis 127.0.0.1:6379> GETSET mynewkey "This is my test key"
    (nil)
    redis 127.0.0.1:6379> GETSET mynewkey "This is my new value to test getset"
    "This is my test key"
    ```

- **GETBIT** > Redis GETBIT command is used to get the bit value at offset in the string value stored at key. When an offset is beyond the string length, the string is assumed to be a contiguous space with 0 bits. When a key does not exist it is assumed to be an empty string, so the offset is always out of range and the value is also assumed to be a contiguous space with 0 bits.

    **Example**

    ```code
    redis 127.0.0.1:6379> SETBIT mykey 7 1
    (integer) 0
    redis 127.0.0.1:6379> GETBIT mykey 0
    (integer) 0
    redis 127.0.0.1:6379> GETBIT mykey 7
    (integer) 1
    redis 127.0.0.1:6379> GETBIT mykey 100
    (integer) 0
    ```

- **MGET** > Redis MGET command is used to get the values of all specified keys. For every key that does not hold a string value or does not exist, the special value nil is returned.

    **Example**

    ```code
    redis 127.0.0.1:6379> SET key1 "hello"
    OK
    redis 127.0.0.1:6379> SET key2 "world"
    OK
    redis 127.0.0.1:6379> MGET key1 key2 someOtherKey
    1) "Hello"
    2) "World"
    3) (nil)
    ```

- **SETBIT** > The Redis SETBIT key is used to sets or clears the bit at offset in the string value stored at key. It is depending on value, which can be either 0 or 1. When the key does not exist, a new string value is created. The string is grown to make sure it can hold a bit at offset. The offset argument is required to be greater than or equal to 0, and smaller than 232.

    **Example**

    ```code
    redis 127.0.0.1:6379> SETBIT mykey 7 1
    (integer) 0
    redis 127.0.0.1:6379> GETBIT mykey 0
    (integer) 0
    redis 127.0.0.1:6379> GETBIT mykey 7
    (integer) 1
    redis 127.0.0.1:6379> GETBIT mykey 100
    (integer) 0
    ```

- **SETEX** > The Redis SETEX command is used to set some string value with specified timeout in seconds in redis key.

    **Example**

    ```code
    redis 127.0.0.1:6379> SETEX mykey 60 redis
    OK
    redis 127.0.0.1:6379> TTL mykey
    60
    redis 127.0.0.1:6379> GET mykey
    "redis
    ```

- **SETNX** > Redis SETNX command is used to set some string value in redis key, if the key does not exist in redis. When key already holds a value, no operation is performed. SETNX is short for "SET if Not eXists".

    **Example**

    ```code
    redis 127.0.0.1:6379> SETNX mykey redis
    (integer) 1
    redis 127.0.0.1:6379> SETNX mykey mongodb
    (integer) 0
    redis 127.0.0.1:6379> GET mykey
    "redis"
    ```

- **SETRANGE** > Redis SETRANGE command is used to overwrite part of a string at key starting at the specified offset for the entire length of the value. If the offset is larger than the current length of the string at the key, the string is padded with zero-bytes to make offset fit. Non-existing keys are considered as empty strings.

    The maximum offset that you can set is 2<sup>29</sup>-1 (536870911), as Redis Strings are limited to 512 megabytes.

    **Example**

    ```code
    redis 127.0.0.1:6379> SET key1 "Hello World"
    OK
    redis 127.0.0.1:6379> SETRANGE key1 6 "Redis"
    (integer) 11
    redis 127.0.0.1:6379> GET key1
    "Hello Redis"
    ```

- **STRLEN** > Redis STRLEN command is used to get the length of the string value stored at key. An error is returned when key holds a non-string value.

    **Example**

    ```code
    redis 127.0.0.1:6379> SET key1 "Redis"
    OK
    redis 127.0.0.1:6379>  STRLEN key1
    (integer) 5
    ```

- **MSET** > Redis MSET command is used to set multiple values to multiple keys. It replaces existing values with new values.

    **Example**

    ```code
    redis 127.0.0.1:6379> MSET key1 "Hello" key2 "World"
    OK
    redis 127.0.0.1:6379> GET key1
    "Hello"
    redis 127.0.0.1:6379> GET key2
    1) "World"
    ```

- **MSETNX** > The Redis MSETNX command is used to set multiple values to multiple keys, only if none of the already exists. If any one from current operation exists in redis then MSETNX does not perform any operation.

    **Example**

    ```code
    redis 127.0.0.1:6379> MSETNX key1 "Hello" key2 "world"
    (integer) 1
    redis 127.0.0.1:6379> MSETNX key2 "worlds" key3 "third key"
    (integer) 0
    redis 127.0.0.1:6379> MGET key1 key2 key3
    1) "Hello"
    2) "world"
    3) (nil)
    ```

- **PSETEX** > Redis PSETEX command is used to set the value of key, with the expiration of time in milliseconds instead of seconds.

    **Example**

    ```code
    redis 127.0.0.1:6379> PSETEX mykey 1000 "Hello"
    OK
    redis 127.0.0.1:6379> PTTL mykey
    999
    redis 127.0.0.1:6379> GET mykey
    1) "Hello"
    ```

- **INCR** > Redis INCR command is used to increment the integer value of a key by one. If the key does not exist, it is set to 0 before performing the operation. An error is returned if the key contains a value of the wrong type or contains a string that can not be represented as an integer. This operation is limited to 64-bit signed integers.

    **Example**

    ```code
    redis 127.0.0.1:6379> SET visitors 1000
    OK
    redis 127.0.0.1:6379> INCR visitors
    (integer) 1001
    redis 127.0.0.1:6379> GET visitors
    (integer) 1001
    ```

- **INCRBY** > Redis INCRBY command is used to increment the number stored at key by specified value. If the key does not exist, it is set to 0 before performing the operation. An error is returned if the key contains a value of the wrong type or contains a string that can not be represented as an integer.

    **Example**

    ```code
    127.0.0.1:6379> SET visitors 1000
    OK
    127.0.0.1:6379> INCRBY visitors 10
    (integer) 1010
    127.0.0.1:6379> GET visitors
    "1010"
    ```

- **INCRBYFLOAT** > Redis INCRBYFLOAT command is used to increments the string representing a floating point number stored at key by the specified increment. If the key does not exist, it is set to 0 before performing the operation. If the key contains a value of the wrong type or current key content or the specified increment are not parsable as floating point number, then an error is returned.

    **Example**

    ```code
    redis 127.0.0.1:6379> SET visitors 1000.20
    OK
    redis 127.0.0.1:6379> INCRBYFLOAT visitors .50
    1000.70
    redis 127.0.0.1:6379> GET visitors
    1000.70
    ```

- **DECR** > Redis DECR command is used to decrement the integer value of a key by one. If the key does not exist, it is set to 0 before performing the operation. If the key contains a value of the wrong type or contains a string that can not be represented as integer an error is returned.

    **Example**

    ```code
    redis 127.0.0.1:6379> SET visitors 1000
    OK
    redis 127.0.0.1:6379> DECR visitors
    (integer) 999
    redis 127.0.0.1:6379> SET visitors "13131312312312312312312rgergerg"
    Ok
    redis 127.0.0.1:6379> DECR visitors
    ERR value is not an integer or out of range
    ```

- **DECRBY** > Redis DECRBY command is used to decrement the number stored at key by specified value. If the key does not exist, it is set to 0 before performing the operation. If the key contains a value of the wrong type or contains a string that can not be represented as integer an error is returned.

    **Example**

    ```code
    redis 127.0.0.1:6379> SET visitors 1000
    OK
    redis 127.0.0.1:6379> DECRBY visitors 5
    (integer) 995
    ```

- **APPEND** > Redis APPEND command is used to add some value in a key. If the key already exists and is a string, this command appends the value at the end of the string. If key does not exist it is created and set as an empty string.

    **Example**

    ```code
    redis 127.0.0.1:6379> SET mykey "hello"
    OK
    redis 127.0.0.1:6379> APPEND mykey "w3resource"
    (integer) 20
    redis 127.0.0.1:6379> GET mykey 
    "hello tutorialspoint"
    ```

# List

Lists in Redis are a collection of ordered values. This is in contrast to Sets which are unordered. Redis lists are implemented via Linked Lists. This means that even if you have millions of elements inside a list, the operation of adding a new element in the head or in the tail of the list is performed in constant time. Here are some common commands associated with lists: 

- LPUSH: Add a value to the begriming of a list
- RPUSH: Add a value to the end of a list
- LPOP: Get and remove the first element in a list
- RPOP: Get and remove the last element in a list
- LREM: Remove elements from a list
- LRANGE: Get a range of elements from a list
- LTRIM: Modifies a list so leave only a specified range

### List Commands:

- **BLPOP** > Redis BLPOP command is used to blocks the connection when there are no elements to pop from any of the given lists or remove and get the first element in a list if available. An element is popped from the head of the first list that is non-empty.

    **Example**

    ```code
    127.0.0.1:6379> RPUSH mycolor1 R G B
    (integer) 3
    127.0.0.1:6379> RPUSH mycolor2 Y O P
    (integer) 3
    127.0.0.1:6379> BLPOP mycolor mycolor1 mycolor2 30
    1) "mycolor1"
    2) "R"
    127.0.0.1:6379> BLPOP mycolor mycolor1 mycolor2 30
    1) "mycolor1"
    2) "G"
    127.0.0.1:6379> BLPOP mycolor mycolor1 mycolor2 30
    1) "mycolor1"
    2) "B"
    127.0.0.1:6379> BLPOP mycolor mycolor1 mycolor2 30
    1) "mycolor2"
    2) "Y"
    127.0.0.1:6379> BLPOP mycolor mycolor1 mycolor2 30
    1) "mycolor2"
    2) "O"
    127.0.0.1:6379> BLPOP mycolor mycolor1 mycolor2 30
    1) "mycolor2"
    2) "P"
    127.0.0.1:6379> BLPOP  mycolor mycolor1 mycolor2 30
    (nil)
    (30.03s)
    ```

- **BRPOP** > Redis BRPOP command is used to block the connection when there are no elements to pop from any of the given lists or remove and get the last element in a list if available. It is a blocking list pop primitive. An element is popped from the tail of the first list that is non-empty.

    **Example**

    ```code
    127.0.0.1:6379> RPUSH mycolor1 R G B
    (integer) 3
    127.0.0.1:6379> RPUSH mycolor2 Y O P
    (integer) 3
    127.0.0.1:6379> BRPOP mycolor mycolor1 mycolor2 30
    1) "mycolor1"
    2) "B"
    127.0.0.1:6379> BRPOP mycolor mycolor1 mycolor2 30
    1) "mycolor1"
    2) "G"
    127.0.0.1:6379> BRPOP mycolor mycolor1 mycolor2 30
    1) "mycolor1"
    2) "R"
    127.0.0.1:6379> BRPOP mycolor mycolor1 mycolor2 30
    1) "mycolor2"
    2) "P"
    127.0.0.1:6379> BRPOP mycolor mycolor1 mycolor2 30
    1) "mycolor2"
    2) "O"
    127.0.0.1:6379> BRPOP mycolor mycolor1 mycolor2 30
    1) "mycolor2"
    2) "Y"
    127.0.0.1:6379> BRPOP mycolor mycolor1 mycolor2 30
    (nil)
    (30.04s)
    ```

- **BRPOPLPUSH** > Redis BRPOPLPUSH command is used to block the connection until another client pushes to it or until a timeout is reached when source is empty. When source contains element then this command Atomically returns and removes the last element (tail) of the list stored at source, and pushes the element at the first element (head) of the list stored at the destination.

    **Example**

    ```code
    127.0.0.1:6379> RPUSH mycolor1 R G B
    (integer) 3
    127.0.0.1:6379> RPUSH mycolor2 Y O P
    (integer) 3
    127.0.0.1:6379> BRPOPLPUSH  mycolor1 mycolor2 100
    "B"
    127.0.0.1:6379> BRPOPLPUSH  mycolor1 mycolor2 100
    "G"
    127.0.0.1:6379> BRPOPLPUSH  mycolor1 mycolor2 100
    "R"
    127.0.0.1:6379> BRPOPLPUSH  mycolor1 mycolor2 100
    (nil)
    (100.06s)
    ```

- **LINDEX** > Redis LINDEX command is used to get the element at index in the list stored at key. The index is zero-based, so 0 means the first element, 1 the second element and so on. Negative indices can be used to designate elements starting at the tail of the list. Here, -1 means the last element, -2 means the penultimate and so forth.

    **Example**

    ```code
    127.0.0.1:6379> LPUSH mycolor1 white black red blue
    (integer) 4
    127.0.0.1:6379> LRANGE mycolor1 0 -1
    1) "blue"
    2) "red"
    3) "black"
    4) "white"
    127.0.0.1:6379> LINDEX mycolor1 0
    "blue"
    127.0.0.1:6379> LINDEX mycolor1 1
    "red"
    127.0.0.1:6379> LINDEX mycolor1 -1
    "white"
    127.0.0.1:6379> LINDEX mycolor1 -2
    "black"
    127.0.0.1:6379> LINDEX mycolor1 6
    (nil)
    ```

- **LINSERT** > Redis LINSERT command inserts value in the list stored at key either before or after the reference value pivot. It is considered an empty list and no operation are performed when the key does not exist. When key exists but does not hold a list value, an error is returned. 

    **Example**

    ```code
    127.0.0.1:6379> LPUSH mycolor1 white black red blue
    (integer) 4
    127.0.0.1:6379> LRANGE mycolor1 0 -1
    1) "blue"
    2) "red"
    3) "black"
    4) "white"
    ```
    ```code
    127.0.0.1:6379> LRANGE mycolor1 0 -1
    1) "blue"
    2) "red"
    3) "black"
    4) "white"
    127.0.0.1:6379> LINSERT mycolor1 after white green
    (integer) 5
    127.0.0.1:6379> LRANGE mycolor1 0 -1
    1) "blue"
    2) "red"
    3) "black"
    4) "white"
    5) "green"
    ```
    ```code
    127.0.0.1:6379> LINSERT mycolor1 after white green
    (integer) 5
    127.0.0.1:6379> LRANGE mycolor1 0 -1
    1) "blue"
    2) "red"
    3) "black"
    4) "white"
    5) "green"
    ```

- **LLEN** > Redis LLEN command returns the length of the list stored at key. If a key does not exist, it is interpreted as an empty list and 0 is returned. An error is returned when the value stored at key is not a list.

    **Example**

    ```code
    127.0.0.1:6379> LPUSH mycolor1 white black red blue
    (integer) 4
    127.0.0.1:6379> LRANGE mycolor1 0 -1
    1) "blue"
    2) "red"
    3) "black"
    4) "white"
    127.0.0.1:6379> LLEN mycolor1
    (integer) 4
    ```

- **LPOP** > Redis LPOP key is used to remove and returns the first element of the list stored at the key.

    **Example**

    ```code
    127.0.0.1:6379> LPUSH mycolor1 white black red blue
    (integer) 4
    127.0.0.1:6379> LRANGE mycolor1 0 -1
    1) "blue"
    2) "red"
    3) "black"
    4) "white"
    127.0.0.1:6379> LPOP mycolor1
    "blue"
    127.0.0.1:6379> LRANGE mycolor1 0 -1
    1) "red"
    2) "black"
    3) "white"
    127.0.0.1:6379> RPOP mycolor1
    "white"
    ```

- **LPUSH** > Redis LPUSH command is used to insert all the specified values at the head of the list stored at key. It is created as an empty list before performing the push operations if tje key does not exist. When key holds a value that is not a list, an error is returned.

    **Example**

    ```code
    127.0.0.1:6379> LPUSH mycolor1 white black
    (integer) 2
    127.0.0.1:6379> LRANGE mycolor1 0 -1
    1) "black"
    2) "white"
    127.0.0.1:6379> LPUSH mycolor1 red blue
    (integer) 4
    127.0.0.1:6379> LRANGE mycolor1 0 -1
    1) "blue"
    2) "red"
    3) "black"
    4) "white"
    ```

- **LPUSHX** > Redis LPUSHX command is used to insert value at the head of the list stored at key, only if a key already exists and holds a list.

    **Example**

    ```code
    127.0.0.1:6379> LPUSH mycolor1 white
    (integer) 1
    127.0.0.1:6379> LPUSHX mycolor1 black
    (integer) 2
    127.0.0.1:6379> LPUSHX mycolor1 red
    (integer) 3
    127.0.0.1:6379> LPUSHX mycolor2 blue
    (integer) 0

    The key mycolor2 not exists, so return 0

    127.0.0.1:6379> EXISTS mycolor2
    (integer) 0
    127.0.0.1:6379> LRANGE mycolor1 0 -1
    1) "red"
    2) "black"
    3) "white"
    ```

- **LREM** > Redis LREM command is used to remove the first count occurrences of elements equal to the value from the list stored at key. The count argument influences the operation describes as below:
    - count > 0: Remove elements equal to value moving from head to tail.
    - count < 0: Remove elements equal to value moving from tail to head.
    - count = 0: Remove all elements equal to value.

    **Example**

    ```code
    127.0.0.1:6379> LPUSH mycolor red red red green
    (integer) 4
    127.0.0.1:6379> LRANGE mycolor 0 -1
    1) "green"
    2) "red"
    3) "red"
    4) "red"
    127.0.0.1:6379> LREM mycolor 1 red
    (integer) 1
    127.0.0.1:6379> LRANGE mycolor 0 -1
    1) "green"
    2) "red"
    3) "red"
    127.0.0.1:6379> LREM mycolor 0 red
    (integer) 2
    127.0.0.1:6379> LRANGE mycolor 0 -1
    1) "green"
    ```
    ```code
    127.0.0.1:6379> LPUSH mycolor red red red green
    (integer) 4
    127.0.0.1:6379> LRANGE mycolor 0 -1
    1) "green"
    2) "red"
    3) "red"
    4) "red"
    127.0.0.1:6379> LREM mycolor -2 red
    (integer) 2
    127.0.0.1:6379> LRANGE mycolor 0 -1
    1) "green"
    2) "red"
    ```

- **LSET** > Redis LSET command is used to set the list element at index to value. The index is zero-based, so 0 means the first element, 1 the second element and so on. Negative indices can be used to designate elements starting at the tail of the list.

    **Example**

    ```code
    127.0.0.1:6379> LPUSH mycolor1 white black red blue
    (integer) 4
    127.0.0.1:6379> LRANGE mycolor1 0 -1
    1) "blue"
    2) "red"
    3) "black"
    4) "white"
    127.0.0.1:6379> LSET mycolor1 2 YELLOW
    OK
    127.0.0.1:6379> LSET mycolor1 -1 GREEN
    OK
    127.0.0.1:6379> LRANGE mycolor1 0 -1
    1) "blue"
    2) "red"
    3) "YELLOW"
    4) "GREEN"

    127.0.0.1:6379> LSET mycolor 2 YELLOW
    (error) ERR index out of range
    ```

- **LRANGE** > Redis LRANGE command is used to return the specified elements of the list stored at key. The offsets start and stop are zero-based indexes, with 0 being the first element of the list, 1 being the next element and so on.

    **Example**

    ```code
    127.0.0.1:6379> LPUSH mycolor1 white black red blue
    (integer) 4
    127.0.0.1:6379> LRANGE mycolor1 0 -1
    1) "blue"
    2) "red"
    3) "black"
    4) "white"
    127.0.0.1:6379> LRANGE mycolor1 0 1
    1) "blue"
    2) "red"
    127.0.0.1:6379> LRANGE mycolor1 2 -1
    1) "black"
    2) "white"
    ```
    ```code
    127.0.0.1:6379> LRANGE mycolor1 -2 -1
    1) "black"
    2) "white"
    127.0.0.1:6379> LRANGE mycolor1 -4 -3
    1) "blue"
    2) "red"
    127.0.0.1:6379> LRANGE mycolor1 0 -3
    1) "blue"
    2) "red"
    ```
    ```code
    127.0.0.1:6379> LRANGE mycolor1 3 20
    1) "white"
    127.0.0.1:6379> LRANGE mycolor1 -20 -10
    (empty list or set)
    ```

- **LTRIM** > Redis LTRIM command is used to trim an existing list so that it will contain only the specified range of elements specified. Both start and stop are zero-based indexes, where 0 is the first element of the list (the head), 1 the next element and so on.

    **Example**

    ```code
    127.0.0.1:6379> LPUSH mycolor1 white black red blue
    (integer) 4
    127.0.0.1:6379> LRANGE mycolor1 0 -1
    1) "blue"
    2) "red"
    3) "black"
    4) "white"
    127.0.0.1:6379> LTRIM mycolor1 1 2
    OK
    127.0.0.1:6379> LRANGE mycolor1 0 -1
    1) "red"
    2) "black"
    ```
    ```code
    127.0.0.1:6379> LPUSH mycolor1 white black red blue
    (integer) 4
    127.0.0.1:6379> LTRIM mycolor1 -2 -1
    OK
    127.0.0.1:6379> LRANGE mycolor1 0 -1
    1) "black"
    2) "white"
    ```
    ```code
    127.0.0.1:6379> LPUSH mycolor1 white black red blue
    (integer) 4
    127.0.0.1:6379> LTRIM mycolor1 10 10
    OK
    127.0.0.1:6379> LRANGE mycolor1 0 -1
    (empty list or set)
    ```

- **RPOP** > Redis RPOP command is used to remove and returns the last element of the list stored at key.

    **Example**

    ```code
    127.0.0.1:6379> LPUSH mycolor1 white black red blue
    (integer) 4
    127.0.0.1:6379> LRANGE mycolor1 0 -1
    1) "blue"
    2) "red"
    3) "black"
    4) "white"
    127.0.0.1:6379> RPOP mycolor1
    "white"
    127.0.0.1:6379> LRANGE mycolor1 0 -1
    1) "blue"
    2) "red"
    3) "black"
    127.0.0.1:6379> RPOP mycolor1
    "black"
    127.0.0.1:6379> LPOP mycolor1
    "blue"
    ```

- **RPOPLPUSH** > Redis RPOPLPUSH command is used to return and remove the last element of the list stored at the source, and pushes the element at the first element of the list stored at the destination.

    **Example**

    ```code
    127.0.0.1:6379> LPUSH mycolor1 white black red blue
    (integer) 4
    127.0.0.1:6379> LPUSH mycolor2 GREEN ORANGE YELLOW PINK
    (integer) 4
    127.0.0.1:6379> RPOPLPUSH mycolor1 mycolor2
    "white"
    127.0.0.1:6379> LRANGE mycolor1 0 -1
    1) "blue"
    2) "red"
    3) "black"
    127.0.0.1:6379> LRANGE mycolor2 0 -1
    1) "white"
    2) "PINK"
    3) "YELLOW"
    4) "ORANGE"
    5) "GREEN"
    ```
    ```code
    127.0.0.1:6379> LPUSH mycolor1 white black red blue
    (integer) 4
    127.0.0.1:6379> LRANGE mycolor1 0 -1
    1) "blue"
    2) "red"
    3) "black"
    4) "white"
    127.0.0.1:6379> RPOPLPUSH mycolor1 mycolor1
    "white"
    127.0.0.1:6379> LRANGE mycolor1 0 -1
    1) "white"
    2) "blue"
    3) "red"
    4) "black"
    ```

- **RPUSH** > Redis RPUSH command is used to insert all the specified values at the tail of the list stored at key. It is created as an empty list before performing the push operation if the key does not exist. An error is returned when key holds such a value that is not a list.

    **Example**

    ```code
    127.0.0.1:6379> RPUSH mycolor1 white black
    (integer) 2
    127.0.0.1:6379> RPUSH mycolor1 red blue
    (integer) 4
    127.0.0.1:6379> LRANGE mycolor1 0 -1
    1) "white"
    2) "black"
    3) "red"
    4) "blue"
    ```

- **RPUSHX** > Redis RPUSHX command is used to insert a value at the tail of the list stored at key, only if a key already exists and holds a list. In contrary to RPUSH, no operation will be performed when a key does not yet exist.

    **Example**

    ```code
    127.0.0.1:6379> del mycolor1
    (integer) 1
    127.0.0.1:6379> RPUSH mycolor1 white
    (integer) 1
    127.0.0.1:6379> RPUSHX mycolor1 black
    (integer) 2
    127.0.0.1:6379> RPUSHX mycolor1 red
    (integer) 3
    127.0.0.1:6379> RPUSHX mycolor2 blue
    (integer) 0

    The key mycolor2 not exists, so return 0

    127.0.0.1:6379> EXISTS mycolor2
    (integer) 0
    127.0.0.1:6379> LRANGE mycolor1 0 -1
    1) "white"
    2) "black"
    3) "red"
    ```

# Sets

Redis Sets are unordered collections of strings. If you want to combine strings, you can do that with REDIS sets. Here are some common commands associated with sets:

- SADD: Add one or members to a set
- SMEMBERS: Get all set members
- SINTER: Find the intersection of multiple sets
- SISMEMBER: check if a value is in a set
- SRANDMEMBER: Get a random set member

### Sets Commands:

- **SADD** > Redis SADD command is used to add members to set stored at key. If the member already exists, then it is ignored. If the key does not exist, then a new set is created and members are added into it. If the value stored at key if not set, then an error is returned.

    **Example**

    ```code
    127.0.0.1:6379> SADD mycolor "White"
    (integer) 1
    127.0.0.1:6379> SADD mycolor "Yellow" "Green"
    (integer) 2
    127.0.0.1:6379> SADD mycolor "Red" "Blue" "Orange"
    (integer) 3
    127.0.0.1:6379> SMEMBERS mycolor
    1) "Yellow"
    2) "White"
    3) "Blue"
    4) "Green"
    5) "Red"
    6) "Orange"
    ```
    ```code
    127.0.0.1:6379> SADD mycolor "Orange"
    (integer) 0
    127.0.0.1:6379> SADD mycolor "Orange" "Pink"
    (integer) 1
    127.0.0.1:6379> SMEMBERS mycolor
    1) "Blue"
    2) "White"
    3) "Yellow"
    4) "Pink"
    5) "Green"
    6) "Red"
    7) "Orange"
    ```

- **SCARD** > Redis SCARD command is used to return the number of elements stored in the set.

    **Example**

    ```code
    127.0.0.1:6379> SADD mycolor "Red" "Green"
    (integer) 2
    127.0.0.1:6379> SADD mycolor "Blue" "Yellow"
    (integer) 2
    127.0.0.1:6379> SCARD mycolor
    (integer) 4
    ```

- **SDIFF** > Redis SDIFF command is used to return the members of the set resulting from the difference between the first set and all the successive sets. If keys do not exist in redis then it is considered as empty sets.

    **Example**

    ```code
    127.0.0.1:6379> SADD mycolor1 R G B
    (integer) 3
    127.0.0.1:6379> SADD mycolor2  G Y
    (integer) 2
    127.0.0.1:6379> sdiff mycolor1 mycolor2
    1) "R"
    2) "B"
    127.0.0.1:6379> SADD mycolor3  B P
    (integer) 2
    127.0.0.1:6379> sdiff mycolor1 mycolor2 mycolor3
    1) "R"
    ```

- **SDIFFSTORE** > Redis SDIFFSTORE command store the members of the set, resulting from the difference between the first set and all the successive sets, into the specified key. If the specified key already exists, it is overwritten.

    **Example**

    ```code
    127.0.0.1:6379> SADD mycolor1 R G B
    (integer) 3
    127.0.0.1:6379> SADD mycolor2 G Y
    (integer) 2
    127.0.0.1:6379> SDIFFSTORE dest_key mycolor1 mycolor2
    (integer) 2
    127.0.0.1:6379> SMEMBERS dest_key
    1) "R"
    2) "B"
    ```
    ```code
    127.0.0.1:6379> SADD mycolor3 B P
    (integer) 2
    127.0.0.1:6379> SDIFFSTORE dest_key mycolor1 mycolor2 mycolor3
    (integer) 1
    127.0.0.1:6379> SMEMBERS dest_key
    1) "R"
    ```

- **SINTER** > Redis SINTER command is used to return the members of the set resulting from the intersection of all the specified sets. Keys that do not exist are considered to be empty sets and if one of the keys being an empty set, the resulting set is also empty.

    **Example**

    ```code
    127.0.0.1:6379> SADD mycolor1 R G B
    (integer) 3
    127.0.0.1:6379> SADD mycolor2 G B Y
    (integer) 3
    127.0.0.1:6379> SINTER mycolor1 mycolor2
    1) "G"
    2) "B"
    ```
    ```code
    127.0.0.1:6379> SADD mycolor3 B W O
    (integer) 3
    127.0.0.1:6379> SINTER mycolor1 mycolor2 mycolor3
    1) "B"
    ```

- **SINTERSTORE** > Redis SINTERSTORE command is used to store the members of the set resulting from the intersection of all the specified sets in a specific key. Keys that do not exist are considered to be empty sets and if one of the keys being an empty set, the resulting set is also empty.

    **Example**

    ```code
    127.0.0.1:6379> SADD mycolor1 R G B
    (integer) 3
    127.0.0.1:6379> SADD mycolor2 G B Y
    (integer) 3
    127.0.0.1:6379> SINTERSTORE dest_key mycolor1 mycolor2
    (integer) 2
    127.0.0.1:6379> SMEMBERS dest_key
    1) "G"
    2) "B"
    ```
    ```code
    127.0.0.1:6379> SADD mycolor3 B W O
    (integer) 3
    127.0.0.1:6379> SINTERSTORE dest_key mycolor1 mycolor2 mycolor3
    (integer) 1
    127.0.0.1:6379> SMEMBERS dest_key
    1) "B"
    ```

- **SISMEMBER** > Redis SISMEMBER command is used to return the member, which is the member of the set stored at key.

    **Example**

    ```code
    127.0.0.1:6379> SADD mycolor "red" "green" "blue"
    (integer) 3
    127.0.0.1:6379> SISMEMBER mycolor "green"
    (integer) 1
    127.0.0.1:6379> SISMEMBER mycolor "orange"
    (integer) 0
    ```

- **SMEMBERS** > Redis SMEMBERS command is used to return all the members of the set value stored at key.

    **Example**

    ```code
    127.0.0.1:6379> SADD mycolor1 "red" "green" "blue"
    (integer) 3
    127.0.0.1:6379> SADD mycolor1 "orange" "yellow"
    (integer) 2
    127.0.0.1:6379> SMEMBERS  mycolor1
    1) "blue"
    2) "orange"
    3) "green"
    4) "red"
    5) "yellow"
    ```

- **SMOVE** > Redis SMOVE command is used to move a member from one set to another set. If the source set does not exist or does not contain the specified element, no operation is performed and 0 is returned. Otherwise, the element is removed from the source set and added to the destination set. When the specified element already exists in the destination set, it is only removed from the source set. An error is returned if source or destination does not hold a set value.

    **Example**

    ```code
    127.0.0.1:6379> SADD source_key 1 2 3
    (integer) 3
    127.0.0.1:6379> SADD dest_key 4
    (integer) 1
    127.0.0.1:6379> SMOVE source_key dest_key 1
    (integer) 1
    127.0.0.1:6379> SMEMBERS source_key
    1) "2"
    2) "3"
    127.0.0.1:6379> SMEMBERS dest_key
    1) "1"
    2) "4"
    ```
    ```code
    127.0.0.1:6379> DEL dest_key
    (integer) 1
    127.0.0.1:6379> SMOVE source_key dest_key 2
    (integer) 1
    127.0.0.1:6379> SMEMBERS source_key
    1) "3"
    127.0.0.1:6379> SMEMBERS dest_key
    1) "2"
    ```
    ```code
    127.0.0.1:6379> SMOVE source_key dest_key 9
    (integer) 0
    127.0.0.1:6379> SMOVE sources_key dest_key 9
    (integer) 0
    ```

- **SPOP** > Redis SPOP command is used to remove and return one or more random member from set stored at specified key.

    **Example**

    ```code
    127.0.0.1:6379> SADD mycolor1 "red" "green" "blue"
    (integer) 3
    127.0.0.1:6379> SPOP mycolor1
    "red"
    127.0.0.1:6379> SMEMBERS mycolor1
    1) "green"
    2) "blue"
    127.0.0.1:6379> SPOP mycolor1
    "green"
    127.0.0.1:6379> SMEMBERS mycolor1
    1) "blue"
    ```

- **SRANDMEMBER** > Redis SRANDMEMBER command is used to return a random element from the set value stored at key when called with just the key argument. When called with the additional count argument, return an array of count distinct elements if  positive and if called with a negative, the behavior changes and the command is allowed to return the same element multiple times. In this case, the number of returned elements is the absolute value of the specified count.

    **Example**

    ```code
    127.0.0.1:6379> SADD mycolor "red" "green" "blue" "yellow"
    (integer) 4
    127.0.0.1:6379> SRANDMEMBER mycolor
    "blue"
    127.0.0.1:6379> SRANDMEMBER mycolor
    "yellow"
    ```
    ```code
    127.0.0.1:6379> SRANDMEMBER mycolor 3
    1) "red"
    2) "green"
    3) "yellow"
    127.0.0.1:6379> SRANDMEMBER mycolor -3
    1) "green"
    2) "blue"
    3) "yellow"
    ```
    ```code
    127.0.0.1:6379> SRANDMEMBER mycolor 6
    1) "red"
    2) "green"
    3) "yellow"
    4) "blue"
    127.0.0.1:6379> SRANDMEMBER mycolor -6
    1) "blue"
    2) "red"
    3) "red"
    4) "blue"
    5) "blue"
    6) "red"
    ```

- **SREM** > Redis SREM command is used to remove the specified members from the set stored at key. Specified members that are not a member of this set are ignored. If the key does not exist, it is treated as an empty set and this command returns 0. An error is returned when the value stored at key is not a set.

    **Example**

    ```code
    127.0.0.1:6379> SADD mycolor "red" "green" "blue" "yellow"
    (integer) 4
    127.0.0.1:6379> SREM mycolor "yellow"
    (integer) 1
    127.0.0.1:6379> SMEMBERS mycolor
    1) "red"
    2) "green"
    3) "blue"
    127.0.0.1:6379> SREM mycolor "orange"
    (integer) 0
    ```

- **SUNION** > Redis SUNION command is used to get the members of the set resulting from the union of all the given sets. Keys that do not exist are considered to be empty sets.

    **Example**

    ```code
    127.0.0.1:6379> SADD mycolor1 R G B
    (integer) 3
    127.0.0.1:6379> SADD mycolor2 G B Y
    (integer) 3
    127.0.0.1:6379> SUNION mycolor1 mycolor2
    1) "G"
    2) "B"
    3) "Y"
    4) "R"
    ```
    ```code
    127.0.0.1:6379> SADD mycolor3 B O P
    (integer) 3
    127.0.0.1:6379> SUNION mycolor1 mycolor2 mycolor3
    1) "Y"
    2) "R"
    3) "O"
    4) "P"
    5) "G"
    6) "B"
    ```

- **SUNIONSTORE** > Redis SUNIONSTORE command is used to store, the members of the set resulting from the union of all the given sets in a specific key. Keys that do not exist are considered to be empty sets and overwritten if specified key already exists.

    **Example**

    ```code
    127.0.0.1:6379> SADD mycolor1 R G B
    (integer) 3
    127.0.0.1:6379> SADD mycolor2 G B Y
    (integer) 3
    127.0.0.1:6379> SUNIONSTORE dest_key mycolor1 mycolor2
    (integer) 4
    127.0.0.1:6379> SMEMBERS dest_key
    1) "G"
    2) "B"
    3) "Y"
    4) "R"
    ```
    ```code
    127.0.0.1:6379> SADD mycolor3 B O P
    (integer) 3
    127.0.0.1:6379> SUNIONSTORE dest_key mycolor1 mycolor2 mycolor3
    (integer) 6
    127.0.0.1:6379> SMEMBERS dest_key
    1) "Y"
    2) "R"
    3) "O"
    4) "P"
    5) "G"
    6) "B"
    ```
    ```code
    127.0.0.1:6379> SUNIONSTORE mycolor1 mycolor1 mycolor2 mycolor3
    (integer) 6
    127.0.0.1:6379> SMEMBERS mycolor1
    1) "Y"
    2) "R"
    3) "O"
    4) "P"
    5) "G"
    6) "B"
    ```

- **SSCAN** > The Redis SSCAN command  is used in order to incrementally iterate over a collection of elements.

    **Example**

    ```code
    127.0.0.1:6379> SADD mytestset M1 M2 M3 N1 N2 N3 O1 O2 O3 P1 P2 P3 Q1 Q2 Q3
    (integer) 15
    127.0.0.1:6379> SSCAN mytestset 0
    1) "3"
    2)  1) "O2"
        2) "Q3"
        3) "P1"
        4) "O3"
        5) "M1"
        6) "N1"
        7) "P2"
        8) "N3"
        9) "Q2"
    10) "N2"
    11) "O1"
    127.0.0.1:6379> SSCAN mytestset 3
    1) "0"
    2) 1) "M2"
    2) "P3"
    3) "M3"
    4) "Q1"
    ```
    ```code
    127.0.0.1:6379> SSCAN mytestset 0 COUNT 18
    1) "0"
    2)  1) "O2"
        2) "Q3"
        3) "P1"
        4) "O3"
        5) "M1"
        6) "N1"
        7) "P2"
        8) "N3"
        9) "Q2"
    10) "N2"
    11) "O1"
    12) "M2"
    13) "P3"
    14) "M3"
    15) "Q1"
    ```
    ```code
    127.0.0.1:6379> SSCAN mytestset 0 MATCH N*
    1) "3"
    2) 1) "N1"
    2) "N3"
    3) "N2"
    127.0.0.1:6379> SSCAN mytestset 0 MATCH *3*
    1) "3"
    2) 1) "Q3"
    2) "O3"
    3) "N3"
    127.0.0.1:6379> SSCAN mytestset 0 MATCH *3* COUNT 20
    1) "0"
    2) 1) "Q3"
    2) "O3"
    3) "N3"
    4) "P3"
    5) "M3"
    ```

# Sorted Sets

Sorted sets are a data type which is similar to a mix between a Set and a Hash. Like sets, sorted sets are composed of unique, non-repeating string elements, so in some sense, a sorted set is a set as well.

However, while elements inside sets are not ordered, every element in a sorted set is associated with a floating point value, called the score (this is why the type is also similar to a hash, since every element is mapped to a value). Here are some common commands associated with sorted sets : 

- ZADD: Adds members to a sorted set
- ZRANGE: Displays the members of a sorted set arranged by index (with the default low to high)
- ZREVRANGE: Displays the members of a sorted set arranged by index (from high to low)
- ZREM: Removes members from a sorted set

### Sorted Sets Commands:

- **ZADD** > Redis ZADD command is used to add all the specified members with the specified scores to the sorted set stored at key. If a specified member is an existing member of the stored set, the score is updated and the element reinserted at the right position to ensure the correct ordering. A new sorted set with the specified members as sole members is created, when key dies not exists or the sorted set was empty. If the key exists but does not hold a sorted set, an error is returned.

    **Example**

    ```code
    127.0.0.1:6379> ZADD mycolorset 1 white
    (integer) 1
    127.0.0.1:6379> ZADD mycolorset 2 black
    (integer) 1
    127.0.0.1:6379> ZADD mycolorset 3 red
    (integer) 1
    127.0.0.1:6379> ZRANGE mycolorset 0 -1
    1) "white"
    2) "black"
    3) "red"
    127.0.0.1:6379> ZRANGE mycolorset 0 -1 WITHSCORES
    1) "white"
    2) "1"
    3) "black"
    4) "2"
    5) "red"
    6) "3"
    ```
    ```code
    127.0.0.1:6379> ZADD mycolorset 4 blue 5 green
    (integer) 2
    127.0.0.1:6379> ZRANGE mycolorset 0 -1 WITHSCORES
    1) "white"
    2) "1"
    3) "black"
    4) "2"
    5) "red"
    6) "3"
    7) "blue"
    8) "4"
    9) "green"
    10) "5"
    ```
    ```code
    127.0.0.1:6379> ZADD mycolorset 1 white 1 black 1 red 1 blue 1 green
    (integer) 5
    127.0.0.1:6379> ZRANGE mycolorset 0 -1 WITHSCORES
    1) "black"
    2) "1"
    3) "blue"
    4) "1"
    5) "green"
    6) "1"
    7) "red"
    8) "1"
    9) "white"
    10) "1"
    ```
    ```code
    127.0.0.1:6379> ZADD mycolorset 1 orange
    (integer) 1
    127.0.0.1:6379> ZRANGE mycolorset 0 -1 WITHSCORES
    1) "black"
    2) "1"
    3) "blue"
    4) "1"
    5) "green"
    6) "1"
    7) "orange"
    8) "1"
    9) "red"
    10) "1"
    11) "white"
    12) "1"
    ```

- **ZCARD** > Redis ZCARD command is used to return the number of elements stored in the set at specified key.

    **Example**

    ```code
    127.0.0.1:6379> ZADD mycolorset 10 white 12 black 14 red 16 blue
    (integer) 4
    127.0.0.1:6379> ZADD mycolorset 18 green 20 orange 22 pink 24 yellow
    (integer) 4
    127.0.0.1:6379> ZCARD mycolorset
    (integer) 8
    ```

- **ZCOUNT** > Redis ZCOUNT command is used to return the number of elements in the sorted set at the key with a score between minimum and maximum.

    **Example**

    ```code
    127.0.0.1:6379> ZADD mycolorset  528 white  2514 black 850 red 128 pink 742 yellow
    (integer) 5
    127.0.0.1:6379> ZADD mycolorset  158 orange 1500 green 645 blue 426 gray
    (integer) 4
    127.0.0.1:6379> ZCOUNT mycolorset -inf +inf
    (integer) 9
    ```
    ```code
    127.0.0.1:6379> ZADD mycolorset  528 white  2514 black 850 red 128 pink 742 yellow
    (integer) 5
    127.0.0.1:6379> ZADD mycolorset  158 orange 1500 green 645 blue 426 gray
    (integer) 4
    127.0.0.1:6379> ZCOUNT mycolorset 100 500
    (integer) 3   
    ```
    ```code
    127.0.0.1:6379> ZADD mycolorset  528 white  2514 black 850 red 128 pink 742 yellow
    (integer) 5
    127.0.0.1:6379> ZADD mycolorset  158 orange 1500 green 645 blue 426 gray
    (integer) 4
    127.0.0.1:6379> ZCOUNT mycolorset -inf 300
    (integer) 2
    ```

- **ZINCRBY** > Redis ZINCRBY command is used to increment the score or member in the sorted set stored at key by an incremental value. If no member exists in the sorted set, it is added with incremental value as specified. If the  key does not exist, a new sorted set with the specified member as its sole member is created. When key exists but does not hold a sorted set an error will return.

    **Example**

    ```code
    127.0.0.1:6379> ZADD myvisit 1200 facebook.com 1800 google.com 1500 stackoverflow.com
    (integer) 3
    127.0.0.1:6379> ZREVRANGE myvisit 0 -1 WITHSCORES
    1) "google.com"
    2) "1800"
    3) "stackoverflow.com"
    4) "1500"
    5) "facebook.com"
    6) "1200"

    The facebook.com increased by 500

    127.0.0.1:6379> ZINCRBY myvisit 500 facebook.com
    "1700"
    127.0.0.1:6379> ZREVRANGE myvisit 0 -1 WITHSCORES
    1) "google.com"
    2) "1800"
    3) "facebook.com"
    4) "1700"
    5) "stackoverflow.com"
    6) "1500"

    The google.com decreased by 200

    127.0.0.1:6379> ZINCRBY myvisit -200 google.com
    "1600"

    The rank has changed

    127.0.0.1:6379> ZREVRANGE myvisit 0 -1 WITHSCORES
    1) "facebook.com"
    2) "1700"
    3) "google.com"
    4) "1600"
    5) "stackoverflow.com"
    6) "1500"
    ```
    ```code
    127.0.0.1:6379> ZADD myliteracy 8.3 Canada 6.7 Brazil 5.1 India 4.2 Koria 7.9 Japan
    (integer) 5
    127.0.0.1:6379> ZREVRANGE myliteracy 0 -1 WITHSCORES
    1) "Canada"
    2) "8.3000000000000007"
    3) "Japan"
    4) "7.9000000000000004"
    5) "Brazil"
    6) "6.7000000000000002"
    7) "India"
    8) "5.0999999999999996"
    9) "Koria"
    10) "4.2000000000000002"
    127.0.0.1:6379> ZINCRBY myliteracy 1.7 India
    "6.7999999999999998"
    ```
    ```code
    127.0.0.1:6379> ZINCRBY dailyviewers 1 450205
    "1"
    127.0.0.1:6379> ZINCRBY dailyviewers 1 450205
    "2"
    127.0.0.1:6379> ZINCRBY dailyviewers 1 450205
    "3"
    127.0.0.1:6379> ZINCRBY dailyviewers 1 450306
    "1"
    127.0.0.1:6379> ZINCRBY dailyviewers 1 450306
    "2"
    127.0.0.1:6379> ZREVRANGE dailyviewers 0 -1 WITHSCORES
    1) "450205"
    2) "3"
    3) "450306"
    4) "2"
    ```

- **ZINTERSTORE** > Redis ZINTERSTORE command is used to compute the intersection of specified number of input keys sorted sets given by the specified keys, and stores the result in a specified key.

    **Example**

    ```code
    127.0.0.1:6379> ZADD srcset1 5 M 6 N 7 O
    (integer) 3
    127.0.0.1:6379> ZADD srcset2 3 N 2 O 4 P
    (integer) 3
    127.0.0.1:6379> ZINTERSTORE desset 2 srcset1 srcset2
    (integer) 2
    127.0.0.1:6379> ZRANGE desset 0 -1
    1) "N"
    2) "O"
    127.0.0.1:6379> ZRANGE desset 0 -1 WITHSCORE
    (error) ERR syntax error
    127.0.0.1:6379> ZRANGE desset 0 -1 WITHSCORES
    1) "N"
    2) "9"
    3) "O"
    4) "9"
    ```
    ```code
    127.0.0.1:6379> ZADD srcset1 5 M 6 N 7 O
    (integer) 3
    127.0.0.1:6379> ZADD srcset2 3 N 2 O 4 P
    (integer) 3
    127.0.0.1:6379> ZINTERSTORE desset 2 srcset1 srcset2 WEIGHTS 2 3
    (integer) 2
    127.0.0.1:6379> ZRANGE desset 0 -1 WITHSCORES
    1) "O"
    2) "20"
    3) "N"
    4) "21"
    ```
    ```code
    127.0.0.1:6379> ZADD srcset1 5 M 6 N 7 O
    (integer) 3
    127.0.0.1:6379> ZADD srcset2 3 N 2 O 4 P
    (integer) 3
    127.0.0.1:6379> ZINTERSTORE desset 2 srcset1 srcset2 AGGREGATE MIN
    (integer) 2
    127.0.0.1:6379> ZRANGE desset 0 -1 WITHSCORES
    1) "O"
    2) "2"
    3) "N"
    4) "3"
    127.0.0.1:6379> ZINTERSTORE desset 2 srcset1 srcset2 AGGREGATE MAX
    (integer) 2
    127.0.0.1:6379> ZRANGE desset 0 -1 WITHSCORES
    1) "N"
    2) "6"
    3) "O"
    4) "7"
    ```
    ```code
    127.0.0.1:6379> ZADD srcset1 5 M 6 N 7 O
    (integer) 3
    127.0.0.1:6379> ZADD srcset2 3 N 2 O 4 P
    (integer) 3
    127.0.0.1:6379> ZADD srcset3 1 O 2 P 3 Q
    (integer) 3
    127.0.0.1:6379> ZINTERSTORE desset 3 srcset1 srcset2 srcset3
    (integer) 1
    127.0.0.1:6379> ZRANGE desset 0 -1 WITHSCORES
    1) "O"
    2) "10"
    ```

- **ZLEXCOUNT** > Redis ZLEXCOUNT command is used to return the number of elements in the sorted set at the key with a value between minimum and maximum when all the elements in a sorted set are inserted with the same score, in order to force lexicographical ordering.
    - min, max are the range of the member. To view all the -, + use.
    - min, if you give the max value must be used '[' or '(' in the front. 
    - '[' Is used to contain a value, and '(' is used to block out. 

    **Example**

    ```code
    127.0.0.1:6379> ZADD mycityset 1 Delhi 1 Mumbai 1 London 1 Paris 1 Tokyo
    (integer) 5
    127.0.0.1:6379> ZRANGEBYLEX mycityset - +
    1) "Delhi"
    2) "London"
    3) "Mumbai"
    4) "Paris"
    5) "Tokyo"
    127.0.0.1:6379> ZLEXCOUNT mycityset "[London" +
    (integer) 4
    127.0.0.1:6379> ZLEXCOUNT mycityset "(London" +
    (integer) 3
    127.0.0.1:6379> ZLEXCOUNT mycityset "(London" "(Paris"
    (integer) 1
    127.0.0.1:6379> ZLEXCOUNT mycityset "[London" "(Paris"
    (integer) 2
    ```

- **ZRANGE** > Redis ZRANGE command is used to return the specified range of elements in the sorted set stored at key. The elements are considered to be ordered from the lowest to the highest score. Lexicographical order is used for elements with equal score.

    Both start and stop are zero-based indexes,

    - 0 is the first element,
    - 1 is the next element and so on.

    They can also be negative numbers indicating offsets from the end of the sorted set,

    - -1 being the last element of the sorted set
    - -2 the penultimate element and so on.

    **Example**

    ```code
    127.0.0.1:6379> ZADD mycity 1 Delhi 2 London 3 Paris 4 Tokyo 5 NewYork 6 Seoul
    (integer) 6
    127.0.0.1:6379> ZRANGE mycity 0 -1
    1) "Delhi"
    2) "London"
    3) "Paris"
    4) "Tokyo"
    5) "NewYork"
    6) "Seoul"
    127.0.0.1:6379> ZRANGE mycity 0 -1 WITHSCORES
    1) "Delhi"
    2) "1"
    3) "London"
    4) "2"
    5) "Paris"
    6) "3"
    7) "Tokyo"
    8) "4"
    9) "NewYork"
    10) "5"
    11) "Seoul"
    12) "6"
    ```

- **ZRANGEBYLEX** > Redis ZRANGEBYLEX command is used to return all the elements in the sorted set at the key with a value between minimum and maximum, when all the elements in a sorted set are inserted with the same score, in order to force lexicographical ordering.

    - min, max are the range of the member. To view all the -, + use.
    - min, if you give the max value must be used '[' or '(' in the front. 
    - '[' Is used to contain a value, and '(' is used to block out. 

    Both start and stop are zero-based indexes,

    - 0 is the first element,
    - 1 is the next element and so on.

    They can also be negative numbers indicating offsets from the end of the sorted set,

    - -1 being the last element of the sorted set
    - -2 the penultimate element and so on.

    **Example**

    ```code
    127.0.0.1:6379> ZADD mycity 1 Delhi 2 London 3 Paris 4 Tokyo 5 NewYork 6 Seoul
    (integer) 6
    127.0.0.1:6379> ZRANGE mycity 0 -1
    1) "Delhi"
    2) "London"
    3) "Paris"
    4) "Tokyo"
    5) "NewYork"
    6) "Seoul"
    127.0.0.1:6379> ZRANGEBYLEX mycity - +
    1) "Delhi"
    2) "London"
    3) "Paris"
    4) "Tokyo"
    5) "NewYork"
    6) "Seoul"
    127.0.0.1:6379> ZRANGEBYLEX mycity "[London" +
    1) "London"
    2) "Paris"
    3) "Tokyo"
    4) "NewYork"
    5) "Seoul"
    127.0.0.1:6379> ZRANGEBYLEX mycity "(London" +
    1) "Paris"
    2) "Tokyo"
    3) "NewYork"
    4) "Seoul"
    127.0.0.1:6379> ZRANGEBYLEX mycity "(London" "(Seoul"
    1) "Paris"
    ```
    ```code
    127.0.0.1:6379> ZADD mycity 1 Delhi 2 London 3 Paris 4 Tokyo 5 NewYork 6 Seoul
    (integer) 6
    127.0.0.1:6379> ZRANGE mycity 0 -1
    1) "Delhi"
    2) "London"
    3) "Paris"
    4) "Tokyo"
    5) "NewYork"
    6) "Seoul"
    127.0.0.1:6379> ZRANGEBYLEX mycity - + LIMIT 0 2
    1) "Delhi"
    2) "London"
    127.0.0.1:6379> ZRANGEBYLEX mycity - + LIMIT 2 3
    1) "Paris"
    2) "Tokyo"
    3) "NewYork"
    ```

- **ZRANGEBYSCORE** > Redis ZRANGEBYSCORE command is used to return all the elements in the sorted set at the key with a score between minimum and maximum. The elements are considered to be ordered from low to high scores. The elements having the same score are returned in lexicographical order.

    **Example**

    ```code
    127.0.0.1:6379> ZADD mysales 1556 Samsung 2000 Nokis 1800 Micromax
    (integer) 3
    127.0.0.1:6379> ZADD mysales 2200 Sunsui 1800 MicroSoft 2500 LG
    (integer) 3
    127.0.0.1:6379> ZRANGEBYSCORE mysales -inf +inf WITHSCORES
    1) "Samsung"
    2) "1556"
    3) "MicroSoft"
    4) "1800"
    5) "Micromax"
    6) "1800"
    7) "Nokis"
    8) "2000"
    9) "Sunsui"
    10) "2200"
    11) "LG"
    12) "2500"
    ```
    ```code
    127.0.0.1:6379> ZRANGEBYSCORE mysales 1500 1800 WITHSCORES
    1) "Samsung"
    2) "1556"
    3) "MicroSoft"
    4) "1800"
    5) "Micromax"
    6) "1800"
    127.0.0.1:6379> ZRANGEBYSCORE mysales 2000 +inf WITHSCORES
    1) "Nokis"
    2) "2000"
    3) "Sunsui"
    4) "2200"
    5) "LG"
    6) "2500"
    127.0.0.1:6379> ZRANGEBYSCORE mysales (1500 (1800 WITHSCORES
    1) "Samsung"
    2) "1556"
    127.0.0.1:6379> ZRANGEBYSCORE mysales (1800 2200 WITHSCORES
    1) "Nokis"
    2) "2000"
    3) "Sunsui"
    4) "2200"
    ```
    ```code
    127.0.0.1:6379> ZRANGEBYSCORE mysales -inf +inf WITHSCORES LIMIT 0 3
    1) "Samsung"
    2) "1556"
    3) "MicroSoft"
    4) "1800"
    5) "Micromax"
    6) "1800"
    127.0.0.1:6379> ZRANGEBYSCORE mysales -inf +inf WITHSCORES LIMIT 4 10
    1) "Sunsui"
    2) "2200"
    3) "LG"
    4) "2500"
    ```

- **ZRANK** > Redis ZRANK command is used to return the rank of member in the sorted set stored at key, with the scores ordered from low to high. The rank is 0-based, which means that the member with the lowest score has rank 0.

    **Example**

    ```code
    127.0.0.1:6379> ZADD mysales 1556 Samsung 2000 Nokia 1800 Micromax
    (integer) 3
    127.0.0.1:6379> ZADD mysales 2200 Sunsui 1800 MicroSoft 2500 LG
    (integer) 3
    127.0.0.1:6379> ZRANGE mysales 0 -1 WITHSCORES
    1) "Samsung"
    2) "1556"
    3) "MicroSoft"
    4) "1800"
    5) "Micromax"
    6) "1800"
    7) "Nokia"
    8) "2000"
    9) "Sunsui"
    10) "2200"
    11) "LG"
    12) "2500"
    127.0.0.1:6379> ZRANGE mysales 0 -1 WITHSCORES
    1) "Samsung"
    2) "1556"
    3) "MicroSoft"
    4) "1800"
    5) "Micromax"
    6) "1800"
    7) "Nokia"
    8) "2000"
    9) "Sunsui"
    10) "2200"
    11) "LG"
    12) "2500"
    127.0.0.1:6379> ZRANK mysales "Sunsui"
    (integer) 4
    127.0.0.1:6379> ZRANK mysales MicroSoft
    (integer) 1
    ```

- **ZREM** > Redis ZREM command is used to remove the specified members from the sorted set stored at key. The number which does not exist are ignored and an error is returned when key exists and does not hold a sorted set.

    **Example**

    ```code
    127.0.0.1:6379> ZADD mysales 1556 Samsung 2000 Nokia 1800 Micromax
    (integer) 3
    127.0.0.1:6379> ZADD mysales 2200 Sunsui 1800 MicroSoft 2500 LG
    (integer) 3
    127.0.0.1:6379> ZREVRANGE mysales 0 -1 WITHSCORES
    1) "LG"
    2) "2500"
    3) "Sunsui"
    4) "2200"
    5) "Nokia"
    6) "2000"
    7) "Micromax"
    8) "1800"
    9) "MicroSoft"
    10) "1800"
    11) "Samsung"
    12) "1556"
    127.0.0.1:6379> ZREM mysales Micromax LG Nokia
    (integer) 3
    127.0.0.1:6379> ZREVRANGE mysales 0 -1 WITHSCORES
    1) "Sunsui"
    2) "2200"
    3) "MicroSoft"
    4) "1800"
    5) "Samsung"
    6) "1556"
    ```

- **ZREMRANGEBYLEX** > Redis ZREMRANGEBYLEX command is used to remove all elements in the sorted set stored at key between the lexicographical range specified by minimum and maximum values, when all the elements in a sorted set are inserted with the same score, in order to force lexicographical ordering.

    **Example**

    ```code
    127.0.0.1:6379> ZADD mycityset 0 Delhi 0 Mumbai 0 Hyderabad 0 Kolkata 0 Chennai
    (integer) 5
    127.0.0.1:6379> ZRANGEBYLEX mycityset - +
    1) "Chennai"
    2) "Delhi"
    3) "Hyderabad"
    4) "Kolkata"
    5) "Mumbai"
    127.0.0.1:6379> ZREMRANGEBYLEX mycityset "(Hyderabad" "(Mumbai"
    (integer) 1
    127.0.0.1:6379> ZRANGEBYLEX mycityset - +
    1) "Chennai"
    2) "Delhi"
    3) "Hyderabad"
    4) "Mumbai"
    127.0.0.1:6379> ZREMRANGEBYLEX mycityset "[Hyderabad" "(Mumbai"
    (integer) 1
    127.0.0.1:6379> ZRANGEBYLEX mycityset - +
    1) "Chennai"
    2) "Delhi"
    3) "Mumbai"
    127.0.0.1:6379> ZREMRANGEBYLEX mycityset "[Hyderabad" "[Mumbai"
    (integer) 1
    127.0.0.1:6379> ZRANGEBYLEX mycityset - +
    1) "Chennai"
    2) "Delhi"
    ```

- **ZREMRANGEBYRANK** > Redis ZREMRANGEBYRANK command is used to remove all elements in the sorted set stored at the key with the rank between start and stop.

    Both start and stop are zero-based indexes,

    - 0 is the first element,
    - 1 is the next element and so on.

    They can also be negative numbers indicating offsets starting at the element with the highest score.

    - -1 being the last element of the sorted set
    - -2 the penultimate element and so on.

    **Example**

    ```code
    127.0.0.1:6379> ZADD mycityset 1 Delhi 2 Mumbai 3 Hyderabad 4 Kolkata 5 Chennai
    (integer) 5
    127.0.0.1:6379> ZRANGE mycityset 0 -1 WITHSCORES
    1) "Delhi"
    2) "1"
    3) "Mumbai"
    4) "2"
    5) "Hyderabad"
    6) "3"
    7) "Kolkata"
    8) "4"
    9) "Chennai"
    10) "5"
    127.0.0.1:6379> ZREMRANGEBYRANK mycityset 1 2
    (integer) 2
    127.0.0.1:6379> ZRANGE mycityset 0 -1 WITHSCORES
    1) "Delhi"
    2) "1"
    3) "Kolkata"
    4) "4"
    5) "Chennai"
    6) "5"
    ```
    ```code
    127.0.0.1:6379> ZADD mycityset 1 Delhi 2 Mumbai 3 Hyderabad 4 Kolkata 5 Chennai
    (integer) 5
    127.0.0.1:6379> ZRANGE mycityset 0 -1 WITHSCORES
    1) "Delhi"
    2) "1"
    3) "Mumbai"
    4) "2"
    5) "Hyderabad"
    6) "3"
    7) "Kolkata"
    8) "4"
    9) "Chennai"
    10) "5"
    127.0.0.1:6379> ZREMRANGEBYRANK mycityset -3 -2
    (integer) 2
    127.0.0.1:6379> ZRANGE mycityset 0 -1 WITHSCORES
    1) "Delhi"
    2) "1"
    3) "Mumbai"
    4) "2"
    5) "Chennai"
    6) "5"
    ```

- **ZREMRANGEBYSCORE** > Redis ZREMRANGEBYSCORE command is used to remove all elements in the sorted set stored at the key with a score between the minimum and maximum value.

    min, max in the range of score. To delete all use -inf, + inf.

    **Example**

    ```code
    127.0.0.1:6379> ZADD mycityset 80 Delhi 60 Mumbai 70 Hyderabad 50 Kolkata 65 Chennai
    (integer) 5
    127.0.0.1:6379> ZRANGEBYSCORE mycityset -inf +inf  WITHSCORES
    1) "Kolkata"
    2) "50"
    3) "Mumbai"
    4) "60"
    5) "Chennai"
    6) "65"
    7) "Hyderabad"
    8) "70"
    9) "Delhi"
    10) "80"
    127.0.0.1:6379> ZREMRANGEBYSCORE mycityset -inf (70
    (integer) 3
    127.0.0.1:6379> ZRANGEBYSCORE mycityset -inf +inf  WITHSCORES
    1) "Hyderabad"
    2) "70"
    3) "Delhi"
    4) "80"
    127.0.0.1:6379> ZREMRANGEBYSCORE mycityset -inf 70
    (integer) 1
    127.0.0.1:6379> ZRANGEBYSCORE mycityset -inf +inf  WITHSCORES
    1) "Delhi"
    2) "80"
    ```

- **ZREVRANGE** > Redis ZREVRANGE command is used to return the specified range of elements in the sorted set stored at key. The elements are considered to be ordered from the highest to the lowest score. The descending lexicographical order is used for elements with equal score.

    **Example**

    ```code
    127.0.0.1:6379> ZADD mycityset 100000 Delhi 850000 Mumbai 700000 Hyderabad 800000 Kolkata
    (integer) 4
    127.0.0.1:6379> ZREVRANGE mycityset 0 -1 WITHSCORES
    1) "Mumbai"
    2) "850000"
    3) "Kolkata"
    4) "800000"
    5) "Hyderabad"
    6) "700000"
    7) "Delhi"
    8) "100000"
    ```
    ```code
    127.0.0.1:6379> ZADD mycityset 100000 Delhi 850000 Mumbai 700000 Hyderabad 800000 Kolkata
    (integer) 4
    127.0.0.1:6379> ZREVRANGE mycityset 0 -1 WITHSCORES
    1) "Mumbai"
    2) "850000"
    3) "Kolkata"
    4) "800000"
    5) "Hyderabad"
    6) "700000"
    7) "Delhi"
    8) "100000"
    127.0.0.1:6379> ZREVRANGE mycityset 0 1 WITHSCORES
    1) "Mumbai"
    2) "850000"
    3) "Kolkata"
    4) "800000"
    127.0.0.1:6379> ZREVRANGE mycityset -2 -1 WITHSCORES
    1) "Hyderabad"
    2) "700000"
    3) "Delhi"
    4) "100000"
    ```

- **ZREVRANGEBYLEX** > Redis ZREVRANGEBYLEX command is used to return the specified range of elements in the sorted set stored at key between maxium and minimum value, when all the elements in a sorted set are inserted with the same score, in order to force lexicographical ordering. The elements are considered to be ordered from the the highest to lowest score.

    - min, max are the range of the member. To view all the -, + use.
    - min, if you give the max value must be used '[' or '(' in the front. 
    - '[' Is used to contain a value, and '(' is used to block out. 

    Both start and stop are zero-based indexes,

    - 0 is the first element,
    - 1 is the next element and so on.

    They can also be negative numbers indicating offsets from the end of the sorted set,

    - -1 being the last element of the sorted set
    - -2 the penultimate element and so on.

    **Example**

    ```code
    127.0.0.1:6379> ZADD mycity 0 Delhi 0 London 0 Paris 0 Tokyo 0 NewYork 0 Seoul
    (integer) 6
    127.0.0.1:6379> ZREVRANGEBYLEX mycity + -
    1) "Tokyo"
    2) "Seoul"
    3) "Paris"
    4) "NewYork"
    5) "London"
    6) "Delhi"
    127.0.0.1:6379> ZREVRANGEBYLEX mycity + "[Paris"
    1) "Tokyo"
    2) "Seoul"
    3) "Paris"
    127.0.0.1:6379> ZREVRANGEBYLEX mycity "(NewYork"  "[London"
    1) "London"
    127.0.0.1:6379> ZREVRANGEBYLEX mycity "(Seoul"  "[Delhi"
    1) "Paris"
    2) "NewYork"
    3) "London"
    4) "Delhi"
    ```
    ```code
    127.0.0.1:6379> ZADD mycity 0 Delhi 0 London 0 Paris 0 Tokyo 0 NewYork 0 Seoul
    (integer) 6
    127.0.0.1:6379> ZREVRANGEBYLEX mycity + -
    1) "Tokyo"
    2) "Seoul"
    3) "Paris"
    4) "NewYork"
    5) "London"
    6) "Delhi"
    127.0.0.1:6379> ZREVRANGEBYLEX mycity + - LIMIT 0 2
    1) "Tokyo"
    2) "Seoul"
    127.0.0.1:6379> ZREVRANGEBYLEX mycity + - LIMIT 2 3
    1) "Paris"
    2) "NewYork"
    3) "London"
    ```

- **ZREVRANGEBYSCORE** > Redis ZREVRANGEBYSCORE command is used to return all the elements in the sorted set at the key with a score between maximum and minimum values. In contrary to the default ordering of sorted sets, for this command, the elements are considered to be ordered from high to low scores. The elements having the same score are returned in reverse lexicographical order.

    **Example**

    ```code
    127.0.0.1:6379> ZADD mysales 1556 Samsung 2000 Nokis 1800 Micromax
    (integer) 3
    127.0.0.1:6379> ZADD mysales 2200 Sunsui 1800 MicroSoft 2500 LG
    (integer) 3
    127.0.0.1:6379> ZREVRANGEBYSCORE mysales +inf -inf WITHSCORES
    1) "LG"
    2) "2500"
    3) "Sunsui"
    4) "2200"
    5) "Nokis"
    6) "2000"
    7) "Micromax"
    8) "1800"
    9) "MicroSoft"
    10) "1800"
    11) "Samsung"
    12) "1556"
    127.0.0.1:6379> ZREVRANGEBYSCORE mysales 2000 1800 WITHSCORES
    1) "Nokis"
    2) "2000"
    3) "Micromax"
    4) "1800"
    5) "MicroSoft"
    6) "1800"
    127.0.0.1:6379> ZREVRANGEBYSCORE mysales +inf 2200 WITHSCORES
    1) "LG"
    2) "2500"
    3) "Sunsui"
    4) "2200"
    ```
    ```code
    127.0.0.1:6379> ZADD mysales 1556 Samsung 2000 Nokis 1800 Micromax
    (integer) 3
    127.0.0.1:6379> ZADD mysales 2200 Sunsui 1800 MicroSoft 2500 LG
    (integer) 3
    127.0.0.1:6379> ZREVRANGEBYSCORE mysales +inf -inf WITHSCORES
    1) "LG"
    2) "2500"
    3) "Sunsui"
    4) "2200"
    5) "Nokis"
    6) "2000"
    7) "Micromax"
    8) "1800"
    9) "MicroSoft"
    10) "1800"
    11) "Samsung"
    12) "1556"
    127.0.0.1:6379> ZREVRANGEBYSCORE mysales (2100 (1800 WITHSCORES
    1) "Nokis"
    2) "2000"
    127.0.0.1:6379> ZREVRANGEBYSCORE mysales 2100 (1800 WITHSCORES
    1) "Nokis"
    2) "2000"
    ```
    ```code
    127.0.0.1:6379> ZADD mysales 1556 Samsung 2000 Nokis 1800 Micromax
    (integer) 3
    127.0.0.1:6379> ZADD mysales 2200 Sunsui 1800 MicroSoft 2500 LG
    (integer) 3
    127.0.0.1:6379> ZREVRANGEBYSCORE mysales +inf -inf  WITHSCORES
    1) "LG"
    2) "2500"
    3) "Sunsui"
    4) "2200"
    5) "Nokis"
    6) "2000"
    7) "Micromax"
    8) "1800"
    9) "MicroSoft"
    10) "1800"
    11) "Samsung"
    12) "1556"
    127.0.0.1:6379> ZREVRANGEBYSCORE mysales +inf -inf  WITHSCORES LIMIT 0 3
    1) "LG"
    2) "2500"
    3) "Sunsui"
    4) "2200"
    5) "Nokis"
    6) "2000"
    127.0.0.1:6379> ZREVRANGEBYSCORE mysales +inf -inf  WITHSCORES LIMIT 3 10
    1) "Micromax"
    2) "1800"
    3) "MicroSoft"
    4) "1800"
    5) "Samsung"
    6) "1556"
    ```

- **ZREVRANK** > Redis ZREVRANK command is used to return the rank of member in the sorted set stored at key, with the scores ordered from high to low. The rank is 0-based, which means that the member with the highest score has rank 0.

    **Example**

    ```code
    127.0.0.1:6379> ZADD mycityset 80 Delhi 60 Mumbai 70 Hyderabad 50 Kolkata 65 Chennai
    (integer) 5
    127.0.0.1:6379> ZREVRANGE mycityset 0 -1 WITHSCORES
    1) "Delhi"
    2) "80"
    3) "Hyderabad"
    4) "70"
    5) "Chennai"
    6) "65"
    7) "Mumbai"
    8) "60"
    9) "Kolkata"
    10) "50"
    127.0.0.1:6379> ZREVRANK mycityset Hyderabad
    (integer) 1
    127.0.0.1:6379> ZREVRANK mycityset Kolkata
    (integer) 4
    ```

- **ZSCORE** > Redis ZSCORE command is used to return the score of a member in the sorted set at the key. If a member does not exist in the sorted set, or key does not exist, nil is returned.

    **Example**

    ```code
    127.0.0.1:6379> ZADD mycityset 80 Delhi 60 Mumbai 70 Hyderabad 50 Kolkata 65 Chennai
    (integer) 5
    127.0.0.1:6379> ZRANGE mycityset 0 -1 WITHSCORES
    1) "Kolkata"
    2) "50"
    3) "Mumbai"
    4) "60"
    5) "Chennai"
    6) "65"
    7) "Hyderabad"
    8) "70"
    9) "Delhi"
    10) "80"
    127.0.0.1:6379> ZSCORE mycityset Kolkata
    "50"
    127.0.0.1:6379> ZSCORE mycityset Chennai
    "65"
    ```

- **ZUNIONSTORE** > Redis ZUNIONSTORE command calculates the union of a number of input keys  sorted sets given by the specified keys, and stores the result in a specified key.

    The WEIGHTS option along with ZUNIONSTORE specify a multiplication factor for each input sorted set. This means that the score of every element in every input sorted set is multiplied by this factor before being passed to the aggregation function. When WEIGHTS is not given, the multiplication factors default to 1.

    **Example**

    ```code
    127.0.0.1:6379> ZADD srcset1 5 M 6 N 7 O
    (integer) 3
    127.0.0.1:6379> ZADD srcset2 3 N 2 O 4 P
    (integer) 3
    127.0.0.1:6379> ZUNIONSTORE desset 2 srcset1 srcset2
    (integer) 4
    127.0.0.1:6379> ZRANGE desset 0 -1 WITHSCORES
    1) "P"
    2) "4"
    3) "M"
    4) "5"
    5) "N"
    6) "9"
    7) "O"
    8) "9"
    ```
    ```code
    127.0.0.1:6379> ZADD srcset1 5 M 6 N 7 O
    (integer) 3
    127.0.0.1:6379> ZADD srcset2 3 N 2 O 4 P
    (integer) 3
    127.0.0.1:6379> ZUNIONSTORE desset 2 srcset1 srcset2 WEIGHTS 2 3
    (integer) 4
    127.0.0.1:6379> ZRANGE desset 0 -1 WITHSCORES
    1) "M"
    2) "10"
    3) "P"
    4) "12"
    5) "O"
    6) "20"
    7) "N"
    8) "21"
    ```
    ```code
    127.0.0.1:6379> ZADD srcset1 5 M 6 N 7 O
    (integer) 3
    127.0.0.1:6379> ZADD srcset2 3 N 2 O 4 P
    (integer) 3
    127.0.0.1:6379> ZUNIONSTORE desset 2 srcset1 srcset2 AGGREGATE MIN
    (integer) 4
    127.0.0.1:6379> ZRANGE desset 0 -1 WITHSCORES
    1) "O"
    2) "2"
    3) "N"
    4) "3"
    5) "P"
    6) "4"
    7) "M"
    8) "5"
    127.0.0.1:6379> ZUNIONSTORE desset 2 srcset1 srcset2 AGGREGATE MAX
    (integer) 4
    127.0.0.1:6379> ZRANGE desset 0 -1 WITHSCORES
    1) "P"
    2) "4"
    3) "M"
    4) "5"
    5) "N"
    6) "6"
    7) "O"
    8) "7"
    ```
    ```code
    127.0.0.1:6379> ZADD srcset1 5 M 6 N 7 O
    (integer) 3
    127.0.0.1:6379> ZADD srcset2 3 N 2 O 4 P
    (integer) 3
    127.0.0.1:6379> ZADD srcset3 1 O 2 P 3 Q
    (integer) 3
    127.0.0.1:6379> ZUNIONSTORE desset 3 srcset1 srcset2 srcset3
    (integer) 5
    127.0.0.1:6379> ZRANGE desset 0 -1 WITHSCORES
    1) "Q"
    2) "3"
    3) "M"
    4) "5"
    5) "P"
    6) "6"
    7) "N"
    8) "9"
    9) "O"
    10) "10"
    ```

- **ZSCAN** > Redis ZSCAN command iterates elements of Sorted Set types and their associated scores.

    Basic usage of SSCAN

    - ZSCAN is a cursor based iterator. This means that at every call of the command, the server returns an updated cursor that the user needs to use as the cursor argument in the next call.
    - An iteration starts when the cursor is set to 0, and terminates when the cursor returned by the server is 0. The following is an example of ZSCAN iteration: 

    **Example**

    ```code
    127.0.0.1:6379> ZADD mytestset 1 M1 2 M2 3 M3 4 N1 5 N2 6 N3 7 O1 8 O2 9 O3
    (integer) 9
    127.0.0.1:6379> ZSCAN mytestset 0
    1) "0"
    2)  1) "M1"
        2) "1"
        3) "M2"
        4) "2"
        5) "M3"
        6) "3"
        7) "N1"
        8) "4"
        9) "N2"
    10) "5"
    11) "N3"
    12) "6"
    13) "O1"
    14) "7"
    15) "O2"
    16) "8"
    17) "O3"
    18) "9"
    ```
    ```code
    127.0.0.1:6379> ZSCAN mytestset 0 COUNT 5
    1) "0"
    2)  1) "M1"
        2) "1"
        3) "M2"
        4) "2"
        5) "M3"
        6) "3"
        7) "N1"
        8) "4"
        9) "N2"
    10) "5"
    11) "N3"
    12) "6"
    13) "O1"
    14) "7"
    15) "O2"
    16) "8"
    17) "O3"
    18) "9"
    ```
    ```code
    127.0.0.1:6379> ZSCAN mytestset 0 COUNT 5
    1) "0"
    2)  1) "M1"
        2) "1"
        3) "M2"
        4) "2"
        5) "M3"
        6) "3"
        7) "N1"
        8) "4"
        9) "N2"
    10) "5"
    11) "N3"
    12) "6"
    13) "O1"
    14) "7"
    15) "O2"
    16) "8"
    17) "O3"
    18) "9"
    127.0.0.1:6379>
    127.0.0.1:6379> ZSCAN mytestset 0 MATCH N*
    1) "0"
    2) 1) "N1"
    2) "4"
    3) "N2"
    4) "5"
    5) "N3"
    6) "6"
    127.0.0.1:6379> ZSCAN mytestset 0 MATCH *3*
    1) "0"
    2) 1) "M3"
    2) "3"
    3) "N3"
    4) "6"
    5) "O3"
    6) "9"
    127.0.0.1:6379> ZSCAN mytestset 0 MATCH *3* COUNT 20
    1) "0"
    2) 1) "M3"
    2) "3"
    3) "N3"
    4) "6"
    5) "O3"
    ```

# Hashes

Hashes in Redis are useful to represent objects with many fields. They are set up to store a vast amount of fields in a small amount of space. Here are some common commands associated with hashes:

- HMSET: Sets up multiple hash values
- HSET: Sets the hash field with a string value
- HGET: Retrieves the value of a hash field
- HMGET: Retrieves all of the values for given hash fields
- HGETALL: Retrieves all of the values for in a hash

### Hashes Commands:

- **HDEL** > Redis HDEL command is used to remove the specified fields from the hash stored at a key and ignored the specified keys that do not exist within this hash. It is treated as an empty hash if the key does not exist, and this command returns 0.

    **Example**

    ```code
    127.0.0.1:6379> HMSET langhash lang1 "PHP" lang2 "JavaScript" lang3 "Python"
    OK
    127.0.0.1:6379> HGET langhash lang1
    "PHP"
    127.0.0.1:6379> HGET langhash lang2
    "JavaScript"
    127.0.0.1:6379> HGET langhash lang3
    "Python"
    127.0.0.1:6379> HDEL langhash lang1
    (integer) 1
    127.0.0.1:6379> HGET langhash lang1
    (nil)
    127.0.0.1:6379> HGET langhash lang2
    "JavaScript"
    127.0.0.1:6379> HGET langhash lang3
    "Python" 
    ```
    ```code
    127.0.0.1:6379> HMSET langhash lang1 "PHP" lang2 "JavaScript" lang3 "Python"
    OK
    127.0.0.1:6379> HGET langhash lang1
    "PHP"
    127.0.0.1:6379> HGET langhash lang2
    "JavaScript"
    127.0.0.1:6379> HGET langhash lang3
    "Python"
    127.0.0.1:6379> HDEL langhash lang1 lang2 lang3
    (integer) 3
    127.0.0.1:6379> HGET langhash lang1
    (nil)
    127.0.0.1:6379> HGET langhash lang2
    (nil)
    127.0.0.1:6379> HGET langhash lang3
    (nil)
    ```
    ```code
    127.0.0.1:6379> HGETALL user
    1) "email"
    2) "example@gmail.com"
    3) "lang"
    4) "English"
    5) "gender"
    6) "Male"
    127.0.0.1:6379> HDEL user email
    (integer) 1
    127.0.0.1:6379> HDEL user lang gender
    (integer) 2
    ```

- **HEXISTS** > Redis HEXISTS command is used to check whether a hash field stored at key or not.

    **Example**

    ```code
    127.0.0.1:6379> HMSET langhash lang1 "PHP" lang2 "JavaScript" lang3 "Python"
    OK
    127.0.0.1:6379> HEXISTS langhash lang1
    (integer) 1
    127.0.0.1:6379> HEXISTS langhash lang4
    (integer) 0
    ```
    ```code
    127.0.0.1:6379> HSET user email example@gmail.com
    (integer) 1
    127.0.0.1:6379> HEXISTS user email
    (integer) 1
    127.0.0.1:6379> HEXISTS user xyz
    (integer) 0
    ```

- **HGET** > Redis HGET command is used to get the value associated with the field in the hash stored at key.

    **Example**

    ```code
    127.0.0.1:6379> HMSET langhash lang1 "PHP" lang2 "JavaScript" lang3 "Python"
    OK
    127.0.0.1:6379> HGET langhash lang1
    "PHP"
    127.0.0.1:6379> HGET langhash lang4
    (nil)
    ```
    ```code
    127.0.0.1:6379> HGETALL user
    1) "email"
    2) "example@gmail.com"
    3) "lang"
    4) "English"
    5) "gender"
    6) "Male"
    127.0.0.1:6379> HGET user email
    "example@gmail.com"
    127.0.0.1:6379> HGET user lang
    "English"
    ```

- **HGETALL** > Redis HGETALL command is used to get all fields and values of the hash stored at key. In the returned value, every field name is followed by its value, so the length of the reply is twice the size of the hash.

    **Example**

    ```code
    127.0.0.1:6379> HMSET langhash lang1 "PHP" lang2 "JavaScript" lang3 "Python"
    OK
    127.0.0.1:6379> HSET langhash lang4 "Golanguage"
    (integer) 1
    127.0.0.1:6379> HGETALL langhash
    1) "lang1"
    2) "PHP"
    3) "lang2"
    4) "JavaScript"
    5) "lang3"
    6) "Python"
    7) "lang4"
    8) "Golanguage"
    ```
    ```code
    127.0.0.1:6379> HSET user email example@gmail.com
    (integer) 1
    127.0.0.1:6379> HSET user lang English
    (integer) 1
    127.0.0.1:6379> HSET user gender Male
    (integer) 1
    127.0.0.1:6379> HGETALL user
    1) "email"
    2) "example@gmail.com"
    3) "lang"
    4) "English"
    5) "gender"
    6) "Male"
    ```

- **HINCRBY** > Redis HINCRBY command is used to increment the number stored at the field in the hash stored at key by increment. If the key does not exist, a new key holding a hash is created. If the field does not exist the value is set to 0 before the operation is performed.

    **Example**

    ```code
    127.0.0.1:6379> HSET langhash lang1 10
    (integer) 1
    127.0.0.1:6379> HINCRBY langhash lang1 1
    (integer) 11
    127.0.0.1:6379> HINCRBY langhash lang1 -1
    (integer) 10
    ```
    ```code
    127.0.0.1:6379> HSET user visits 5
    (integer) 1
    127.0.0.1:6379> HINCRBY user visits 15
    (integer) 20
    127.0.0.1:6379> HINCRBY user visits -25
    (integer) -5
    ```

- **HINCRBYFLOAT** > Redis HINCRBYFLOAT command is used to increment the specified field of a hash stored at key, and representing a floating point number, by the specified increment. If the field does not exist, it is set to 0 before performing the operation. If the field contains a value of wrong type or specified increment is not parsable as floating point number, then an error has occurred.

    **Example**

    ```code
    127.0.0.1:6379> HSET langhash lang1 10.25
    (integer) 1
    127.0.0.1:6379> HINCRBYFLOAT langhash lang1 0.2
    "10.45"
    127.0.0.1:6379> HSET langhash lang1 6.0e4
    (integer) 1
    127.0.0.1:6379> HINCRBYFLOAT langhash lang1 3.0e3
    "63000"
    ```
    ```code
    127.0.0.1:6379> HINCRBYFLOAT user height 85.5
    "85.5"
    127.0.0.1:6379> HINCRBYFLOAT user height 14.2
    "99.7"
    127.0.0.1:6379> HINCRBYFLOAT user height -12.5
    "87.2"
    127.0.0.1:6379> HINCRBYFLOAT user height 85.5
    "172.7"
    127.0.0.1:6379> HINCRBYFLOAT user height -85.7
    "87"
    ```

- **HKEYS** > Redis HKEYS command is used to get all field names in the hash stored at key.

    **Example**

    ```code
    127.0.0.1:6379> HSET langhash lang1 "PHP"
    (integer) 1
    127.0.0.1:6379> HSET langhash lang2 "Javascript"
    (integer) 1
    127.0.0.1:6379> HSET langhash lang3 "Python"
    (integer) 1
    127.0.0.1:6379> HSET langhash lang4 "Golanguage"
    (integer) 1
    127.0.0.1:6379> HKEYS langhash
    1) "lang1"
    2) "lang2"
    3) "lang3"
    4) "lang4"
    ```
    ```code
    127.0.0.1:6379> HKEYS user
    1) "email"
    2) "lang"
    3) "gender"
    4) "visits"
    5) "height"
    127.0.0.1:6379> HMSET user-x name Sweaty gender Female lang English
    OK
    127.0.0.1:6379> HKEYS user-x
    1) "name"
    2) "gender"
    3) "lang"
    ```

- **HLEN** > Redis HLEN command is used to get the number of fields contained in the hash stored at key.

    **Example**

    ```code
    127.0.0.1:6379> HSET langhash lang1 "PHP"
    (integer) 1
    127.0.0.1:6379> HSET langhash lang2 "Javascript"
    (integer) 1
    127.0.0.1:6379> HSET langhash lang3 "Python"
    (integer) 1
    127.0.0.1:6379> HSET langhash lang4 "Golanguage"
    (integer) 1
    127.0.0.1:6379> HLEN langhash
    (integer) 4
    ```
    ```code
    127.0.0.1:6379> HMSET user-x name Sweaty gender Female lang English
    OK
    127.0.0.1:6379> HLEN user-x
    (integer) 3
    ```

- **HMGET** > Redis HMGET command is used to get the values associated with the specified fields in the hash stored at key. If a field does not exist in redis hash, then nil value is returned.

    **Example**

    ```code
    127.0.0.1:6379> HSET langhash lang1 "PHP"
    (integer) 1
    127.0.0.1:6379> HSET langhash lang2 "Javascript"
    (integer) 1
    127.0.0.1:6379> HSET langhash lang3 "Python"
    (integer) 1
    127.0.0.1:6379> HMGET langhash lang1 lang2 lang3 lang4
    1) "PHP"
    2) "Javascript"
    3) "Python"
    4) (nil)
    ```
    ```code
    127.0.0.1:6379> HMSET user-y email example@gmail.com language English gender Male
    OK
    127.0.0.1:6379> HMGET user-y email language gender
    1) "example@gmail.com"
    2) "English"
    3) "Male"
    ```

- **HMSET** > Redis HMSET command is used to set the specified fields to their respective values in the hash stored at key. This command overwrites any existing fields in the hash. If the key does not exist, a new key holding a hash is created.

    **Example**

    ```code
    127.0.0.1:6379> HMSET langhash lang1 "PHP" lang2 "JavaScript" lang3 "Redis"
    OK
    127.0.0.1:6379> HGET langhash lang1
    "PHP"
    127.0.0.1:6379> HGET langhash lang2
    "JavaScript"
    127.0.0.1:6379> HGET langhash lang3
    "Redis"
    ```
    ```code
    127.0.0.1:6379> HMSET user-y email example@gmail.com language English gender Male
    OK
    127.0.0.1:6379> HGETALL user-y
    1) "email"
    2) "example@gmail.com"
    3) "language"
    4) "English"
    5) "gender"
    6) "Male"
    ```

- **HSET** > Redis HSET command is used to set the field in the hash stored at key to value. If the key does not exist, a new key holding a hash is created. If the field already exists in the hash, it is overwritten.

    **Example**

    ```code
    127.0.0.1:6379> HSET langhash lang1 "PHP"
    (integer) 1
    127.0.0.1:6379> HSET langhash lang2 "Javascript"
    (integer) 1
    127.0.0.1:6379> HGET langhash lang1
    "PHP"
    127.0.0.1:6379> HGET langhash lang2
    "Javascript"
    ```
    ```code
    127.0.0.1:6379> HSET user email example@gmail.com
    (integer) 1
    127.0.0.1:6379> HSET user lang English
    (integer) 1
    127.0.0.1:6379> HSET user gender Male
    (integer) 1
    127.0.0.1:6379> HGETALL user
    1) "email"
    2) "example@gmail.com"
    3) "lang"
    4) "English"
    5) "gender"
    6) "Male"
    ```

- **HSETNX** > Redis HSETNX command is used to set the field in the hash stored at the key to value, only if the field does not yet exist. If the key does not exist, a new key holding a hash is created. If the field already exists, this operation has no effect.

    **Example**

    ```code
    127.0.0.1:6379> HSETNX langhash lan1 "example"
    (integer) 1
    127.0.0.1:6379> HSETNX langhash lan2 "Tutorial"
    (integer) 1
    127.0.0.1:6379> HSETNX langhash lan1 "PHP"
    (integer) 0
    127.0.0.1:6379> HSETNX langhash lan2 "JavaScript"
    (integer) 0
    127.0.0.1:6379> HGET langhash lan1
    "example"
    127.0.0.1:6379> HGET langhash lan2
    "Tutorial"
    ```
    ```code
    127.0.0.1:6379> HSETNX user-y email example@gmail.com
    (integer) 0
    127.0.0.1:6379> HSETNX user-v email example1@gmail.com
    (integer) 1
    127.0.0.1:6379> HGET user-y email
    "example@gmail.com"
    127.0.0.1:6379> HGET user-v email
    "example1@gmail.com"
    ```

- **HVALS** > Redis HVALS command is used to get all values in the hash stored at key.

    **Example**

    ```code
    127.0.0.1:6379> HSET langhash lang1 "PHP"
    (integer) 1
    127.0.0.1:6379> HSET langhash lang2 "JavaScript"
    (integer) 1
    127.0.0.1:6379> HSET langhash lang3 "Python"
    (integer) 1
    127.0.0.1:6379> HVALS langhash
    1) "PHP"
    2) "JavaScript"
    3) "Python"
    ```
    ```code
    127.0.0.1:6379> HVALS user-y
    1) "example@gmail.com"
    2) "English"
    3) "Male"
    127.0.0.1:6379> HMSET user-x name Chima gender Male language English
    OK
    127.0.0.1:6379> HVALS user-x
    1) "Chima"
    2) "Male"
    3) "English"
    ```

- **HSCAN** > HSCAN array of elements contains two elements, a field, and a value, for every returned element of the Hash.

    **Example**ก

    ```code
    127.0.0.1:6379> HMSET langhash lang1 "PHP" lang2 "JavaScript" lang3 "Python" lang4 "GoLanguage"
    OK
    127.0.0.1:6379> HSCAN langhash 0
    1) "0"
    2) 1) "lang1"
    2) "PHP"
    3) "lang2"
    4) "JavaScript"
    5) "lang3"
    6) "Python"
    7) "lang4"
    8) "GoLanguage"
    ```
