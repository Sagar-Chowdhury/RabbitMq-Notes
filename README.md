
![image](https://github.com/Sagar-Chowdhury/RabbitMq-Notes/assets/76145064/3383839c-138a-42ca-9201-1694ad6c15aa)


**What is RabbitMQ?**

(`Overall Architecture`)
![image](https://github.com/Sagar-Chowdhury/RabbitMq-Notes/assets/76145064/09b156f9-d671-4eff-99d5-d61c54bfc8d4)


* **Message Broker:** RabbitMQ is a robust, open-source message broker that acts as a central hub for asynchronous communication between applications. 
* **Message Queuing:** At its core, it stores and forwards messages ensuring reliable delivery, even if parts of your system temporarily fail.
* **AMQP:**  RabbitMQ primarily implements the Advanced Message Queuing Protocol (AMQP), an industry standard for message-oriented middleware, providing high compatibility with various technologies. However, it supports other protocols like MQTT and STOMP.

**Key Concepts**

1. **Producers:** Applications that send messages to RabbitMQ.
2. **Consumers:**  Applications that receive messages from RabbitMQ.
3. **Queues:** Named buffers within RabbitMQ where messages are stored. A producer sends messages to a queue, and consumers fetch them from the queue.
4. **Exchanges:**  Entry points for messages coming from producers. Exchanges route messages to appropriate queues based on rules called bindings.
5. **Bindings:**  Associations between exchanges and queues that establish the routing logic.
6. **Routing Key:** A piece of data included in a message sent by a producer. The exchange uses this routing key, along with bindings, to determine where to send the message.
7. **V-Host:** Vhosts (Virtual Hosts) in RabbitMQ provides a way to segregate applications using the same RabbitMQ instance. RabbitMQ vhosts creates a logical group of connections, exchanges, queues, bindings, user permissions, etc. within an instance.

**Messaging Patterns**

* **One-to-One (Simple Queues):**  One producer sends messages to a queue, and one consumer receives them.
* **Work Queues:** Multiple consumers share the work from a single queue, distributing tasks.
* **Publish/Subscribe (Fanout Exchange):** An exchange broadcasts a message to all bound queues and their consumers.
* **Topic-Based Routing (Topic Exchange):** Messages have routing keys. Exchanges route messages to queues based on matching patterns between the routing key and the binding key.

**Why RabbitMQ?**

* **Decoupling:** Applications communicate indirectly,  increasing system flexibility and maintainability.
* **Reliability:** RabbitMQ ensures message delivery with features like acknowledgments and durable queues.
* **Scalability:** RabbitMQ handles high message throughput and can cluster for horizontal scaling.
* **Cross-Platform:** Implementations in many languages, ensuring integration across diverse systems.

**Getting Started - Basic Message Sending and Receiving**

![image](https://github.com/Sagar-Chowdhury/RabbitMq-Notes/assets/76145064/1cb63ac9-15ae-487f-aa23-0e216f3133c8)

```JavaScript
// send.js

const amqp = require("amqplib/callback_api");

amqp.connect("amqp://localhost", function (error0, connection) {
  if (error0) throw error0;

  connection.createChannel(function (error1, channel) {
    if (error1) throw error1;

    var queue = "hello";
    var msg = "Hello World";

    channel.assertQueue(queue, {
      durable: false,
    });

    channel.sendToQueue(queue, Buffer.from(msg));
    console.log(" [x] Sent %s", msg);
  });
  setTimeout(function() {
    connection.close();
    process.exit(0);
}, 500);
});
```
```JavaScript
//receive.js

var amqp = require('amqplib/callback_api');

amqp.connect('amqp://localhost', function(error0, connection) {
    if (error0) {
        throw error0;
    }
    connection.createChannel(function(error1, channel) {
        if (error1) {
            throw error1;
        }

        var queue = 'hello';

        channel.assertQueue(queue, {
            durable: false
        });

        console.log(" [*] Waiting for messages in %s. To exit press CTRL+C", queue);

        channel.consume(queue, function(msg) {
            console.log(" [x] Received %s", msg.content.toString());
        }, {
            noAck: true
        });
    });
});

```
**Explantion**
Let's break down your RabbitMQ code example and the concepts involved:

**Architecture**

This example demonstrates the classic publish/subscribe pattern using RabbitMQ:

* **Producer (`send.js`)**: Sends messages to a queue.
* **Queue (`hello`)**:  A named buffer within RabbitMQ that stores the messages.
* **Consumer (`receive.js`)**:  Subscribes to the queue and processes received messages.

**Components and Concepts**

* **AMQP:**  The core messaging protocol used by RabbitMQ.
* **amqplib:**  The Node.js client library to interact with RabbitMQ.
* **Connection:**  A TCP connection established between your applications and the RabbitMQ server.
* **Channel:**  A virtual connection within an AMQP connection. It's a lightweight communication pathway for sending and receiving messages.
* **Queue (`hello`)**: The named message buffer on the RabbitMQ server. In both scripts, you declare the queue to ensure its existence.
* **Durable: false:**   The queue is not persistent. Messages are lost if the RabbitMQ server restarts.
* **Message (`Hello World`)**: The data payload sent from the producer to the consumer.
* **noAck: true:** Automatic message acknowledgment. You signal to RabbitMQ that the message was successfully processed (not ideal for production scenarios where you might want to handle failures).

**Code Breakdown**

**send.js**

1. **Imports `amqplib`**.
2. **Connects to RabbitMQ** (`amqp://localhost` is the default RabbitMQ connection string).
3. **Creates a channel**.
4. **Asserts the queue** (`channel.assertQueue`). This ensures the queue exists before sending messages.
5. **Sends the message** (`channel.sendToQueue`).
6. **Logs to console and closes the connection**.

**receive.js**

1. Imports `amqplib`.
2. **Connects to RabbitMQ**.
3. **Creates a channel**.
4. **Asserts the queue**.
5. **Logs a waiting message for visual indication**.
6. **Defines a consumer** (`channel.consume`). This subscribes to the queue. Each time a message arrives:
     * **Logs the message to the console**.
     * **Automatically acknowledges the message (not ideal in production environments)**.



**Work Queues**

The main idea behind Work Queues (aka: Task Queues) is to **avoid doing a resource-intensive task immediately** and having to wait for it to complete. Instead we schedule the task to be done later. We encapsulate a task as a message and send it to a queue. A worker process running in the background will pop the tasks and eventually execute the job. When you run many workers the tasks will be shared between them.

This concept is especially useful in web applications where it's impossible to handle a complex task during a short HTTP request window.

**Illustrative Example**

(`task-queue-producer.js`)

```JavaScript



const amqp = require("amqplib/callback_api")

amqp.connect("amqp://localhost",function(error0,connection){
    if(error0)throw error0;
    
    connection.createChannel(function(error1,channel){
        if(error1) throw error1

        var queue = "task_queue"
        
        // Simulate multiple tasks
        for(var i=1;i<=20;i++){
            var msg = "Task number"+i
            channel.sendToQueue(queue,Buffer.from(msg))
            console.log(" [x] Sent %s",msg)
        }
        
        setTimeout(function(){
           connection.close()
           process.exit(0) 
        },500)

    })

})

```
(`work-consumer.js`)

```JavaScript
const amqp = require("amqplib/callback_api");

amqp.connect("amqp://localhost", function (error0, connection) {
  if (error0) throw error0;

  connection.createChannel(function (error1, channel) {
    if (error1) throw error1;

    var queue = "task_queue";

    channel.assertQueue(queue, {
      durable: false,
    });

    channel.consume(queue, function (msg) {
      console.log("[x] Received %s", msg);

      // Simulate processing time
      setTimeout(function () {
        console.log(" [x] Done", msg.content.toString());
        channel.ack(msg); // Acknowledge the message after processing
      }, 1000),
        {
          // Prefetch count: Limit the number of unacknowledged messages per worker
          prefetch: 1,
        };
    });
  });
});

```

**Explanation**

* **task-queue-producer.js**:
    * Connects to RabbitMQ.
    * Creates a channel and defines a queue named "task_queue".
    * Sends 10 messages (simulating tasks) to the queue.
    * Closes the connection after sending messages.

* **work-consumer.js**:
    * Connects to RabbitMQ.
    * Creates a channel and asserts the "task_queue".
    * Defines a consumer that listens for messages on the queue.
    * When a message arrives:
        * Logs the message content (task).
        * Simulates processing time (replace with your actual work).
        * Logs completion and acknowledges the message with `channel.ack(msg)`. Unacknowledged messages might be redelivered if a worker crashes.
    * Sets `prefetch: 1` to ensure only one unacknowledged message is delivered to the worker at a time.



**Key Concepts**

**Round-robin dispatching** :- By default, RabbitMQ will send each message to the next consumer, in sequence. On average every consumer will get the same number of messages. This way of distributing messages is called round-robin.

**Message Acknowledgement** :- 

In order to make *sure a message is never lost*, RabbitMQ supports message acknowledgments. *An ack(nowledgement) is sent back by the consumer* to tell RabbitMQ that a particular message has been received, processed and that RabbitMQ is *free to delete it*.

If a consumer dies (its channel is closed, connection is closed, or TCP connection is lost) without *sending an ack*, RabbitMQ will understand that a message wasn't processed fully and *will re-queue it*. If there are other consumers online at the same time, it will then quickly redeliver it to another consumer. That way you can be sure that no message is lost, even if the workers occasionally die.

```JavaScript
channel.consume(queue, function(msg) {
  var secs = msg.content.toString().split('.').length - 1;

  console.log(" [x] Received %s", msg.content.toString());
  setTimeout(function() {
    console.log(" [x] Done");
    channel.ack(msg);
  }, secs * 1000);
  }, {
    // manual acknowledgment mode,
    // see /docs/confirms for details
    noAck: false
  });

```
Using this code, you can ensure that even if you terminate a worker using CTRL+C while it was processing a message, *nothing is lost*. Soon after the worker terminates, all unacknowledged messages are delivered.

**Message Durability**

We have learned how to make sure that even if the consumer dies, the task isn't lost. But our tasks will **still be lost if RabbitMQ server stops**.

When RabbitMQ quits or crashes it will forget the queues and messages unless you tell it not to. Two things are required to make sure that messages aren't lost: we need to mark both the queue and messages as durable.

```JavaScript
channel.assertQueue('task_queue', {durable: true});
```
```JS
channel.sendToQueue(queue, Buffer.from(msg), {persistent: true});
```

### Fair Dispatch

You might have noticed that the dispatching still doesn't work exactly as we want. For example in a situation with two workers, when all odd messages are heavy and even messages are light, one worker will be constantly busy and the other one will do hardly any work. Well, RabbitMQ doesn't know anything about that and will still dispatch messages evenly.

This happens because *RabbitMQ just dispatches a message when the message enters the queue*. It doesn't look at the **number of unacknowledged messages for a consumer**. It just blindly dispatches every n-th message to the n-th consumer.

![image](https://github.com/Sagar-Chowdhury/RabbitMq-Notes/assets/76145064/f5ac81f2-359b-4b33-a987-db553594436d)



To defeat that we can use the prefetch method with the value of 1. This tells RabbitMQ **not to give more than one message to a worker at a time**. Or, in other words, don't dispatch a new message to a worker until it has processed and acknowledged the previous one. Instead, it will dispatch it to the next worker that is not still busy.

### Exchanges

An exchange is a very simple thing. On one side it *receives messages from producers* and the other side it *pushes them to queues*. The exchange must know exactly what to *do with a message it receives*. Should it be appended to a particular queue? Should it be appended to many queues? Or should it get discarded. The rules for that are defined by the exchange type.

![image](https://github.com/Sagar-Chowdhury/RabbitMq-Notes/assets/76145064/86304d40-bae3-474a-8917-07631d21dcdc)


Basic **Fanout** Exchange. (`Broadcasting to all Consumers`) 

(`emit-log-producer.js`)

```JS
var amqp = require("amqplib/callback_api");

amqp.connect("amqp://localhost", function (error0, connection) {
  if (error0) throw error0;

  connection.createChannel(function (error1, channel) {
    if (error1) throw error1;

    var exchange = "logs";
    var msg = "Message Sent Via Exchange";

    channel.assertExchange(exchange, "fanout", {
      durable: false,
    });
   
    for(var i=1;i<=50;i++){
    channel.publish(exchange, "", Buffer.from(msg));
    console.log("[x] Sent %s", msg);
    }
  });


  setTimeout(function () {
    connection.close();
    process.exit(0);
  }, 500);
});

```

(`receive-log-consumer.js`)

```JS
var amqp = require("amqplib/callback_api");

amqp.connect("amqp://localhost", function (error0, connection) {
  if (error0) {
    throw error0;
  }
  connection.createChannel(function (error1, channel) {
    if (error1) {
      throw error1;
    }
    var exchange = "logs";

    channel.assertExchange(exchange, "fanout", {
      durable: false,
    });

    channel.assertQueue(
      "",
      {
        exclusive: true,
      },
      function (error2, q) {
        if (error2) {
          throw error2;
        }
        console.log(
          " [*] Waiting for messages in %s. To exit press CTRL+C",
          q.queue
        );
        channel.bindQueue(q.queue, exchange, "");

        channel.consume(
          q.queue,
          function (msg) {
            if (msg.content) {
              console.log(" [x] %s", msg.content.toString());
            }
          },
          {
            noAck: true,
          }
        );
      }
    );
  });
});

```

***Points to Note***

   ```JS
   channel.publish('logs', '', Buffer.from('Hello World!'));
   ```
The empty string as second parameter means that we don't want to send the message to *any specific queue*. We want only to publish it to our 'logs' exchange.     
```JS
channel.assertQueue('', {
  exclusive: true
});
```

The empty string as second parameter means that we don't want to send the message to *any specific queue*. We want only to publish it to our 'logs' exchange.
Giving a queue a *name is important when you want to share the queue between producers and consumers*.
But that's not the case for our logger. We want to hear about all log messages, not just a subset of them. We're also interested only in currently flowing messages 
not in the old ones. To solve that we need two things.


*Randomly generated queue-names*

![image](https://github.com/Sagar-Chowdhury/RabbitMq-Notes/assets/76145064/599a7302-3c86-47b7-b5f8-59e16a553a1e)



(`Output Illustration` - ` A Typical Fanout Exchange`)

![image](https://github.com/Sagar-Chowdhury/RabbitMq-Notes/assets/76145064/91ec6589-4f61-404d-b3f4-d0ca49bec795)

**Exchange Types**

In RabbitMQ there are four main types of Exchanges:

Direct
Topic
Fanout
Headers

**Taxi Company Example: RabbitMQ Exchange Types**

**Direct Exchange**

* **Purpose:** Assigning specific taxi requests where the user wants a particular driver by preference.
* **Routing Logic:** Exact matches between the message's routing key and a queue's binding key.
* **Example:**
    * **Routing key:** "driver-id-1234" 
    * **Queue:** Bound to "driver-id-1234"
    * **Message:** Ride request for driver with ID 1234

**Topic Exchange**

* **Purpose:** Routing ride requests based on taxi attributes or customer preferences. 
* **Routing Logic:** Pattern matching between message's routing key and queues' binding keys. Uses '*' (match a single word) and '#' (match zero or more words).
* **Example 1 (Environmental Taxi):**
    * **Routing key:** "vehicle.large.eco"
    * **Queues:** 
         * Bound to "vehicle.large.eco" (exact match)
         * Bound to "vehicle.large.*" (all large vehicles) 
* **Example 2 (Large Taxi, Any Type):**
    * **Routing key:** "vehicle.large"
    * **Queues:** Bound to "vehicle.large.*" 

**Fanout Exchange**

* **Purpose:** Broadcast-style messages to all taxis. 
* **Routing Logic:** Ignores routing keys; sends copies of the message to all bound queues.
* **Example:**
    * **Routing key:** (Not relevant)
    * **Queues:** All taxi driver queues
    * **Message:** Urgent traffic alert about a road closure

**Headers Exchange**

* **Purpose:** Filtering messages based on message headers rather than simple routing keys. Allows for more complex routing scenarios.
* **Routing Logic:** Matches messages based on header values (key-value pairs)
* **Example:**
    * **Header:**  "priority" : "high"
    * **Queues:** Bound to headers with "priority" set to "high"
    * **Message:** High-paying ride opportunity 


**Virtual Hosts (vhosts) in RabbitMQ**

* **Purpose:**  Provide logical segregation of applications using the same RabbitMQ instance.

* **Concept:** Think of vhosts as mini-RabbitMQ servers within a single instance. Each vhost has its own:
    * Connections
    * Exchanges
    * Queues
    * Bindings
    * User permissions

* **Client Connection:** When a client connects to RabbitMQ, it specifies the target vhost.

* **Isolation:** Resources are not shared between vhosts, ensuring separation.

* **Creation:** Vhosts can be created via:
    * Management portal
    * HTTP API
    * `rabbitmqctl` command

* **Default vhost:** A default vhost (named "/") exists upon installation.

**Benefits of Using Vhosts**

* **Application Separation:** Isolate different applications on the same RabbitMQ broker.
* **Environment Management:** Create separate vhosts for production, staging, etc.
* **Improved Security:** Enforce granular user permissions per vhost.
* **Resource Management:** Manage the topology and resource usage of individual services.

**Important Note:** While vhosts offer logical separation, performance of one vhost can potentially impact others since they share the same physical resources. 





