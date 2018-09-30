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

