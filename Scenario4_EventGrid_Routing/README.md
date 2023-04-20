# Scenario 4 – Route MQTT data through Event Grid subscription
This scenario showcases how to configure routing to send filtered messages from the MQTT Broker to an [Azure storage queue](https://learn.microsoft.com/en-us/azure/storage/queues/storage-quickstart-queues-portal). Consider a use case where one needs to identify the location of vehicles, and want to route the vehicles’ location data from a specific client to a separate storage queue.

## Scenario:

|Client | Role | Topic/Topic Filter|
| ------------ | ------------ | ------------ |
|s4-vehicle1 | Publisher | vehicles/vehicle1/GPS/position|

## Resource Configuration:
|Client| Client Group| PermissionBinding (Role)| TopicSpaces|
| ------------ | ------------ | ------------ | ------------ |
|s4-vehicle1 (Attributes: “Type”:”vehicle”)| vehicle | vehicle-publisher|  VehiclesLocation: (Topic Templates: vehicles/{client.name}/GPS/position  -Subscription Support: Not supported)|'

Follow the instructions in the Prerequisites to test this scenarios. Use the following instructions to configure the resources and test the scenario.

## Configure the resources:

- Configure your Event Grid Topic where your messages will be routed.
```bash
# Create the topic in the East US 2 EUAP Region and set the input schema to CloudEvent Schema v1.0
az eventgrid topic create -g ${rg_name} --name <topic name> -l centraluseuap --input-schema cloudeventschemav1_0
# Register the Event Grid resource provider
az provider register --namespace Microsoft.EventGrid
```
- Set EventGrid Data Sender role to your user ID
	- In the portal, go to the created Event Grid topic resource. 
	- In the "Access control (IAM)" menu item, select "Add a role assignment".
	- In the "Role" tab, select "EventGrid Data Sender", then select "Next".
	- In the "Members" tab, click on "+Select members", then type your AD user name in the "Select" box that will appear (e.g. user@contoso.com).
	- Select your AD user name, then select "Review + assign"

- Edit NS_Scenario4.json to add the reference to the topic that you just created
	- Navigate to the resources directory through `cd ./resources`
	- Edit the file NS_Scenario4.json to add the topic reference as shown in the following example:`"routeTopicResourceId": "/subscriptions/<Subscription ID>/resourceGroups/<Resource Group Name>/providers/Microsoft.EventGrid/topics/<Event Grid Topic Name>"`
	- Navigate back to the "Scenario4_Route MQTT data through Event Grid subscription" directory through `cd ..`

- Set a unique namespace name in the following varibale to be used in the following commands
```bash
export resource_prefix="${ns_id_prefix}/<unique namespace name>"
```
- Create a namespace with a reference to the Event Grid Topic that you just created
```bash
az resource create --resource-type ${base_type} --id ${resource_prefix} --is-full-object --api-version 2022-10-15-preview --properties @./resources/NS_Scenario4.json
```
- Create the CA Certificate:
```bash
az resource create --resource-type ${base_type}/caCertificates --id ${resource_prefix}/caCertificates/test-ca-cert --api-version 2022-10-15-preview --properties @./resources/CAC_test-ca-cert.json
```
- Generate the client certificate using the cert-gen scripts. You can skip this step if you're using your own certificate.
```bash
pushd ../cert-gen
./certGen.sh create_leaf_certificate_from_intermediate s4-vehicle1
popd
```
- Register the following clients:
	- s4-vehicle1
		- Attribute: Type=vehicle
```bash
az resource create --resource-type ${base_type}/clients --id ${resource_prefix}/clients/s4-vehicle1 --api-version 2022-10-15-preview --properties @./resources/C_vehicle1.json
```
- Create the following client groups:
	- vehicle to include s4-vehicle1 client
		- Query: ${client.attribute.Type}= “vehicle”
```bash
az resource create --resource-type ${base_type}/clientGroups --id ${resource_prefix}/clientGroups/vehicle --api-version 2022-10-15-preview --properties @./resources/CG_vehicle.json
```
- Create the following topic space:
	- vehicle-publish 
		- Topic Template: vehicles/${client.name}/GPS/#
		- Subscription Support: Not supported
```bash
az resource create --resource-type ${base_type}/topicSpaces --id ${resource_prefix}/topicSpaces/vehicle-publish --api-version 2022-10-15-preview --properties @./resources/TS_vehicle-publish.json
```
- Create the following permission bindings:
	- vehicle-publisher: to grant access for the client group vehicle to publish to the topic space vehicle-publish
```bash
az resource create --resource-type ${base_type}/permissionBindings --id ${resource_prefix}/permissionBindings/vehicle-publisher --api-version 2022-10-15-preview --properties @./resources/PB_vehicle-publisher.json
```

## Test the scenario:
Use the following steps to set up a subscription on your created event grid topic, run the python scripts to send messages, and observe the messages on your endpoint.

### Set up an Event Grid Subscription to your storage queue:
- In the portal, go to the created Event Grid topic resource, and select "+ Event Subscription" in the Overview menu item.
- In the Basics tab, provide the following fields:
	- Name: your event subscription name
	- Event Schema: Cloud Event Schema v1.0
	- Endpoint type: Storage Queues
	- Endpoint: your Storage Queues endpoint
- Go to the Filters tab, and “Enable subject filtering”
	- In the field “Subject Begins With”, type “vehicles/s4-vehicle1”
		- The MQTT topic is represented by the Subject field in the routed Cloud Event Schema so this configuration will filter all the messages with the MQTT Topic that starts with “vehicles/s4-vehicle1”.

- Select "Create"
		
### Test the scenario using the python scripts:		
1. If you haven't installed the required modules, follow the instructions in the [python README file](../python/README.md).
2. Make sure you have the `mqtt-broker` virtual environment activated by running `source ~/env/mqtt-broker/bin/activate`
3. In a terminal window, set up the following variable: `export gw_url="<namespace name>.centraluseuap-1.ts.eventgrid.azure.net"` and run the sample script through the following command: `python ./publish.py`
4. In the portal, go to your Storage account > Queues. Select your queue and observe the incoming messages.
