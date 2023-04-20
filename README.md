# MQTT broker functionality in Event Grid

The Microsoft Azure Messaging team invites you and your organization to preview the MQTT broker functionality in Event Grid.  During this preview, we will provide full support with a high level of engagement from the Azure Messaging product group.  Please note that this preview is available by invitation only and requires an NDA.  By participating in the private preview, you agree to the [Terms of Use](https://www.microsoft.com/legal/terms-of-use).  Please submit the [form](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRxdDENSpgZtIq581m55eAQpURURXNEw4UkpTOEdNVTVXSllLQVhBUUo0US4u) to signup for private preview.  We look forward to your feedback as you leverage this capability. You can submit your feedback using this [form](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRxdDENSpgZtIq581m55eAQpURDA2RVRTV1VBSUQ2MDBCM1g3WkY4Q1k2Sy4u).  

**The Private Preview will be closing on 3/30/2023.**

<mark>You can continue testing till the end of March.  After 3/30/2023, your Event Grid Namespace will be disabled. Please feel free to delete any other resources you may have created during the private preview.</mark>

Please reach out to us with questions and comments at askmqtt@microsoft.com.

## Overview
MQTT broker functionality in Event Grid delivers the flexibility to leverage hierarchical topics and supports messaging using the light weight MQTT protocol.  Clients (both devices and cloud applications) can publish and subscribe over these flexible topics for scenarios such as command and control and high scale broadcast.

|Concepts|
| ------------ |
| [MQTT standard protocol](https://mqtt.org/) |
| [Client Authentication](#client-authentication) |
| [Client Groups](#client-groups) |
| [Topic Spaces](#topic-spaces)|

## Private preview program information
The private preview is only for testing.  Please do NOT use it for your production.

**Engagement:**  We will actively engage with you during the preview. At any point, feel free to connect with us for questions/concerns by emailing to askmqtt@microsoft.com.

**Feedback:**  At the end of the preview, we will capture additional feedback using this [form](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRxdDENSpgZtIq581m55eAQpURDA2RVRTV1VBSUQ2MDBCM1g3WkY4Q1k2Sy4u).

**Cost to use:**  For this release, MQTT broker functionality in Event Grid is available for no additional charge. You will be charged for routing MQTT messages through Event Grid subscriptions.  Please check the Event Grid pricing [here](https://azure.microsoft.com/en-us/pricing/details/event-grid/).

**Supported Region**
This private preview is currently supported only in East US 2 EUAP region.

**Post private preview program**
When the private preview program ends, or when your tests are completed, you can choose to either cleanup your configuration or retain the configuration in private preview East US 2 EUAP region.


## Capabilities available in this preview
This private preview provides the following capabilities
- Cloud MQTT broker functionality in Event Grid enabling publish and subscribe on flexible topic structure: support of wildcards in topic structure to allow subscription to filtered messages
- MQTT v3.1.1 compliance: 
	- Persistent sessions ensure that a client's subscription information and messages are preserved for 2 days when a client disconnects, and the messages are resent to the client when it reconnects on the same session.
	- Limitations:  LWT, Retain messages, Message ordering and QoS 2 are not supported. [Learn more](#mqttv311-level-of-support-and-limitations) 
- MQTT v5 compliance, below are some key features we support: 
	- User Properties on publish packet: Allows you to add additional information on publish packets to provide more context about the message.
	- Message Expiry Interval: You can set the message expiry interval (seconds) for each PUBLISH message and the message is stored by the broker for the defined period of time for any subscribing clients that are not connected currently.
	- Negative Acknowledgements:  Server can notify the publishing client, if the server cannot currently process the message, along with the reason.
	- Server-sent disconnect packet:  Server communicates the reason (in most scenarios) for disconnect to the client, to help client handle the disconnect better.
	- Topic Aliases:  Helps reduce the size of published packets by reducing the size of the topic field, making the data transfer more efficient.
	- Maximum message size:  The client can now tell the server the maximum message size the client can receive and any messages larger than this are dropped by the server.
	- Request-response:  We support response topic and correlation data properties on publish packet allowing clients to model request-response over MQTT per specification.
	- Flow control:  Helps adjust the message flow depending on device capabilities such as processing speed or storage capabilities by limiting number of QoS 1 messages to be dispatched to it simultaneously.
	- Clean Start and Session Expiry:  Helps preserve client's subscription details and messages for up to 2 days if configured (configurable between 0 and 172,800 seconds), in case the client disconnets.  When the client reconnects on the same session within the session expiry interval timeframe, all the messages are resent to the client.  Also, client doesn't need to resubscribe to topics.
	- Limitations:  LWT, Retain messages, Message ordering, QoS 2, Shared subscriptions, Subscription IDs, Auth packet, and Assigned Client ID are not supported. [Learn more](https://github.com/Azure/MQTTBrokerPrivatePreview#mqttv5-level-of-support-and-limitations)
- QoS 0, 1: QoS 0 level guarantees a best-effort delivery. QoS1 guarantees that the message will be delivered at least once.
- Flexible access control model:  Grouping clients into ClientGroups and topic references into TopicSpaces to ease access control management.  See the [concepts](#concepts) section for a fuller description of all functionality
- Fine-grained access control model:  Introducing TopicTemplates with variables support to enable fine-grained access control.
- Support for one-one, one-many, many-one messaging patterns to accommodate for a variety of scenarios
- Compatibility with standard MQTT client libraries (ex. Eclipse Paho) allows users to migrate configuration much faster
- Route MQTT messages through Event Grid subscriptions to integrate data with Azure services or custom endpoints for flexibility to further process the data
- Support for TLS 1.2 and TLS 1.3 endpoints for data plane operations to keep the data transmission secure
- Also, see [throttle limit tables](#mqtt-messages-limits) below

## Capabilities coming up in future releases
The following features are not in scope for this release, but they will be supported in future -
- Azure Portal UX, CLI, custom Azure SDK libraries along with APIs
- Ability to publish messages to topics using HTTP
- Edge MQTT Broker bridging
- Extended MQTT v3.1.1 support: LWT and Retain support
- Extended MQTT v5 support: LWT, Retain, Shared subscriptions, Subscription IDs, Auth packet, and Assigned Client ID support
- Metrics and diagnostic logs 
- Enhanced performance and scale limits 
- Pay As You Go Billing


## Prerequisites

- We will enable the feature for the subscription ID you shared in the sign up form. If you haven't responded, please fill out this [form](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRxdDENSpgZtIq581m55eAQpURURXNEw4UkpTOEdNVTVXSllLQVhBUUo0US4u)

- You can use any ARM client or the [Azure Portal experience](/Azure%20Portal/README.md) to deploy the service's resources and any compliant MQTT client to test the service. However, this guide will only provide commands and scripts using [Azure CLI](/Azure%20CLI/README.md) for deploying resouces and Paho client in python for testing in Ubuntu. Follow these steps to take advantage of the provided instructions: 
	1. Use any Linux environment for testing.
		- To easily install and run Linux on Windows, run `wsl --install -d Ubuntu`. After installation is complete, you can run `wsl` to start running commands on your Ubuntu subsystem whenever you open a new terminal window. [Learn more](https://learn.microsoft.com/en-us/windows/wsl/)
	2. Clone this repository to any directory in your Linux environment: `git clone https://github.com/Azure/MQTTBrokerPrivatePreview.git`
	3. Follow the instructions in the [Environment_configuration README](/Environment_configuration/README.md) to register Azure CLI and set common variables for resources deployment.
	4. Follow the instructions in the [Python README](https://github.com/Azure/MQTTBrokerPrivatePreview/tree/main/python#azure-mqtt-broker-sample-python-instructions) to be able to test the scenarios.
	5. Navigate to each of the scenario folders and follow its README.md instructions to run the scenario.

- To connect to the new MQTT broker functionality in Event Grid, the clients must be authenticated using X.509 certificates. Clients can be authenticated using a CA-signed certificate or a self-signed certificate.  Please see the [client authentication section](#client-authentication). You can use your own certificates or the certificate generation script provided in this guide and referenced in the scenarios' instructions.

- To route your messages from your clients to different Azure services or your custom endpoint, an Event Grid topic needs to be created and referenced in namespace to forward the messages to that topic for routing. [Scenario4](/Scenario4_Route%20MQTT%20data%20through%20Event%20Grid%20subscription) showcases an example of the configuration to take advantage of the routing functionality and the [Routing and Namespace sections of the generic Azure CLI instructions](https://github.com/Azure/MQTTBrokerPrivatePreview/tree/main/Azure%20CLI#event-grid-topic) also provide instructions on the generic routing configuration.  

## Demo

https://user-images.githubusercontent.com/59579210/215560807-df83c50f-38a8-49d7-a61d-d9d1b0ca3ddb.mp4

## QuickStart
Let us get started with a simple scenario, with a publisher and subscriber communicating on a topic. 

|Client | Role | Topic/Topic Filter|
| ------------ | ------------ | ------------ |
|Pub_client | Publisher | samples/topic |
|Sub_client | Subscriber | samples/topic |

After following the instructions in the [Prerequisites](#prerequisites), navigate to the Scenario0_Hello_World folder in your cloned repo through `cd ./MQTTBrokerPrivatePreview/Scenario0_Hello_World/`

Run the following commands to run the script, creating the resources: 
```bash
chmod 700 create_resources.sh
./create_resources.sh
```
To test the scenario:
1. If you haven't installed the required modules, follow the instructions in the [python README file](../python/README.md).
2. Make sure you have the `mqtt-broker` virtual environment activated by running `source ~/env/mqtt-broker/bin/activate` in Linux or `env/mqtt-broker/bin/activate` in Windows
3. In a terminal window, set up the following variable: `export gw_url="<namespace name>.southcentralus-1.ts.eventgrid.azure.net"` and run the sample script through the following command: `python ./subscribe.py`
4. In a different terminal window, set up the following variable: `export gw_url="<namespace name>.southcentralus-1.ts.eventgrid.azure.net"` and run the sample script through the following command: `python ./publish.py`


## Scenarios
Here are a few scenarios that showcase the functionality of the service. Follow the instructions in the [Prerequisites](#Prerequisites) to test these scenarios.

| # | Scenario | Description |
| ------------ | ------------ | ------------ |
| 1 | Fan-out (one-to-many) messages  | This scenario simulates cloud-to-client commands to several clients and can be leveraged for use cases such as sending alerts to clients. Consider the use case where a fleet management service needs to send a weather alert to all the vehicles in the fleet. For instructions see [README](/Scenario1_Fan-Out/README.md).  |
| 2 | Fan-in (many to one) messaging  | This scenario simulates publishing messages from multiple clients to a single client.  Consider a use case where one needs to identify location of vehicles on a map.  For instructions see [README](/Scenario2_Fan-In/README.md). |
| 3 | One to one messaging  | This scenario simulates publishing messages from one client to another.  Consider a use case where a user can unlock their car from a mobile app.  For instructions see [README](/Scenario3_One-to-One/README.md).  |
| 4 | Route MQTT data through Event Grid subscription  | This scenario showcases how to configure route to send filtered messages from MQTT to the endpoint: Event Hubs through EG subscription.  Consider a use case where one needs to identify location of vehicles.  For instructions see [README](/Scenario4_EventGrid_Routing/README.md).  |


Please share your feedback using this [form](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRxdDENSpgZtIq581m55eAQpURDA2RVRTV1VBSUQ2MDBCM1g3WkY4Q1k2Sy4u) or email us at askmqtt@microsoft.com with any questions you may have.


## Terminology
Some of the key terms relevant for private preview are explained below.

| Term| Definition |
| ------------ | ------------ |
| MQTT Broker| An MQTT broker is an intermediary entity that enables MQTT clients to communicate. Specifically, an MQTT broker receives messages published by clients, filters the messages by topic, and distributes them to subscribers. |
| Namespace| A namespace is a declarative space that provides a scope to the resources (certificates, clients, client groups, topic spaces, permission bindings, etc.) inside it.  Namespaces are used to organize the resources into logical groups. |
| Client| Client can be a device or a service that will publish and/or subscribe MQTT messages |
| Certificate / Cert| Certificate is a form of asymmetric credential. They are a combination of a public key from an asymmetric keypair and a set of metadata describing the valid uses of the keypair.  If the keypair of the issuer is the same keypair as the certificate, the certificate is said to be "self-signed". Third-party certificate issuers are sometimes called Certificate Authorities (CA). |
| Client attributes| Client attributes represent a set of key-value pairs that provide descriptive information about the client.  Client attributes are used in creating client groups and as variables in Topic Templates.   For example, client type is an attribute that provides the client's type. |
| Client group| Client group is a collection of clients that are grouped by a set of common client attribute(s) using a query string, and could be allowed to publish and/or subscribe to a specific Topic Spaces |
| Topic spaces | Topic spaces is a set of topic templates (defined below). It is used to simplify access control management by enabling you to grant publish or subscribe access to a group of topics at once instead of individual topics. |
| Topic filter| An MQTT topic filter is an MQTT topic that can include wildcards for one or more of its segments, allowing it to match multiple MQTT topics. It is used to simplify subscriptions declarations as one topic filter can match multiple topics. |
| Topic template| Topic templates are an extension of the topic filter that supports variables. It is used for fine-grained access control within a client group. |
| Permission bindings| A Permission Binding grants access to a specific client group to either publish or subscribe on a specific topic spaces.  |

## Concepts

### Namespace
A namespace is a declarative space that provides a scope to all the nested resources such as certificates, clients, client groups, topic spaces, permission bindings, etc. inside it.  Namespaces are used to organize the resources into logical groups.  Namespace is a tracked resource with 'tags' and a 'location' properties, and once created can be found on resources.azure.com.

Currently, Namespace resource type is enabled in following regions only.  

- East US2
- West US3
- Canary East US 2 EUAP
- Canary East US 2 EUAP

**After creating a Namespace, you will need to explicitly enable MQTT for the Namespace on the Configuration page.**

Once MQTT is enabled, it cannot be disabled for the Namespace.  Also, after enabling MQTT, you can choose client authentication settings.  

We recommend that you include the Client Authentication Name in the Username field of the client's CONNECT packet.  Using this authentication name along with the client certificate, service will be able to authenticate the client.

If you anticipate any client not having the authentication name in the Username field while connecting to the service, you can choose alternative source fields in client certificate for the client authentication name.  Service will look for the client identity from the client certificate in the order the fields are mentioned on the Configuration page.

We support 5 certificate fields, you can find the information [here](#client-authentication-source-options).  


### Client Authentication
For private preview, we are supporting authentication of clients using X.509 certificates.  X.509 certificate will provide the credentials to associate a particular client with the tenant.  In this model, authentication generally happens once during session establishment.  Then, all future operations using the same session are assumed to come from that identity. 
The following credential types are supported:
- Certificates issued by a Certificate Authority (CA) 
- Certificate thumbprint

**CA signed certificates:**  In this method, a root or intermediate X.509 certificate is registered with the service.  Essentially, the root or intermediary certificate that is used to sign the client certificate, must be registered with the service first.  Later, clients are authenticated if they have a valid leaf certificate that's signed by the root or intermediate certificate that was supplied to the service.  While registering the clients, the subject common name of the leaf certificate needs to be supplied for authentication. Service will validate the subject values match the subject values from the client certificate and also validate the client certificate is signed the root or intermediary certificate that was registered earlier.  

**Certificate Thumbprint:**  Clients are onboarded to the service using the certificate thumbprint alongside the identity record. In this method of authentication, the client registry will store the exact ID of the certificate that the client is going to use to authenticate. 


**Client Authentication Name:**  For the client, you can provide a unique identifier which is not restricted by Azure Resource Manager naming constraints.  It is a mandatory field and if not explicitly provided, it will be defaulted to the Client Name.

No two clients can have same authentication name within a Namespace.  While authenticating a client, we treat Client authentication name as case insensitive.

We preserve the original case of client authentication name that you configure in the client.  We use the original client authentication name (case sensitive) that was provided when client was created, in routing enrichments, topic space matching, etc.  


**Client Certificate Authentication Validation Scheme:**
To use CA certificate for authentication, you can choose from one of following options to specify the location of the client identity in the client certificate.  When the client tries to connect to the service, service will find the client identify from this certificate field and matches it with the client authentication name to authenciate the client.

We support 5 certificate fields, you can find the information [here](#client-authentication-source-options) corresponding to below options.  

- Subject Matches Authentication Name
- Dns Matches Authentication Name
- Uri Matches Authentication Name
- IP Matches Authentication Name
- Email Matches Authentication Name

Use the "Thumbprint Match" option while using the client certificate thumbprint to authenticate the client.


#### Multi-Session Support
To create multiple sessions per client, provide the Username property in the CONNECT packet to signify your client name, and the Client Identifier (ClientID) property in the CONNECT packet to signify the session name such as there are one or more values for the ClientID for each Username.
- If the Username property is not present in the CONNECT packet, the service will use the ClientID as the client name and session name.
- ClientID cannot be empty.

If a client tries to take over another client's active session by presenting its session name, its connection request will be rejected with an unauthorized error. E.g. if Client B tries to connect to session 123 that is assigned at that time for client A, Client B's connection request will be rejected. 

If a client resource is deleted without ending its session, other clients will not be able to use its session name until the session expires. E.g. If client B creates a session with session name 123 then client B deleted, client A will not be able to connect to session 123 until it expires. 
 
### Client Groups
Client group is a new concept introduced to make management of client access controls (publish/subscribe) easy – multiple clients can be grouped together based on certain commonalities to provide similar levels of authorization to publish and/or subscribe to Topic spaces.

Clients can be devices or applications, such as devices or vehicles that send/receive MQTT messages. We provide a default client group, named $all, which all clients are associated with. For ease of testing, you can considering using $all for premissions.

For example, a fleet management company with hundreds of trucks and other shipment delivery vehicles across the country, can improve their routing, tracking, driver safety and predictable maintenance capabilities by sending and receiving MQTT messages to/from the vehicles to/from monitoring applications on cloud.

In this scenario, vehicles can be configured as clients that publish/subscribe to various topics such as weather information, road conditions, geo location, engine performance and other wear-and-tear aspects of the vehicle.  And, while configuring the vehicle as a client, a set of attributes such as vehicle type, year, make & model, max load capacity can also be included as part of the client metadata via client attributes.

These client attributes can be used to create the client groups.  For example, if the vehicles that can carry loads over 2 tons are prone to accidents if driven with low braking fluid, in icy road conditions, then all such vehicles can be grouped together to continuously monitor and alert the drivers in case of potential hazardous conditions based on weather at their locations.

### Client group considerations:
The main purpose of client groups is to provide common authorization to a set of clients to either publish and/or subscribe on one or more Topic spaces.  Every client needs to be part of a client group to be able to publish and subscribe on topic spaces.  The goal is to keep the quantity of client groups very small to make permissions manageable.

For this preview, we will be supporting a maximum of 10 client groups per namespace.

While grouping clients, please ensure that it's easier to reuse the group to publish and subscribe across multiple topic spaces.  To this end, it is important to think through the end-to-end scenarios to identify the topics every client will publish to and subscribe to.  We recommend identifying the commonalities across the scenarios, to avoid over fragmentation of client groups and topic spaces.  Set the client attributes generic enough to achieve simple grouping and avoid highly complex group queries.

**What are client attributes?**
Client attributes are a set of user defined key-value pairs or tags that provide information about the client.  These attributes will be the main ingredient in the client group filtering expressions.  Attributes could be describing the physical or functional characteristics of the client.  Typical attribute could be type of the client (client type).  
Here is an example:
- Type: Values could be "sensor" or "thermostat" or "vehicle"

Here’s a sample schema for the attribute definition: 

```bash
{  
    "id": "device123",  
    "attributes": {  
        "type": "home-sensors",
        "sensors": ["motion", "noise", "light"]  
     }  
}
```

While configuring the client attributes, consider the topics that the clients will be publishing (subscribing) to.  Thinking backwards from topics to clients will help identifying the commonalities across client roles much easier and defining the client attributes to make the client grouping effortless.  

**How to create client group queries?**
To setup a client group, user needs to build a query that filters a set of clients based on their attribute values.

Here are a few sample queries:
- (attributes.sensors = "motion" or attributes.sensors = "humidity") or attributes.type = "home-sensors"
- attributes.sensors IN ["motion", "humidity", "temperature"] and attributes.floor <= 5
- authenticationName IN ['client1', 'client2']

In group queries, following operands are allowed:
- Equality operator "="
- Not equal operator in 2 forms "<>" and "!=" 
- Less than "<", greater than ">", less than equal to "<=", greater than equal to ">=" for long integer values
- "IN" to compare with a set of values

Please refer to the [naming considerations table](#naming-considerations) for details on allowed characters and patterns.


### Topic Spaces
A topic space is a set of topic templates (defined below). It is used to simplify access control management by enabling you to grant publish or subscribe access to a group of topics at once instead of individual topics.  
It is important to note that the publishing is a supported action on all topic spaces by default; however, you need to configure the subscription support as detailed below.

#### Topic filter:
An [MQTT topic filter](http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html#_Toc398718106) is an MQTT topic that can include wildcards for one or more of its segments, allowing it to match multiple MQTT topics. It is used to simplify subscriptions declarations as one topic filter can match multiple topics.

The service supports all the MQTT wildcards defined by the [MQTT specification](https://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html#_Toc398718107) as follows:
- +: which matches a single segment.
	- E.g. topic filter: "machines/+/alert" will match the following topics:
		- machines/temp/alert
		- machines/humidity/alert
- #: which matches zero or more segments at the end of the topic. 
	- E.g. topic filter: "machines/#" will match the following topics:
		- machines
		- machines/temp
		- machines/humidity
		- machines/temp/alert etc

See [Topic Wilcards](https://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html#_Toc398718107) from the MQTT spec for more details.

#### Topic template: 
Topic templates are an extension of the topic filter that supports variables. It is used for fine-grained access control within a client group. You can provide access to a client group to publish or subscribe on a topic space with multiple topic templates. If any of the topic templates include a variable, only the clients associated with this variable will have access to the corresponding topic.

For example, you can provide access to client group "machines" to the topic space "machinesTelemetry" that includes the topic template "machines/${client.authenticationName}/temp". Only the machine with client name = machine1 will be able to publish on topic "machines/machine1/temp", and only the machine with client name = machine2 will be able to publish on topic "machines/machine2/temp", and so on. This prevents machine2 from publishing false information on behalf of machine1, even though it has access to the same topic space, and vice versa. 

##### Supported variables:
- ${client.authenticationName}: this variable represents the name of the client assigned during client creation.
- ${client.attributes.x}: this variable represents any of the assigned attributes to a client during client creation/update, so as "x" would be equal to the exact string of the attribute key. Read more about client attributes in the Terminology section.

**Note:** A variable can represent a portion of a segment or an entire segment but cannot cover more than one segment. E.g. a topic template could include "machines/${client.authenticationName|.factory1}/temp" will match topics "machines/machine1.factory1/temp", "machines/machine2.factory1/temp", etc

#### Subscription support:
Subscription support is used to optimize the service’s mode of  message delivery to your clients based on your scenario. There are three modes: 
- Not supported: will indicate that topic space could be used only for publishing. This will be helpful in scenarios when you expect your corresponding topic templates to overlap since this is the only mode that will allow your topic templates to overlap with any other topic template.
- Low fanout: will specify a subscription delivery mode optimized for having a maximum of 10 subscribers per topic, with very low latency. 
	- Example scenario: There is a group of vehicles with each vehicle subscribing to its own command delivered to its own topic.
- High fanout: will specify a subscription delivery mode optimized for having unlimited number of subscribers per topic, with higher latency.
	- Example scenario: There is a large group of vehicles in a city subscribing to the same weather alerts that get broadcasted on the same topic.

#### Topic space considerations
- **Default Behavior:**
To publish or subscribe to any topic, a matching topic space must be configured, and a permission binding needs to be set for the client group(s) that include the clients that need publish/subscribe access to this topic space. 
- **Topic templates Overlap:**
If you set your topic space with a low fanout or high fanout subscription modes, the corresponding topic templates cannot overlap with each other, but they can overlap with a topic space with "not supported" subscription support. The overlap exists if a topic could match more than one topic template: 
	- Examples:
		- "machines/${client.authenticationName}/temp" and "machines/+/temp" overlap because the second template covers the first one via wildcard. 
		- vehicles/vehicle1/telemetry/# and vehicles/${client.authenticationName}/telemetry/# conflict because in the second template the segment with variable is treated as single level wildcard + and hence, covers the first topic template. PublishOnly topic spaces can overlap with LowFanout topic spaces.
- **Configuration:**
	- Topic templates use special characters $ and | and these need to be escaped differently based on the shell being used. In PowerShell, $ can be escaped with vehicles/${dollar}telemetry/#. If you’re using PowerShell, you can accomplish this as shown in the examples below: 
		- '"vehicles/${client.authenticationName|dollar}/#"'
		- 'vehicles/${client.authenticationName"|"dollar}/#'
	- Subscription support is immutable. To reconfigure the subscription support, delete the topic space and create a new topic space with the desired subscription support.
	- Topic space updates may take up to 5 minutes to propagate.


### Access Control

Client Groups and Topic Spaces are designed to simplify access control. Consider the following as you design Client Groups and Topic Spaces: 
- Group a set of clients into 'Client Groups' such that each Client Group represents the clients that need the same access to publish and/or subscribe to the same set of topics.
- Group a set of Topic Templates into 'Topic Spaces' such that each Topic Space represents messages meant for the same audience (a specific Client Group).

Examples:

The following examples detail how to configure the access control model based on the following requirements. 

- Example 1:

    A factory has multiple areas with each area including machines that need to communicate with each other. However, machines from other areas of the factory are not allowed to communicate with them.

	| Client  | Role  | Topic/Topic Filter  |
	| ------------ | ------------ | ------------ |
	|Area1_Machine1| Publisher| areas/area1/machines/machine1|
	|Area1_Machine2| Subscriber| areas/area1/machines/#|
	|Area2_Machine1| Publisher| areas/area2/machines/machine1|
	|Area2_Machine2| Subscriber| areas/area2/machines/#|

- Configuration
	- Create a Client Group for each factory area’s machines.
	- Create a Topic Space for each area representing the topics that the area’s machines will communicate over.
	- Create 2 Permission Bindings for each area’s Client Group to publish and subscribe to its corresponding area’s Topic Space.


	|Client| Client Group| Permission Binding| Topic Space|
	| ------------ | ------------ | ------------ | ------------ |
	|Area1_Machine1| Area1Machines| Area1-Pub| Area1Messages -Topic Template: areas/area1/machines/#|
	|Area1_Machine2| Area1Machines| Area1-Sub| Area1Messages -Topic Template: areas/area1/machines/#|
	|Area2_Machine1| Area2Machines| Area2-Pub| Area2Messages -Topic Template: areas/area2/machines/#|
	|Area2_Machine2| Area2Machines| Area2-Sub| Area2Messages -Topic Template: areas/area2/machines/#|

- Example 2:

    Let’s assume an extra requirement for the example above: each area has management nodes along with the machines, and the machines must not have publish access in case any of them gets compromised. On the other hand, the management nodes need publish access to send commands to the machines and subscribe access to receive telemetry from the machines.

	| Client | Role | Topic/Topic Filter | 
	| ------------ | ------------ | ------------ |
	| Area1_Machine1 | Publisher | areas/area1/machines/machine1
	|| Subscriber | areas/area1/mgmt/# | 
	| Area1_Mgmt1 | Publisher | areas/area1/mgmt/machine1
	|| Subscriber | areas/area1/machines/# | 
	| Area2_Machine1 | Publisher | areas/area2/machines/machine1
	| |Subscriber | areas/area2/mgmt/# | 
	| Area2_ Mgmt1 | Publisher | areas/area2/mgmt/machine1 | 
	||Subscriber | areas/area2/machines/# |

- Configuration:
	- Create 2 Client Groups per area: one for the management nodes and another for the machines.
	- Create 2 Topic Spaces for each area: one representing telemetry topics and another representing commands topics.
	- Create 2 Permission Bindings for each area’s management nodes to publish to the commands Topic Space and subscribe to the telemetry Topic Space.
	- Create 2 Permission Bindings for each area’s machines to subscribe to the commands Topic Space and publish to the telemetry Topic Space.


	|Client | Client Group | Permission Binding | Topic/Topic Filter|
	| ------------ | ------------ | ------------ | ------------ |
	|Area1_Machine1 | Area1Machines | Area1Machines-Pub | Area1Telemetry -Topic Template: areas/area1/machines/#|
	| |  | Area1Machines-Sub | Area1Commands -Topic Template: areas/area1/mgmt/#|
	|Area1_MgmtClient1 | Area1Mgmt | Area1Mgmt-Pub | Area1Commands -Topic Template: areas/area1/mgmt/#|
	| |  | Area1Mgmt-Sub | Area1Telemetry -Topic Template: areas/area1/machines/#|
	|Area2_Machine1 | Area2Machines | Area2Machines-Pub | Area2Telemetry -Topic Template: areas/area2/machines/#|
	| |  | Area2Machines-Sub | Area2Commands -Topic Template: areas/area2/mgmt/#|
	|Area2_ MgmtClient1 | Area2Mgmt | Area2Mgmt-Pub | Area2Commands -Topic Template: areas/area2/mgmt/#|
	| |  | Area2Mgmt-Sub | Area2Telemetry -Topic Template: areas/area2/machines/#|
 
### Routing
This functionality will enable you to route your messages from your clients to different Azure services like Event hubs, Service Bus, etc or your custom endpoint. This functionality is achieved through [Event Grid](https://docs.microsoft.com/en-us/azure/event-grid/), by sending all your messages from your clients to an [Event Grid topic](https://docs.microsoft.com/en-us/azure/event-grid/custom-topics), and using [Event Grid subscriptions](https://docs.microsoft.com/en-us/azure/event-grid/subscribe-through-portal) to route the messages from that Event Grid topic to the [supported endpoints](https://docs.microsoft.com/en-us/azure/event-grid/event-handlers).

Event Grid is a highly scalable, serverless event broker that you can use to integrate applications using events. Events are delivered by Event Grid to subscriber destinations such as applications, Azure services, or any endpoint to which Event Grid has network access. [Learn more](https://docs.microsoft.com/en-us/azure/event-grid/)

**Note:**  To be able to take advantage of this feature, you have to either use the ARM template to create the Event Grid custom topic first and provide that topic’s ARM Id during the namespace creation.

[Scenario4](https://github.com/Azure/MQTTBrokerPrivatePreview/blob/main/Scenario4_EventGrid_Routing/README.md) showcases an example of taking advantage of the routing functionality and the [Routing and Namespace section of the generic Azure CLI instructions](https://github.com/Azure/MQTTBrokerPrivatePreview/tree/main/Azure%20CLI#event-grid-topic) also provides instructions on the routing configuration.  

#### How can I use the routing feature?
Routing the messages from your clients to an Azure service or your custom endpoint enables you to maximize the benefits of this data. The following are some of many use cases to take advantage of this feature:
- Data Analysis: extract and analyze the routed messags from your clients to optimize your solution. e.g. analyze your machines' telemetry to predict when to schedule maintenance before failures happen to avoid delays and further damage.
- Serverless applications: trigger a serverless function based on the routed messages from your clients. E.g. when a motion sensor detects a motion, send a notification to security personnel to address it.
- Data Visualizations: build visualizations of the routed data from your clients to easily represent and understand the data as well as highlight trends and outliers.

#### Routing Considerations:
- The Event grid topic that will be used for routing need to fulfil the following requirements:
	- It needs to be set to use the Cloud Event Schema v1.0
	- It needs to be in the same region as the namespace
	- You need to assign "EventGrid Data Sender" role to yourself on the Event Grid Topic.
		- In the portal, go to the created Event Grid topic resource. 
		- In the "Access control (IAM)" menu item, select "Add a role assignment".
		- In the "Role" tab, select "EventGrid Data Sender", then select "Next".
		- In the "Members" tab, click on "+Select members", then type your AD user name in the "Select" box that will appear (e.g. user@contoso.com).
		- Select your AD user name, then select "Review + assign"
> **Warning**

>  If you move the Event Grid Topic resource to a different resource group or subscription after configuration, the routing experience will not function as expected.

- **Filtering:**
	- You can use the Event Grid Subscription’s filtering capability to filter the routed messages based on the MQTT topic through filtering on the "subject" property in the Cloud Event schema. Event Grid Subscriptions supports free simple subject filtering by specifying a starting or ending value for the subject. For example, 
		- You can specify the subject ends with "gps" to only route messages reporting on location. 
		- You can filter the subject begins with "factory1/Area2/" to route only the messages that belong to facotry1 and area 2 to a specific endpoint and you can replicate this configuration to route messages from other factories/areas to different endpoints.
	- You can also take advantage of the [Event Subscription’s advanced filtering](https://docs.microsoft.com/en-us/azure/event-grid/event-filtering#advanced-filtering) to filter based on the MQTT topic through filtering on the subject property in the Cloud Event Schema. This enable you to set more complex filters by secifying a comparison operator, key, and value.

#### CloudEvent Schema for the routed MQTT messages:
MQTT Messages will be routed to the Event Grid Topic as CloudEvents according to the following logic:
- For MQTT v3 messages or MQTT v5 messages of a payload format indicator=0, the payload will be forwarded in the data_base64 object and encoded as a base 64 string according to the following schema sample.
```bash
{
   "specversion": "1.0",
   "id": "9aeb0fdf-c01e-0131-0922-9eb54906e20", // unique id stamped by the service
   "time": "2019-11-18T15:13:39.4589254Z", // timestamp when messages was received by the service
   "type": "MQTT.EventPublished", // set type for all MQTT messages enveloped by the service
   "source": "testnamespace", // namespace name
   "subject": "campus/buildings/building17", // topic of the MQTT publish request 
   "data_base64": {
          IlRlbXAiOiAiNzAiLAoiaHVtaWRpdHkiOiAiNDAiCg==
  }
 }
 ```

- For MQTT v5 messages of content type= “application/json; charset=utf-8” or of a payload format indicator=1, the payload will be forwarded in the data object, and the message will be serialized as a JSON or a JSON string if the payload not a JSON. This will enable you to filter on your payload properties. For example, you would be able to add this filter for the following sample: "advancedFilters": [{"operatorType": "NumberGreaterThan","key": "data.Temp","value": 100}] 
```bash
{
   "specversion": "1.0",
   "id": "9aeb0fdf-c01e-0131-0922-9eb54906e20", // unique id stamped by the service
   "time": "2019-11-18T15:13:39.4589254Z", // timestamp when messages was received by the service
   "type": "MQTT.EventPublished", // set type for all MQTT messages enveloped by the service
   "source": "testnamespace", // namespace name
   "subject": "campus/buildings/building17", // topic of the MQTT publish request 
   "data": {
         "Temp": "70",
         "humidity": "40"
  }
 }
 ```
- For MQTT v5 messages that are already enveloped in a CloudEvent according to the [MQTT Protocol Binding for CloudEvents](https://github.com/cloudevents/spec/blob/v1.0/mqtt-protocol-binding.md) whether using the binary content mode or the structured content mode in JSON encoding (utf-8), the event will be forwarded with the original default CloudEvents attributes after enrichments according to the following schema sample.
```bash
{
  "specverion": "1.0",
  "id": "9aeb0fdf-c01e-0131-0922-9eb54906e20", // original id stamped by the client 
  "time": "2019-11-18T15:13:39.4589254Z", // timestamp when messages was received by the client
  "type": "Custom.Type", // original type value stamped by the client
  "source": "Custom.Source", // original source value stamped by the client
  "subject": " Custom.Subject", // original subjectvalue stamped by the client
   "data": {
         "Temp": "70",
         "humidity": "40"
  }
 }
```
#### Enrichments

The enrichments support enable you to add up to 10 custom key-value properties to your messages before they are sent to the Event Grid topic. These enrichments enable you to:
- Add contextual data to your messages. For example, enriching the message with the client's name or the namespace name could provide endpoints with information about the source of the message.
- Reduce computing load on endpoints. For example, enriching the message with the MQTT publish request's payload format indicator informs endpoints how to process the message's payload without trying multiple parsers first.
- Filter your routed messages through Event Grid Subscriptions based on the added data. For example, enriching a client attribute will enable you to filter the messages to be routed to the endpoint based on the different attribute's values.

The enrichment key is a string that needs to comply with these requirements:
- Include only lower-case alphanumerics: only (a-z) and (0-9)
- Must not be "specversion", "id", "time", "type", "source", "subject", "datacontenttype", "dataschema", or "data".
- Must not start with “azsp”.
- Must not be duplicated.
- Must not be more than 20 characters.

The enrichment value could be a static string for static enrichments or one of the supported values that represent the client attributes or the MQTT message properties for dynamic enrichment. Enrichment values must not be more than 128 characters.
The following is a list of the supported values:

###### Client attributes
- ${client.authenticationName}: the name of the publishing client.
- ${client.attributes.x}: an attribute of the publishing client, where x is the attribute key name. 

###### MQTT Properties
- ${mqtt.message.userProperties.x}: user properties in the MQTTv5 publish request, where x is the user property key name 
  - Type: string
  - Use the following variable format instead if your user property includes special characters ${mqtt.message.userProperties['x']}. You will still need to escape an apostrophe and backslash as follows: and "PN\t" becomes "PN\\\\t".
- ${mqtt.message.topicName}: the topic in the MQTT publish request.
  - Type: string
- ${mqtt.message.responseTopic}: the response topic in the MQTTv5 publish request.
  - Type: string
- ${mqtt.message.correlationData}: the correlation data in the MQTTv5 publish request.
  - Type: binary
- ${mqtt.message.pfi}: the payload format indicator in the MQTTv5 publish request.
  - Type: integer

##### Enrichments Configuration

Enrichment can be configured on the namespace creation/update through Azure CLI. The following is a sample JSON for the namespace payload:

 ```
{
    "properties": {
        "topicSpacesConfiguration": {
            "state": "Enabled",
            "routeTopicResourceId": "/subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/test-resrouce-group/providers/Microsoft.EventGrid/topics/test-topic",
            "routingEnrichments": {
                "static": [
                    {
                        "key": "namespaceid",
                        "value": "123",
                        "valueType": "string"
                    }
                ],
                "dynamic": [
                    {
                        "key": "clientname",
                        "value": "${client.authenticationName}"
                    },
                    {
                        "key": "clienttype",
                        "value": "${client.attributes.type}"
                    },
                    {
                        "key": "address",
                        "value": "${mqtt.message.userProperties['client.address']}"
                    },
                    {
                        "key": "region",
                        "value": "${mqtt.message.userProperties.location}"
                    },
                    {
                        "key": "mqtttopic",
                        "value": "${mqtt.message.topicName}"
                    },
                    {
                        "key": "mqttresponsetopic",
                        "value": "${mqtt.message.responseTopic}"
                    },
                    {
                        "key": "mqttcorrelationdata",
                        "value": "${mqtt.message.correlationData}"
                    },
                    {
                        "key": "mqttpfi",
                        "value": "${mqtt.message.pfi}"
                    }
                ]
            }
        }
    },
    "location": "centraluseuap",
    "tags": {},
}
```
	
##### Sample Output

The following is a sample output of a MQTTv5 message with PFI=0 after applying the enrichment configuration above: 

```bash
{
   "specversion": "1.0",
   "id": "9aeb0fdf-c01e-0131-0922-9eb54906e20", // unique id stamped by the service
   "time": "2019-11-18T15:13:39.4589254Z", // timestamp when messages was received by the service
   "type": "MQTT.EventPublished", // set type for all MQTT messages enveloped by the service
   "source": "testnamespace", // namespace name
   "subject": "campus/buildings/building17", // topic of the MQTT publish request 
   "namespaceid": "123", // static enrichment
   "clientname": "client1", // dynamic enrichment of the name of the publishing client
   "clienttype": "operator", // dynamic enrichment of an attribute of the publishing client
   "address": "1 Microsoft Way, Redmond, WA 98052", // dynamic enrichment of a user property in the MQTT publish request
   "region": "North America", // dynamic enrichment of another user property in the MQTT publish request
   "mqtttopic": "campus/buildings/building17", // dynamic enrichment of the topic of the MQTT publish request
   "mqttresponsetopic": "campus/buildings/building17/response", // dynamic enrichment of the response topic of the MQTT publish request
   "mqttcorrelationdata": "cmVxdWVzdDE=", // dynamic enrichment of the correlation data of the MQTT publish request encoded in base64
   "mqttpfi": 0, // dynamic enrichment of the payload format indicator of the MQTT publish request
   "datacontenttype": "application/octet-stream", //content type of the MQTT publish request
   "data_base64": {
          IlRlbXAiOiAiNzAiLAoiaHVtaWRpdHkiOiAiNDAiCg==
  }
 }
```
###### Handling special cases:
- Unspecified clinet attributes/user properties: if a dynamic enrichment pointed to a client attribute/user property that doesn’t exist, the enrichment will include the specified key with an empty string for a value. e.g. "emptyproperty": ""
- Arrays: Arrays in client attributes and duplicate userproperties will be transformed to a comma-separated string. For example: if the enriched client attribute is set to be “array”: “value1”, “value2”, “value3”, the resulting enriched property will be “array”: “value1,value2,value3”. Another example: if the same MQTT publish request has the following user properties >  "userproperty1": "value1", "userproperty1": "value2", resulting enriched property will be “userproperty1”: “value1,value2”.

## Limitations
### MQTTv3.1.1 Level of Support and Limitations
MQTT v3.1.1 support is limited in following ways:
- Will Message is not supported yet. Receiving a CONNECT request with Will Message will result in a connection failure.
- QoS2 and Retain Flag are not supported yet. A publish request with a retain flag or with a QoS2 will fail and close the connection.
- Message ordering is not guaranteed.
- Keep Alive Maximum is 1160 seconds. 
### MQTTv5 Level of Support and Limitations
MQTT v5 support is limited in following ways (communicated to client via CONNACK properties unless explicitly noted otherwise):
- Shared Subscriptions are not supported yet.
- Retain flag is not supported yet.
- Will Message is not supported yet. Receiving a CONNECT request with Will Message will result in CONNACK with 0x83 (Implementation specific error).
- Maximum QoS is 1.
- Maximum Packet Size is 512 KiB
- Message ordering is not guaranteed.
- Subscription Identifiers are not supported.
- Assigned Client Identifiers are not supported yet.
- Client-specified Session Expiry is not supported. The server will override any Session Expiry Interval value coming in the  CONNECT request (other than 0) with a fixed expiry interval (3600) in the CONNACK’s Session Expiry Interval property.
- Since the only supported authentication mode is through certificates, the server will respond to a CONNECT request with either Authentication Method or Authentication Data with a CONNACK with code 0x8C (Bad authentication method) or 0x87 (Not Authorized) respectively. 
- Topic Alias Maximum is 10. The server will not assign any topic aliases for outgoing messages at this time. Clients can assign and use topic aliases within set limit. 
- CONNACK doesn't return Response Information property even if the  CONNECT request contains Request Response Information property.
- If the server recieves a PUBACK from a client with non-success response code, the connection will be terminated.
- Keep Alive Maximum is 1160 seconds.

### MQTT Messages Limits
For this release, the following limits are supported.  Please do not stress test beyond the limits mentioned below and for other scenarios.  These limits will be revised for future releases. 

|Limit Description | Limit |
| ------------ | ------------ |
|Max Message size | 512 KiB |
|Topic Size| 256 B | 
|Topic Alias | 10 Topic Aliases| 
|New Connect requests | 500 requests/second |
|Subscribe requests | 5000 requests/second |
|Total number of subscriptions per connection | 50 |
|Total inbound publish requests | 4000 messages/second |
|Total outbound publish requests | 4000 messages/second |
|Total inbound Publish requests per connection | 100 messages/second |
|Total outbound Publish requests per connection | 100 messages/second |

**Note:**  A message is counted in 1 KiB increments. For example, a 6 KiB message is counted as 6 messages.

### Resources Limits
| Resource Name | Description| Limit| 
| ------------ | ------------ | ------------ |
| Name spaces | Maximum number of name spaces per subscription | 10 |
| Clients | Maximum number of clients | 10,000 |
| CA Certificates | Maximum number of registered CA root certificates | 2 |
| Client Groups | Maximum number of client groups| 10 |
| Topic spaces | Maximum number of topic spaces | 10 |
| Topic templates | Maximum number of topic templates within a topic space | 10 |
| Permission bindings | Maximum number of permission bindings | 100 |

## Naming considerations
All the names are of String type

| Category| Name length| Allowed characters| Other considerations|
| ------------ | ------------ | ------------ | ------------ |
| Namespace| 3-50 characters| Alphanumeric, and hyphens(-); no spaces|  Name needs to be unique per region | 
|CA Certificate| 3-50 characters| Alphanumeric, hyphen(-) and, no spaces| Name needs to be unique per namespace | 
| Client| 1-128 characters| Alphanumeric, hyphen(-), colon(:), dot(.), and underscore(_), no spaces| Case sensitive; Name needs to be unique per namespace | 
| Client authentication name| 1-128 characters| UTf-8 strings, no restrictions | Case sensitive; Authentication Name needs to be unique per namespace (case is ignored while determining uniqueness)  | 
| Client attributes| Total size of the attributes is <=4KB| Alphanumeric and underscores(_)| Case sensitive; Attribute values can be strings, string arrays, integers; Name needs to be unique per namespace| 
| Client Group| 3-50 characters| Alphanumeric, hyphen(-) and, no spaces| $all is the default client group that includes all the clients.  This group cannot be edited or deleted; Name needs to be unique per namespace| 
| TopicSpace| 3-50 characters| Alphanumeric, hyphen(-) and, no spaces| |
| Permission Bindings| 3-50 characters| Alphanumeric, hyphen(-) and, no spaces| Name needs to be unique per namespace | 


## Client authentication source options 
| Authentication Source Option | Certificate field | Description |
| ------------ | ------------ | ------------ |
| Certificate Subject Name | tls_client_auth_subject_dn | The subject distinguished name of the certificate. |
| Certificate Dns | tls_client_auth_san_dns | The dNSName SAN entry in the certificate. |
| Certificate Uri | tls_client_auth_san_uri | The uniformResourceIdentifier SAN entry in the certificate. |
| Certificate Ip | tls_client_auth_san_ip | The IPv4 or IPv6 address present in the iPAddress SAN entry in the certificate. |
| Certificate Email | tls_client_auth_san_email | The rfc822Name SAN entry in the certificate. |



## Frequently asked questions 
- Is Azure monitoring metrics and logging available? 
	- Not in this release.  We will add monitoring metrics and diagnostic logs in the next release.
- What happens if client attempts to publish on a topic when a matching topic space is not found? 
	- Client connection will be closed. Identify which client groups the client belongs to.  Based on the permission bindings, identify the topic spaces to which the client groups have access to publish.
- How do I connect an MQTT client that requires username and password for authentication?
	- Username and password based authentication is not supported in this release.  It will be supported in a future release.
- What happens if I have more than 10 subscribers per topic for a low fanout topic space?
    - In this release, the 11th subscription request for the same topic will be rejected.
- How do Azure IoT Hub and Azure Event Grid compare?
    - While Azure IoT Hub and Azure Event Grid (with new MQTT Broker capability) enable messaging between devices and services, that are necessary for IoT solutions, there are some key differences. Both services support device-to-cloud and cloud-to-device communication but Event Grid also enables device-to-device communication, via the cloud MQTT Broker. IoT Hub provides richer device management and bulk-device provisioning through integration with Azure Device Provisioning Service. In terms of protocols Event Grid supports HTTP & MQTT and IoT Hub supports HTTP, MQTT & AMQP. For messaging use cases using MQTT protocol, you can start with Azure Event Grid. If your solution requires bulk device provisioning and device management, then you can start with Azure IoT Hub. In future, Azure Event Grid will integrate with Azure Device Provisioning Service to enable these bulk provisioning capabilities.




Please share your feedback using this [form](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRxdDENSpgZtIq581m55eAQpURDA2RVRTV1VBSUQ2MDBCM1g3WkY4Q1k2Sy4u) or email us at askmqtt@microsoft.com with any questions.

