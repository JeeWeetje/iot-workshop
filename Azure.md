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
4. Press Select `Add`. A list with available services appears

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
3. Press Select `Add`. A list with available services appears

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
13. Creating an namespace will take some time. We wait a moment so we can complete this step. Navigate back to the resource group (repeate step 1 and 2) and check the namespace creation.










## Connecting the Azure Stream Analytics input



## Create an Azure Function



