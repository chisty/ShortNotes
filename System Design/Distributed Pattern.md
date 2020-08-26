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