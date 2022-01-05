# Standard Definitions of Non-Functional Requirements 

### Elasticity
It is the ability to increase or decrease the resources available to an application (such as computing processing, memory, storage, etc.) dynamically as required to adapt to the workload fluctuations at runtime. A software system can scale horizontally and vertically. By vertical (scale-up) we mean, adding more CPU and memory to the existing machines. Horizontal (scale-out) means adding more nodes with CPU and memory (but not always linear).

Elasticity is the ability of the cloud.

### Scalability
It is the ability of an application (i.e., ESP) to serve an increasing workload using the scaled infrastructure while adhering to Service-Level Agreement (SLA) guarantees.
 
### Redundancy
It is to have replicated hardware or software resources that can be used when the main resource fails. Redundancy doesn’t always imply highly available, as the time required to perform failover (or takeover) might exceed HA SLA (see below).    

### Availability
It is a measure of the time that a service is functioning normally. It is a measure of the percentage of uptime (considering the downtime due to faults + maintenance). High Availability: covers services that have availability above 99.99++%. Such services are often called as always on, always available. 
 
### Failover/Takeover
The process of switching to a redundant resource when the primary resource fails due to a software or hardware issue. Having a redundant system (i.e., secondary or standby)  allows to resume service with minimal downtime (ideally without breaching availability SLA) It is one of the most common ways to achieve high availability.
 
*Point to NOTE: The concepts of High Availability and Failover do not imply any guarantees on message delivery by themselves. Additional mechanisms are required that avoid message loss or duplication during the failover process.*
 
### Reliability
It is measured as MTBF (Mean time between failures).  It is driven by the frequency of failures and their impact. It maintains data reliability during the episodes of failure. Reliability is the ability of the application/system to produce correct output within the desired time period.
 
### Fault Tolerance 
It is the ability to tolerate any fault (including software errors or hardware failures) without any observable faults by the user including data loss/duplication. Failover is a subset of Fault tolerance in a distributed system.
 
*Point to NOTE: An application can be highly available but not fault-tolerant. Moreover, an application can be fault-tolerant but not Highly Available (Fault Tolerance does not put any limits on the time it takes to recover from a fault).*
 
### Resiliency 
It is a weaker form of Fault Tolerance. Resiliency only guarantees that persistent data/state will not be affected by any types of faults. Whereas uncommitted (cached) data might be lost/duplicated/corrupted in the case of a failure. An application is not resilient if it is not reliable. 
 
*Point to NOTE: Reliability is the outcome and resiliency is the way to achieve the outcome.*
 
### Disaster Recovery 
It is to have a secure backup strategy that stores and maintains data and state replication.
