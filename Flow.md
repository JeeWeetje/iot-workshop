# The Things Network & Azure IoT in unison
## Pushing telemetry messages to Microsoft Flow and beyond

This is an example of how messages from Azure Functions can be passed to an external service, eg. Mail, using Microsoft Flow. 

Microsoft Flow is a brand new SaaS offering, available today in preview, for automating workflows across the growing number of applications and SaaS services that business users rely on. How much time do we spend sifting through streams of messages for the few notifications that matter? How much manual labor is spent transferring information from one place to another, or entering data into tracking systems? Too often these tasks are done manually, or not done at all.

We will pass the telemetry to an email address provided by you.

*Note: in this workshop, you are introduced into Microsoft Flow. See for [more information](https://flow.microsoft.com/).*

### Prerequisites

1. An Azure Function written in C#, recieving telemetry from The Things Network
2. Telemetry arriving at the Azure Function
3. Azure account [create here](https://azure.microsoft.com/en-us/free/) _(Azure passes will be present for those who have no Azure account)_
4. A Microsoft account for Microsoft Flow [signup here](https://flow.microsoft.com/)

## Create an endpoint in Microsoft Flow

![alt tag](img/msft/Picture12-connect-anything-using-flow.png)

Follow these steps to create an endpoint in Microsoft Flow to send telemetry data to. From there we can use the data in an "If Then Else" flow.

*Note: If you have no account yet, please sign up first (You can sign up for free using the button at the top of the Microsoft Flow portal).*

1. `Log into` the [Microsoft flow portal](https://flow.microsoft.com/). You will be asked to provide Azure credentials if needed

    ![alt tag](img/flow-introduction.png)

2. Select `Make a flow`. A lot of pre-defined flows are shown. Scroll through the list to get an impression of all combination possible. We will *not* use option, we will create a flow from scratch
3. Select `My flows`

    ![alt tag](img/flow-portal-my-flows.png)

4. Normally, all your flows will appear here. Because there is no flow yet, a short introduction is shown

    ![alt tag](img/flow-my-flows-create.png)

5. Select `Create from blank` 

    ![alt tag](img/flow-portal-create-from-blank.png)

6. An empty flow is shown
7. First give the flow a proper name. Name it `Mail telemetry conditionally` (by replacing 'Untitled')

    ![alt tag](img/flow-input-more-with-name.png)

8. You are invited to select one of many flow steps. `Scroll down` until a step called 'Request' is shown

    ![alt tag](img/flow-input-request2.png)

9. Then select the `Request` step. The following fields are shown

    ![alt tag](img/flow-request-step-init.png)

10. This is an incoming API call and we will use Azure Functions to trigger this flow. *Note: The URL will be generated after save*
11. The Azure Function will post a Json object to this Request URL. Flow can not handle this Json object directly. Therefore, enter the following 'Request Body JSON Schema'. This is used to transform the Json object into an entity. This way, Microsoft Flow can handle the separate fields in the message

    ```json
    {
      "type": "object",
      "properties": {
        "level": {
          "type": "integer"
        },
        "deviceId": {
          "type": "string"
        },
        "time": {
          "type": "string"
        }
      },
      "required": [
        "deviceId"
      ]
    }
    ```

12. The Request step is ready now. We will retrieve the URL after creation of the Flow
13. Select `New step`

    ![alt tag](img/flow-portal-new-step.png)

14. In this flow, we will mail conditionally. So select `Add a condition` 

    ![alt tag](img/flow-portal-add-a-condition.png)

15. This is the heart of the Flow. We provide a condition (like 'Level is lower than 42'). And if it's true, a certain step will be executed. Otherwise, the other step will be executed. *Note: The first or the latter are optional*

    ![alt tag](img/flow-portal-condition-init.png)

16. Enter the left field with 'Choose a value'. The previous Request step will output an entity with fields like 'deviceId', 'time' and 'level'. So here you can compare one of the fields with another value

    ![alt tag](img/flow-portal-condition-fields.png)

17. Select the `level` field
18. Because we want to be warned when the level is less than a certain value, select `is less then` operator

    ![alt tag](img/flow-portal-condition-less-then.png)

19. Finally, enter `42` in the right field

    ![alt tag](img/flow-portal-condition-less-then-42.png)

20. We have created a condition, now we can fill in the 'if then else' blocks with steps. In the left block, Change 'IF YES, DO NOTHING' into 'IF YES' by selecting `Add an Action`

    ![alt tag](img/flow-condition-true-add-action.png)

21. Select the `Mail - Send email` step

    ![alt tag](img/flow-condition-true-add-mail.png)

22. Create a connection for Mail. `Accept` the SendGrid terms and privacy policy *Note: SendGrid is a third party email provider*

    ![alt tag](img/flow-condition-true-mail-step.png)

23. Enter `your own email address` in the 'To' field 
24. Enter `Check the level of device ` plus the entity field 'deviceId' in the 'Subject' field 
25. Enter `Hurry up, the level just got below ` plus the entity field 'level' in the 'Email body' field 

    ![alt tag](img/flow-condition-true-mail-step-filled-in.png)

26. This Mail step is ready, and so is the flow. Select `Create flow`

    ![alt tag](img/flow-portal-create-flow.png)

27. The flow is now being created. We have to wait a moment to get it starting up. Select `Done`

    ![alt tag](img/flow-portal-flow-creation-done.png)

Although the flow is created and almost available we still need to do one thing more.

## Getting the API endpoint of the Request step

By this time, the endpoint of the Request step is created. Before we can pass telemetry to this flow, we need to have that API endpoint.

1. Select `My flows`

    ![alt tag](img/flow-portal-my-flows.png)

2. All your flows will appear here. The flow we just created is shown

    ![alt tag](img/flow-my-flows-list.png)

3. Edit the flow by selecting `the pen` icon

    ![alt tag](img/flow-my-flows-list-flow-edit.png)

4. The flow is shown, in detail. Select the first step, the `Request step` so the fields are shown

    ![alt tag](img/flow-request-step-url.png)

5. The URL of the Request step endpoint is now available to copy
6. **Copy and write down** the Url in the `HTTP POST to this URL` field
7. Select `X Close' to close this page. Do *not* save any changes

    ![alt tag](img/flow-portal-close.png)

Now we have the URL of the endpoint. We will call it inside the Azure Function.

## Altering the azure function for Microsoft Flow access

Right now, the Azure Function is only showing the telemetry as a message in the logging. Let's make the Azure function more meaningful. Let's 'talk' to Microsoft Flow  

1. On the left, select `Resource groups`. A list of resource groups is shown

    ![alt tag](img/azure-resource-groups.png)

2. Select the ResourceGroup `TechDays42rg`. It will open a new blade with all resources in this group
3. Select the Azure Function App `TechDays42fa`
4. Select the function `TechDaysEventHubTriggerFunction`
5. The develop page is shown. In the middle, you will see the function in the 'Code' panel
6. Replace 'the code' with

    ```csharp
    using System;
    using System.Text;

    public static void Run(string myEventHubMessage, TraceWriter log)
    {
      log.Info($"My TechDays trigger function processed this message: {myEventHubMessage}");

      using (var client = new System.Net.Http.HttpClient())
      {

        var response = client.PostAsync(
                        "[PASTE THE REQUEST URL HERE]",
                        new StringContent(myEventHubMessage, 
                                          Encoding.UTF8,
                                          "application/json")
                                          ).Result;
        log.Info($"Microsoft Flow accepts the message: {response.IsSuccessStatusCode.ToString()}");
      }
    }
    ```

7. And `replace` '[PASTE THE REQUEST URL HERE]' with the *remembered* `HTTP POST URL`
8. Select `Save`. The changed C# code will be recompiled immediately
9. Just below 'Code', in the 'Logs' panel, verify the outcome of the compilation

    ```
    2016-09-25T12:23:35.380 Script for function 'TechDaysEventHubTriggerFunction' changed. Reloading.
    2016-09-25T12:23:35.427 Compilation succeeded.
    ```

10. When new telemetry arrives, the Azure Function log shows how the message is picked up by Microsoft Flow

    ```
    2016-09-30T08:29:04.325 Function started (Id=dbd7a0c4-3710-464d-a6fe-bc32d5a856c8)
    2016-09-30T08:29:04.404 My TechDays trigger function processed this message: {"level":45,"time":"2016-09-30T08:28:59.4658651Z","deviceId":"goatTrough"}
    2016-09-30T08:29:05.252 Microsoft Flow accepts the message: True
    2016-09-30T08:29:05.252 Function completed (Success, Id=dbd7a0c4-3710-464d-a6fe-bc32d5a856c8)
    ```

Now the Azure Function is passing the telemetry towards your Microsoft Flow.

## Receiving mail from your device

Microsoft Flow is passing the telemetry to the email address you provided. Microsoft flow provides a list of runs. We can check how our flow is doing.

1. Go to the Microsoft Flow portal
2. Select `My flows`

    ![alt tag](img/flow-portal-my-flows.png)

3. All your flows will appear here. The flow we created will be shown

    ![alt tag](img/flow-my-flows-list.png)

4. Select the run list of the flow by selecting `the i` (for Information) icon

    ![alt tag](img/flow-portal-run-list.png)

5. By now, one or more runs must be listed

    ![alt tag](img/flow-mail-runs-list.png)

6. Select a row and `click' on it
7. The execution of the flow is shown in detail. Green ticks will mark which steps are executed successfully. In the following example, the telemetry is received, the condition is checked but the 'IF NO, DO NOTHING' block is executed (the 'IF YES' block has no green tick)

    ![alt tag](img/flow-mail-sent-mail-false-condition.png)

8. But is the condition is right, the level is too low, both steps will have a green tick

    ![alt tag](img/flow-mail-sent-mail-true-condition.png)

9. Finally, open an email client and check your mail. Did you receive any mail?
10. The mail is only sent conditionally when the level is too low. You will receive an email message like this

    ![alt tag](img/flow-mail-inbox-message.png)

Now we get an email from a TTN device. But you are free to play with Microsoft Flow. For example, you can alter the condition or you can add more steps to sent the telemetry to. Be creative.

This concludes this part of the workshop. Thank you for checking out Microsoft Flow. You have now acquired your IoT chops!

![alt tag](img/msft/Picture13-make-the-world-a-better-place.png)

But... There is one more [thing](Webjob.md)!
![Workshop provided by Microsoft, The Things Network and Atos](img/logos/microsoft-ttn-atos.png)
