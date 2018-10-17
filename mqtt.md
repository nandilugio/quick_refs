MQTT
====

Pub/Sub messaging protocol.

Topics
    No need to configure: publish or subscribe directly
    Hierarchical, `/`-separated
    Subscriptions
        Wildcards
            `+` One hierarchical level: `a/+/b` matches `a/x/b` and `a/y/b`
            `#` All remaining hierarchical levels `a/b/#` matches `a/b/x`, `a/b/y` and `a/b/c/d`
QoS
    The broker/client will deliver the message:
        0: once, with no confirmation
        1: at least once, with confirmation required
        2: exactly once by using a four step handshake
Other
    https://mosquitto.org/man/mqtt-7.html
        Retained Messages
        Clean session / Durable connections
        Wills

Broker: Mosquitto
-----------------

Bridges
    Connections between borkers.
        Can be constrained to particular topics, and particular directions (in|out|both).
        Remote topics can be prefixed
        

