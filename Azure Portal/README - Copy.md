The following instructions describes how to use the Azure portal to configure the Event Grid Namespace with MQTT support.

# Configure Event Grid for Routing
1. Create an Event Grid topic according to [these instructions](https://learn.microsoft.com/en-us/azure/event-grid/create-custom-topic), while selecting the following values:
   - On the Basics page of Create Topic, select the Region as East US 2 EUAP
   - On the Advanced tab of Create Topic, select the Cloud Event Schema v1.0 as the topic's Event Schema.
2. Set EventGrid Data Sender role to your user ID on the created topic
   - In the portal, go to the created Event Grid topic resource. 
   - In the "Access control (IAM)" menu item, select "Add a role assignment".
   - In the "Role" tab, select "EventGrid Data Sender", then select "Next".
   - In the "Members" tab, click on "+Select members", then type your AD user name in the "Select" box that will appear (e.g. user@contoso.com).
   - Select your AD user name, then select "Review + assign"
3. Create Event Subscriptions on the created topic according to [these instructions](https://learn.microsoft.com/en-us/azure/event-grid/subscribe-through-portal)

# Get started 
1. Use only [this](https://portal.azure.com/?microsoft_azure_marketplace_ItemHideKey=PubSubNamespace&microsoft_azure_eventgrid_assettypeoptions={"PubSubNamespace":{"options":""}}) link to reach the Azure portal for the service. 
2. On the Azure homepage, search for "Event Grid Namespace" to land on the service's page.

![Namespace_Search.png](https://github.com/Azure/MQTTBrokerPrivatePreview/blob/main/Azure%20Portal/Namespace_Search.png)

# Create a Namespace
1. Once you land on the service's page, click "Create"
2. On the Basics tab, complete the fields as follows:
   -  Subscription: Select the enabled subscription for private preview.
   - Resource group: Select a resource group or create a new one. For more information, see [Manage Azure Resource Manager resource groups](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-portal).
   - Name: Enter a name for your namespace. This name must be globally unique, with a length between 3 and 50 alphanumeric characters. The name can also include the hyphen ('-') character.
   - Region: Select East US 2 EUAP
   
![Namespace_Creation-Basics.png](https://github.com/Azure/MQTTBrokerPrivatePreview/blob/main/Azure%20Portal/Namespace_Creation-Basics.png)   

3. On the Routing tab:
   - Check Enable Routing
   - Select the Event Grid topic that you created above from the drop-down list.
     - If you don't have 'EventGrid Data Sender' permission to the selected topic, click "Add permission" to assign this permission, then click "Refresh" to update the list.
     - Note that it might take a few minutes for the permission to be reflected after assignment.

![Namespace_Creation-Routing.png](https://github.com/Azure/MQTTBrokerPrivatePreview/blob/main/Azure%20Portal/Namespace_Creation-Routing.png)   

4. Select Review + create to review your choices.
5. Select Create to start the deployment of your new namespace. Your deployment will be in progress a few minutes while the namespace is being created. Once the deployment is complete, select Go to resource to open the new namespace.
6. On the Overview page, you will be able to see the hostname that your MQTT clients need to send the messages to.

# Create CA Certificates
 1. In your namespace, go to the Ca certificates menu item and click + Certificate.
 2. In the creation blade:
    - Add a Certificate Name to the Ca certificate
    - Upload your Ca Certificate .pem or .cer file
    - Click Upload.
    
![CACertificate_Creation.png](https://github.com/Azure/MQTTBrokerPrivatePreview/blob/main/Azure%20Portal/CACertificate_Creation.png)   

# Create Clients
 1. In your namespace, go to the Clients menu item and click + Client.
 2. In the creation page:
    - Add a Client Name to the client
    - Add a Client Descriptionto the client
    - Select the Authentication type
      - Select "X.509 Certificate subject" if the client uses CA-signed certificate 
      - Select "X.509 Certificate thumbprint" if the client uses self-signed certificate
    - Check "Use certificate to get the certificate values" and upload the client's cerficate or manually enter the certificate values as follows
    - Click Add attribute to add Client Attributes up to 4 KB of attributes' size.
      - Key: key of the client attribute
      - Type: type of the attribute. Choose between String, Integers, or Array of strings
      - Value: value of the client attribute
    - Click Create.

![Client_Creation.png](https://github.com/Azure/MQTTBrokerPrivatePreview/blob/main/Azure%20Portal/Client_Creation.png)   
    
# Create Client Groups
 1. In your namespace, go to the Client Groups menu item and click + Client group.
 2. You will see $all client group that includes all clients by default for convenience.
 3. In the creation blade:
    - Add a Group Name.
    - Add a Query based on the client attributes according to these guidelines.
    - Add a Descripiton to your client group.
    - Click Create.
    
![ClientGroup_Creation.png](https://github.com/Azure/MQTTBrokerPrivatePreview/blob/main/Azure%20Portal/ClientGroup_Creation.png)   
    
# Create Topic Spaces
 1. In your namespace, go to the Topic Spaces menu item and click + Topic Space.
 2. In the creation blade:
    - Add a Name to the topic space
    - Add up to 10 topic templates to represent your MQTT topics. A topic templates are a flexible topic filters with variable support. [Learn more](https://github.com/Azure/MQTTBrokerPrivatePreview/blob/main/README.md#topic-template)
    - Select the Subscription support. The subscription support is used to optimize the mode of message delivery to your clients. Note that your clients can publish to all topic spaces by default, but you need to enable subscriptions on your topic spaces below and choose the message delivery mode.
      - Not supported: this will indicate that topic space could be used only for publishing. This will be helpful in scenarios when you expect your corresponding topic templates to overlap since this is the only mode that will allow your topic templates to overlap with any other topic template.
      - Low fanout: will specify a subscription delivery mode optimized for having a maximum of 10 subscribers per topic, with very low latency. E.g. there is a group of vehicles with each vehicle subscribing to its own command delivered to its own topic.
      - High fanout: will specify a subscription delivery mode optimized for having unlimited number of subscribers per topic, with higher latency. E.g. there is a large group of vehicles in a city subscribing to the same weather alerts that get broadcasted on the same topic.
    - Click Create.

![TopicSpace_Creation.png](https://github.com/Azure/MQTTBrokerPrivatePreview/blob/main/Azure%20Portal/TopicSpace_Creation.png)   

# Create Permission Bindings
 1. In your namespace, go to the Permission bindings menu item and click +  Permission binding 
 2. In the creation blade:
    - Add a Name for your permission binding.
    - Client group name: the client group that needs access to a topic space.
    - Topic space name: the topic space that the client group needs access to.
    - Permission: Publisher or Subscriber permission that the client group needs.
    - Click Create.

![PermissionBinding_Creation.png](https://github.com/Azure/MQTTBrokerPrivatePreview/blob/main/Azure%20Portal/PermissionBinding_Creation.png)   
