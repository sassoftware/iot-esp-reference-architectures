# Streaming Architecture Design Guide Questionnaire

## Table of Contents
* [Overview](#overview)
    - [Description](#description)
* [Requirements](#requirements)
* [Data Ingestion](#data-ingestion)
* [Analytics](#analytics)
* [Storage Considerations](#storage-considerations)
* [Displaying and Capturing Results](#displaying-and-capturing-results)
* [Platform](#platform)

## Overview 
<p align="center">
 <img src="https://github.com/sassoftware/iot-esp-reference-architectures/blob/3e15c82ebca767fb95e794af60e07056ab978468/05_Architecture_Guide_Questions/images/Streaming_design_considerations.jpg"/>
    <br>
    <em>Streaming System Design Considerations</em>
</p>

### Description

Building complete and functional IoT projects can be a puzzle full of surprises if you are not aware of all the 
building pieces that must be taken into account. 
IoT projects contain many parts which must work together to solve specific requirements.
Connecting these options can be confusing when each part of the whole is considered individually.
Fortunately, all IoT projects share the same building blocks from an architectural point of view.
Once you have a clear understanding of these building blocks, it will be easier for you to design a successful IoT project.
The purpose of this guide is to provide a list of questions that will help turn requirements into successes. 

The graphic above is an example of how an IoT project may be broken into building blocks.
Every project starts with a need or requirements.  What problem are you trying to solve?
Once requirements are understood we move onto data ingestion.
Here we define all the data sources, their data types, and the transport types and used to move data from
where it is produced to the analytical engine which will be used to transform and analyze it.
Next we transform and analyze the data.
Our analytical results need to be saved.  This is the purpose of the storage building block.
Now all our calculations are complete and saved we need to display our results or take actions.
Last but not least we need to understand the platform which will be used to host our solution.


## Requirements

- What problem are you trying to solve? What is the business use-case?
- System User description
    - Data Science vs Alerting​
    - Will models be developed in house?
- How many projects will be produced and how often?
- What types of analytics are required​?
    - Can groups of devices use similar analytics?
- What are the availability requirements?

## Data Ingestion

- Where does the data originate?​
- How many data sources are there?​
- From message broker/bus?​ 
    - If using Kafka, how many topics are there and how many partitions per topic?
    - Can we add more partitions if necessary?
- Directly from data source?​
- What are the incoming data rates?​ What is the expected cumulative number of events per second?
- Are they steady or will there be significant fluctuation?​
- Can the incoming data stream be processed in parallel?​
    - How are the events distributed?
    - Are the events ordered or unordered?
    - Should the ordering be maintained?
- Are the data sources stationary or mobile?
- What is the expected throughput?
- What is the maximum acceptable latency?

## Analytics
- What data transformations are necessary​?
- What type of analytics used​?
- List Analytic base table requirements​
- Will the deployment be stateless or will it retain some state?​
- Will the deployment need to be modularized?​
- Will there be multiple ESP projects that need to communicate with each other?​
- Will users be creating new modules or removing old ones?​
- Do we need auto-scaling of the ESP servers to handle the fluctating workload?

## Storage Considerations

- What are the storage requirements​
    - Long term ​
    - Short term ​
    - In memory​
- What are the retention periods?
- How big is the reference data?

## Displaying and Capturing Results

- How are alerts/notifications being delivered?​
- Can duplicated alerts be tolerated?​
- What is the format of the reporting tables?​
- How will reports and alerts be delivered?​
- How and where are the results stored or streamed to?

## Platform

- Where is the processing power needed?​
    - Edge, Fog, Cloud?​
- How much processing power (compute resources) is needed at the Edge IoT Devices, Edge Gateways, and Cloud?​
- How many IoT devices/sensors are there?
- Hosting:​
    - On-prem​ises
    - Public/Private Cloud ​
    - Hybrid-cloud
- Is a MapReduce framework an option?
- Who will manage the 3rd party applications?










