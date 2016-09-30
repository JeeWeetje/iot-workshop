# The Things Network Azure IoT Hub Integration Bridge

This is an example integration between The Things Network and Azure IoT Hub. This integration will be offered as a bridge, which features creating devices in the Azure IoT Hub device registry as well as sending events from uplink messages.

<<<<<<< HEAD
*Note: in this workshop we will build a simple bridge, running on your own computer. See the [full example](https://github.com/TheThingsNetwork/examples/tree/master/integrations/azure) on how to deploy this bridge as a WebJob to Azure.*

*Note: in this workshop we will create uniquely named Azure resources. The suggested names could be reserved already.*

### Prerequisites

1. A running TTN node connected to the TTN network
2. NodeJs (https://nodejs.org/en/). _(We prefer Version 6.6)_
3. Azure account [create here](https://azure.microsoft.com/en-us/free/) _(Azure passes will be present for those who have no Azure account)_
4. TTN account (https://account.thethingsnetwork.org/)
5. Device Explorer _(for UI based usage)_ (https://github.com/Azure/azure-iot-sdks/blob/master/tools/DeviceExplorer/)
6. IoT Hub Explorer _(for Command-Line based usage)_ (https://github.com/Azure/azure-iot-sdks/tree/master/tools/iothub-explorer)
=======
### Prerequisites

1. A running TTN node connected to the TTN network
2. Azure account [create here](https://azure.microsoft.com/en-us/free/) _(Azure passes will be present for those who have no Azure account)_
3. The Things Network account [create here](https://staging.account.thethingsnetwork.org/) _(The Things Network accounts are available for every participant)_
4. Device Explorer (https://github.com/Azure/azure-iot-sdks/blob/master/tools/DeviceExplorer/doc/how_to_use_device_explorer.md)
>>>>>>> refs/remotes/origin/fix/text


## Create an Azure IoT Hub

Follow these steps to create an Azure IoT Hub.

1. Log into the [Azure portal](https://portal.azure.com/). You will be asked to provide Azure credentials if needed
2. On the left, a number of common Azure services are shown. Select `More Services` to open a list with all available services

    ![alt tag](img/azure-more-services.png)

3. Filter it with `IoT Hub`

    ![alt tag](img/azure-search-iot-hub.png)

4. Select `IoT Hub` and a new blade will be shown. Select `Add` and you will be asked to enter the information needed to create an IoT Hub

    ![alt tag](img/azure-portal-add.png)

5. Enter a unique IoT Hub name eg. `TechDays42ih`. A green sign will be shown if the name is unique
6. Enter a unique Resource Group eg. `TechDays42rg`. A green sign will be shown if the name is unique
7. Select `West Europe` for the location

    ![alt tag](img/azure-new-iot-hub-scaled.png)

8. Press `Create` and the portal will start creating the service. Once it is created, a notification is shown. In the right upper corner, a bell represents the list of all notifications shown

    ![alt tag](img/azure-notifications.png)

<<<<<<< HEAD
Creating an IoT Hub takes some time. Meanwhile, we will create the bridge.
=======
Creating an IoT Hub takes some time. Meanwhile we will connect the device and create the bridge.
>>>>>>> refs/remotes/origin/fix/text


## Create a bridge

Follow these steps to create the integration bridge between The Things Network and Azure IoT Hub. NPM will be used to create a folder structure and install packages.

1. Create a new folder eg. `c:\techdays42`
2. In Command Prompt, navigate to the new folder 
3. In this new folder, run `npm init` to initialize a new Node.js application. Some values will be presented to be changed; accept the initial values, only use `server.js` (instead of _index.js_ as entry point, if proposed)
   
   ![alt tag](img/npm-init.png)
   
4. Accept the changes to be written in a json file with yes (default option)
<<<<<<< HEAD
5. Run `npm install --save ttn-azure-iothub@1.0.0-3` to install this package
=======
5. Run `npm install --save ttn-azure-iothub@preview` to install this package
>>>>>>> refs/remotes/origin/fix/text
6. Create a new file named `server.js` in the folder you created

This server.js file will be edited below but we need some secrets first. We have to collect unique keys of the TTN app and the Azure IoT Hub first.


<<<<<<< HEAD
### Collect TTN App secrets
=======
### TTN Application
>>>>>>> refs/remotes/origin/fix/text

The integration requires an application and device configured in The Things Network.

1. Log into the [The Things Network dashboard](https://preview.dashboard.thethingsnetwork.org). You will be asked to provide TTN credentials if needed
2. Add a new application. Pick a unique Application ID

    ![alt tag](img/ttn-application.png)

3. Go to **Manage devices** and click **Register device**
4. Enter a **Device ID** and click **Randomize** to use a random Device EUI
5. Click **Settings**
6. Check **Disable frame counter checks**
7. Click **Personalize device** and confirm by clicking **Personalize**

    ![alt tag](img/ttn-device.png)

<<<<<<< HEAD
5. the `App EUI` and the `Access Keys` are shown. **Write down** both the App EUI and access keys

    ![alt tag](img/ttn-application-cred.png)

These are the secrets needed from the TTN app.
=======
8. Go back to your application by clicking its name in the navigation bar
9. Scroll down to **Access Keys**. **Write down** the access key

    ![alt tag](img/ttn-application-cred.png)

The `Application ID` and `Access Key` are required to get data from The Things Network.
>>>>>>> refs/remotes/origin/fix/text


### Collect Azure IoT Hub secrets

The integration requires an Azure IoT Hub Shared access policy key name with `Registry read, write and Device connect` permissions. In this example, we use the **iothubowner** policy which has these permissions enabled by default.

1. Check the Azure portal. The resource group and the IoT Hub should be created by now

    ![alt tag](img/azure-notifications.png)

2. On the left, select `Resource groups`. A list of resource groups is shown

    ![alt tag](img/azure-resource-groups.png)

3. Select the resource group `TechDays42rg`. It will open a new blade with all resources in this group
4. Select the IoT Hub `TechDays42ih`. It will open a new blade with the IoT Hub

    ![alt tag](img/azure-iot-hub-initial.png)

5. The IoTHub has not received any messages yet. Check the general settings for `Shared access policies`

    ![alt tag](img/azure-iot-hub-share-access-policy.png)

6. **Write down** the `name` of the IoT Hub eg. `TechDays42ih`
7. Navigate to the `iothubowner` policy and **write down** the primary key
<<<<<<< HEAD
=======
8. In the last step op this tutorial, the 'Connection String-Primary Key' is needed. **Write down** this `Connection String-Primary Key`
>>>>>>> refs/remotes/origin/fix/text

    ![alt tag](img/azure-iothubowner-policy.png)

8. In the last step op this Bridge tutorial, the 'Connection string-primary key' is needed. **write down** this `Connection String-Primary Key` too

These are the secrets needed from the Azure IoT Hub.


### Edit server.js

Edit the file named server.js in the new folder. `Fill in` the secrets and `save` the file.

```js
'use strict';

const ttnazureiot = require('ttn-azure-iothub');

// Replace with your AppEUI and App Access Key
const appEUI = '<insert AppEUI>';
const appAccessKey = '<insert App Access Key>';

// Replace with your Azure IoT Hub name and key
const hubName = '<insert hub name>';
const keyName = 'iothubowner';
const key = '<insert key>';

const bridge = new ttnazureiot.Bridge(appEUI, appAccessKey, hubName, keyName, key);

bridge.on('ttn-connect', () => {
  console.log('TTN connected');
});

bridge.on('error', err => {
  console.warn('Error', err);
});

bridge.on('uplink', data => {
  console.log('Uplink', data);
});
```

This is the most basic example of a bridge between TTN and Azure. 

## Start the bridge

Run `npm start` to verify that the bridge works in the new folder. This is example output:

```
TTN connected
goat: Handling uplink
Uplink { devEUI: 'goat',
  message: '{"water":19,"deviceId":"goat","time":"2016-06-14T16:19:15.402956092Z"}' }
goat: Handling uplink
Uplink { devEUI: 'goat',
  message: '{"water":19,"deviceId":"goat","time":"2016-06-14T16:19:37.546601639Z"}' }
...
```

*Note: the message consists of valid Json telemetry.*

*Note: Keep the bridge running untill the end of the complete workshop.*  


## Monitoring the arrival of the telemetry in Azure

We can check the arrival of messages in the Azure IoT Hub. This can be done using a UI app named Device Explorer or using a Command-Line tool named IoT Hub Explorer. `Choose one` 

### Monitoring using UI

We can check the arrival of the messages in the Azure IoT Hub using the Device Explorer. This tool is UI based, please check the installation requirements.

1. Start the `Device Explorer`
2. On the Configuration Tab, insert the IoT Hub `Connection String-primary key` and the `name` of the IoT Hub (as Protocol Gateway Hostname)
3. Press `Update`
4. On the Management tab, your device should already be available. It was registered by the bridge the very first time, telemetry arrived
5. On the Data tab, Select your `Device ID` and press `Monitor`

```
Receiving events...
09/23/16 21:43:47> Device: [goat], Data:[{"water":10}]
09/23/16 21:43:51> Device: [goat], Data:[{"water":15}]
09/23/16 21:43:53> Device: [goat], Data:[{"water":14}]
```

### Monitoring using Command-line

We can check the arrival of the messages in the Azure IoT Hub using the IoT Hub Explorer. This tool is Command-Line based, please check the installation requirements. 

*Note : See the [full example](https://github.com/Azure/azure-iot-sdks/tree/master/tools/iothub-explorer) for more options of this tool.*

1. Create a new folder eg. `c:\iothubexplorer`
2. In a dos-box, navigate to the new folder 
1. In this folder, to install the latest (pre-release) version of the iothub-explorer tool, run the following command `npm install -g iothub-explorer@latest` in your command-line environment
2. Login to the IoT Hub Explorer by supplying your IoT Hub `Connection String-primary key` using the command `iothub-explorer login "your connection string"`
3. A session with the IoT Hub will start and it will last for approx. one jour:

```
Session started, expires Tue Sep 27 2016 18:35:37 GMT+0200 (W. Europe Daylight Time)
```

4. To monitor the device-to-cloud messages from a device, use the following command and fill in your `Connection String-primary key` and device name in `iothub-explorer "your connection string" monitor-events goat`

```
Monitoring events from device goat
Event received:
{
  "water": 12
}
```

## Conclusion

The messages are shown here too. These messages are now available in Azure.

You are now ready to process your data in an Azure Stream Analytics job. Continue to [Handling The Things Network telemetry in Azure](Azure.md)

![Workshop provided by Microsoft, The Things Network and Atos](img/logos/microsoft-ttn-atos.png)

        
        
    
