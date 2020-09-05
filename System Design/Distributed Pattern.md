## Sidecar Pattern

Sidecar pattern is a single node pattern made up of two container. The first one is application container(where the application exist) & the second one is sidecar container. The role of sidecar is to improve, augment existing application. Both will act as a pair & deployed to a single node. Both container shares same network, hostname, filesystem.

### Use Cases

- Adding HTTPS to a legacy application. Legacy will be exposed to only localhost, and sidecar will handle the https and route to legacy.
- Dynamic configuration with sidecar. Both legacy & sidecar share the same host/directory. Legacy will read configuration from host. Sidecar will update the configuration from cloud and notify the legacy. Worst case, sidecar may send SIGKILL signal to abort the legacy application. Once aborted, the container orchestration will restart the legacy app at which point it will load the new configuration.
- Building a simple PAAS is possible. Like app container may run a node application with `nodemon` command. And sidecar is always running a loop to pull from a git repo. So, when we push something to git, sidecar will pull and get the new JavaScript files which will replace the old app since `nodemon` is running.
- Modular application container. Sidecar can help with modularity and reuse of components. Suppose, we need to write our own Http/Top service (to readout resource usage). Then for different service written in different language, we need to develop on every language and deploy it with the same service. But, we can write a separate sidecar application which can introspect all running processes and provide a consistent user interface. We can deploy one single service and deploy it everywhere with sidecar.

### Notes

- To make sidecar modular and reusable, implement it in a way to accept parameters. By parameterizing sidecar containers, we can configure it for its environment. We can think it like a function. Like for HTTPS support to legacy application, we need minimum 2 parameter for sidecar container: the certificate being used to provide SSL/HTTPS, the port of the legacy application server. Without these parameters, sidecar will not work.
- Each container should have its own API and support versioning.



## Ambassador / Service Broker

 Ambassador container broker interacts between the application and the rest of the world. Like other single node pattern, this two container  is tightly coupled in a symbiotic pairing and scheduled to a single machine. 

### Use Cases

- Modular, reusable container. Also supports separation of concerns so that easier to maintain, deploy.
- To shard a service. When data is too big to handle for a single machine, we may need to shard storage layer. **Sharding** splits up the storage layer into multiple disjoint pieces, each hosted by a separate machine. We can either write the sharding logic into client side code or middleware code. However, often time it is hard to fit a sharded client into existing code/app which expects to connect to a single storage backend. One approach might be, write all the sharding logic into shard service. The service will direct traffic to each shard. This will make the shard services complicated. But either way if we try to write all the logic is client side, it will make the client complicated too. So, it is a design decision which one to pick since both has trade offs.  We can introduce an *Ambassador* container that contains all the logic to route requests to the appropriate shard. Thus frontend/middleware only connects to what appears to be a single storage backend running on localhost. However, the server is actually Sharding Ambassador Proxy, which will route all the request  from the client to appropriate shard. 
- Using an Ambassador for Service Brokering. **Service discovery**, configuration is a primary challenge at Cloud environment. For example, a Mysql instance can be provided as a Software-as-a-service in Public cloud, but it might be necessary to spin up dynamically in a new virtual machine in a private cloud. Also, any application running on the cloud requires to introspect its environment and find the appropriate services to connect to. This process is called **Service Discovery**, and the system that performs this discovery and linking is called **Service Broker**. So, an ambassador is kind of *service broker.* Ambassador pattern helps to separate the application container to service broker container. The application will connect to local instance of Mysql. Its Ambassador broker's job to introspect the environment and service.
- Using Ambassador to do experiment or *request splitting*. Most often this is done to perform experiment with new, beta versions of the service to determine if the new version is *reliable or performant*. Additionally, request splitting is sometimes used to tee or split traffic so that all traffic go to *both production system as well as undeployed, new system*. The responses from the production system are returned to the user, while the response from the undeployed version is *ignored*. Most often this form of request splitting is done *to simulate production load* on new version. 
- The separation of concerns keeps each service slim and we can reuse the ambassador broker on multiple service which can accelerate development. 

## Adapters

Adapter pattern is a single node pattern which helps to connect external service to application container unlike Ambassador. 

### Use Cases

- Helps to modify the interface of application container so that it can conforms to some predefined interface. Real world application development is heterogeneous, some services are developed from scratch, some from vendor, some from open source, some from library. Adapter helps to make a common interface.
- Helps for monitoring. Prometheus is a open source monitoring aggregator which collects metrics and aggregates them into a single time series database. On top of that, it provides visualization, query language. But, it expects every container to expose a specific metrics API which enables Prometheus to monitor wide variety of different programs through single interface. However, Redis does not expose any compatible metrics for Prometheus. Here, an Adapter pattern is quite useful to fetch from Redis container and adapting it to Prometheus metrics.
- Helps to aggregate logging. In many cases, we use containers produced by another party. So, we can't change the logging behavior. So, an Adapter might be very handy here. Additionally, sometimes we have different types of logging in different files (error, fatal, warning, info), and also in stderr, stdout, console etc. So, if we need to aggregate all log into a single place, Adapters can help us to process different types of log into a specific format.
- Helps to add customized Health check. 



## Replicated Load Balanced Service

The pattern consists of scalable amount of servers with a load balancer in front of them. The load balancer can choose round-robin or session stickiness.

### Use Cases

- Stateless service. No state is necessary. It could serve static content or complex middleware/API service to receive response from multiple server and aggregate.
- Stateless services are replicated to provide *redundancy* and *scale*. To make a service highly available, each service should have **at least 2 replica**. [*For 3-9 (99.9%), we have 1.4min/day |8hr 31m/year downtime (24600.001). For 4-9(99.99%)  52.56 min/year. For 5-9 (99.999%), 5.25min/year  downtime is allowed :(* ] So, if the service never crashes, we need those replicas to upgrade/deploy new versions.
- Like Health probes, readiness probes for load balancing is important to inform the load balancer. Readiness probes determines if a service/load balancer is ready to serve requests. Its different than health probes since many application requires some time to initialized / populate data before they serve. They may need to connect db, pull request, initialize stores etc. So, we must include a special url to check readiness probe.
- Session tracked service. Often there is some reason that we want to ensure that a particular user's request should go to a specific server machine. The reason could be like caching user's data or long running state cache etc. Usually, the *session stickiness* is performed by hashing the source & destination ip address and using that key to identify server. This IP based session-stickiness works fine within a cluster (internal IP), not for external ip because of NAT (network address translation). For external session stickiness, application-level tracking like cookies preferred. Often, this is accomplished by consistent hashing. 
- Application layer replicated service. Like network load balancer, sometimes we can do application level load balancing by protocol mapping, request etc.
- Caching layer. Varnish is a good caching service.
- Sidecar pattern caching implementation is simple. But, disadvantage of this approach is we need to create *same* *number of cache replica* like server. If each replica has 1 GB ram and we have 10 server. It is always better to have 2 cache layer with 5GB ram each rather than 10 server with 1GB ram since we can cache more data in 5 GB ram whereas we will redundantly keep same page in 10 server with minimum data. 
- Caching can break session tracking if not careful. Like, if we use default IP address affinity and load balancing, *all request to API server will go from cache server, not from the end user IP.* So, it could led in a situation that  some replica of API service *will not see any traffic*. So, it is better to use something like cookie or http header for session tracking. 
- Rate Limiting and Denial-of-Service defense mechanism. 
- *SSL termination* at load balancer is performed to improve performance since decryption is resource and cpu intensive. So, API layer can do more processing. Also, if we want to communicate using https with different layers, certificates should be different in each layer. Each individual internal service should use its own certificate. Varnish can't do SSL termination, so we can use Nginx to do the termination and then send request from Nginx server to Varnish web cache.

## Sharded Service

In Replicated services, each replica is capable of serving every request. But, in sharded service, each replica/shard is capable of serving a subset of all request. Sharded services are used for stateful services whereas Replicated services are used for stateless services. 

### Hot Sharding System

If any data (photo,video,post) goes viral or receives huge traffic, the cache shard contains the data becomes "HOT". If there is 3 shard in total, initially each service receives equal traffic. However, when Shard A gets 4 times much traffic than B,C; The hot sharding system moves B shard to same machine of Shard C and replicates Shard A to the second machine of Shard B. So, now 2 shard contains "Hot" A cache, and 1 shard contains B & C. So, now traffic is shared equally among each shard.

## Scatter/Gather

Requests are sent to each replica and each replica will do a small amount of work. Root server will process various partial result to form a single complete response and send back to the client. 

### Use Cases

- Implementation is easy when the process is embarrassingly parallel.
- To process a 60s job in a single core, we can make it parallel job with 32 core machine which will do the processing within less than 2 second.
- However, Still using multiple core in a single machine is not ideally perfect since lot of IO, network, disk access cannot be performed  in parallel. In that case, using  multiple machine and different node will make the process faster and improve overall latency.
- Distributed Document Search. It creates index which is basically a hashtable with  each word to map the document id. So, to get the desired documents with "cat"  & "dog" each node process the data separately and pull document IDS and then intersect between two result set to get the final result and then pull the documents by ID.
- Adding more leaf to scatter/gather may not actually speed up processing, it also suffers from "straggler " problem. Since the root node waits for each leaf node result, the overall time it takes to processing document is defined by the slowest leaf node.

## Owner Election

There are 2 ways to implement master election. First is to implement a distributed consensus algorithm like **PAXOS & RAFT**, which is very complex. Fortunately, there are large number of distributed key-value stores already implemented such consensus algorithm like **etcd, zookeeper, consul**. The basic primitive is to perform *compare & swap* operation for a particular key. 

- If there are 3 replica A, B, C and A is master and if for some reason, A is fails, Replica C or B will be master. However, if replica is back after some time, B/C will still be the master. 
- Compare-and-swap atomically writes a new value if the existing value matches the expected value. If the value doesn’t match, it returns false. If the value doesn’t exist and `*currentValue*` is not null, it returns an error. 
- In addition to compare-and-swap, the key-value stores allow you to set a time-to-live (TTL) for a key. Once the TTL expires, the key is set back to empty  .
- **etcd**, is a distributed lock server developed by CoreOS. It is used in production at high scale and used by various projects including **Kubernetes**. We need to use distributed locking mechanism for the distributed key-value stores.
- However, only locking will not achieve anything. Since it is distributed system, a process could fail while holding the lock and there will be no one to release the lock which will make the system stuck. To resolve this, we need to use TTL functionality of the Key-value store. If we don't unlock within a given time, the TTL will unlock it. 
- Adding TTL also introduce a potential *bug*. For example, Process A obtains the *Lock* with TTL *t1* and ran really slow, longer than t1. Hence the *Lock* expires and Process B acquire the *Lock* since Process A lost the *Lock* due to TTL. After sometime Process A finishes and calls `Unlock().` Process C acquires the *Lock* listening Process B's notification. Process A believes it actually Unlocked the *Lock* since it does not know it lost the lock in the middle of the operation due to TTL. Now, both Process B & C believes they have the *Lock*. This issue can be solved since the key-value store provides a *resource version.*
- The replica sending request as master needs to be validated if it is still the master or not! To do this, the **hostname** of replica which is holding the Lock, should be *saved* in the *key-value store*. So, if any replica sends any request as a master, the lock-server verifies if that replica is a master or not!

## Work Queue System