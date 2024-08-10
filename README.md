# custom-NATS

NATS is an open-source messaging broker (also sometimes referred to as message-oriented middleware). It’s written in Go and you can find it’s source code in the nats.io [GitHub repository](https://github.com/nats-io/nats-server/).

Step Zero

In the majority of common programming languages we index arrays from the element zero onwards. Coding Challenges is the same, we start with Step 0 - setup your IDE / editor of choice and programming language of choice.

Step 1

In this step your goal is to parse the NATS protocol. Before jumping in it’s worth reading the NATS Protocols documentation. Then dust off your favourite telnet client and work through the Protocol Demo.

Once you’ve done that it’s time to begin the challenge! In this step, much like for the Redis Coding Challenge, we’re going to build a protocol parser.

NATS uses a a zero allocation byte parser which you can see in the NATS source code. If you’ve not come across it before this is an interesting technique to learn.

I would suggest using TDD to handle this, so you can develop a parser to handle the following commands:

CONNECT - just with an empty set of options for now.
PING and PONG - an easy place to start.
SUB - For the moment feel free to ignore the non-required fields.
PUB - For the moment feel free to ignore the non-required fields.
Some example test cases to consider are:

PING\r\n

PONG\r\n

SUB FOO 1\r\n

PUB CodingChallenge 11\r\nHello John!\r\n

When you’re happy with your parser move on to Step 2.

Step 2

In this step your goal is to create the bones of the server, handle a client connection and respond to PING and PONG messages.

Your server should start up and begin listening for clients connecting to port: 4222

When a client connects you should send the INFO message, followed by the host, port and client IP address, then wait for commands. For this step your server should accept CONNECT {} and respond with +OK. It should also respond to PING requests with PONG.

When you’ve completed this step you should be able to use telnet to connect to your server and carry out this sequence:

% telnet localhost 4222
Trying ::1...
Connected to localhost.
Escape character is '^]'.
INFO {"host":"0.0.0.0","port":4222,"client_ip":"172.17.0.1"}
CONNECT {}
+OK
PING
PONG
Step 3

In this step your goal is to allow a client to subscribe to a topic, publish a message to the topic and receive messages published to that topic.

Your server will need to parse the SUB command and add the client to a list of subscribers to a topic. It will also need to parse the PUB command and then publish the message it receives to all of the subscribers of that topic.

Your goal is to be able to connect to do something like this (the client is both the publisher and the subscriber):

CONNECT {}
+OK
SUB CC 10
+OK
PUB CC 4
JOHN
+OK
MSG CC 10 4
JOHN
Step 4

In this step your goal is to handle multiple concurrent clients. If you haven’t already done so, now is the time to make your server handle concurrent clients. You have two basic options here; have one thread per client or use the asynchronous programming support offered by your chosen programming language. If you have time, give both a go, both have pros and cons.

Test your solution by connecting two telnet sessions and when you’re happy you can check out the nats bench tool to benchmark the performance of your server.

For example:

% nats bench coding.challenge --pub 1 --size 16 --msgs 1000
16:07:22 Starting Core NATS pub/sub benchmark [subject=coding.challenge, multisubject=false, multisubjectmax=0, msgs=1,000, msgsize=16 B, pubs=1, subs=0, pubsleep=0s, subsleep=0s]
16:07:22 Starting publisher, publishing 1,000 messages
Finished      0s [==============================================================================================================] 100%

Pub stats: 251,140 msgs/sec ~ 3.83 MB/sec
Step 5

In this step your goal is to extend you server to handle a client unsubscribing. Once the client is unsubscribed they should still be able to publish messages, but they will no longer see the message (as they no longer have an active subscription.

That should look like this:

CONNECT {}
+OK
SUB CC 10
+OK
PUB CC 5
Hello
+OK
MSG CC 10 5
Hello
UNSUB 1-
+OK
PUB CC 3
bye
+OK
You should now have the core of the NATS MoM server - try connecting a few clients and sending messages from a published to a couple of fanned out clients.
