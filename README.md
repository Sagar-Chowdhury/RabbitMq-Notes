
![image](https://github.com/Sagar-Chowdhury/RabbitMq-Notes/assets/76145064/3383839c-138a-42ca-9201-1694ad6c15aa)


**What is RabbitMQ?**

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






