---
title: "Redis"
date: 2021-10-18T14:16:55Z
draft: false
tags: ["database"]
categories: ["note"]
---

## Keyword

1. `KEYS pattern`  (glob wildcard)
    1. ? any char
    2. \* any chars
    3. \[a-z] scope
    4. \x x

2. `EXISTS key` if exists return 1 else return 0 (integer)
3. `DEL key` [key ...] return the count of the keys deleted
4. `TYPE key` get the type of the key
   * return string,hash,list,set,zset

### String  

1. max 512MB
2. `SET key`  
3. `GET key`  
4. `INCR key` increase integer
5. `DECR key` decrease integer
6. `INCRBY key increment`
7. `INCRBYFLOAT` increase float
8. `APPEND key value` append at the end
9. `STRLEN key`xxx return the length of the key if key not exists return 0 (UTF-8 one Chinese char is 3)
10. `MGET key [key ...]` get many keys at the same time
11. `MSET key value [key value ...]`
12. `GETBIT key offset`  
    `SETBIT key offset value`  
    `BITCOUNT key [start] [end]` get the num of binary bit 1 start and end refer byte
    `BITOP operation destkey key [key ...]` bit operation result is destkey  
    `BITOPS key 0/1 [start] [end]` get the first position of 0/1

### Hash  

1. `HSET key field value`  
2. `HGET key field`  
3. `HMSET key field value [field value ...]`  
4. `HMGET key field [field ...]`  
5. `HGETALL key`  
6. `HEXISTS key field`  
7. `HSETNX key field value` if key exists do nothing
8. `HINCRBY key field increment`
9. `HDEL key field [field ..]` return the num of deleted fielded
10. `HKEYS key` get all field names without values
11. `HVALS key` get all field values without names
12. `HLEN key` get the num of fields

### List

1. `LPUSH key value [value ...]` push on left
2. `RPUSH key value [value ...]` push on right
3. `LPOP key`
4. `RPOP key`
5. `LLEN key` get length
6. `LRANGE key start stop` get a range from start to stop (support negative number)
7. `LREM key count value` delete *count* members values *value*  if count > 0 begin on left, < 0 begin on right, = 0 delete all
8. `LINDEX key index` get the value specified by index
9. `LSET key index value` set value
10. `LTRIM key start end` only save the specified members
11. `LINSERT key BEFORE|AFTER pivot value` return the length of the list
12. `RPOPLPUSH source destination` rpop from source to lpush destination
13. `BRPOP` block

### Set

1. MAX 2^32-1 hash table
2. `SADD key member [member ...]`
3. `SREM key member [member ...]` remove
4. `SMEMBERS key` get all members
5. `SISMEMBER key member` judge if exist
6. `SDIFF key [key ,,]` difference operation
7. `SINTER key [key ,,]` intersection operation
8. `SUNION key [key ,,]` union operation
9. `SCARD key` get the num of members of the set
10. `SDIFFSTORE destination key [key ...]` storage the result
11. `SINTERSTORE destination key [key ...]`
12. `SUNIONSTORE destination key [key ...]`
13. `SRANDMEMBER key [count]` get *count* members randomly if count is negative return repeatable members
14. `SPOP key` pop randomly

### Zset

1. `ZADD key score member [score member ...]`
2. `ZSCORE key member` get the score of the member
3. `ZRANGE key start stop [WITHSCORES]` get scores from small to large
4. `ZREVRANGE key start stop [WITHSCORES]` from large to small
5. `ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]` if wants the result without point add ( before *min* *max*
6. `ZREVRANGEBYSCORE key min max [WITHSCORE] [LIMIT offset count]`
7. `ZINCRBY key increment member`
8. `ZCARD key` get the num of zset
9. `ZCOUNT key min max`
10. `ZREM ket member [member ...]`
11. `ZREMRANGEBYRANK ket start stop` from small to large
12. `ZREMRANGEBYSCORE key min max`
13. `ZRANK key member` get rank
14. `ZREVRANK key member`

## Advanced

### Transaction

1. `MULTI` the start of a transaction
2. `EXEC` the end of a transaction
3. handle error

    * grammar error: all commands won't be executed
    * run error: other commands will be executed

    redis doesn't support rollback

4. `WATCH` can watch one or more keys, if one of them was changed, the transaction won't be executed  
`UNWATCH` cancel watch

### Expiration

1. `EXPIRE key seconds` if success return 1, else return 0
2. `TTL key` get the time left of the key if the key doesn't have a expiration return -1, else if the key doesn't exist return -2
3. `PERSIST key` cancel the expiration SET and GETSET will also clear expiration
4. `PEXPORE key milliseconds`
5. `EXPIREAT key unixtime`  
    `PEXPOREAT key unixtime`

### Sort

1. `SORT key [LIMIT start end] [BY prefix:* (-> field)] [GET prefix:* (->field)] [STORE key]`
2. a SORT can have many GET parameters
3. `GET #` return the main key

### Message

1. `LPOP key [key ...] [timeout]` priority
2. `PUBLISH channel msg`
3. `SUBSCRIBE channel` after executing SUBSCRIBE will enter publish and subscribe mode
4. `UNSUBSCRIBE channel`
5. `PSUBSCRIBE channle(glob)`
6. `PUNSUBSCRIBE channel(glob)`

### Script

#### Lua

1. data type

    * nil
    * boolean
    * number
    * string
    * table
    * function

2. variable
    * default is nil
    * global variable (cannot be used in redis)
        * `a = 1` assign 1 to the variable
        * `a = nil` delete a variable
    * local variable

        * `local c` declare a variable
        * `local d = 1` declare and assign a variable
        * `local e,f` declare many variables at the same time
        * save a function to a variable

             ```lua
                local say_hi = function ()
                    print 'hi'
                end
             ```

        * local variable's scope begins from the declaration, ends with the layer
    * comment
        * one line `--`
        * multi-line `--[[ comment ]]`
    * assign
        * `local a, b = 1, 2` a=1 b=2
        * `local c, d = 1, 2, 3` c=1 d=2
        * `local e, f = 1` e=1, f=nil
        * as executing multiple assignment Lua will calculate expression's value
    * operator
        * `+ - * / % ^` number string will be changed to number
        * `== ~= < > <= >=` string cannot be transformed to number
        * `not and or` 0 is
        * `..` connect two string
        * `#` get the size of string or table
    * if

        ```lua
            if expression then
                ...
            elseif expression then
                ...
            else
                ...
            end
        ```

    * cycle

        ```lua
            while expression do
                ...
            end

            repeat
            ...
            until expression

            for variable = initial, endvalue, step(default = 1) do
                ...
            end

            for variable1, variable2, ..., vraiableN in literator do
                ..
            end
        ```

    * table
        * `a = {}`
        * `a['filed'] = 'value'`
        * index is from 1
        * `ipairs()` like a literator
        * `pairs()` traverse the table not array
    * function

        ```Lua
            function (list of parameters)
                ...
                return
            end

            local function name (list of patters)
                return 
            end
        ```

        `...` in () means many parameters

3. standard library

    Redis support Base String Table Math Debug

    1. String lib
        * `string.len(string)` get the length of the string
        * `string.lower(string)` `string.upper(string)` convert the case
        * `string.sub(string start [end default = -1])` get the substring
    2. Table lib
        * `table.concat(table [sep] [i] [j])` toString
        * `table.insert(table [pos default=length+1] value)`
        * `table.remove(table [pos default=length])` pop a element
    3. Math lib
        * abs sin cos tan ceil floor max min pow sqrt random random seed
    4. other libs
        * cjson.encode(table)
        * cjson.decode(string)
        * cmsgpack.pack(table)
        * cmsgpack.unpack(string)

4. Redis with Lua

    |redis return value|Lua data type|
    |--|--|
    |integer|number|
    |string|string|
    |multi-line string|table|
    |status|table(only a ok field)|
    |error|table(only a err field)|

    * `EVAL script keyNum [key ...] [arg ...]` through *KEYS* *ARGV* to get the parameters
    * `EVALSHA`
5. KEYS and ARGV
    * KEYS refers to key names  
    * ARGV refers to not key names
6. other commands about scripts
    1. `SCRIPT LOAD script` load script to cache
    2. `SCRIPT EXISTS script [scripts...]` judge if the script cached
    3. `SCRIPT FLUSH` clear scripts
    4. `SCRIPT KILL` kill script

### Persistence

#### RDB

1. `SAVE m n` m is time, n is in the time changed key number
2. `SAVE`
3. `BGSAVE` save in background
4. `LASTSAVE` get the lastest snapshot time
5. `FLUSHALL` clear all data and if hte auto save is not null, will snapshot

#### AOF

`appendonly yes`

### Cluster

#### Replication

## *Design And Implementation*

### Simple Dynamic String

save string, use as buffer: AOF buffer, client input buffer

```c
struct sdshdr{
    int len;
    int free;
    char buf[]
}
```

1. O(1) get string length
2. avoid buffer overflow
3. reduce the number of memory reallocations required to modify the length of a string
4. binary security
5. compatible with part of `C` string function

### Link List

one of list key's implementation, publish and subscribe, slow search, monitor

```c
typedef struct list{
    listNode *head;
    listNode *tail
    unsigned long len;
    void *(*dup) (vpid *ptr);   //copy
    void (*free) (void *ptr);
    int (*match) (void *ptr, void *key);
}
```

### Dictionary

redis database, hash key

```c
typedef struct dictht{
    dicEntry **table;
    unsigned long size;
    unsigned long sizemask;
    unsigned long used;
}
```
