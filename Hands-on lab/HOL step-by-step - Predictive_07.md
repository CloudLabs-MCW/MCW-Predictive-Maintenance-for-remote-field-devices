## Exercise 7: Create an Azure Function to predict pump failure

Duration: 45 minutes

We will be using an Azure Function to read incoming telemetry from IoT Hub and send it to the HTTP endpoint of our predictive maintenance model. The function will receive a 0 or 1 from the model indicating whether or not a pump should be maintained to avoid possible failure. A notification will also be initiated through Flow to notify Field Workers of the maintenance required.

### Task 1: Create an Azure Function Application

1. Return to the [Azure Portal](https://portal.azure.com).

2. Open your resource group for this lab.

3. From the top menu, select the **+ Add** button, and search for Function App.

4. Configure the Function App as follows, then select **Next: Hosting >**:

    | Field         | Value                                           |
    | ------------- | ----------------------------------------------- |
    | App Name      | _your choice, must be globally unique_          |
    | Subscription  | _select the appropriate subscription_           |
    | Resource Group | use existing, and _select Fabrikam_Oil_           |
    | Publish       | _select Code_ |
    | Runtime Stack | _select .Net_                                       |
    | Version | _select 3.1_ |
    | Location      | _select the location nearest to you_            |

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/create-azure-function-app-form.png)

5. Configure the Hosting options as follows, then select **Review + create**:

    | Field         | Value                                           |
    | ------------- | ----------------------------------------------- |
    | Storage Account | _retain the default value of create new_ |
    | Operating System | Windows |
    | Plan Type | Consumption (Serverless)                                     |

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/create-azure-function-app-form2.png)

6. On the Review blade, select **Create**, then **wait until the Function App is created** before continuing.

### Task 2: Create a notification table in Azure Storage

One of the things we would like to avoid is sending repeated notifications to the workforce in the field. Notifications should only be sent once every 24-hour period per device. To keep track of when a notification was last sent for a device, we will use a table in a Storage Account.

1. In the [Azure portal](https://portal.azure.com), select **Resource groups** from the left-hand menu, then select the **Fabrikam_Oil** link from the listing.

2. Select the link for the storage account that was created with the Function App in Task 1. The name will start with "storageaccount".

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/select-function-storage-account.png)

3. From the Storage Account left-hand menu, select **Tables** from the _Data storage_ section, then select the **+ Table** button, and create a new table named **DeviceNotifications**.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/create-storage-table-menu.png)

4. Keep the Storage Account open in your browser for the next task.

### Task 3: Create a notification queue in Azure Storage

There are many ways to trigger flows in Microsoft Power Automate. One of them is monitoring an Azure Queue. We will use a Queue in our Azure Storage Account to host this queue.

1. From the Storage Account left-hand menu, select **Queues** located beneath the _Data storage_ section, then select the **+ Queue** button, and create a new queue named **flownotificationqueue**.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/create-storage-queue-menu.png)

2. Navigate to the storage account. Obtain the Shared Storage Key for the storage account by selecting the left-hand menu item **Access keys**. Copy the _Key_ value of _key1_ and retain this value. We will be using it later in the next task. Also, retain the name of your Storage Account. (in the image below, the name of the Storage Account is _storageaccountfabribce9_)

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/copy-function-storage-access-key.png)

### Task 4: Create notification service in Microsoft Power Automate

We will be using [Microsoft Power Automate](https://flow.microsoft.com/) as a means to email notifications to the workforce in the field. This flow will respond to new messages placed on the queue that we created in Task 3.

1. Access [Microsoft Power Automate](https://flow.microsoft.com) and sign in (create an account if you don't already have one).

2. From the left-hand menu, select **+ Create**, then choose **Instant cloud flow**.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/create-flow-menu.png)

3. When the dialog displays, select the **Skip** link at the bottom to dismiss it.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/instant-flow-skip-dialog.png)

4. From the search bar, type _queue_ to filter connectors and triggers. Then, select the **When there are messages in a queue** item from the filtered list of Triggers.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/select-flow-trigger-type.png)

5. Fill out the form as follows, then select the **Create** button:

   | Field                | Value                                      |
   | -------------------- | ------------------------------------------ |
   | Connection Name      | Notification Queue                         |
   | Storage Account Name | _enter the generated storage account name_ |
   | Shared Storage Key   | _paste the Key value recorded in Task 3_   |

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/create-flow-queue-step.png)

6. In the queue step, select the **flownotificationqueue** item, then select the **+ New step** button.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/create-flow-select-queue.png)

7. In the search box for the next step, search for _email_, then select the **Send an email notification (V3)** item from the filtered list of Actions.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/create-flow-email-step.png)

8. You may need to accept the terms and conditions of the SendGrid service, a free service that provides the underlying email capabilities of this email step.

9. In the Send an email notification (v3) form, fill it out as follows, then select the **+ New Step** button.

   | Field      | Value                                                                                |
   | ---------- | ------------------------------------------------------------------------------------ |
   | To         | _enter your email address_                                                           |
   | Subject    | Action Required: Pump needs maintenance                                              |
   | Email Body | _put cursor in the field, then select **Message Text** from the Dynamic Content menu_ |

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/create-flow-email-form.png)

10. In the search bar for the next step, search for _queue_ once more, then select the **Delete message** item from the filtered list of Actions.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/create-flow-delete-message-step.png)

11. In the Delete message form, fill it out as follows, then select the **Save** button.

    | Field       | Value                                                                               |
    | ----------- | ----------------------------------------------------------------------------------- |
    | Queue Name  | flownotificationqueue                                                               |
    | Message ID  | _put cursor in the field, then select **Message ID** from the Dynamic Content menu_  |
    | Pop Receipt | _put cursor in the field, then select **Pop Receipt** from the Dynamic Content menu_ |

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/create-flow-delete-message-form.png)

12. Microsoft Power Automate will automatically name the Flow. You are able to edit this Flow in the future by selecting **My flows** from the left-hand menu.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/new-flow-created.png)

### Task 5: Obtain connection settings for use with the Azure Function implementation

1. Once the Function App has been provisioned, open the **Fabrikam_Oil** resource group and select the link for the storage account that was created with the Function App in Task 1. The name will start with "storageaccount".

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/select-function-storage-account.png)

2. From the left-hand menu, select **Access Keys** and copy the key 1 Connection String, keep this value handy as we'll be needing it in the next task.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/copy-function-storage-access-key-connection-string.png)

3. Return to the **Fabrikam_Oil** resource group and select the link for the Event Hubs Namespace.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/select-event-hubs-namespace.png)

4. With the Event Hubs Namespace resource open, in the left-hand menu select the **Shared access policies** item located in Settings, then select the **RootManageSharedAccessKey** policy.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/open-event-hub-namespace-access-keys-menu.png)

5. A blade will open where you will be able to copy the Primary Connection string. Keep this value handy as we'll be needing it in the next task.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/copy-event-hub-namespace-access-key.png)

### Task 6: Create the local settings file for the Azure Functions project

It is recommended that you never check in secrets, such as connection strings, into source control. One way to do this is to use settings files. The values stored in these files mimic environment values used in production. The local settings file is never checked into source control.

1. Using Visual Studio Code, open the `C:\MCW-Predictive-Maintenance-for-remote-field-devices-master\Hands-on lab\Resources\FailurePredictionFunction` folder.

2. Upon opening the folder in Visual Studio Code, you may be prompted to restore unresolved dependencies. If this is the case, select the **Restore** button.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/unresolveddependencies.png)

3. In this folder, create a new file named _local.settings.json_ and populate it with the values obtained in the previous task as follows, then save the file (note: prediction model endpoint was obtained in Exercise 6, Task 1 - step 8):

   ```json
   {
      "IsEncrypted": false,
      "Values": {
        "AzureWebJobsStorage": "<paste storage account connection string>",
        "FUNCTIONS_WORKER_RUNTIME": "dotnet",
        "fabrikam-oil_RootManageSharedAccessKey_EVENTHUB": "<paste event hub namespace connection string>",
        "PredictionModelEndpoint": "<paste prediction model endpoint>"
      }
   }
   ```

### Task 7: Review the Azure Function code

1. Observe the static _Run_ function located in **PumpFailurePrediction.cs**. It identifies that the function will run every time the _iot-central-feed_ event hub receives a message. It also uses a specific Consumer Group. These groups are used when there is a possibility that more than one process may be utilizing the data received in the hub. This ensures that dependent processes receive all messages once - thus avoiding any contingency problems. The method receives an array (batch) of events from the hub on each execution, as well as an instance of a logger in case you wish to see values in the console (locally or in the cloud).

   ```c#
   public static async Task Run([EventHubTrigger("iot-central-feed", Connection = "fabrikam-oil_RootManageSharedAccessKey_EVENTHUB",
                                ConsumerGroup = "ingressprocessing")] EventData[] events, ILogger log)
   ```

2. On line 29-36, the message body received from the event is deserialized into a Telemetry object. The Telemetry class matches the telemetry sent by the pumps. The Telemetry class can be found in the `Models/Telemetry.cs` file.

3. From lines 48 - 60, we group the telemetry by Device ID and calculate the averages for each sensor reading. This helps us reduce the number of calls we send to the scoring service that contains our deployed prediction model.

4. Lines 66 through 75 sends the received telemetry to the scoring service endpoint. This service will respond with a 1 - meaning the pump requires maintenance, or a 0 meaning no maintenance notifications should be sent.

5. Lines 81 through 109 checks Table storage to ensure a notification for the specific device hasn't been sent in the last 24 hours. If a notification is due to be sent, it will update the table storage record with the current timestamp and send a notification by queueing a message onto the _flownotificationqueue_ queue.

### Task 8: Run the Function App locally

1. Select <kbd>Ctrl</kbd>+<kbd>F5</kbd> to run the Azure Function code.

2. After some time, you should see log statements indicating that a message has been queued (indicating that Microsoft Power Automate will send a notification email).

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/azure-function-output.png)

3. Once a message has been placed on the _flownotificationqueue_, it will trigger the notification flow that we created and send an email to the field workers. These emails are sent in 5-minute intervals.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/flow-email-receipt.png)

4. You can now exit the locally running functions by selecting the Terminal window by pressing the <kbd>Ctrl</kbd>+<kbd>c</kbd> keys.

### Task 9: Prepare the Azure Function App with settings

1. In Task 6, we created a local settings file to hold environment variables that are used in our function code. We need to mirror these values in the Azure Function App as well. In the Azure portal, access the **Fabrikam_Oil** resource group, and open the **pumpfunctions** Function Application.

2. Select the **Configuration** option in the left-hand menu.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/functionconfigurationsettingsmenu.png)

3. In the **Application Settings** section, we will add the following application settings to mimic those that are in our *local.settings.json* file. Add a new setting by selecting the **New application setting** button.

    | Setting     | Value                                                                               |
    | ----------- | ----------------------------------------------------------------------------------- |
    | fabrikam-oil_RootManageSharedAccessKey_EVENTHUB  | _event hub shared access key value from the local.settings.json file_                  |
    | PredictionModelEndpoint  | _prediction model endpoint value from local.settings.json file_   |

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/functionconfigurationnewsetting.png)

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/functtionappaddremoveappsetting.png)

4. Once complete, select the **Save** button from the top menu to commit the changes to the application configuration.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/functionconfigurationsave.png)

### Task 10: Deploy the Function App into Azure

1. Now that we have been able to successfully run our Functions locally, we are ready to deploy them to the cloud. The first step to deployment is to ensure that you are logged in to your Azure Account. To log into your Azure account, select the following shortcut to display the command palette: <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>p</kbd>.

2. In the textbox of the command palette, type in *Azure:Sign In*, and select enter (or select the command from the list). This will open a Microsoft Authentication webpage in your default browser. Logging into this window will authenticate Visual Studio Code with your ID.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/commandpalettesignin.png)

3. Once authenticated, we are ready to deploy - once again select <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>p</kbd> to open the command palette. Type *Azure Functions: Deploy* and select the *Azure Functions: Deploy to Function App* command from the list.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/commandpalettedeploytofunctionapp.png)

4. The first step of this command is to identify where we are deploying the function to. In our case, we have already created a Function App to house our function called **pumpfunctions**. Select this value from the list of available choices.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/funcdeploydestinationapp.png)

5. You may be prompted if you want to deploy to **pumpfunctions**, select the **Deploy** button in this dialog.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/funcdeploymentdeploydialog.png)

6. After some time, a notification window will display indicating the deployment has completed.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/funcdeploycompleted.png)

7. Returning to the Azure Portal, in the **Fabrikam_Oil** resource group, open the **pumpfunctions** function app and observe that our function that we created in Visual Studio Code has been deployed.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/azurefunctiondeployed.png)

