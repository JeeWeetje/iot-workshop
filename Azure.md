# Handling The Things Network telemetry in Azure

This is an example of how uplink messages from The Things Network can be handled in Azure. In this workshop you will pass telemetry to Azure Stream Analytics and Azure Functions. To connect Azure Stream Analytics with Azure Functions, you will add an Azure Event Hub too.

*Note: This workshop has an open end. It provides a solid solution on how to handled telemetry programmatically in Azure. From there on it's up to you to add more Azure services.*

### Prerequisites

1. A running TTN node connected to the TTN network
2. Azure account [create here](https://azure.microsoft.com/en-us/free/) _(Azure passes will be present for those who have no Azure account)_
3. An Azure IoT Hub
4. A running TTN bridge on your PC and connected to an IoT Hub
5. A running Device Explorer, connected to the IoT Hub, showing the telemetry comming in

## Create Azure Stream Analytics Job

Follow these steps to create an Azure Stream Analytics Job which takes messages from your IoT Hub. These jobs can handle great amounts of messages and handled with a SQL-like query language. Stream Analytics Jobs are great for time window related queries.

*Note: in this workshop we will not dive too deep into Stream Analytics. See for [more information](https://azure.microsoft.com/en-us/documentation/articles/stream-analytics-real-time-event-processing-reference-architecture/).*

1. Log into the [Azure portal](https://portal.azure.com/). You will be asked to provide Azure credentials if needed
2. On the left, select `Resource groups`. A List of resource groups is shown

    ![alt tag](img/azure-resource-groups.png)

3. Select the ResourceGroup `TechDays42rg`. It will open a new blade with all resources in this group
4. Select `Add`. A list with available services appears

    ![alt tag](img/azure-portal-add.png)

5. Filter it with `Stream Analytics` and select `Stream Analytics job`

    ![alt tag](img/azure-filter-stream-analytics.png)

6. An introduction will be shown. Select `Create`

    ![alt tag](img/azure-stream-analytics-intro.png)

7. A dialog for a new Stream Analytics is shown. Enter a unique name eg. `TechDays42sa`. A green sign will be shown if the name is unique
8. The Resource Group eg. `TechDays42rg` is already filled in
9. Select `West Europe` for the location

    ![alt tag](img/azure-create-stream-analytics.png)

10. Select `Create` and the portal will start creating the service. Once it is created, a notification is shown

Creating an Azure Stream analytics job will take some time. Input is already known, the already existing IoT Hub, So let's create the service to send the output to, an azure Event Hub.

## Create an Azure Event Hub

Follow these steps to create an Azure Event Hub which can pass large amounts of (transformed) messages to other services.

1. On the left, select `Resource groups`. A List of resource groups is shown

    ![alt tag](img/azure-resource-groups.png)

2. Select the ResourceGroup `TechDays42rg`. It will open a new blade with all resources in this group
3. Select `Add`. A list with available services appears

    ![alt tag](img/azure-portal-add.png)

4. Filter it with `Stream Analytics` and select `Event Hubs`

    ![alt tag](img/azure-filter-event-hub.png)

5. An introduction will be shown. Select `Create`
6. Event Hub live within namespaces. So first a new namespace has to be created
7. A dialog for a new namespace is shown. Enter a unique name eg. `TechDays42ns`. A green sign will be shown if the name is unique
8. The Resource Group eg. `TechDays42rg` is already filled in
9. Select `West Europe` for the location

    ![alt tag](img/azure-create-eventhub-namespace.png)

10. Select `Create`
11. The creation will fail for now, the pricing tier is not entered yet. Click the pricing tier selection. A "Choose your pricing tier" section will be shown. Select the Basic tier and press `select`

    ![alt tag](img/azure-namespace-pricingtier.png)

12. Select `Create` again and the portal will start creating the namespace. Once it is created, a notification is shown
13. Creating an namespace will take some time, but we want to complete this step
14. Navigate back to the resource group (repeate step 1 and 2) and check the namespace creation
15. If the namespace is listed, select it. Otherwise, refresh the list a few times
16. You are now in the namespace blade. It should be shown like this (otherwise, refresh a few times):

    ![alt tag](img/azure-namespace.png)

17. At the top, select `Add Event Hub`

    ![alt tag](img/azure-namespace-add.png)

18. A dialog for a new namespace is shown. Enter a unique name eg. `Techdays42eh`. A green sign will be shown if the name is unique
19. Select `Create` again and the portal will start creating the namespace. Once it is created, a notification is shown

The Event Hub is now created. But before we leave this namespace, we need some secrets for later usage.

## Azure Event Hub secrets

Below we will access the Event Hub from Azure Functions. At this moment the Azure functions are not able to autmatically connect to an Event Hub.

1. Within the namespace blade, select the general setting `Share access policies`
2. select the `RootManageSharedAccessKey` policy

    ![alt tag](img/azure-eventhub-policy.png)

3. **Remember** the `Connection String-Primary Key`
4. **Remember** the `name` of the Event Hub eg. `TechDays42eh`

*Note: The Event Hub itself has Shared access policies too. We do not need to remember those, just the policy of the namespace.*

### Connecting the Azure Stream Analytics input

As shown above, the Azure Stream Analytics will connect the IoT Hub and the Event Hub. Both are created now. Follow these steps to define the input, the output and the query of Azure Stream Analytics.

1. On the left, select `Resource groups`. A List of resource groups is shown

    ![alt tag](img/azure-resource-groups.png)

2. Select the ResourceGroup `TechDays42rg`. It will open a new blade with all resources in this group
3. Select the Azure Stream Analytics `TechDays42sa`. At this moment there are no Inputs or Outputs.

    ![alt tag](img/azure-stream-analytics-empty.png)

4. Select `Inputs`
5. Select `Add`. A dialog to add a new input is shown

    ![alt tag](img/azure-portal-add.png)

6. Enter `hubinput` as Input alias
7. Select `IoT Hub` as Source. Because we have only one IoT Hub in our account, all other fields are automatically filled in with the right IoT Hub, `TechDays42rg`

    ![alt tag](img/azure-stream-analytics-add-input.png)

8. Select `Create`
9. The input will be created and the connection to the hub is tested automatically. 
10. Select `Outputs`
11. Select `Add`. A dialog to add a new output is shown

    ![alt tag](img/azure-portal-add.png)

12. Enter `huboutput` as Output alias
13. The `Event Hub` is alreadt selected as Source and all other fields are automatically filled in with the right Event Hub, `TechDays42eh`

    ![alt tag](img/azure-stream-analytics-add-output.png)

14. Select `Create`
15. The Output will be created and the connection to the hub is tested automatically. 

## Create an Azure Function



