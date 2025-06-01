RUNNING KAFKA ON COMMAND LINE:

Install kafka from the website: https://kafka.apache.org/downloads

This command calculates the SHA-512 checksum of the kafka_2.13-3.9.1 file.:
You can compare this hash with a published SHA-512 hash from the Kafka website to:
Verify file integrity (i.e., the file wasn’t corrupted during download).


Ensure authenticity (i.e., it wasn’t tampered with or modified).
shasum -a 512 kafka_2.13-3.9.1

extracts the contents of the kafka_2.13-3.9.1.tgz archive file.
tar -xvf kafka_2.13-3.9.1.tgz

START ZOOKEEPER:
Start zookeeper in a new tab. 

Move to the kafka directory where we unpacked the .tgz file. 

For example: cd ~/Downloads/kafka_2.13-3.9.1
Ls shows these files:
(base) poojamanjunatha@poojas-air kafka_2.13-3.9.1 % ls
LICENSE		bin		libs		site-docs
NOTICE		config		licenses

All of the libraries are in the libs directory 
Example configuration files in config directory 
Bin directory has files that help run kafka, zookeeper and admin commands. 

ALWAYS RUN ZOOKEEPER FIRST AND END IT LAST.

./bin/ZooKeeper-server-start.sh config/zookeeper.properties

This command starts the zookeeper and we pass default values present in the config file for it to be up and running. 











What we look for in this file is a message that looks like this: 
INFO binding to port 0.0.0.0/0.0.0.0:2181 (org.apache.zookeeper.server.NIOServerCnxnFactory)- which means the zookeeper is up and running










3. START KAFKA
Open a new tab in the same directory and run the command:
./bin/kafka-server-start.sh config/server.properties




This time we are looking for a message that says : 
2025-05-28 12:34:37,717] INFO [KafkaServer id=0] Start processing authorizer futures (kafka.server.KafkaServer)




CREATE A TOPIC:
Open a new tab and move to the kafka directory and stat the topic called “mytopic”
replication_factor=1
partitions=1 

./bin/kafka-topics.sh --create --topic mytopic --replication-factor 1 --partitions 1 --bootstrap-server localhost:9092


Part
Meaning
./bin/kafka-topics.sh
This is the Kafka script used to manage topics (create, list, delete, etc.). It’s located inside the bin/ folder of your Kafka directory.
--create
This tells Kafka you want to create a new topic.
--topic mytopic
This sets the name of the topic you're creating. In this case, the topic will be called mytopic.
--replication-factor 1
This defines how many replicas of the topic’s data Kafka should maintain across brokers. 1 means no replication (just one copy). Useful for local setups or single-node clusters.
--partitions 1
This sets the number of partitions the topic will have. Partitions help with parallelism and scalability.
--bootstrap-server localhost:9092
This tells the Kafka CLI tool how to connect to your Kafka broker. In this case, it's saying: “Connect to a Kafka broker running on localhost at port 9092.”





List all topics
./bin/kafka-topics.sh --list --bootstrap-server localhost:9092
Mytopic




Produce messages to the topic:
./bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic mytopic

Once the prompt appears (>), type messages line by line. Example:
> hello
> world
> new event



Consume messages from the topic:
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic mytopic --from-beginning

--bootstrap-server localhost:9092: Connects to the Kafka broker.


--topic mytopic: Specifies the topic to read messages from.


--from-beginning: Tells Kafka to consume all messages from the beginning of the topic, not just new ones.


Configure second kafka broker: by copying the file: cp config/server.properties config/server-1.properties


Change #listeners= PLAINTEXT://:9092 to PLAINTEXT:://:9093 as shown in the figure above


Add-1 to log.dirs=/tmp/kafka-logs and change it to log.dirs=/tmp/kafka-logs-1 as mentioned above

When you try to run this broker, it throws an error:
Because it requires unique broker id 



./bin/kafka-server-start.sh config/server-1.properties


Create three brokers with the same idea, changing the port to 9093 and 9094. 
Changing broker id to 2 and 3 and logs -2,-3

Created a new topic called mytopic1 with 3 replication factor:
 ./bin/kafka-topics.sh --describe --topic mytopic1 --bootstrap-server localhost:9092




Field
Description
Topic
Name of the Kafka topic: mytopic1
TopicId
Unique ID assigned to the topic internally
PartitionCount
Number of partitions: 1
ReplicationFactor
Number of replicas for each partition: 3
Leader
Broker currently acting as the leader for partition 0: Broker 2
Replicas
List of broker IDs that hold replicas: [2, 0, 1]
Isr (In-Sync Replicas)
Replicas that are fully caught up and synchronized: [2, 0, 1]
Elr (Eligible Leader Replicas)
Deprecated (shows N/A)
LastKnownElr
Deprecated (shows N/A)


Leader: This is the broker responsible for handling all reads and writes for this partition.


Replicas: These are brokers storing a copy of the partition. Even if not leaders, they ensure high availability.


ISR (In-Sync Replicas): Replicas that are up-to-date with the leader. All 3 brokers are in sync — this is ideal.


This implies you have 3 Kafka brokers running with broker.ids 0, 1, and 2 — necessary to support replication factor 3.


Custom Kafka Logging Setup and Console Producer Test

This command sets the KAFKA_OPTS environment variable to use a custom Log4j configuration file located at config/log4j.properties. It ensures that all Kafka logs follow the structure and log levels defined in this file, which helps in debugging and log management

export KAFKA_OPTS="-Dlog4j.configuration=file:config/log4j.properties"


Produce messages to kafka
./bin/kafka-console-producer.sh --topic mytopic1 --bootstrap-server localhost:9093

This command launches the Kafka Console Producer that connects to the Kafka broker running on localhost:9093 and sends messages to the topic named mytopic1.
After running the command, you can type messages in the terminal and press Enter to send each message to the Kafka topic.
The producer keeps running until you stop it (using Ctrl+C).


These steps confirm that:
Your Kafka broker is running and reachable.
The topic mytopic1 is properly created.
The custom logging configuration is active and logs Kafka events as specified.


Kafka requires each broker in the cluster to have a unique, persistent identity. This identity is defined by the broker’s broker.id, which is used to manage partitions, replication, and coordination within the Kafka cluster.
In Kubernetes, if you use a regular Deployment, pods are treated as interchangeable and may get new names or IPs when restarted. This can break Kafka’s expectations about broker identity and lead to issues with data consistency and cluster stability.
To solve this, StatefulSets are used. A StatefulSet ensures:
Each Kafka broker pod has a stable, unique name (like kafka-0, kafka-1, etc.)


Each broker gets its own dedicated persistent volume, which is not shared and is preserved across restarts


The broker ID can be mapped directly from the pod name (e.g., broker.id=0 for kafka-0).
This persistent identity and storage are critical for Kafka’s correct operation and for maintaining data integrity.












9. Download docker
 Move to the folder it is downloaded in and run the command
docker run hello-world




	
Install Kubernetes
Go to settings in docker and enable kubernetes 


Run this command in kubernetes to check if it is working 
kubectl get nodes
The output should contain docker-desktop as shown in the image below



Open a new filecalled statefulset-kafka.yaml and add the content: 
The command to create a new file in the terminal-
vi statefulset-kafka.yaml              

The content of the file will be:
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: kafka
spec:
  selector:
    matchLabels:
      app: kafka
  serviceName: "kafka"
  replicas: 1
  template:
    metadata:
      labels:
        app: kafka
    spec:
      containers:
        - name: kafka
          image: debezium/kafka
          ports:
            - containerPort: 9092
              name: kafka
          env:
            - name: BROKER_ID
              value: "0"
            - name: ZOOKEEPER_CONNECT
              value: "host.docker.internal:2181"

Run the command  kubectl apply -f statefulset-kafka.yaml
This applies kubernetes to the cluster 


Run the command:kubectl get pods lists all the current pods in your Kubernetes cluster (in the current namespace).


The -w (or --watch) flag tells Kubernetes to continuously watch and stream updates about the pods as they happen in real-time.



kafka_2.13-3.9.1 % kubectl get pods -w



The command below is  used to open an interactive Bash shell inside the running Kafka pod named kafka-0 in your Kubernetes cluster. It allows you to directly access the container's command line interface, just like logging into a remote server. This is helpful for performing tasks such as inspecting files, running Kafka commands, checking environment variables, or debugging issues within the pod. The -it flag makes the session interactive and user-friendly by enabling input and displaying output in a terminal-like environment.

kafka_2.13-3.9.1 % kubectl exec -it kafka-0 -- /bin/bash


Create a new topic inside the container, called “mynewtopic” and then describe it using the command 

./bin/kafka-topics.sh --describe --topic mynewtopic --bootstrap-server poojas-air.lan:9092


Produce messages in this topic:


Send and produce messages 
To check the status of the pod- delete the pod 
kubectl delete pod kafka-0

Run this command in another tab and see the status 
kubectl get pods -w 



Kubernetes, when a pod (like your Kafka pod kafka-0) is terminated (intentionally or due to an error), the controller managing the pod, typically a StatefulSet (for Kafka) or Deployment, will automatically recreate it to maintain the desired state.
However, this automatic recreation does not retain data unless:
Persistent Volumes (PVs) are properly configured and mounted via Persistent Volume Claims (PVCs).


Kafka is configured to write logs and data to those volumes.


If Kafka is not using persistent storage, then on pod restart, it will lose all topic data (e.g., messages, logs, consumer offsets), and it will start fresh — which explains why your consumer might not receive anything.
To summarize:
Yes, the pod is recreated automatically.


But unless persistent storage is used, data is lost on each termination/restart.


You can check if a PVC is mounted with:



