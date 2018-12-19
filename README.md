# kafka-sf
Kafka Service Fabric App

Deploy using:
```powershell
Remove-ServiceFabricComposeDeployment -DeploymentName app -Force
New-ServiceFabricComposeDeployment -DeploymentName app -Compose kafka.yaml
```

Note:
If you change the `DeploymentName` above, change the `dns` name of the service in `kafka.yaml` as well.

Verify zookeeper is up:
```bash
`ssh` into nodes and see that there is a leader and multiple followers.

nt1vm000000:~$ echo stat | nc localhost 2181
Zookeeper version: 3.4.13-2d71af4dbe22557fda74f9a9b4309b15a7487f03, built on 06/29/2018 04:05 GMT
Clients:
 /172.17.0.1:59886[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/0/0
Received: 4
Sent: 3
Connections: 1
Outstanding: 0
Zxid: 0x100000000
Mode: leader
Node count: 4
Proposal sizes last/min/max: -1/-1/-1
asnegi@nt1vm000000:~$


nt1vm000001:~$ echo stat | nc localhost 2181
Zookeeper version: 3.4.13-2d71af4dbe22557fda74f9a9b4309b15a7487f03, built on 06/29/2018 04:05 GMT
Clients:
 /172.17.0.1:55516[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/0/0
Received: 3
Sent: 2
Connections: 1
Outstanding: 0
Zxid: 0x100000000
Mode: follower
Node count: 4
asnegi@nt1vm000001:~$

# connect with zkCli
asnegi@nt1vm000001:~$ sudo docker exec -it bd bash
bash-4.4# bin/zkCli.sh
[zk: localhost:2181(CONNECTED) 9] ls /brokers/ids
[1165, 1162, 4010]
```

Verify kafka
```
asnegi@nt1vm000001:~$ sudo docker exec -it 5f22 bash

bash-4.4# $KAFKA_HOME/bin/kafka-topics.sh --create --topic test1 --partitions 1 --zookeeper $KAFKA_ZOOKEEPER_CONNECT --replication-factor 2
Created topic "test1".
bash-4.4#
bash-4.4# $KAFKA_HOME/bin/kafka-topics.sh --describe --topic test1 --zookeeper $KAFKA_ZOOKEEPER_CONNECT
Topic:test1     PartitionCount:1        ReplicationFactor:2     Configs:
        Topic: test1    Partition: 0    Leader: 1165    Replicas: 1165,1162     Isr: 1165,1162
bash-4.4#

# on one node.
bash-4.4# $KAFKA_HOME/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test1
[2018-12-19 05:54:39,559] WARN Property topic is not valid (kafka.utils.VerifiableProperties)
so whats^H's up
hello
goodbye
^Cbash-4.4#
bash-4.4#

# on another node
bash-4.4#  $KAFKA_HOME/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test1 --from-beginning --zookeeper $KAFKA_ZOOKEEPER_CONNECT
'bootstrap-server' is not a recognized option
bash-4.4#  $KAFKA_HOME/bin/kafka-console-consumer.sh --topic test1 --from-beginning --zookeeper $KAFKA_ZOOKEEPER_CONNECT                            hi
bye
so what's up
hello
goodbye

```

Todo :
* Make sure that no instances of zookeeper are on same node.
* Generate ApplicationManifest and checkin those for deployment using linux.
* Get powershell command equivalent of zookeeper.