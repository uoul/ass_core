 # Alert Support System - Core
 The ASS-Core contains the basic setup of ass using Docker-Compose. All configuration is done via the docker-compose environment.
 >**_NOTE:_** This Repository contains the basic setup! If you need more configuration, read the documentations of every used service!
## Used services
1. RabbitMQ
2. ass_gateway
3. ass_sms77notifier

### RabbitMQ
The RabbitMQ is used as messaging service between all components in ASS. So it's the heart of all ASS parts.

### ass_gateway
The ass_gateway is responsible for message producing on RabbitMQ. It reads the current messages via TCP socket from the Alu2g device and publish the current alerts plus new alerts on RabbitMQ.

### ass_sms77notifier
The ass_sms77notifier is a service, that consumes the "new alerts" from RabbitMQ and sends sms notifications every new alert. It supports two groups of messaged:
1. Full information -> that means, that sent sms contains detailed information about the alert
2. Min information -> only notification about new alerts (without any detail)

## Docker-Compose minimal setup
```yaml
version: "3.2"
services:
  rabbitmq:
    image: rabbitmq:3-management-alpine
    container_name: 'rabbitmq'
    ports:
      - 5672:5672
      - 15672:15672
    restart: on-failure
    healthcheck:
        test: [ "CMD", "rabbitmqctl", "status"]
        interval: 5s
        timeout: 20s
        retries: 5
    networks:
      - ass_net
  ass_gateway:
    image: uoul/ass-core-gateway:latest
    container_name: 'ass_gateway'
    depends_on:
      rabbitmq:
        condition: service_healthy
    environment:
      - RABBITMQ_HOST=rabbitmq
    restart: on-failure
    networks:
      - ass_net
  ass_sms77notifier:
    image: uoul/ass-core-sms77notifier:latest
    container_name: 'ass_sms77notifier'
    depends_on:
      rabbitmq:
        condition: service_healthy
    environment:
      - RABBITMQ_HOST=rabbitmq
      - SMS77NOTIFIER_API_KEY=<SMS77_API_KEY>
      - SMS77NOTIFIER_SENDER=<SENDER_NAME>
      - SMS77NOTIFIER_SMSGROUP_MAX_INFO=<SMS77_CONTACT_GROUP>
      - SMS77NOTIFIER_SMSGROUP_MIN_INFO=<SMS77_CONTACT_GROUP>
    restart: on-failure
    networks:
      - ass_net
networks:
  ass_net:
    driver: bridge
```

## How to setup?
1. save Docker-compose file to your machine
2. Do necessary configuration --> minimal is
   1. <SMS77_API_KEY> = generated api key from sms77 portal
   2. <SENDER_NAME> = sender name, that should be seen as sender on mobile
   3. <SMS77_CONTACT_GROUP> = in your sms77 phonebook, the group name, that should get corresponding information
3. Start Docker-compose 
```shell
docker-compose -f "<your_docker_compose_file.yml>" up
```
1. Finished!