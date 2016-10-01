# The Things Network & Azure IoT in unison
## Deploying The Things Network Bridge to Azure

Remember creating the TTN bridge locally on your computer? Will it be always on? Most likely not, therefore it seems reasonable to deploy the TTN bridge in the Azure cloud.

![Build a bridge from TTN to Azure](img/msft/Picture08-build-a-bridge-frm-ttn-to-azure.png)

### Prerequisits

1. A TTN bridge deployed/running on your computer _(like the one created earlier in the workshop)_

### Make the bridge ready for transport

Follow these steps to make the bridge ready to be deployed to Azure.

1. If the bridge is still running, stop the running bridge with `ctrl c` and confirm the cancelation of the process with `Y`.
    
    ![Canceling the running bridge](img/bridge-cancelation.png)

2. Open a File Explorer and browse to the directory `c:\techdays42`.
3. Compress all files (including the `node_modules` folder) in `c:\techdays42` as ZIP file
    
    ![Zipping all files](img/zipping-all-files.png)


### Deploy Azure WebJob

Follow these steps to deploy an Azure WebJob using Node.js that runs the integration between The Things Network and Azure IoT Hub.

1. `Log into` the [Azure portal](https://portal.azure.com/). You will be asked to provide Azure credentials if needed
2. On the left, select `Resource groups`. A list of resource groups is shown

    ![alt tag](img/azure-resource-groups.png)

3. Select the ResourceGroup `TechDays42rg`. It will open a new blade with all resources in this group

4. Select `Add`. A list of available services appears

    ![alt tag](img/azure-portal-add.png)

5. Filter it with `web app` and select `Web App`

    ![alt tag](img/azure-filter-web-app.png)

6. An introduction will be shown. Select `Create`
7. A dialog for the new Web App is shown
8. Enter a unique Web App name eg. `TechDays42wa`. A green sign will be shown if the name is unique
9. The Resource Group eg. TechDays42rg is already filled in
10. The App Service plan blade eg. TechDays42asp is already filled in 

    ![alt tag](img/azure-web-app-create.png)

11. Select `Create`

12. Creating a Web App will take some time, but we want to complete this step
13. So navigate back to the resource group (repeat step 1 and 2) and check the Web app creation in the resource group
14. If the Web App becomes listed, select `TechDays42wa`. Otherwise, 'refresh' the list a few times

    ![alt tag](img/azure-portal-refresh.png)

15. You are now in the Web App blade. It should be shown like this, with all information available (otherwise, refresh a few times):

    ![alt tag](img/azure-web-app-blade.png)

16. A Web App has dozens of settings. Filter the settings for `webjobs`

    ![alt tag](img/azure-web-app-filter-webjobs.png)

17. Select `WebJobs`. An empty list is presented
18. Select `Add`

    ![alt tag](img/azure-portal-add.png)

19. Enter a unique Web App name eg. `TtnBridgeWebJob`. A green sign will be shown if the name is unique
20. Select 'your ZIP file` as file to upload
21. Ensure that the type is set to `Continuous`
22. Set the scale to `Single Instance`

    ![alt tag](img/azure-web-job-add.png)

23. Select `Ok`


11. Once deployed, select the WebJob and click **Logs** to verify that the bridge works. This is example output:
```
[06/14/2016 16:27:47 > 996af8: INFO] TTN connected
[06/14/2016 16:28:07 > 996af8: INFO] 0004A30B001B442B: Handling uplink
[06/14/2016 16:28:10 > 996af8: INFO] Uplink { devEUI: '0004A30B001B442B',
[06/14/2016 16:28:10 > 996af8: INFO]   message: '{"lux":1000,"temperature":19.82,"deviceId":"0004A30B001B442B","time":"2016-06-14T16:28:06.766772461Z"}' }
[06/14/2016 16:28:29 > 996af8: INFO] 0004A30B001B442B: Handling uplink
[06/14/2016 16:28:29 > 996af8: INFO] Uplink { devEUI: '0004A30B001B442B',
[06/14/2016 16:28:29 > 996af8: INFO]   message: '{"lux":1000,"temperature":19.82,"deviceId":"0004A30B001B442B","time":"2016-06-14T16:28:28.908965124Z"}' }
```

You have now deployed the whole upstream to the Azure cloud. You have succesfully acomplished all available steps of this workshop.

![Workshop provided by Microsoft, The Things Network and Atos](img/logos/microsoft-ttn-atos.png)