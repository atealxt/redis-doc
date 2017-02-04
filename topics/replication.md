Replication
===

Redis replication is a very simple to use and configure master-slave
replication that allows slave Redis servers to be exact copies of
master servers. The following are some very important facts about Redis
replication:

* Redis uses asynchronous replication. Starting with Redis 2.8,
however, slaves periodically acknowledge the amount of data
processed from the replication stream.

* A master can have multiple slaves.

* Slaves are able to accept connections from other slaves. Aside from
connecting a number of slaves to the same master, slaves can also be
connected to other slaves in a cascading-like structure.

* Redis replication is non-blocking on the master side. This means that
the master will continue to handle queries when one or more slaves perform
the initial synchronization.

* Replication is also non-blocking on the slave side. While the slave is performing the initial synchronization, it can handle queries using the old version of
the dataset, assuming you configured Redis to do so in redis.conf.
Otherwise, you can configure Redis slaves to return an error to clients if the
replication stream is down. However, after the initial sync, the old dataset
must be deleted and the new one must be loaded. The slave will block incoming
connections during this brief window (that can be as long as many seconds for very large datasets).

* Replication can be used both for scalability, in order to have
multiple slaves for read-only queries (for example, slow O(N)
operations can be offloaded to slaves), or simply for data redundancy.

* It is possible to use replication to avoid the cost of having the master write the full dataset to disk: a typical technique involves configuring your master `redis.conf` to avoid persisting to disk at all, then connect a slave configured to save from time to time, or with AOF enabled. However this setup must be handled with care, since a restarting master will start with an empty dataset: if the slave tries to synchronized with it, the slave will be emptied as well.

Safety of replication when master has persistence turned off
---

In setups where Redis replication is used, it is strongly advised to have
persistence turned on in the master, or when this is not possible, for example
because of latency concerns, instances should be configured to **avoid restarting automatically** after a reboot.

To better understand why masters with persistence turned off configured to
auto restart are dangerous, check the following failure mode where data
is wiped from the master and all its slaves:

1. We have a setup with node A acting as master, with persistence turned down, and nodes B and C replicating from node A.
2. A crashes, however it has some auto-restart system, that restarts the process. However since persistence is turned off, the node restarts with an empty data set.
3. Nodes B and C will replicate from A, which is empty, so they'll effectively destroy their copy of the data.

When Redis Sentinel is used for high availability, also turning off persistence
on the master, together with auto restart of the process, is dangerous. For example the master can restart fast enough for Sentinel to don't detect a failure, so that the failure mode described above happens.

Every time data safety is important, and replication is used with master configured without persistence, auto restart of instances should be disabled.

How Redis replication works
---

If you set up a slave, upon connection it sends a PSYNC command.

If this is a reconnection and the master has enough *backlog*, only the difference (what the slave missed) is sent. Otherwise what is called a *full resynchronization* is triggered.

When a full resynchronization is triggered, the master starts a background
saving process in order to produce an RDB file. At the same time it starts to
buffer all new write commands received from the clients. When the background
saving is complete, the master transfers the database file to the slave,
which saves it on disk, and then loads it into memory. The master will
then send all buffered commands to the slave. This is done as a
stream of commands and is in the same format of the Redis protocol itself.

You can try it yourself via telnet. Connect to the Redis port while the
server is doing some work and issue the `SYNC` command. You'll see a bulk
transfer and then every command received by the master will be re-issued
in the telnet session.

Slaves are able to automatically reconnect when the master-slave
link goes down for some reason. If the master receives multiple
concurrent slave synchronization requests, it performs a single
background save in order to serve all of them.

Partial resynchronization
---

Starting with Redis 2.8, master and slave are usually able to continue the
replication process without requiring a full resynchronization after the
replication link went down.

This works by creating an in-memory backlog of the replication stream on the
master side. The master and all the slaves agree on a *replication
offset* and a *master run ID*, so when the link goes down, the slave will
reconnect and ask the master to continue the replication. Assuming the
master run ID is still the same, and that the offset specified is available
in the replication backlog, replication will resume from the point where it left off.
If either of these conditions are unmet, a full resynchronization is performed
(which is the normal pre-2.8 behavior). As the run ID of the connected master is not persisted to disk, a full resynchronization is needed when the slave restarts.

The new partial resynchronization feature uses the `PSYNC` command internally,
while the old implementation uses the `SYNC` command. Note that a Redis
slave is able to detect if the server it is talking with does not support
`PSYNC`, and will use `SYNC` instead.

Diskless replication
---

Normally a full resynchronization requires to create an RDB file on disk,
then reload the same RDB from disk in order to feed the slaves with the data.

With slow disks this can be a very stressing operation for the master.
Redis version 2.8.18 is the first version to have support for diskless
replication. In this setup the child process directly sends the
RDB over the wire to slaves, without using the disk as intermediate storage.

Configuration
---

To configure replication is trivial: just add the following line to the slave
configuration file:

    slaveof 192.168.1.1 6379

Of course you need to replace 192.168.1.1 6379 with your master IP address (or
hostname) and port. Alternatively, you can call the `SLAVEOF` command and the
master host will start a sync with the slave.

There are also a few parameters for tuning the replication backlog taken
in memory by the master to perform the partial resynchronization. See the example
`redis.conf` shipped with the Redis distribution for more information.

Diskless replication can be enabled using the `repl-diskless-sync` configuration
parameter. The delay to start the transfer in order to wait more slaves to
arrive after the first one, is controlled by the `repl-diskless-sync-delay`
parameter. Please refer to the example `redis.conf` file in the Redis distribution
for more details.

Read-only slave
---

Since Redis 2.6, slaves support a read-only mode that is enabled by default.
This behavior is controlled by the `slave-read-only` option in the redis.conf file, and can be enabled and disabled at runtime using `CONFIG SET`.

Read-only slaves will reject all write commands, so that it is not possible to write to a slave because of a mistake. This does not mean that the feature is intended to expose a slave instance to the internet or more generally to a network where untrusted clients exist, because administrative commands like `DEBUG` or `CONFIG` are still enabled. However, security of read-only instances can be improved by disabling commands in redis.conf using the `rename-command` directive.

You may wonder why it is possible to revert the read-only setting
and have slave instances that can be target of write operations.
While those writes will be discarded if the slave and the master
resynchronize or if the slave is restarted, there are a few legitimate
use case for storing ephemeral data in writable slaves.

For example computing slow set or zset operations and storing them into local
keys is an use case for writable slaves that was observed multiple times.

However note that **writable slaves before version 4.0 were uncapable of expiring keys with a time to live set**. This means that if you use `EXPIRE` or other commands that set a maximum TTL for a key, the key will leak, and while you may no longer see it while accessing it with read commands, you will see it in the count of keys and it will still use memory. So in general mixing writable slaves (previous version 4.0) and keys with TTL is going to create issues.

Redis 4.0 RC3 and greater totally resolve this problem and now writable
slaves are able to evict keys with TTL as masters do, with the exceptions
of keys written in DB numbers greater than 63 (but by default Redis instances
only have 16 databases).

Also note that since Redis 4.0 slave writes are only local, and are not propagated to sub-slaves attached to the instance. Sub slaves instead will always receive the replication stream identical to the one sent by the top-level master to the intermediate slaves. So for example in the following setup:

    A ---> B ---> C

Even if `B` is writable, C will not see `B` writes and will instead have identical dataset as the master instance `A`.

Setting a slave to authenticate to a master
---

If your master has a password via `requirepass`, it's trivial to configure the
slave to use that password in all sync operations.

To do it on a running instance, use `redis-cli` and type:

    config set masterauth <password>

To set it permanently, add this to your config file:

    masterauth <password>

Allow writes only with N attached replicas
---

Starting with Redis 2.8, it is possible to configure a Redis master to
accept write queries only if at least N slaves are currently connected to the
master.

However, because Redis uses asynchronous replication it is not possible to ensure
the slave actually received a given write, so there is always a window for data
loss.

This is how the feature works:

* Redis slaves ping the master every second, acknowledging the amount of replication stream processed.
* Redis masters will remember the last time it received a ping from every slave.
* The user can configure a minimum number of slaves that have a lag not greater than a maximum number of seconds.

If there are at least N slaves, with a lag less than M seconds, then the write will be accepted.

You may think of it as a relaxed version of the "C" in the CAP theorem, where
consistency is not ensured for a given write, but at least the time window for
data loss is restricted to a given number of seconds.

If the conditions are not met, the master will instead reply with an error and the write will not be accepted.

There are two configuration parameters for this feature:

* min-slaves-to-write `<number of slaves>`
* min-slaves-max-lag `<number of seconds>`

For more information, please check the example `redis.conf` file shipped with the
Redis source distribution.

How Redis replication deals with expires on keys
---

Redis expires allow keys to have a limited time to live. Such a feature depends
on the ability of an instance to count the time, however Redis slaves correctly
replicate keys with expires, even when such keys are altered using Lua
scripts.

To implement such a feature Redis cannot rely on the ability of the master and
slave to have synchronized clocks, since this is a problem that cannot be solved
and would result into race conditions and diverging data sets, so Redis
uses three main techniques in order to make the replication of expired keys
able to work:

1. Slaves don't expire keys, instead they wait for masters to expire the keys. When a master expires a key (or evict it because of LRU), it synthesizes a `DEL` command which is transmitted to all the slaves.
2. However because of master-driven expire, sometimes slaves may still have in memory keys that are already logically expired, since the master was not able to provide the `DEL` command in time. In order to deal with that the slave uses its logical clock in order to report that a key does not exist **only for read operations** that don't violate the consistency of the data set (as new commands from the master will arrive). In this way slaves avoid to report logically expired keys are still existing. In practical terms, an HTML fragments cache that uses slaves to scale will avoid returning items that are already older than the desired time to live.
3. During Lua scripts executions no keys expires are performed. As a Lua script runs, conceptually the time in the master is frozen, so that a given key will either exist or not for all the time the script runs. This prevents keys to expire in the middle of a script, and is needed in order to send the same script to the slave in a way that is guaranteed to have the same effects in the data set.

As it is expected, once a slave is turned into a master because of a fail over, it start to expire keys in an independent way without requiring help from its old master.

Configuring replication in Docker and NAT
---

When Docker, or other types of containers using port forwarding, or Network Address Translation is used, Redis replication needs some extra care, especially when using Redis Sentinel or other systems where the master `INFO` or `ROLE` commands output are scanned in order to discover slaves addresses.

The problem is that the `ROLE` command, and the replication section of
the `INFO` output, when issued into a master instance, will show slaves
as having the IP address they use to connect to the master, which, in
environments using NAT may be different compared to the logical address of the
slave instance (the one that clients should use to connect to slaves).

Similarly the slaves will be listed with the listening port configured
into `redis.conf`, that may be different than the forwarded port in case
the port is remapped.

In order to fix both issues, it is possible, since Redis 3.2.2, to force
a slave to announce an arbitrary pair of IP and port to the master.
The two configurations directives to use are:

    slave-announce-ip 5.5.5.5
    slave-announce-port 1234

And are documented in the example `redis.conf` of recent Redis distributions.
