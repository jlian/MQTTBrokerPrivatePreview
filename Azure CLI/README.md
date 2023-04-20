### Get started with Azure CLI:

This quickstart requires Azure CLI version 2.17.1 or later. Run the following command to find the cuurent installed version
    
    az --version
    
To install or upgrade, see [Install Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
    

- Set Azure cloud
    ```bash
    az cloud set --name AzureCloud
    ```
- Login to Azure
    ```bash
    az login
    ```

- Set your subscription
    ```bash
    az account set -s <Subscription ID>
    ```
### Event Grid Topic
To route your messages from your clients to different Azure services or your custom endpoint, an Event Grid topic needs to be created and referenced during namespace creation to forward the messages to that topic for routing; this reference cannot be added/updated afterwards. Use the following commands to create the topic and set the right permissions to take advantage of the routing functionality.

~~~
# Create the topic in the East US 2 EUAP Region and set the input schema to CloudEvent Schema v1.0
az eventgrid topic create -g <resource group> --name <topic name> -l centraluseuap --input-schema cloudeventschemav1_0
# Register the Event Grid resource provider
az provider register --namespace Microsoft.EventGrid
# Set EventGrid Data Sender role to your user ID
az role assignment create --assignee "<alias>@contoso.com" --role "EventGrid Data Sender" --scope "/subscriptions/<subscription ID>/resourcegroups/<resource group>/providers/Microsoft.EventGrid/topics/<event grid topic name>"
~~~

### Namespace

#### ARM Contract
~~~
{
    "properties": {
        "inputSchema": "CloudEventSchemaV1_0",
        "topicSpacesConfiguration": {
            "state": "Enabled",
            
            "clientAuthentication": {
                "alternativeAuthenticationNameSources": [
                    "ClientCertificateSubject",
                    "ClientCertificateIp",
                    "ClientCertificateDns"
                ]   // Optional. Required only if client CONNECT packet doesn't have client authentication name in username. 
                    //Allowed Values: ClientCertificateSubject, ClientCertificateDns, ClientCertificateUri, ClientCertificateIp,ClientCertificateEmail
            }
            
            "routeTopicResourceId": "/subscriptions/<Subscription ID>/resourceGroups/<Resource Group>/providers/Microsoft.EventGrid/topics/<Event Grid Topic name>",
            "routingEnrichments": {
                "static": [
                    {
                        "key": "statickey",
                        "value": "staticvalue",
                        "valueType": "string"
                    }
                ],
                "dynamic": [
                    {
                        "key": "clientname",
                        "value": "${client.name}"
                    },
                    {
                        "key": "userproperty1",
                        "value": "${mqtt.message.userProperties.userProperty1}"
                    }
                ]
            }
        }
    },
    "location": "centraluseuap",
    "tags": {
        "tag1": "value1",
        "tag2": "value2",
        "tag3": "value3"
    }
}
~~~


#### Commands
| Action           | Azure CLI |
| ---------------- | --------- |
| Create namespace |  az resource create --resource-type Microsoft.EventGrid/namespaces --id /subscriptions/\<Subscription ID>/resourceGroups/\<Resource Group>/providers/Microsoft.EventGrid/namespaces/\<Namespace Name> --is-full-object --api-version 2023-06-01-preview --properties @./resources/NS.json | 
Get namespace | az resource show --id /subscriptions/\<Subscription ID>/resourceGroups/\<Resource Group>/providers/Microsoft.EventGrid/namespaces/\<Namespace Name> |
Delete Namespace | az resource delete --id /subscriptions/\<Subscription ID>/resourceGroups/\<Resource Group>/providers/Microsoft.EventGrid/namespaces/\<Namespace Name> |


### CA Certificate

#### ARM Contract
~~~
{
    "properties": {
        "description": "This is a test certificate",
        "encodedCertificate": "-----BEGIN CERTIFICATE-----
{{ Base64 encoded Certificate}}
-----END CERTIFICATE-----"
    }
}
~~~
#### Commands
| Action           | Azure CLI |
| ---------------- | --------- |
| Create caCertificate |  az resource create --resource-type Microsoft.EventGrid/namespaces/caCertificates --id /subscriptions/\<Subscription ID>/resourceGroups/\<Resource Group>/providers/Microsoft.EventGrid/namespaces/\<Namespace Name>/caCertificates/\<Ca Certificate Name> --is-full-object --api-version 2023-06-01-preview --properties @./resources/CAC_test-ca-cert.json | 
Get caCertificate | az resource show --id /subscriptions/\<Subscription ID>/resourceGroups/\<Resource Group>/providers/Microsoft.EventGrid/namespaces/\<Namespace Name>/caCertificates/\<Ca Certificate Name> |
Delete caCertificate | az resource delete --id /subscriptions/\<Subscription ID>/resourceGroups/\<Resource Group>/providers/Microsoft.EventGrid/namespaces/\<Namespace Name>/caCertificates/\<Ca Certificate Name> |



### Clients

#### ARM Contract
> Note: Only one authentication property type (from CertificateThumbprint or CertificateSubject) must be provided in the Create/Update Payload for Client.

##### For CA Certificate Subject based authentication
~~~
{
    "properties": {

        "state": "Enabled",

        "authenticationName": "@abc_123:y", //New client identifier apart from client name.  Defaulted to client name value, but can be changed.

        "clientCertificateAuthentication": {
            
            "validationScheme": "SubjectMatchesAuthenticationName", 
                // Required, Possible Values: [ThumbprintMatch, SubjectMatchesAuthenticationName, DnsMatchesAuthenticationName, UriMatchesAuthenticationName, IpMatchesAuthenticationName, EmailMatchesAuthenticationName]
            
            "allowedThumbprints": [
                "8E7968B9C434EAA8139BE58A21F2E7FB15D94C344B3B2ED8F8BC02E4C5FEB7E7", //Primary thumbprint required if validationScheme is ThumbprintMatch.  Sample thumbprint shown.
                "6C93323180FC326618437231435DFD4CEB9C2192" //Secondary thumbprint is optional.  Sample thumbprint shown.
            ] // Only relevant if ThumbprintMatch is selected for validationScheme

        }, 
    },

    "attributes": {
        "attribute1": "345",
        "arrayAttribute": [
            "value1",
            "value2",
            "value"
        ]
    },
    "description": "This is a test client"
    }
}
~~~


#### Commands
| Action           | Azure CLI |
| ---------------- | --------- |
| Create Client | az resource create --resource-type Microsoft.EventGrid/namespaces/clients --id /subscriptions/\<Subscription ID>/resourceGroups/\<Resource Group>/providers/Microsoft.EventGrid/namespaces/\<Namespace Name>/clients/\<Client Name> --is-full-object --api-version 2023-06-01-preview --properties @./resources/client.json | 
Get Client | az resource show --id /subscriptions/\<Subscription ID>/resourceGroups/\<Resource Group>/providers/Microsoft.EventGrid/namespaces/\<Namespace Name>/clients/\<Client Name> |
Delete Client | az resource delete --id /subscriptions/\<Subscription ID>/resourceGroups/\<Resource Group>/providers/Microsoft.EventGrid/namespaces/\<Namespace Name>/clients/\<Client Name> |


### Client Groups

#### ARM Contract
~~~
{
    "properties": {
        "description": "This is a test client group",
        "query": "attributes.b IN ['a', 'b', 'c']"
    }
}
~~~

#### Commands
| Action           |Azure CLI |
| ---------------- | --------- |
| Create clientGroup |  az resource create --resource-type Microsoft.EventGrid/namespaces/clientGroups --id /subscriptions/\<Subscription ID>/resourceGroups/\<Resource Group>/providers/Microsoft.EventGrid/namespaces/\<Namespace Name>/clientGroups/\<Client Group Name> --is-full-object --api-version 2023-06-01-preview --properties @./resources/CG.json | 
Get clientGroup | az resource show --id /subscriptions/\<Subscription ID>/resourceGroups/\<Resource Group>/providers/Microsoft.EventGrid/namespaces/\<Namespace Name>/clientGroups/\<Client Group Name> |
Delete clientGroup | az resource delete --id /subscriptions/\<Subscription ID>/resourceGroups/\<Resource Group>/providers/Microsoft.EventGrid/namespaces/\<Namespace Name>/clientGroups/\<Client Group Name>|


### Topic Spaces

#### ARM Contract
~~~
{ 
    "properties": {
        "subscriptionSupport": "LowFanout",
        "topicTemplates": [
            "segment1/+/segment3/${client.name}",
            "segment1/${client.attributes.attribute1}/segment3/#"
        ]
    }
}
~~~
#### Commands
| Action           | Azure CLI |
| ---------------- | --------- |
| Create topicSpace |  az resource create --resource-type Microsoft.EventGrid/namespaces/topicSpaces --id /subscriptions/\<Subscription ID>/resourceGroups/\<Resource Group>/providers/Microsoft.EventGrid/namespaces/\<Namespace Name>/topicSpaces/\<Topic Space Name> --is-full-object --api-version 2023-06-01-preview --properties @./resources/TS.json | 
Get topicSpace | az resource show --id /subscriptions/\<Subscription ID>/resourceGroups/\<Resource Group>/providers/Microsoft.EventGrid/namespaces/\<Namespace Name>/topicSpaces/\<Topic Space Name> |
Delete topicSpace | az resource delete --id /subscriptions/\<Subscription ID>/resourceGroups/\<Resource Group>/providers/Microsoft.EventGrid/namespaces/\<Namespace Name>/topicSpaces/\<Topic Space Name> |



### Permission Binding

#### ARM Contract
~~~
{
    "properties": {
        "clientGroupName": "clientGroup1",
        "permission": "Publisher", //or Subscriber
        "topicSpaceName": "topicSpace1"
    }
}
~~~
#### Commands
| Action           | Azure CLI |
| ---------------- | --------- |
| Create permissionBinding |  az resource create --resource-type Microsoft.EventGrid/namespaces/permissionBindings --id /subscriptions/\<Subscription ID>/resourceGroups/\<Resource Group>/providers/Microsoft.EventGrid/namespaces/\<Namespace Name>/permissionBindings/\<Permission Binding Name> --api-version 2023-06-01-preview --properties @./resources/PB.json | 
Get permissionBinding | az resource show --id /subscriptions/\<Subscription ID>/resourceGroups/\<Resource Group>/providers/Microsoft.EventGrid/namespaces/\<Namespace Name>/permissionBindings/\<Permission Binding Name> |
Delete permissionBinding | az resource delete --id /subscriptions/\<Subscription ID>/resourceGroups/\<Resource Group>/providers/Microsoft.EventGrid/namespaces/\<Namespace Name>/permissionBindings/\<Permission Binding Name> |




