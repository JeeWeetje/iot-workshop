# The Things Network & Azure IoT in unison
## The Things Network Azure IoT Hub Integration Bridge

![alt tag](img/msft/Picture01-overview.png)

This is an example integration between The Things Network and Azure IoT Hub. This integration will be offered as a bridge, which features creating devices in the Azure IoT Hub device registry as well as sending events from uplink messages.

*Note: in this workshop, we will create uniquely named Azure resources. The suggested names could be reserved already.*

### Prerequisites

1. A computer with internet access
2. A The Things Uno, a Grove - Water Sensor, a Grove - PIR Motion Sensor, wiring & a micro USB cable
3. Arduino IDE [http://arduino.cc](http://arduino.cc)
4. Node.js [https://nodejs.org/en/](https://nodejs.org/en/). _(We prefer Version 6.6)_
5. Azure account [create here](https://azure.microsoft.com/en-us/free/) _([Azure passes](https://www.microsoftazurepass.com/howto) will be present for those who have no Azure account)_
6. TTN account [https://account.thethingsnetwork.org/](https://account.thethingsnetwork.org/)
7. [IoT Hub Explorer](https://github.com/Azure/azure-iot-sdks/tree/master/tools/iothub-explorer) _(for Command-Line interface  based usage)_ or [Device Explorer](https://github.com/Azure/azure-iot-sdks/blob/master/tools/DeviceExplorer/) _(for GUI based usage)_  

## Connect your device

![alt tag](img/msft/Picture02-build-the-hardware.png)

Follow the workshop facilitator connecting the sensors. A few important things:

- The passive infrared sensor (PIR) VCC pin is connected to the `5v` pin on the The Things Uno
- The passive infrared sensor (PIR) SIG pin is connected to the digital pin `2` (third pin in the pin header) on the The Things Uno
- The passive infrared sensor (PIR) GND pin is connected to one of the `GND` pins on the The Things Uno
- The water sensor VCC pin is connected to the `3v3` pin on the The Things Uno 
- The water sensor SIG pin is connected to the analog pin `A0` on the The Things Network Uno
- The water sensor GND pin is connected to one of the `GND` pins on the The Things Uno

*Note: On the pictures, there is a white wire from the water sensor; the white wire is not connected by intention*

Your device and sensors should be connected as follows:

   ![alt tag](img/ttn-device-overview.JPG)

   ![alt tag](img/ttn-device-vcc-gnd.JPG)

## Read sensors

![alt tag](img/msft/Picture03-read-sensor-data.png)

Open the Arduino IDE and follow these steps.

1. Connect The Things Uno via the micro USB cable to your computer
2. In the **Tools** menu, click **Board** and select **Arduino Leonardo**
3. In the **Tools** menu, click **Port** and select the serial port of your **COMx (Arduino Leonardo)**
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
      uint16_t waterLevel = analogRead(PIN_WATER);

      // Only print the water level value when there is motion
      if (motion == HIGH) {
        Serial.print("Water level: ");
        Serial.println(waterLevel);
      }

      // Wait one second
      delay(1000);
    }
    ```

5. In the **Sketch** menu, click **Verify/Compile**
6. In the **Sketch** menu, click **Upload**
7. Once the sketch has been uploaded, go to the **Tools** menu and open the **Serial Monitor**
8. You should see output like this, only new lines when there is motion (your PIR sensor lights up red):

    ```
    Water level: 572
    Water level: 573 
    ...
    ```

## Create The Things Network application

![alt tag](img/msft/Picture04-create-a-ttn-device.png)

Follow the steps to create an application and register your device.

1. Log into the [The Things Network dashboard](https://preview.dashboard.thethingsnetwork.org). You will be asked to provide TTN credentials if needed
2. A TTN application is a logical container of several devices, providing the same telemetry. There are no TTN applications yet

    ![alt tag](img/ttn-app-list-empty.png)

3. Add a new application. Pick a unique Application ID (for example `azure-demo` in lower case) and fill in a description

    ![alt tag](img/ttn-application.png)

4. Go to **Manage devices**

    ![alt tag](img/ttn-devices-list-empty.png)

5. Click **Register device**
6. Enter a **Device ID** (for example `goattrough` in lower case) and click **Randomize** to use a random Device EUI (the 'Randomize' link will disappear)

    ![alt tag](img/ttn-device-add.png)

7. Click **Register** 
8. The device is now created

    ![alt tag](img/ttn-device-created.png)

7. Click **Settings** in the upper right corner
8. Check **Disable frame counter checks**

    ![alt tag](img/ttn-disable-frame-counter-check.png)

9. Click **Save**

9. Again Click **Settings** and click **Personalize device** and just confirm by clicking **Personalize**. The following page is shown

    ![alt tag](img/ttn-device.png)

8. Keep this page open, you need the device address, network session key and application session key in a minute

## Send data from your device

![alt tag](img/msft/Picture05-submit-data-to-ttn.png)

The sensor data is read, now it is time to send the sensor data to The Things Network. 

1. In the Arduino IDE, from the **File** menu, choose **New** to create a new sketch and paste the following code:

    ```c
    #include <TheThingsNetwork.h>

    // Set your DevAddr
    const byte devAddr[4] = { ... }; //for example: {0x02, 0xDE, 0xAE, 0x00};

    // Set your NwkSKey and AppSKey
    const byte nwkSKey[16] = { ... }; //for example: {0x2B, 0x7E, 0x15, 0x16, 0x28, 0xAE, 0xD2, 0xA6, 0xAB, 0xF7, 0x15, 0x88, 0x09, 0xCF, 0x4F, 0x3C};
    const byte appSKey[16] = { ... }; //for example: {0x2B, 0x7E, 0x15, 0x16, 0x28, 0xAE, 0xD2, 0xA6, 0xAB, 0xF7, 0x15, 0x88, 0x09, 0xCF, 0x4F, 0x3C};

    TheThingsNetwork ttn;

    // Define the pins of your sensors
    #define PIN_PIR 2
    #define PIN_WATER A0

    // Setup runs once
    void setup() {
      // The PIR is a digital input
      pinMode(PIN_PIR, INPUT);

      Serial.begin(9600);
      Serial1.begin(57600);

      ttn.init(Serial1, Serial);
      ttn.reset();

      //the device will configure the LoRa module
      ttn.personalize(devAddr, nwkSKey, appSKey);

      ttn.showStatus();
      Serial.println("Setup for The Things Network complete");
    }

    // Loop runs indefinitely
    void loop() {
      // Read sensors
      uint8_t motion = digitalRead(PIN_PIR);
      uint16_t waterLevel = analogRead(PIN_WATER);

      // Check if there is motion
      if (motion == HIGH) {
        // Print the water level value
        Serial.print("Water level: ");
        Serial.println(waterLevel);

        // Send data to The Things Network
        byte buffer[2];
        buffer[0] = highByte(waterLevel);
        buffer[1] = lowByte(waterLevel);
        ttn.sendBytes(buffer, sizeof(buffer));
      }

      // Wait 10 seconds
      delay(10000);
    }
    ```

2. Insert your device address in `devAddr`, network session key in `nwkSkey` and application session key in `appSKey`. You can use the handy `<>` button in the dashboard to copy it quickly as a C-style byte array; exactly what Arduino wants
3. In the **Sketch** menu, click **Upload**
4. Open the **Serial Monitor** again from the **Tools** menu once upload has completed. Your device should now be sending data to The Things Network
5. In The Things Network dashboard, go to **Data**. You see packets coming in:
    
    ![alt tag](img/ttn-device-payload-binary.png)

## Decode data on TTN

![alt tag](img/msft/Picture06-decode-data-on-ttn.png)

Now, the binary payload is not really useful in upstream. Therefore, we have payload functions.

1. In the application overview, click **Payload Functions**
2. Add the following **decoder** function to decode the two bytes back to a 16-bit integer called `waterLevel`:

    ```c
    function Decoder(bytes) {
      var waterLevel = (bytes[0] << 8) | bytes[1];
      return {
        waterLevel: waterLevel
      };
    }
    ```

3. We want to invert the resistance of the water sensor so that more water resembles a higher value. The maximum value of 3v3 analog ADC converter is `682`, so use the following as the **converter**:
    
    ```
    function Converter(decodedObj) {
      decodedObj.waterLevel = 682 - decodedObj.waterLevel;
      return decodedObj;
    }
    ```

4. Go back to your data overview. Now you should see something like this:

    ![alt tag](img/ttn-device-payload-fields.png)

Now we have clean data ready to be processed in Azure IoT Hub and upstream.

## Create an Azure IoT Hub

![alt tag](img/msft/Picture07-prepare-azure-integration.png)

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

Creating an IoT Hub takes some time. Meanwhile, we will connect the device and create the bridge.

## Create a bridge

![alt tag](img/msft/Picture08-build-a-bridge-frm-ttn-to-azure.png)

Follow these steps to create the integration bridge between The Things Network and Azure IoT Hub. NPM will be used to create a folder structure and install packages.

1. Create a new folder eg. `c:\techdays42`
2. In Command Prompt, navigate to the new folder 
3. In this new folder, run `npm init` to initialize a new Node.js application. Some values will be presented to be changed; accept the initial values, only use `server.js` (instead of _index.js_ as entry point, if proposed)
   
   ![alt tag](img/npm-init.png)
   
4. Accept the changes to be written in a JSON file with yes (default option)
5. Run `npm install --save ttn-azure-iothub@preview` to install this package
6. Create a new file named `server.js` in the folder you created

This server.js file will be edited below but we need some secrets first. We have to collect unique keys of the TTN app and the Azure IoT Hub first.

### TTN Application

The integration requires an application and device configured in The Things Network.

1. Go to your application by clicking its name in the navigation bar
2. Scroll down to **Access Keys**. **Write down** the application ID and access key

    ![alt tag](img/ttn-application-cred.png)

The `Application ID` and `Access Key` are required to get data from The Things Network.

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

    ![alt tag](img/azure-iothubowner-policy.png)

8. In the last step op this Bridge tutorial, the 'Connection string-primary key' is needed. **write down** this `Connection String-Primary Key` too

These are the secrets needed from the Azure IoT Hub.

### Edit server.js

Edit the file named server.js in the new folder. `Fill in` the secrets and `save` the file.

```js
'use strict';

const ttnazureiot = require('ttn-azure-iothub');

// Insert your AppId and App Access Key
const region = 'eu';
const appId = '<insert App Id>';
const appAccessKey = '<insert App Access Key>';

// Insert with your Azure IoT Hub name and primary key
const hubName = '<insert IoT Hub name>';
const keyName = 'iothubowner';
const primaryKey = '<insert primary key>';

const bridge = new ttnazureiot.Bridge(region, appId, appAccessKey, hubName, keyName, primaryKey);

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

### Start the bridge

In the new folder, run `npm start` to verify the bridge works. This is example output:

```
TTN connected
goattrough: Handling uplink
Uplink { devEUI: 'goattrough',
  message: '{"waterLevel":19,"deviceId":"goattrough","time":"2016-06-14T16:19:15.402956092Z"}' }
goatThrough: Handling uplink
Uplink { devEUI: 'goattrough',
  message: '{"waterLevel":19,"deviceId":"goattrough","time":"2016-06-14T16:19:37.546601639Z"}' }
...
```

*Note: the message consists of valid JSON telemetry.*

*Note: Keep the bridge running until the end of the complete workshop.*  


## Monitoring the arrival of the telemetry in Azure

![alt tag](img/msft/Picture09-monitor-incoming-data.png)

We can check the arrival of messages in the Azure IoT Hub. This can be done using a UI app named Device Explorer or using a Command-Line tool named IoT Hub Explorer. `Choose one` 

### Monitoring using UI

We can check the arrival of the messages in the Azure IoT Hub using the Device Explorer. This tool is UI based, please check the installation requirements.

1. Start the `Device Explorer`
2. On the Configuration Tab, insert the IoT Hub `Connection String-primary key` and the `name` of the IoT Hub (as Protocol Gateway Hostname)
3. Press `Update`
4. On the Management tab, your device should already be available. It was registered by the bridge the very first time, telemetry arrived
5. On the Data tab, Select your `Device ID` and press `Monitor`
6. This will result in the following messages

```
Receiving events...
09/23/16 21:43:47> Device: [goattrough], Data:[{"waterLevel":10}]
09/23/16 21:43:51> Device: [goattrough], Data:[{"waterLevel":15}]
09/23/16 21:43:53> Device: [goattrough], Data:[{"waterLevel":14}]
```

### Monitoring using Command-line

We can check the arrival of the messages in the Azure IoT Hub using the IoT Hub Explorer. This tool is Command-Line based, please check the installation requirements. 

*Note : See the [full example](https://github.com/Azure/azure-iot-sdks/tree/master/tools/iothub-explorer) for more options of this tool.*

1. Create a new folder eg. `c:\iothubexplorer`
2. In a dos-box, navigate to the new folder 
3. In this folder, run the following command `npm install -g iothub-explorer@latest` in your command-line environment, to install the latest (pre-release) version of the iothub-explorer tool
4. Login to the IoT Hub Explorer by supplying your IoT Hub *remembered* `Connection String-primary key` using the command `iothub-explorer login "[your connection string]"`
5. A session with the IoT Hub will start and it will last for approx. one hour:

    ```
    Session started, expires Tue Sep 27 2016 18:35:37 GMT+0200 (W. Europe Daylight Time)
    ```

6. To monitor the device-to-cloud messages from a device, use the following command `iothub-explorer "[your connection string]" monitor-events [device name]`  and `fill in` your  *remembered* 'Connection String-primary key' and *remember* device name
7. This will result in the following messages

    ```
    Monitoring events from device goattrough
    Event received:
    {
      "waterLevel": 12
    }
    ```

## Conclusion

The messages are shown here too. These messages are now available in Azure.

Next Step: You are now ready to process your data in an Azure Function. Continue to [Handling The Things Network telemetry in Azure](Azure.md)

![Workshop provided by Microsoft, The Things Network and Atos](img/logos/microsoft-ttn-atos.png)

        
        
    
