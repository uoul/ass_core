 # Alert Support System - Core
 The ASS-Core contains the basic setup of ass using Docker-Compose. All configuration is done via the docker-compose environment.
 >**_NOTE:_** This Repository contains the basic setup! If you need more configuration, read the documentations of every used service!
## Used services
1. [RabbitMQ](https://hub.docker.com/_/rabbitmq)
2. [ass_core_gateway](https://hub.docker.com/repository/docker/uoul/ass-core-gateway/general)
3. [ass_core_sms77notifier](https://hub.docker.com/repository/docker/uoul/ass-core-sms77notifier/general)
4. [ass_core_ui_backend](https://hub.docker.com/r/uoul/ass-core-ui-backend)
5. [ass_core_ui_frontend](https://hub.docker.com/r/uoul/ass-core-ui-frontend)

### RabbitMQ
The RabbitMQ is used as messaging service between all components in ASS. So it's the heart of all ASS parts.

### ass_core_gateway
The ass_core_gateway is responsible for message producing on RabbitMQ. It reads the current messages via TCP socket from the Alu2g device and publish the current alerts plus new alerts on RabbitMQ.

### ass_core_sms77notifier
The ass_core_sms77notifier is a service, that consumes the "new alerts" from RabbitMQ and sends sms notifications every new alert. It supports two groups of messaged:
1. Full information -> that means, that sent sms contains detailed information about the alert
2. Min information -> only notification about new alerts (without any detail)

### ass_core_ui_backend
The ass_core_ui_backend provides the information cached from RabbitMq for ass_core_ui_frontend.

### ass_core_ui_frontend
The ass_core_ui_frontend is a small React application for visualizing current alerts. The ui initialy fetch alert-data from backend and subscribes for updates on backend,
so that a page reload is not necessary(using server-sent-events).

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
  ass_core_gateway:
    image: uoul/ass-core-gateway:latest
    container_name: 'ass_core_gateway'
    depends_on:
      rabbitmq:
        condition: service_healthy
    environment:
      - RABBITMQ_HOST=rabbitmq
    restart: on-failure
    networks:
      - ass_net
  ass_core_sms77notifier:
    image: uoul/ass-core-sms77notifier:latest
    container_name: 'ass_core_sms77notifier'
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
  ass_core_ui_backend:
    image: uoul/ass-core-ui-backend:latest
    container_name: 'ass_core_ui_backend'
    ports:
      - 3500:3500
    depends_on:
      rabbitmq:
        condition: service_healthy
    environment:
      - RABBITMQ_HOST=rabbitmq
    restart: on-failure
    networks:
      - ass_net
  ass_core_ui_frontend:
    image: uoul/ass-core-ui-frontend
    container_name: 'ass_core_ui_frontend'
    ports:
      - 3000:3000
    environment:
      - REACT_APP_API_ADDR=<ASS_UI_BACKEND_SERVER_ADDRESS>
      - REACT_APP_API_PORT=3500
      - REACT_APP_SRC_ADDR=<NAV_SRC_ADDRESS>
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
   4. <ASS_UI_BACKEND_SERVER_ADDRESS> = server address (name/ip) of backend server - normaly this should be the docker-host ip address/name
   5. <NAV_SRC_ADDRESS> = source address for google maps module (ex.: Musterweg 12, 9999 Musterhausen)
3. Start Docker-compose 
```shell
docker-compose -f "<your_docker_compose_file.yml>" up
```