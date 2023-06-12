<!-- TOC -->
* [How to Deal with Memory Pressure in Redis](#how-to-deal-with-memory-pressure-in-redis)
    * [Redis Overview and Benefits](#redis-overview-and-benefits)
    * [Redis Hosting and Configuration Options](#redis-hosting-and-configuration-options)
        * [On Premises](#on-premises)
        * [Managed Service in the AWS Cloud](#managed-service-in-the-aws-cloud)
    * [Memory Considerations](#memory-considerations)
        * [Limited Storage Capacity](#limited-storage-capacity)
        * [Non-Durability](#non-durability)
    * [Real World Case Study](#real-world-case-study)
        * [Project Setup](#project-setup)
        * [Events Which Occurred](#events-which-occurred)
        * [Addressing the Issue](#addressing-the-issue)
            * [Monitoring and Alerting](#monitoring-and-alerting)
            * [Optimising Data Structures and Writing Logic](#optimising-data-structures-and-writing-logic)
            * [Vertical Upscaling](#vertical-upscaling)
            * [Backups](#backups)
            * [Data Partitioning and Horizontal Scaling](#data-partitioning-and-horizontal-scaling)
            * [Availability](#availability)
            * [Appropriate Technology Usage According to Your Requirements](#appropriate-technology-usage-according-to-your-requirements)
    * [Lessons Learned](#lessons-learned)
<!-- TOC -->

# How to Deal with Memory Pressure in Redis

Redis has emerged as a popular and powerful open-source in-memory data structure store.
With its fast and efficient performance, Redis is widely adopted for various use cases, ranging from caching to real-time analytics and pub/sub messaging.
As Redis handles large volumes of data in memory, it becomes crucial for system administrators and developers to proactively address memory issues to ensure smooth operations and optimal performance.

In this article, we will explore what Redis is, delve into its suitability for different applications, and focus on the steps necessary to effectively deal with memory overload issues.
Drawing inspiration from a real production scenario, we will highlight practical strategies to optimize memory utilization and mitigate the impact of memory pressure on Redis instances, primarily focusing on managing the Redis cluster provided as a managed service in AWS - because of configuration options and simple instruments the cloud provides.

## Redis Overview and Benefits

Redis, an acronym for Remote Dictionary Server, is an open-source, in-memory data structure store that offers remarkable performance and versatility. 
It serves as a valuable tool in modern application development, enabling developers to address various data storage and retrieval requirements efficiently.

At its core, Redis is designed to deliver blazing-fast operations by keeping all data in memory. 
Unlike traditional disk-based databases, Redis's reliance on memory allows it to achieve sub-millisecond response times, making it ideal for latency-sensitive applications that demand real-time data access.

Redis supports a rich set of data structures, including strings, lists, sets, sorted sets, hashes, and more. 
These data structures are more than just containers; they come with powerful and specialized operations that enable developers to perform complex computations and data manipulations directly within Redis. 
For instance, Redis allows set intersections, union operations, and rank-based sorting, making it well-suited for a wide range of use cases.

One of the key benefits of Redis is its simplicity. 
The Redis API is straightforward and intuitive, enabling developers to quickly grasp its concepts and leverage its capabilities. 
Redis can be easily integrated into existing applications, acting as a caching layer to improve performance or as a primary data store for specific use cases.

Redis's versatility extends beyond its in-memory nature and data structures. 
It also provides built-in features like replication, persistence, and pub/sub messaging, making it a comprehensive solution for various application requirements. 
Redis replication enables high availability and fault tolerance by allowing the creation of replicas that can take over if the master node fails. 
Persistence options, such as snapshotting and append-only file (AOF) mode, provide durability to the data stored in Redis, ensuring data persistence across restarts or failures.

Additionally, Redis excels at pub/sub messaging, allowing for real-time communication between different components of a distributed system. 
Its publish/subscribe mechanism enables message broadcasting and real-time event processing, making it a powerful tool for building scalable and responsive applications.

Overall, Redis offers a combination of speed, simplicity, and versatility that makes it a compelling choice for a wide range of use cases. 
By leveraging its in-memory nature, diverse data structures, and built-in features, developers can harness the full potential of Redis to enhance application performance, scalability, and responsiveness.

## Redis Hosting and Configuration Options

Before jumping into memory usage specifics, let's get some context on the options where could we potentially host our Redis storage.
We will review two primary concepts: on premises hosting and using a managed service in the cloud.

### On Premises

On-premises Redis hosting involves setting up and managing Redis instances within your own infrastructure. 
This option offers complete control over the Redis environment but requires significant upfront investment in hardware, networking, and ongoing maintenance. 
Some key considerations for on-premises Redis hosting include:

a. Hardware and Networking: You have the flexibility to choose hardware specifications tailored to your specific requirements, including CPU, RAM, and storage. Network configuration can be optimized for low latency and high throughput, ensuring optimal Redis performance.

b. Security and Compliance: With on-premises hosting, you have direct control over security measures and can enforce compliance standards specific to your organization's needs. This includes implementing firewalls, encryption, access controls, and auditing mechanisms.

c. Scalability and Flexibility: Scaling Redis in an on-premises environment requires careful planning and provisioning of additional hardware resources. Upgrades and capacity adjustments may involve additional costs and effort, making it less flexible compared to cloud-based options.

d. Operational Overhead: On-premises hosting places the responsibility of infrastructure setup, maintenance, backups, and monitoring on your organization's IT team. This requires expertise and resources for ongoing management and support.

### Managed Service in the AWS Cloud

The most straightforward and simple way to have a Redis storage in AWS cloud is using AWS ElastiCache service. It is a managed Redis service provided by Amazon Web Services (AWS).
It offers a hassle-free way to deploy and operate Redis clusters in the cloud. 
Key advantages of AWS ElastiCache include:

a. Managed Service: ElastiCache handles infrastructure provisioning, configuration, scaling, and maintenance tasks, allowing you to focus on application development instead of operational aspects. AWS takes care of patching, backups, and software updates, ensuring a reliable and secure Redis environment.

b. Scalability and Elasticity: ElastiCache makes it easy to scale Redis clusters vertically (by adding more memory to individual nodes) or horizontally (by adding or removing nodes) based on changing workload demands. This enables seamless scalability and accommodates fluctuating traffic patterns.

c. High Availability and Fault Tolerance: ElastiCache provides automatic failover and replication capabilities, ensuring high availability of Redis clusters. It employs features like Multi-AZ replication, which synchronously replicates data across availability zones to withstand failures.

d. Integration with AWS Ecosystem: ElastiCache seamlessly integrates with other AWS services, enabling you to leverage additional capabilities. For example, you can integrate ElastiCache with AWS Identity and Access Management (IAM) for fine-grained access control, or use AWS CloudWatch for monitoring and performance analysis.

e. Cost-Effectiveness: AWS ElastiCache follows a pay-as-you-go pricing model, where you pay for the resources consumed. This eliminates the need for upfront infrastructure investment and allows you to optimize costs based on actual usage.

It's important to note that while AWS ElastiCache offers convenience and scalability, it also introduces dependencies on the AWS platform and incurs ongoing operational costs. 
Organizations must evaluate their specific requirements, cost considerations, and expertise before choosing between on-premises hosting or a managed service like AWS ElastiCache for Redis.

## Memory Considerations

When utilizing Redis, it is crucial to take into account few memory considerations which can impact your application's data management strategy.

### Limited Storage Capacity

Redis stores data primarily in memory, which provides unparalleled speed and performance. 
However, this also means that the amount of data you can store is constrained by the available memory on the hosting machine. 
Unlike disk-based databases that can scale storage capacity easily, Redis's memory capacity is directly tied to the physical or virtual machine it runs on. 
It is essential to carefully estimate your data requirements and provision sufficient memory to accommodate your dataset.

To optimize memory usage in Redis, consider employing strategies such as data compression, data partitioning, or using Redis modules like Redis Streams or RedisGears to process data on the fly. 
These techniques can help reduce the memory footprint and allow you to store more data within the available memory.

### Non-Durability

Redis, by default, prioritizes speed and performance over durability. 
This means that Redis does not guarantee persistent storage of data on disk like traditional databases. 
While Redis offers persistence options like snapshotting and append-only file (AOF) mode, they introduce additional disk I/O overhead and can impact performance. 
Moreover, AOF can't protect against all failure scenarios. For example, in AWS, if a node fails due to a hardware fault in an underlying physical server, ElastiCache provisions a new node on a different server. 
In this case, the AOF file is no longer available and can't be used to recover the data. Thus, Redis restarts with a cold cache.
It's often recommended to have one or more read replicas in multiple availability zones in the cloud for Redis to be on a safer side.

In the event of a Redis restart or failure, data stored solely in memory may be lost. 
Therefore, it is crucial to have proper backup and recovery mechanisms in place to protect against data loss. 
This can include regular snapshots, using Redis AOF persistence mode, or implementing replication and high-availability configurations to ensure data redundancy across multiple Redis instances.

To address the durability concern, some organizations choose to combine Redis with other databases for a hybrid approach. 
For example, critical data can be stored both in Redis for fast access and in a durable database for long-term persistence.

## Real World Case Study

### Project Setup

Now having the basics of Redis and its memory concerns in mind, let's dive into a real scenario, when memory pressure greatly impacted operation excellence of the business, analyze the mistakes which were made in the initial design and the steps which helped in issues mitigation.

Let's start with the business context: we are dealing with a large eCommerce company which exposes the multi-tenant online shopping system online, selling consumer goods like shoes, home decor and other things.
We will be taking a closer look at products management part of the business: the software which is responsible for getting the information about items, their prices, available configurations like sizes and other things from the supplier systems, applying necessary data enriching, aggregation and transformation business logic, and making this updated data available via an API, so that the shopping frontend could show the items to the end users.

Technologically, that part of the system is represented by an event-driven solution, having:

- Apache Kafka as the intermediate data store for keeping all the items' state updates which are coming, along with applied intermediate data manipulations;
- Kafka Stream applications as data aggregators, processors, transformers;
- Kafka Connect application which helps to sink data from Kafka topics into a storage;
- AWS Elasticache Redis as prepared items data storage;
- Java Spring Boot application as an API for shopping frontend to access the items' data. The diagram below shows the concept view of the system.

```TODO: scheme```

Here we need to bear in mind that data retention period in Kafka is set to 1 month, which is a fairly large period which allows to replay the data to the storage if anything happens.
And we also have to consider that Redis here exists as a non-cluster instance with a read replica in a separate availability zone. That means that the only way to scale the storage up is to do the vertical scaling of the nodes, adding more memory capacity.

### Events Which Occurred

The first trouble with such a setup occurred just during the Easter peak season: all the items in the shop just went offline (or not available), and sadly the business was the first to be notified about the issue.

Once tech department got the hands dirty, they found out that Redis storage is completely empty, all the data there was vanished, and the API acted with the fallback strategy for the frontend: if the item is not in the storage, then it just tells that the item is not available.
After the investigation, the tem has figured out that the memory usage has spiked up from 62% to 100% within few hours and Redis stopped accepting more write commands. 
Moreover, once there has occurred a service disruption, the primary node which accepts writes has restarted, which resulted in complete data loss.
Even having a read replica in another availability zone didn't help: once the primary node became non-responding to writes and went to restart, the replica has substituted the primary node and became primary itself.
However, as the data sync was happening continuously for the replica, it was 100% occupied regarding memory capacity too, and very soon it became unavailable too. And consequently, it was restarted too, and substituted with newly cleaned up node.
Eventually, both nodes went back online, but they were completely empty.

To be fair, AWS does not tell that the ElastiCache node will always be restarted once the memory usage capacity is reached. 
These events are connected but not necessarily happen together all the time. 
Once the memory capacity is reached, Redis would just stop accepting more writes if that increases the memory usage.
But in our case, unfortunately both nodes restarted as well, and AWS Support did not really tell the exact reason for that (and they are not really obliged to).
But anyway the storage became completely empty which disrupted the normal work of the whole shop.

To fix that, they had to replay all the items data from Kafka topics to Redis, and that included:
- Stopping Kafka Connectors
- Deleting Kafka Connectors
- Resetting the offsets of corresponding Consumer Groups by executing the command:
```
./bin/kafka-consumer-groups.sh --bootstrap-server :9092 --group connectorName --reset-offsets --to-earliest --execute --topic topicName
```
- Adding the connectors and their configuration back
- Starting the connectors again

This whole process resulted in few hours of downtime which obviously had a major business impact, and after returning the system to the business-as-usual state, the team had to make sure that thing won't happen again.

### Addressing the Issue

#### Monitoring and Alerting

First of all, the team had to improve their observability of the system they are working with - to become more proactive, to foresee the issues before they are occurring or starting impacting the business.
Having Redis as a managed service in AWS, it is a relatively simple task to achieve, as there is a metric for Elasticache called "DatabaseMemoryUsagePercentage" - indicating how much memory is used by the storage.
The only remaining thing is to make sure that monitoring of that metric is automated and if it exceeds the approved threshold, the team would be notified:

1. Create an Amazon CloudWatch Alarm: CloudWatch is a monitoring service in AWS that can collect and track metrics. You'll start by creating an alarm that will trigger when the DatabaseMemoryUsagePercentage metric exceeds a specified threshold. This can be done in the Alarms screen of CloudWatch service. There you'd be able to select the desired metric, the threshold indicating the percentage of memory used when you want to be notified, and the action to be taken (for example, an email to the team or a Lambda function trigger which would send an alert to the corporate messenger).
2. Enable "Enhanced Monitoring" for your ElastiCache cluster, via the Modify options of the cluster - to make sure that DatabaseMemoryUsagePercentage metric is available to you and to CloudWatch.
3. Configure additional actions on alarm if needed.

Additionally, you may want to be notified about major events happening to your Redis storage, like failover, node restart, backup events, node replacements and others.
To do that, you can:
1. Set up an SNS topic where these events will be sinked by ElastiCache service
2. Enable the SNS notifications in ElastiCache service
3. Specify your new topic as the destination
4. Configure the actions to be taken once an event lands to the topic - similarly to CloudWatch alarms (email, Lambda trigger etc.)

You can read more about the concept of SNS notifications on important ElastiCache events [here](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/ECEvents.SNS.html).

If you have your Redis storage on premises, you may want to leverage some software like Datadog to configure the monitoring for your internal resources in a similar way like it's done for AWS.

#### Optimising Data Structures and Writing Logic

If you are responsible for the operational excellence of your Redis cluster, you really should know how your system works, which processes and integrations does it have regarding your target datastore.
Ideally, you should have few ,conceptual views on your system in form of diagrams, or a list telling who's reading from your datastore, who's writing into your datastore, and if we are talking about a key-value storage, it's also really nice to know the key formats different systems write in or read from.

Knowing the integrated apps and key patterns they used, and by running few scan queries for counting the records with key patterns, the team was able to find an old outstanding Kafka Streams application made for data analytics which was no longer needed for business and which was generating huge loads of data which was absolutely unnecessary.
The team removed that application and cleaned up the store from the unused records.

Apart from that, you may want to review your data structures and ensure they are optimized for memory usage. 
For example, consider using Redis' more memory-efficient data structures like Redis Hashes instead of individual keys for storing object fields. 
Additionally, you may want to leverage Redis' data compression capabilities, such as using Redis Modules like RedisGears, to compress data before storing it in Redis, reducing the overall memory footprint.

#### Vertical Upscaling

In the situation when your business is growing, as well as the amounts of data stored, you may want to consider scaling up the memory capacity of your Redis cluster using ElastiCache's vertical scaling capabilities, or by providing a machine with bigger capacity and performing the migration if your Redis is on premises. 
In AWS, you can modify the instance type or increase the number of shards to add more memory to your cluster. 
This approach allows your Redis deployment to accommodate increased data volumes and mitigate memory pressure.
However, you need to bear in mind that vertical scaling takes time and in case of a sudden peak memory pressure this might not really manage to help you in time.

#### Backups

If we are talking about ElastiCache, backups for Redis are done as snapshots which are stored in AWS S3. 
This most probably won't serve as a memory pressure resolution mechanism for an active storage. 
However, this can be a safety measure for cases when you have a recovery point you can allow your system to roll back to without major business impact, and you are not under memory pressure.
And you need to bear in mind that if you are restoring from the backup, AWS would create a new cluster with the data from the backup - it's not able to seed the data into your existing cluster, which can result in more configuration changes for your system.

#### Data Partitioning and Horizontal Scaling

If your workload exhibits a skewed data distribution pattern, you can implement data partitioning to evenly distribute the data across multiple Redis shards. 
By partitioning your data, you can distribute the memory load and prevent excessive memory usage on specific shards.
In AWS, you can do so by creating a Redis storage with Cluster Mode Enabled, or by migrating your existing storage from Cluster Mode Disabled to Cluster Mode Enabled, including making all the integration preparations like changing the SDK.
The following [article](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/modify-cluster-mode.html) has more information on the migration process.

However, keep in mind the considerations and complexities involved in managing partitioned data. 
For example, in AWS, you have to manage your key slots partitions, i.e. configure which shards would host which range of keys - either in even or in custom distribution.
In addition, you need to make sure that your integrated applications support the cluster protocol for Redis

Having your data partitioned across shards, you can then consider horizontal scaling by adding more shards to your Redis cluster. 
Horizontal scaling allows you to distribute data across multiple nodes, thereby increasing the overall memory capacity and performance of your Redis deployment. 
ElastiCache makes it seamless to add or remove shards without disrupting your application.
Horizontal scaling is a simpler and quicker process than vertical scaling in most of the cases, and it can happen automatically without service disruptions. 
However, you have to know that the process could bring some performance degradation during scaling.

#### Availability

First of all, in AWS, consider having a Read Replica or even few replicas for your cluster, to distribute the overall load across the nodes. 
The primary node can accept write commands, which few Read Replicas can handle the read load. Plus, if you also enable data partitioning, multiple nodes (shards) would handle write load, which also removes the bottlenecks in your system.
In addition to that you'll have a failover scenario we reviewed above: if anything happens with the primary node, it gets replaced with one of ready replicas in minimal downtime, while AWS handles the provision of another Read Replica instead of the missing one.

Moreover, to enhance resiliency, consider expanding your Redis cluster across multiple Availability Zones (AZs) within the same region. 
ElastiCache's Multi-AZ deployments automatically replicate data synchronously to standby replicas in different AZs, increasing availability and data durability. 
This approach also helps distribute the memory load across multiple AZs, reducing pressure on individual nodes.

#### Appropriate Technology Usage According to Your Requirements

The last, but not the least point to consider is to know your business processes and, most importantly, requirements very well to be able to assess your system, try to map the solution to the requirements and to see if they fit together.
For example, for the reviewed eCommerce case, the primary requirements for the storage were data durability and read performance.
While Redis is indeed a highly performant storage due to its work with memory, the data durability and resilience is questioned for such a technology.

## Lessons Learned

