AMQP - RabbitMQ
===============

[AMPQ](https://www.amqp.org/)
[AMPQ - Wikipedia](https://en.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol)
[RabbitMQ](https://www.rabbitmq.com/)
[RabbitMQ Docs](https://www.rabbitmq.com/documentation.html)

Producer --> Exchange --Binding--> Queue --> Consumer

Messages are sent by Producers to an Exchange, with a Routing Key. That key is used by the Exchange to decide which of its binded queues need to receive the message.

Bindings link Exchanges and Queues. Depending on the Exchange type, this bindings can be parametrized and aid in the routing decision.

One or more Consumers read from each Queue.

### Exchanges

Act like routers, forwarding messages to relevant queues. Takes decisions based (TODO: solely?) on the Routing Key.

- *Direct (empty)*: Forwards the message to the queue binded with __exactly__ the message's routing key.
- *Topic*: Decides destination queues by matching their routing keys (in this case a pattern) to the message's. [Tutorial](https://www.rabbitmq.com/tutorials/tutorial-five-python.html)
- *Fanout*: Distribute messages between binded queues (TODO: round robin right?)
- *Headers*: TODO

### Queues

[Docs](https://www.rabbitmq.com/queues.html)

Can be declared by both Producers and Consumers.

Can be named so they can be suscribed by different Consumers.

Can be set as `exclusive` for a client, so it's deleted after the client disconnects.


Messaging Patterns
------------------

### Publish/Subscribe

[RabbitMQ's tutorial](https://www.rabbitmq.com/tutorials/tutorial-three-python.html)

Exchange routes all messages to all queues, one per subscribed consumer.

                      +-> Queue --> Consumer
Producer --> Exchange-+
                      +-> Queue --> Consumer

### Work queues

[RabbitMQ's tutorial](https://www.rabbitmq.com/tutorials/tutorial-two-python.html)

Exchange routes messages to one queue to distribute work to equivalent workers. Message acknowledgment ensures no tasks (messages) are lost (at-least-once semantics).

                                 +-> Consumer
Producer --> Exchange --> Queue -+
                                 +-> Consumer

### Remote Procedure Calls

[RabbitMQ's tutorial](https://www.rabbitmq.com/tutorials/tutorial-six-python.html)

      /--request--> Queue --\
Client                       Server
      \- Queue <--response--/



Durability
----------

Queues (and messages), can be set as `durable`, so they survive broker restarts/crashes.

Note that this setting is not a 100% warranty. There is a short time between accepting the message and persisting it to disk, so if failure occurs in that period, the message will be lost. See Publisher Confirms to overcome this.


Consumer Acknowledgements and (RabbitMQ's) Publisher Confirms
-------------------------------------------------------------

[Docs](https://www.rabbitmq.com/confirms.html)

Acks can be issued by the consumer once the message is fully processed (doesn't need to be fast). If not ack'ed in a given timeout (TODO), messages are brought back to the queue for re-consumption.

In a similar manner, RabbitMQ publishers can request acks from the brocker ([protocol extension](https://www.rabbitmq.com/extensions.html)) and resend the message if a timeout elapses.


Transactions
------------

TODO


Other interesting topics
------------------------

From RabbitMQ's docs:
- [Production Checklist](https://www.rabbitmq.com/production-checklist.html)
- [Monitoring](https://www.rabbitmq.com/monitoring.html)