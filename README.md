# The Things Network Azure IoT Hub Integration

This is an example integration between The Things Network and Azure IoT Hub. This integration will be offered as a bridge, which features creating devices in the Azure IoT Hub device registry as well as sending events from uplink messages.

*Note: in this workshop we will build a simple bridge, running on your own computer. See the [full example](https://github.com/TheThingsNetwork/examples/tree/master/integrations/azure) on how to deploy this bridge as a WebJob to Azure.*

## Prerequisites

1. Nodejs [download here](https://nodejs.org/en/). Version 4.5 should be fine
2. Azure account
3. TTN account
4. Device Explorer [download here](https://github.com/Azure/azure-iot-sdks/blob/master/tools/DeviceExplorer/doc/how_to_use_device_explorer.md)

## Create an Azure IoT Hub

Follow these steps to create an Azure IoT Hub.

1. Log into the [azure portal](https://portal.azure.com/). The portal will be shown
2. On the left a number of Azure services os shown. Select `More Services` to open a list with all services. Filter it with `Iot Hub`
3. Select `Iot Hub` and a new blade will be shown. Select `Add` and you will be asked to enter the information needed to create an IoT Hub
4. Enter a unique IoT Hub name eg. `TechDays42ih`
5. Create a new Resource Group eg. `TechDays42rg`
6. Select `West Europe` for the location
7. Press `Create` and the portal will start creating the service. Once it is created, a notification is shown. In the right upper corner a bell represents all notifications shown

Creating an IoT Hub takes some time. Meanwhile we will create the bridge.


## Create a bridge

Follow these steps to create the integration bridge between The Things Network and Azure IoT Hub.

1. In a new folder, run `npm init` to initialize a new WebJob using Node.js. Use `server.js` as entry point
2. Run `npm install --save ttn-azure-iothub` to install this package
3. Create a new file `server.js`

This server file will be created below but we need some secrets. We will have to collect the unique keys of the TTN app and the Azure IoT Hub first.


### TTN App secrets

This integration requires the AppEUI and App Access Key from the TTN portal.

TODO - new portal????

These are the secrets needed from TTN.

### Azure IoT Hub secrets

This integration requires an shared access policy key name with Registry write and Device connect permissions. In this example, we use the **iothubowner** policy which has these permissions enabled by default.

1. Check the Azure portal notifications. The IoT Hub should be created by now.
2. On the left, select `Resource groups`
3. select the ResourceGroup `TechDays42rg`. It will open a new blade with all resources.
4. select the IoT Hub `TechDays42ih` It will open a new blade with the IoT Hub.
5. The IoTHub has not received any messages yet. Check the general settings for `Shared access policies`
6. Navigate to the 'iothubowner' policy and remember the primary key.
7. Also remember the name of the IoT Hub eg 'TechDays42ih'

These are the secrets needed from Azure.

### Create server.js

Create a file named server.js in the new folder.

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

## Start the bridge

Run `npm start` to verify that the bridge works in the new folder. This is example output:

```
TTN connected
0004A30B001B442B: Handling uplink
Uplink { devEUI: '0004A30B001B442B',
  message: '{"lux":1000,"water":19.82,"deviceId":"0004A30B001B442B","time":"2016-06-14T16:19:15.402956092Z"}' }
0004A30B001B442B: Handling uplink
Uplink { devEUI: '0004A30B001B442B',
  message: '{"lux":1010,"water":19.72,"deviceId":"0004A30B001B442B","time":"2016-06-14T16:19:37.546601639Z"}' }
...
```

Keep the bridge running till the end of the workshop.    

## Check the arrival of the telemetry

We can check the arrival of the messages in the Azure IoT Hub:

1. Go to the Azure Portal. Navigate to the 'iothubowner' policy and this time remember the primary connection string
2. Install the Device Manager. Start the device manager.
3. On the Configuration Tab, insert the IoT Hub Connection String and the name of the IoT Hub (as Protocol Gateway Hostname)
4. Press 'Update'
5. On the Management tab, your device should already be visible. It is registered by the bridge
6. On the Data tab, Select your 'Device ID' and press 'Monitor'.

The messages should be visible here too. These messages are now available in Azure.

You are now ready to process your data in an Azure Stream Analytics job.
