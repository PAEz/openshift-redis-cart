OpenShift Redis Cartridge
=========================
Redis - 3.0.2
------------

Runs [Redis](http://redis.io) on [OpenShift](https://openshift.redhat.com/app/login) using downloadable cartridge support.  To install to OpenShift from the CLI (you'll need version 1.9 or later of rhc), create your app and then run:

    rhc add-cartridge http://cartreflect-claytondev.rhcloud.com/reflect?github=PAEz/openshift-redis-cart -a 

Any log output will be generated to $OPENSHIFT_REDIS_DIR/logs/redis.log

Supports clustering and persistence, with monitoring from Redis Sentinel.  Does not automatically pull the latest security updates - please monitor the Redis upstream 2.6 branches.


How it Works
------------

`conf/redis.conf.erb` is run before Redis starts and generates a configuration file.  It will attempt to load the file `.openshift/redis.conf` if it exists.  The default configuration will snapshot the db to disk into the <code>$OPENSHIFT_DATA_DIR/.redis/dbs</code> directory.

At cart creation, a unique password will be generated to the environment variable `REDIS_PASSWORD` - the server will require this to connect.

To access `redis-cli` from the SSH session, use the $REDIS_CLI environment variable to get the correct config on the gear

    $ ssh <gear with redis>
    Connectiing to....
    $ redis-cli $REDIS_CLI
    127.0.32.124 6379> 

To connect to Redis from your local shell, run the <code>cartridge-status</code> command to get the port and authorization info:

    $ rhc cartridge-status redis -a <yourapp>

    RESULT:
    Redis is running
      master (receives writes), mode sharded
      Connect to: 343928-abc.dev.rhcloud.com:35546 password: 80o9euk80oeu90834

and then run <code>rhc port-forward</code> to your app:

    $ rhc port-forward <yourapp>
    ....
    redis   127.0.0.1:35546   => 343928-abc.dev.rhcloud.com:35546

In another shell window, run <code>redis-cli</code>

    $ redis-cli -p 35546 -a 80o9euk80oeu90834
    redis 127.0.0.1:35546>

A number of Redis configuration values can be tuned via environment variables:

*  <code>REDIS_PASSWORD</code>

   The password for accessing Redis.

*  <code>REDIS_MAXMEMORY</code>

   The maximum memory this instance will allow.  No default value.

*  <code>REDIS_APPENDONLY</code>

   The value of <code>appendonly</code> in redis.conf.  Defaults to 'no'

*  <code>REDIS_APPENDFSYNC</code>

   The value of <code>appendfsync</code> in redis.conf.  Defaults to 'everysec'

Always restart each gear after setting these environment variables.


Upgrading
---------

If you install this cartridge from source, you will be using a precompiled version of Redis 2.6 for RHEL6.  You can run the <code>bin/control update</code> script on each gear to build and update to the latest version of the Redis 2.6 tree.  

    $ rhc ssh <yourapp> --gears 'cd redis && ./bin/control update'
    $ rhc restart-cartridge redis -a <yourapp>'

We hope to add a more natural update process at a later point.

[more about this cartridge see here](https://github.com/smarterclayton/openshift-redis-cart "more about this cartridge")
