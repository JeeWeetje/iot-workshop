# Pushing telemetry messages to Slack

This is an example of how messages from Azure Functions can be passed to an external service, eg. Slack, using Microsoft Flow. 

Microsoft Flow is a brand new SaaS offering, available today in preview, for automating workflows across the growing number of applications and SaaS services that business users rely on. How much time do we spend sifting through streams of messages for the few notifications that matter? How much manual labor is spent transferring information from one place to another, or entering data into tracking systems? Too often these tasks are done manually, or not done at all.

![alt tag](img/arch/flow-overview.png)

*Note: in this workshop we will not dive too deep into Microsoft Flow. See for [more information](https://flow.microsoft.com/).*

### Prerequisites

1. An Azure Function written in C# to recieve telemetry from The Things Network
2. Telemetry arriving at the Azure Function
3. Azure account [create here](https://azure.microsoft.com/en-us/free/) _(Azure passes will be present for those who have no Azure account)_
4. A Microsoft account for Microsoft Flow
5. A Slack account _(optional)_
6. A Twitter account _(optional)_

## Create an endpoint in Microsoft Flow

Follow these steps to create an endpoint in Microsoft Flow to send telemetry data to. From there we can use the data in an "If Then Else" flow.

1. 
