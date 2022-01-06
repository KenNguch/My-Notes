# **RabbitMQ**

<p style="text-align: right"> **Saturday, 01. January 2022 09:32pm** </p>

RabbitMQ is an open source message broker software that implements the Advanced Message Queuing Protocol (AMQP). RabbitMQ is lightweight and easy to deploy. A message broker is an architectural pattern for message validation, transformation and routing. It gives applications a common platform to send and receive messages and a safe place for messages to live until received. 

There are 3 important parts in the RabbitMQ setup:

- **Producer**. A program that sends messages is a producer. 
- **Queue**. Messages are stored inside a queue. It’s essentially a large message buffer. A queue is only bound by the host’s memory and disk limits.
- **Consumer**. A consumer is a program that mostly waits to receive messages.

    **Producer -> Queue -> Consumer(C1 ,C2 can be more than one)**
        
There can be multiple producers and consumers to a queue. But in normal use cases, we don’t need multiple producers, as putting messages into a queue is quite simple and fast. But consuming a message from the queue and performing a task based on the message normally takes time.

 **RabbitMQ Benefits**

- **Connectability** and scalability through messaging (i.e., applications can connect to each other as components of a larger application)
- **Asynchronous** messaging (decoupling applications by separating sending and receiving data)
- **Robust** messaging for applications
- **Support**s a huge number of developer platforms
- **Open source** and commercially supported

#### **Installing RabbitMQ**
* For this case i will use docker (To make your code more portable and share the same version of RabbitMQ with your developer colleagues, It is highly recommend to use Docker).
* For linux users (sudo apt install docker docker-compose)
* Add the following service to your project **docker-compose.yml** services :		
  ```
  dawnstar-rabbitmq:
    image: rabbitmq:3.9.11-management-alpine
    container_name: 'rabbitmq'
    ports:
      - "5672:5672"
      - "15672:15672"
    volumes:
      - ~/.docker-conf/rabbitmq/data/:/var/lib/rabbitmq/
      - ~/.docker-conf/rabbitmq/log/:/var/log/rabbitmq
    networks:
      - dawnstar
  ```

* This command will pull the **rabbitmq:3-management-alpine** image, create the container rabbitmq and start the service and webUI.
	
## USING NODE JS

What will we be building?
In this article, we will build two simple Node.js apps, that will communicate via RabbitMQ. First app, let’s call it sender, will provide an API to send something to the second app. Second app, let’s call it receiver, will be receiving messages from the sender app and printing them to the console.
Communication
In this article, we will use RabbitMQ to build communication between these apps. RabbitMQ is a message broker written in Erlang and using multiple protocols to communicate with a user (we will use AMQP).
Communication strategy is deadly simple. First app, aka sender, will check if a queue exists, create it if it doesn’t, and send a message to it. Second app, aka receiver, will subscribe for new messages from this queue and print received ones.

Action
I will skip the routing setup part, full code is available here. Actually, all we need to do is write an interface that will allow us to work with RabbitMQ easily. This interface will be a singleton class, that will have methods like subscribe to consume messages from a given queue, send to send messages to the given queue, and a static getInstance method to get singleton instance.
Let’s start with the getInstance method. This method should return a class instance (I called this class MessageBroker) with established RabbitMQ connection and create a channel. To communicate with RabbitMQ using AMQP protocol we will use the amqplib package. The connection method looks like this:

On the line 6 we create a connection and one the line 7 we create a channel to communicate with RabbitMQ.
Next method that we will add is the getInstance method, that will return singleton:

On the line 7 we declared an instance variable, that will contain a singleton instance. On the line 27 we check if instance variable already contains the singleton instance. If so, the method return instance variable, that may hold pending Promise, which will subsequently return us MessageBroker instance. If not, we will initialise created MessageBroker instance on the line 29, and assign returned pending Promise to instance value. This way we can avoid double connection.
Sending messages
Let’s add the send method to the MessageBroker class.

On the line 10, we use assertQueue method, that check a queue existence, and if it doesn’t, assertQueue will create the queue. This operation is idempotent given identical arguments. We set durable option to true, if true, the queue will survive broker restarts. On the line 11, we use sendToQueue method, that sends a single message with the content given as a buffer to the specific queue.
Subscription
To make subscription method the right way, we need to understand how RabbitMQ sending messages to its subscribers.
When an app subscribes for queue’s messages, RabbitMQ treats it as a worker. RabbitMQ will send new messages to the workers using round-robin algorithm.

So, our app should subscribe to a queue only once, but be able to register as many handlers as needed.

Firstly, we created queue class member and defined as an empty object. This member will store queues’ handlers, where a key is a queue name and a value is an array of registered handlers. On the line 23, we check if queues variable holds something. If true, we check if given handler already registered, if so, return unsubscribe function, if it’s not, push the handler to the array of handlers this.queues[queue].push(handler) and return unsubscribe function. If this.queues[queue] is not defined, we check the queue existence(line 32), and define this.queues[queue] with an array, that holds the handler (line 33).
Finally, we subscribe for the queue messages using this.channel.consume method. First argument is the queue’s name. Second is a message handler. To understand what is going on the lines 36–39, we should get to know about Message acknowledgment.
Message acknowledgment
If you kill a worker we will lose the message it was just processing. We’ll also lose all the messages that were dispatched to this particular worker but were not yet handled.
But we don’t want to lose any tasks. If a worker dies, we’d like the task to be delivered to another worker.
An ack(nowledgement) is sent back by the consumer to tell RabbitMQ that a particular message had been received, processed and that RabbitMQ is free to delete it.
There aren’t any message timeouts; RabbitMQ will redeliver the message when the consumer dies. It’s fine even if processing a message takes a very, very long time.
Triggering handlers
So, when new message is arrived, we create the function ack, that mark message as acknowledged, using _.once to make the function execute only once. Then, we iterate over the handlers array this.queues[queue].forEach(h => h(msg, ack)), triggering every handler with the message and acknowledge function.
Sending and receiving
Sender controller:

Here, we are getting the MessageBroker instance and calling send method with queue name equal to test and message equal to Buffer with stringified body.
Receiving controller:

Here, we are getting the MessageBroker instance and calling subscribe method with queue name and handler function. The handler will be printing incoming messages’ contents and acknowledge them right after.
Let’s send a message.

Printed in console of receiver app:
Message: {“msg”:”Hello”}



#### Credits.
- [RabbitMQ](https://www.rabbitmq.com/)
- [How To Set Up Rabbitmq With Docker Compose](https://x-team.com/blog/set-up-rabbitmq-with-docker-compose/) 