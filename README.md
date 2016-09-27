# The Things Network Azure IoT Hub Integration

This is an example integration between The Things Network and Azure IoT Hub. This integration will be offered as a bridge, which features creating devices in the Azure IoT Hub device registry as well as sending events from uplink messages.

### Prerequisites

1. A running TTN node connected to the TTN network
2. Azure account [create here](https://azure.microsoft.com/en-us/free/) _(Azure passes will be present for those who have no Azure account)_
3. The Things Network account [create here](https://staging.account.thethingsnetwork.org/) _(The Things Network accounts are available for every participant)_
4. Device Explorer (https://github.com/Azure/azure-iot-sdks/blob/master/tools/DeviceExplorer/doc/how_to_use_device_explorer.md)


## Connect your device

Follow the workshop facilitator connect the sensors. A few important things:

- The passive infrared sensor (PIR) should be connected to the `5v` and digital pin `2`
- The water sensor should be connected to the `3v3` and analog pin `0` (`A0`)

Your device and sensors should be connected as follows:

   ![alt tag](img/device-overview.JPG)

   ![alt tag](img/device-vcc-gnd.JPG)


## Read sensors

Open the Arduino IDE and follow these steps.

1. Connect The Things Uno to your computer
2. In the **Tools** menu, click **Board** and select **Arduino Leonardo**
3. In the **Tools** menu, click **Port** and select the serial port of your Arduino Leonardo
4. Paste the following code in a new sketch:
```c
// Define the pins of your sensors
#define PIN_PIR 2
#define PIN_WATER A0

// Setup runs once
void setup() {
  pinMode(PIN_PIR, INPUT);
}

// Loops runs indefinitely
void loop() {
  // Read the sensors
  uint8_t motion = digitalRead(PIN_PIR);
  uint16_t water = analogRead(PIN_WATER);

  // Only print the water value when there is motion
  if (motion == HIGH) {
    Serial.print("Water: ");
    Serial.println(water);
  }

  // Wait one second
  delay(1000);
}
```
5. In the **Sketch** menu, click **Upload**
6. Once the sketch has been uploaded, go to the **Tools** menu and open the **Serial Monitor**
7. You should see output like this, only new lines when there is motion (your PIR sensor lights up red):
```
Water: 572
Water: 573 
...
```

## Create The Things Network application

Follow the steps to create an application and register your device.

1. Log into the [The Things Network dashboard](https://preview.dashboard.thethingsnetwork.org). You will be asked to provide TTN credentials if needed
2. Add a new application. Pick a unique Application ID
    ![alt tag](img/ttn-application.png)
3. Go to **Manage devices** and click **Register device**
4. Enter a **Device ID** and click **Randomize** to use a random Device EUI
5. Click **Settings**
6. Check **Disable frame counter checks**
7. Click **Personalize device** and confirm by clicking **Personalize**
    ![alt tag](img/ttn-device.png)


## Create an Azure IoT Hub

Follow these steps to create an Azure IoT Hub.

1. Log into the [Azure portal](https://portal.azure.com/). You will be asked to provide Azure credentials if needed
2. On the left, a number of common Azure services is shown. Select `More Services` to open a list with all available services

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

Creating an IoT Hub takes some time. Meanwhile we will connect the device and create the bridge.


## Create a bridge

Follow these steps to create the integration bridge between The Things Network and Azure IoT Hub. NPM will be used to create a folder structure and imstall packages.

1. Create a new folder eg. `c:\techdays42`
2. In Command Prompt, navigate to the new folder 
3. In this new folder, run `npm init` to initialize a new Node.js application. Some values will be presented to be changed; accept the initial values, only use `server.js` (instead of _index.js_ as entry point, if proposed)
   
   ![alt tag](img/npm-init.png)
   
4. Accept the changes to be written in a json file with yes (default option)
5. Run `npm install --save ttn-azure-iothub@preview` to install this package
6. Create a new file named `server.js` in the folder you created

This server.js file will be edited below but we need some secrets first. We have to collect unique keys of the TTN app and the Azure IoT Hub first.


### TTN Application

The integration requires an application and device configured in The Things Network.

1. Go to your application by clicking its name in the navigation bar
2. Scroll down to **Access Keys**. **Write down** the access key
    ![alt tag](img/ttn-application-cred.png)

The `Application ID` and `Access Key` are required to get data from The Things Network.


### Azure IoT Hub secrets

The integration requires an Azure IoT Hub Shared access policy key name with `Registry read, write and Device connect` permissions. In this example, we use the **iothubowner** policy which has these permissions enabled by default.

1. Check the Azure portal. The resource group and the IoT Hub should be created by now

    ![alt tag](img/azure-notifications.png)

2. On the left, select `Resource groups`. A list of resource groups is shown

    ![alt tag](img/azure-resource-groups.png)

3. Select the resource group `TechDays42rg`. It will open a new blade with all resources in this group
4. Select the IoT Hub `TechDays42ih`. It will open a new blade with the IoT Hub
5. The IoTHub has not received any messages yet. Check the general settings for `Shared access policies`

    ![alt tag](img/azure-iot-hub-initial.png)

6. **Write down** the `name` of the IoT Hub eg. `TechDays42ih`
7. Navigate to the `iothubowner` policy and **write down** the primary key
8. In the last step op this tutorial, the 'Connection String-Primary Key' is needed. **Write down** this `Connection String-Primary Key`

    ![alt tag](img/azure-iothubowner-policy.png)

These are the secrets needed from Azure.


### Edit server.js

Edit the file named server.js in the new folder.

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
goat: Handling uplink
Uplink { devEUI: 'goat',
  message: '{"water":19,"deviceId":"goat","time":"2016-06-14T16:19:15.402956092Z"}' }
goat: Handling uplink
Uplink { devEUI: 'goat',
  message: '{"water":19,"deviceId":"goat","time":"2016-06-14T16:19:37.546601639Z"}' }
...
```

*Note: the message consists of valid Json telemetry.*

Keep the bridge running till the end of the workshop.  


## Monitoring the arrival of the telemetry in Azure

We can check the arrival of the messages in the Azure IoT Hub. This can be done using a UI app named Device Explorer or using a Command-Line tool named Hub Explorer.

### Monitoring using UI

We can check the arrival of the messages in the Azure IoT Hub.

1. Start the `Device Explorer`
2. On the Configuration Tab, insert the IoT Hub `Connection String-primary key` and the `name` of the IoT Hub (as Protocol Gateway Hostname)
3. Press `Update`
4. On the Management tab, your device should already be available. It was registered by the bridge when the very first telemetry arrived
5. On the Data tab, Select your `Device ID` and press `Monitor`

```
Receiving events...
09/23/16 21:43:47> Device: [goat], Data:[{"water":10}]
09/23/16 21:43:51> Device: [goat], Data:[{"water":15}]
09/23/16 21:43:53> Device: [goat], Data:[{"water":14}]
```

### Monitoring using Command-line

Via https://github.com/Azure/azure-iot-sdks/tree/master/tools/iothub-explorer

## Conclusion

The messages are shown here too. These messages are now available in Azure.

You are now ready to process your data in an Azure Stream Analytics job. Continue to [Handling The Things Network telemetry in Azure](Azure.md)

![Workshop provided by Microsoft, The Things Network and Atos](img/logos/microsoft-ttn-atos.png)

        
        
    
