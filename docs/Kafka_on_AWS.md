```
I want to deploy an environment in AWS.
This environment will contain three EC2 instances called publisher, broker and subscriber.
On publisher i want to run two python scripts which sends messages to the Kafka broker on instance broker.
One script sends a continuous stream of letters to the yopic letters on the broker, one script sends a contiinous stream of numbers to the toopic numbers on the broker.
On broker i want to run the Kafka software configured to run two topics called letters and numbers.
On subscriber i want to run three python scripts:
- Script 1 subscribes to the numbers topic and displays what it receives.
- Script 2 subscribes to the numbers topic and displays what it receives.
- Script 3 subscribes to both topics and displays what it receives.
All scripts can be console based.
```

Certainly! Below is a detailed guide on how to deploy the environment you've described in AWS, utilizing Amazon EC2 instances for your Publisher, Broker, and Subscriber. Below are the steps you'll need to follow.

## Step 1: Set Up Your AWS Account
1. Sign in to your AWS account or create one if you do not have an account.
2. Ensure you have appropriate permissions to launch EC2 instances and create security groups.

## Step 2: Create EC2 Instances
1. Launch EC2 Instances:
    - Go to the EC2 Dashboard.
    - Click on Launch Instance.
    - Choose an Amazon Machine Image (AMI), ideally a lightweight Linux distribution like Amazon Linux 2 or Ubuntu.
    - Choose an instance type (e.g., t2.micro for testing).
    - Configure instance details, including the desired number of instances (3 instances).
    - Set up your storage and tags if desired.
2. Configure Security Group:
    - Create a new security group allowing the following inbound rules:
      - SSH (port 22): from your IP (for management).
      - Kafka (default port 9092): from any IP or limited to the publishers/subscribers’ IPs.
      - Customize other necessary ports based on your needs.
3. Assign Public IP Addresses:
   - Ensure that each instance has a public IP assigned.

## Step 3: Install Kafka on the Broker Instance
1. SSH into the Broker instance:
 
    `ssh -i your-key.pem ec2-user@your-broker-public-ip`

2. Install Java (Kafka requires Java):
   
    `sudo yum update -y`  # For Amazon Linux
    `sudo yum install java -y`

3. Download and Install Kafka:

    `wget http://apache.mirrors.pair.com/kafka/2.13-2.8.0/kafka_2.13-2.8.0.tgz`
  
    `tar -xvf kafka_2.13-2.8.0.tgz`
  
    `cd kafka_2.13-2.8.0`

4. Start Kafka Server:
  - Start the Zookeeper server:

      `bin/zookeeper-server-start.sh config/zookeeper.properties &`
    
  - Start the Kafka server:

      `bin/kafka-server-start.sh config/server.properties &`

5. Create Topics:
- Run the following commands to create topics:

    `bin/kafka-topics.sh --create --topic letters --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1`

    `bin/kafka-topics.sh --create --topic numbers --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1`

## Step 4: Set Up the Publisher Instance
1. SSH into the Publisher instance:

    `ssh -i your-key.pem ec2-user@your-publisher-public-ip`

2. Install Python & Necessary Packages:

    `sudo yum install python3 -y`
   
    `pip3 install kafka-python`

4. Create Python Scripts:

- Create publish_letters.py:
```
from kafka import KafkaProducer
import time
import string

producer = KafkaProducer(bootstrap_servers='your-broker-public-ip:9092')

while True:
    for letter in string.ascii_letters:
        producer.send('letters', value=letter.encode())
        time.sleep(1)
```

- Create publish_numbers.py:

```
from kafka import KafkaProducer
import time

producer = KafkaProducer(bootstrap_servers='your-broker-public-ip:9092')

num = 1
while True:
    producer.send('numbers', value=str(num).encode())
    num += 1
    time.sleep(1)
```

## Step 5: Set Up the Subscriber Instance
1. SSH into the Subscriber instance:

    `ssh -i your-key.pem ec2-user@your-subscriber-public-ip`
  
2. Install Python & Necessary Packages:

    `sudo yum install python3 -y`
   
    `pip3 install kafka-python`

3, Create Python Scripts:

- Create subscribe_numbers.py:
```
from kafka import KafkaConsumer

consumer = KafkaConsumer('numbers', bootstrap_servers='your-broker-public-ip:9092')

for message in consumer:
    print(f"Received number: {message.value.decode()}")
```

- Create subscribe_letters.py:
```
from kafka import KafkaConsumer

consumer = KafkaConsumer('letters', bootstrap_servers='your-broker-public-ip:9092')

for message in consumer:
    print(f"Received letter: {message.value.decode()}")
```

- Create subscribe_both.py:
```
from kafka import KafkaConsumer

consumer = KafkaConsumer('letters', 'numbers', bootstrap_servers='your-broker-public-ip:9092')

for message in consumer:
    print(f"Received {message.topic}: {message.value.decode()}")
```

## Step 6: Run Your Scripts
1. On the Publisher Instance:

- Open two terminal sessions and run:

  `python3 publish_letters.py`

    `python3 publish_numbers.py`

2. On the Subscriber Instance:

- Open three terminal sessions and run:

    `python3 subscribe_numbers.py`

    `python3 subscribe_letters.py`

    `python3 subscribe_both.py`

## Step 7: Verify Communication
Watch the subscriber instances for messages being printed from the corresponding topics.
Monitor the publisher instance to ensure letters and numbers are being sent to the correct topics.

## Conclusion
You now have a fully functional Kafka setup on AWS with three EC2 instances for the publisher, broker, and subscriber. Each component is running its intended scripts and you can expand this further based on your requirements. Make sure to manage AWS resources properly to avoid unnecessary costs and shut down instances when not needed.
