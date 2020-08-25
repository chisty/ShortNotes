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