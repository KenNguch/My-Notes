# **RabbitMQ**

<p style="text-align: right">**Saturday, 01. January 2022 09:32pm**</p>

RabbitMQ is an open source message broker software that implements the Advanced Message Queuing Protocol (AMQP). RabbitMQ is lightweight and easy to deploy. A message broker is an architectural pattern for message validation, transformation and routing. It gives applications a common platform to send and receive messages and a safe place for messages to live until received. 

There are 3 important parts in the RabbitMQ setup:

- **Producer**. A program that sends messages is a producer. 
- **Queue**. Messages are stored inside a queue. It’s essentially a large message buffer. A queue is only bound by the host’s memory and disk limits.
- **Consumer**. A consumer is a program that mostly waits to receive messages.

       <p style="text-align: center">  **Producer -> Queue -> Consumer(C1 ,C2 can be more than one)**</p>
        
There can be multiple producers and consumers to a queue. But in normal use cases, we don’t need multiple producers, as putting messages into a queue is quite simple and fast. But consuming a message from the queue and performing a task based on the message normally takes time.

**RabbitMQ Benefits**

- **Connectability** and scalability through messaging (i.e., applications can connect to each other as components of a larger application)
- **Asynchronous** messaging (decoupling applications by separating sending and receiving data)
- **Robust** messaging for applications
- **Support**s a huge number of developer platforms
- **Open source** and commercially supported


11. Install RabbitMQ
	-     For this case i will use docker (To make your code more portable and share the same version of RabbitMQ with your developer colleagues, It is highly recommend to use Docker)
	-     For linux users (sudo apt install docker docker-compose)
	-     Add the following service to your project **docker-compose.yml** services :
			
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
	
	- This command will pull the rabbitmq:3-management-alpine image, create the container rabbitmq and start the service and webUI.
	
	
#### Credits.
- [RabbitMQ](https://www.rabbitmq.com/)
- [How To Set Up Rabbitmq With Docker Compose](https://x-team.com/blog/set-up-rabbitmq-with-docker-compose/) 