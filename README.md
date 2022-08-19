 # Alert Support System - Core
 The ASS-Core contains the basic setup of ass using Docker-Compose. All configuration is done via the docker-compose environment.
 >**_NOTE:_** This Repository contains the basic setup! If you need more configuration, read the documentations of every used service!
## Used services
1. RabbitMQ
2. was_gateway
3. sms77notifier

### RabbitMQ
The RabbitMQ is used as messaging service between all components in ASS. So it's the heart of all ASS parts.

### was_gateway
The was_gateway is responsible for message producing on RabbitMQ. It reads the current messages via TCP socket from the Alu2g device and publish the current alerts plus new alerts on RabbitMQ.

### sms77notifier
The sms77notifier is a service, that consumes the "new alerts" from RabbitMQ and sends sms notifications every new alert. It supports two groups of messaged:
1. Full information -> that means, that sent sms contains detailed information about the alert
2. Min information -> only notification about new alerts (without any detail)

## Docker-Compose basic setup
```yaml
version: "3.2"
services:
  rabbitmq:
    image: rabbitmq:3-management-alpine
    container_name: 'rabbitmq'
    ports:
      - 5672:5672
      - 15672:15672
  was_gateway:
    image: uoul/ass-core-was_gateway:latest
    container_name: 'was_gateway'
    ports:
      - 8080:8080
    depends_on:
      - rabbitmq
    environment:
      - RABBITMQ_HOST=rabbitmq
  sms77notifier:
    image: uoul/ass-core-sms77notifier:latest
    container_name: 'sms77notifier'
    ports:
      - 8081:8081
    depends_on:
      - rabbitmq
      - was_gateway
    environment:
      - RABBITMQ_HOST=rabbitmq
      - SMS77NOTIFIER_API_KEY=<sms77_api_key>
      - SMS77NOTIFIER_SENDER=<name>
      - SMS77NOTIFIER_SMSGROUP_FULL_INFO=<full_info_group_name>
      - SMS77NOTIFIER_SMSGROUP_MIN_INFO=<min_info_group_name>
```

## How to setup?
1. save Docker-compose file to your machine
2. Do necessary configuration --> minimal is
   1. <sms77_api_key> = generated api key from sms77 portal
   2. <name> = sender name, that should be seen as sender on mobile
   3. <full_info_group_name> = in your sms77 phonebook, the group name, that should get the full information
   4. <min_info_group_name> = in your sms77 phonebook, the group name, that should only get minimal notification
3. Start Docker-compose 
```shell
docker-compose -f "<your_docker_compose_file.yml>" up
```
4. Finished!