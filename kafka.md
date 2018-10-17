Delivery semantics
  Offsets are committed by publishing to a topic named `__consumer_offsets`
  - At most once: Offset commited on receive
    Message is lost if processing goes wrong
  - At least once: Offset commited after processing
    Messages are never lost
    There's risk of a message being processed twice (fetch and process, dies before commit, then fetch and process again):
      Processing needs to be _idempotent_
  - Exactly once: more complicated (TODO: how?)
    Implemented by Kafka Streams API (and therefore KSQL)

Broker discovery
  Every broker is a "bootstrap server", meaning
    Brokers deliver that metadata to clients on connect
    - All brokers know _all_ the metadata about the others
    - Clients decide which brocker to connect to reach a particular partition


Zookeper
  Handles metadata synchronization in the Kafka cluster
  It works with an odd number of nodes (3, 5, 7...), and doesn't need to be 1-to-1 with Kafka nodes
  Its cluster has a Leader handling writes and Folowers handling reads

###############################################################################

Apache
    Kafka (core)
        PubSub messaging system
        Fast
            ZeroCopy
            Sequential disk writes to immutable Log
        Scalable
            Nodes are all equal
            Read/writes are load-balanced by sharding of Log Partitions in machines/disks
        Fault-tolerant
            Log Partitions can be replicated
            All nodes can take over
            Clients see all cluster through any node ??
        Durable
            Data is persisted in the Topic Logs
                Re-reading to recover from failures is easy
            Retention can be done by time, size or compaction (kep last timestamp for key/second)
        Flexible
            Exactly-once guarantees ??
            Ordering preserved at shard (Partition) level
            Many subscription paradigms
                Enables fault-tolerant data consumption and workload distribution
                - Queue (consumption within consumer group)
                - PubSub topics MOM-like (broadcasting between consumer group)s

Confluent
    Kafka Connect
    Kafka Streams
        Distributed Streaming Platflorm
    Kafka REST Proxy
    Schema Registry


Datastream decoupling.....

Topic/Feed
    Is a name
    Log: persistence method
        Immutable list of Records
        Consist of Partitions
            Scalability
            Replication of partitions for falut-tolerance
                Topic Replication factor = number of replicas per partition ??
                Kafka replication: failover
                Mirror Maker: Disaster recovery (e.g. mirroring whole cluster between AWS availability zones)

Partition
    Can be distributed (sharded) for parallel insertion and consumption of topic's records
    Can be replicated for parallel consumption and for fault-tolerance
        Leader: for writes
        Follower: catches up with changes on leader. If it's an ISR (in-sync replica) it can take leadership over
    Producer PICKS the partition
        Strategies
            Key
            Data
            Hash
            Priority
            Round robin
    Producer decides consistency level
        ack=0
        ack=1
        ack=all
    Offset pointers:
        Log end offset: end, where producers write to next
        High watermark: behind it, all records are all successfully replicated to all followers



Record
    Immutable

Producer

Consumer group
    Its a suscriber
    Has a unique ID
    Maintains its own offset pointer
        In `__consumer_offset` topic
    Record goes to one of the consumer of each group (Pub/Sub)
        Consumers in group load-balance consumption (each consume a fair-share of partitions)
        Only one consumer can read a particular partition
            If more partitions than consumer
                Some consumers read from more than one partition
            If more consumers than partitions, extra are idle
                Idle ones serve as failover
    Consumers ack processing (advancing the offset pointer)
        If the consumer dies before acking, another consumer gets the data
            This is at-least-once ?? #TODO now exactly-once no ??
                If so: Messages should be idempotent!
    Group coordinator
        A Broker handling addition/dissapearing of consumers in a group
            Redistributes shares automatically
    Consumers can see only records already replicated to all replicas (high-watermark)
    Reset policy #TODO




Cluster
    ZooKeeper
        Cluster coordination
    Broker: Node
        Has a numeric ID
        Contain Log Partitions

Sharding
    Partition decided normally by key