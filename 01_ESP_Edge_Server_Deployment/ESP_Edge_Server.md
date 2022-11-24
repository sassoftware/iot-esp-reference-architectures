# ESP Edge Server Deployment

* [Overview](ESP_Edge_Server.md#overview)
* [Deployment Flavors](ESP_Edge_Server.md#deployment-flavors)
  * [Standard Deployment](ESP_Edge_Server.md#standard-deployment)
  * [Stateless Models with DB for Persistent State](ESP_Edge_Server.md#stateless-models-with-db-for-persistent-state)
  * [Multiple Cascading Projects](ESP_Edge_Server.md#multiple-cascading-projects)
    * [Multiple Cascading Projects Using Message Bus](ESP_Edge_Server.md#multiple-cascading-projects-using-message-buses)
    * [Multiple Cascading Projects Using ESP Router](ESP_Edge_Server.md#multiple-cascading-projects-using-esp-router)
    * [Multiple Cascading Projects Using ESP Project Publish Connectors](ESP_Edge_Server.md#multiple-cascading-projects-using-esp-project-publish-connectors)


## Overview
In this section, we will present several architectures where the ESP edge server runs inside a docker container. If required we can start multiple ESP edge servers from inside the container running on different ports just like a bare-metal deployment. However, we have to take care that the required ports are mapped. In this deployment, we can also run individual dockers for the ESP web-clients, i.e., ESP Studio, ESP Streamviewer, and Event Stream Manager (ESM). All the ESP docker images can be deployed on-premises or in a cloud virtual machine. Multiple ESP projects can run on the same ESP server as done traditionally. 

**NOTE**: *Edge can be an IoT device with enough (minimum) resources to run ESP edge server or it can be a full-fledge server/virtual machine (at the edge gateway) with sufficient resources.*

ESP edge server docker image is packaged with limited connectors and adapters to publish and subscribe to the events. ESP adapters and ESP connectors are quite similar except ESP adapters must be started as a separate process from the ESP edge server while ESP connectors start when the ESP project starts (also managed by connector orchestration to some extent). ESP adapters must be used instead of ESP connectors when the administrator needs to control the start and stop of sending data to the ESP project.

**NOTE:** *During events of failure or crash, ESP edge server, ESP projects, connectors, adapters, etc. must be manually restarted. There are no automatic restart mechanisms for these architectures. Custom scripts can be written to achieve some automation.*

### ESP deliverables
- Linux Docker images of ESP edge server, ESP Studio, ESP Streamviewer and ESM. Available from version 6.2 onwards.

## Deployment Flavors

### Standard Deployment

<p align="center">
 <img src="images/espedgeserver_highlevel_architecture.png"/>
    <br>
    <em>Figure 1: ESP Edge Server Standard Deployments</em>
</p>

Figure 1 presents the ESP Edge server deployment on a single virtual machine (VM) in a cloud/on-premises or at the IoT device. This is the simplest deployment strategy with one ESP Edge server and the ESP web clients, all running in their containers. The only requirement is to have docker running in the VM.

#### Characteristics
- This architecture is closest to the traditional way we have been working with ESP. There will a single ESP edge server with its hostname and port. The web clients (ESP Studio, ESP Streamviewer, and ESM) settings will point to that ESP edge server.
- To handle the required rate of incoming data and to achieve the desired throughput, edge device/server must be vertically scaled with more resources (CPU and memory).
- This architecture is suitable if high availability and state persistence are not an issue in case of failover of the ESP edge server. 
- The deployment is more deterministic as we know exactly where the projects are running. Additionally, we can interact directly with the running ESP projects using the REST calls. This is helpful to find issues like memory leaks in the model due to the wrong state management. 
- Any extra libraries or plugins are installed on the edge device/server. They are shared by the ESP edge servers through shared volumes. This does not require any changes in the ESP edge docker image. This is helpful in the case of some ESP connectors, for example, the MQTT connector which has dependencies on the 3rd party libraries.
- The ESP web-clients dockers can run in the same server as that of the ESP edge server in case of the gateway or on a different server if the ESP edge server is on the IoT device itself.

#### Limitations
- Performance of the server is limited by the configuration of the machine where the docker is running. ESP uses thread pools for increasing the performance which in turn depends on the number of CPU cores on the machine. All the projects share the same set of resources (CPU and memory). The only way to increase the performance is have more CPU cores and internal memory. Therefore, the sizing effort must be considered.
- The amount of state retained in the ESP model depends on the size of available RAM. ESP edge server stores data in-memory for better performance, so if the RAM is full, the ESP edge server will stop. Multiple projects running in the same ESP edge server share the resources, therefore, the RAM availability is sum of the state retained by all the ESP projects.
- If an ESP project crashes/fails and leads to shut down of the ESP server, then all the other running projects will stop too.
- No multi-tenancy and multi-users support is present. 
- No scalability of ESP edge server.

#### Discussion
It is not recommended to connect to the CAS server using ESP CAS connector from the ESP edge server running at the IoT device/server.

### Stateless Models with DB for Persistent State

<p align="center">
 <img src="images/espedgeserver_db_persistent_state.png"/>
    <br>
    <em>Figure 2: ESP edge server with DB for persistent state</em>
</p>

#### Description
Figure 2 illustrates the architecture of an ESP edge server with an in-memory database for state persistence. SAS provides two windows, StateDB Reader to read from and StateDB Writer to write to the low latency and high throughput in-memory database to persist state and data for join and aggregation operations. 

**NOTE**: *At present, Redis is the only in-memory with small footprints that can be deployed on the edge.*

#### Characteristics
- This architecture provides recovery of ESP edge server without losing any state in case of server crash or machine failure. Persisted state and data in the Redis in-memory database are used to recover from failure.
- The ESP projects can be designed stateless for aggregation and join operations as the states are stored in the in-memory database.
- [RedisEdge](https://redis.com/blog/redisedge-iot-database-for-edge-computing/) can be deployed on the edge with defined memory and ESP projects can connect to it using the StateDB Windows. 


#### Limitations
- Throughput and latency must be tested thoroughly in this setup. Due to the network-level communication with the 3rd party in-memory database, we are bound to see some performance degradation (**NOTE**: *it might not be an issue for some of the use cases*).
- It is an ESP edge server deployment, therefore, the ESP model cannot scale up horizontally to achieve higher performance.
- Extra cost is incured due to the in-memory database deployment. 
- State information related to open patterns, geofences, and streaming analytics are not stored in the in-memory database.

#### Discussion
The performance of this architecture must be thoroughly tested to ensure it matches the required SLOs. With the increase in the streaming data, the performance may degrade.

### Multiple Cascading Projects
It is a good practice and strategy to break a huge project into sub-projects based on the use case modeling requirements. To understand it better, let's consider an example. If the use case demands finding a pattern in the streaming events followed by some calculations/transformations based on the pattern then we can have two ESP projects instead of one. The first project will find the pattern and the second project will perform the calculations. The output from the first project is fed as input to the second project. 

Having multiple cascading projects allows easy maintenance of the whole project flow. Projects are independent of each other which allows debugging the specific sub-project without touching the other sub-projects. Sub-projects can be reused across multiple projects if two separate projects require similar data processing.

The major drawback of having cascading projects is that if one sub-project stops then the whole project cascade stops. There might be some performance degradation due to the communication between sub-projects. Breaking a project into sub-projects needs to be handled carefully to avoid duplication of scenarios. Having extra Source windows is inevitable.

There can be various ways by which sub-projects can communicate with each other. We discuss them next.

#### Multiple Cascading Projects Using Message Buses

<p align="center">
 <img src="images/espedgeserver_message_bus.png"/>
    <br>
    <em>Figure 3: ESP edge server with cascading projects using message buses</em>
</p>

##### Characteristics
- This can be the same message bus bringing in the data to the first sub-project or a different one. 
- The message bus can run anywhere, in the VM or outside as long as ESP has access to it. However, one must consider the network latency induced because of the communication.
- Having message buses for communication allows loosing coupled sub-projects. 
- If the message bus has a configurable retention policy, for example, Kafka, then the streaming events can be stored in the configured Kafka topics. This allows sub-projects to start independently of each other without any data loss.

##### Limitations
- 3rd party message buses are required to be configured and managed (either in the same machine as of the ESP edge server or different). This can be an open-source or licensed version of the message bus. 
- Customers/Partners are responsible for the management and adminstration of the message buses. 
- There is a chance of reduced overall performance because of induced network latency for the communication between ESP and message buses. Performance must be thoroughly tested to ensure it complies with the desired SLOs.
- Availability of the message bus becomes critical for the projects to run. Customers/Partners must consider the high availability and resiliency configurations for the message bus. This would have added cost for additional resources.

#### Multiple Cascading Projects Using ESP Router

<p align="center">
 <img src="images/espedgeserver_router.png"/>
    <br>
    <em>Figure 4: ESP edge server with cascading projects using ESP router</em>
</p>

##### Description 
In Figure 4, we present the high-level architecture of multiple cascading projects connected via the [ESP router](https://go.documentation.sas.com/doc/en/espcdc/v_029/espxmllayer/n0hygnviv2kt6mn1vhnkgtsolxt7.htm). In the diagram, we have all the ESP projects and ESP router running in the same ESP edge server. ESP router enables disintegrating larger ESP projects into multiple and connects them seamlessly to perform the end-to-end stream processing.

##### Characteristics
- The ESP Router is capable of distributing events from the upstream project to downstream project(/s) where these downstream project/(s) could be running in the same ESP edge server or different.
- ESP router allows user-defined condition/(s) based routing. 
- We have better performance when the ESP router and downstream project/(s) co-habits in the same ESP edge server because now the events will be injected directly to the downstream ESP project/(s) without going through Pub/Sub APIs.

##### Limitations
- ESP Studio does not support creating/designing the ESP Router. XML router definition must be created manually.
- All the ESP projects and ESP router must be started manually and in the right sequence. For example, starting downstream projects first, followed by starting ESP router and then starting the upstream project.
- If one of the projects stops/fails/crashes, then the restart procedure must be followed due to the dependencies. 

#### Multiple Cascading Projects Using ESP Project Publish Connectors

<p align="center">
 <img src="images/espedgeserver_project_connector.png"/>
    <br>
    <em>Figure 5: ESP edge server with cascading projects using ESP project publish connectors</em>
</p>

##### Description
Another solution to stream events from one project to another is via [ESP Project Publish Connector](https://go.documentation.sas.com/doc/en/espcdc/v_029/espca/p0nd0g1i4np1h1n15zmar9fhln8u.htm) as shown in Figure 5. ESP project connector can connect to the project running on the same ESP edge server or a different one. 

##### Characteristics
- The ESP project connector is configured in the sub-project which gets the events from a different window of a different project. Connector starts automatically when the sub-project starts.
- The ESP project publish connector properties can be modified from ESP Studio just like we do for other ESP connectors.
- If all the projects are running in the same ESP edge server, then only a single copy of the event is stored in the memory as the ESP project publish connector uses a reference-counted copy of the events. Therefore, memory-optimized.  

##### Limitations
- ESP project connector causes tight coupling between the projects. If the source project crashes/stops, then the flow of events to the downstream projects stops as well. They must then be restarted manually.
- Sequence of starting the projects matters as the connectors need to have the source project running. There is a chance of data loss by the time all the sub-projects start because the source project needs to be running for the destination project connector to connect to.

















