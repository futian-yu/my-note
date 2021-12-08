**kafka命令行查看consumer以及topic消费情况**

```shell
#####其中
TOPIC：该group里消费的topic名称
PARTITION：分区编号
CURRENT-OFFSET：该分区当前消费到的offset
LOG-END-OFFSET：该分区当前latest offset
LAG：消费滞后区间，为LOG-END-OFFSET-CURRENT-OFFSET，具体大小需要看应用消费速度和生产者速度，一般过大则可能出现消费跟不上，需要引起应用注意
CONSUMER-ID：server端给该分区分配的consumer编号
HOST：消费者所在主机
CLIENT-ID：消费者id，一般由应用指定

#0.查看所有topic
bin/kafka-topics.sh --zookeeper 10.2.8.72:2181 --list
#1.获取group列表，一般而言，应用是知道消费者group的，通常在应用的配置里，如果已知，该步骤可以省略
bin/kafka-consumer-groups.sh --bootstrap-server 10.2.8.72:9092 --list
#2.查看指定group下各个topic的具体消费情况
bin/kafka-consumer-groups.sh --bootstrap-server 10.2.8.72:9092 --group crs-protocol-group --describe
#3.查看某个topic的内容
bin/kafka-console-consumer.sh --bootstrap-server 10.2.8.72:9092 --topic crs-protocol-flight-platform --from-beginning
#4.创建一个新的topic
bin/kafka-topics.sh --zookeeper 10.2.8.72:2181 --create --topic crs-flight-platform

```

