# Kafka 설치 및 테스트하기

## 1. JDK 1.8+ 설치

```sh
java -version
sudo yum -y install java-1.8.0
sudo yum -y remove java-1.7.0-openjdk
```

## 2. Confluent Platform 설치하기

Confluent Platform 은 Kafka 와 보조라이브러리를 포함하는 Open Source 플렛폼이다.

```sh
$ sudo rpm --import http://packages.confluent.io/rpm/3.2/archive.key

$ sudo vi /etc/yum.repos.d/epel.repo
-----------------------------------------------------------
......
[Confluent.dist]
name=Confluent repository (dist)
baseurl=http://packages.confluent.io/rpm/3.2/7
gpgcheck=1
gpgkey=http://packages.confluent.io/rpm/3.2/archive.key
enabled=1

[Confluent]
name=Confluent repository
baseurl=http://packages.confluent.io/rpm/3.2
gpgcheck=1
gpgkey=http://packages.confluent.io/rpm/3.2/archive.key
enabled=1
......
-----------------------------------------------------------

$ sudo yum clean all

$ sudo yum -y install confluent-platform-oss-2.11
```

## 3. Kafka 실행하기

아래 명령으로 zookeeper 와 Kafka 를 데몬으로 실행시킬 수 있다.

```sh
$ sudo /usr/bin/zookeeper-server-start -daemon /etc/kafka/zookeeper.properties
$ tail -100 /var/log/kafka/zookeeper.out
......
[2018-07-16 11:36:08,225] INFO binding to port 0.0.0.0/0.0.0.0:2181 (org.apache.zookeeper.server.NIOServerCnxnFactory)

$ sudo /usr/bin/kafka-server-start -daemon /etc/kafka/server.properties
$ tail -100 /var/log/kafka/kafkaServer.out
......
[2018-07-16 11:39:18,769] INFO [Kafka Server 0], started (kafka.server.KafkaServer)
[2018-07-16 11:39:18,786] INFO Waiting 10070 ms for the monitored broker to finish starting up... (io.confluent.support.metrics.MetricsReporter)
[2018-07-16 11:39:28,857] INFO Monitored broker is now ready (io.confluent.support.metrics.MetricsReporter)
[2018-07-16 11:39:28,858] INFO Starting metrics collection from monitored broker... (io.confluent.support.metrics.MetricsReporter)
```

## 4. 토픽 생성하기

토픽은 RDB 에서 테이블과 비슷한 개념이다.

```sh
$ /usr/bin/kafka-topics --create \
    --zookeeper localhost:2181 \
    --replication-factor 1 \
    --partitions 1 \
    --topic test
Created topic "test".
```

아래 명령으로 생성된 토픽을 확인할 수 있다.

```sh
$ /usr/bin/kafka-topics --list \
    --zookeeper localhost:2181
test
```

아래 명령으로 토픽에 데이타를 전송(producer)할 수 있다.
Ctrl-C 를 입력해 종료할 수 있다.

```sh
$ /usr/bin/kafka-console-producer \
    --broker-list localhost:9092 \
    --topic test
```

아래 명령으로 생성된 데이타를 수신(consumer)할 수 있다.
Ctrl-C 를 입력해 종료할 수 있다.

```sh
$ /usr/bin/kafka-console-consumer \
    --zookeeper localhost:2181 \
    --topic test \
    --from-beginning
```