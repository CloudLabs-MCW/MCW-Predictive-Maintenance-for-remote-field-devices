## Exercise 1: Configuring IoT Central with devices and metadata

Duration: 45 minutes

[Azure IoT Central](https://azure.microsoft.com/en-us/services/iot-central/) is a Software as a Service (SaaS) offering from Microsoft. The aim of this service is to provide a frictionless entry into the Cloud Computing and IoT space. The core focus of many industrial companies is not on cloud computing; therefore, they do not necessarily have the personnel skilled to provide guidance and to stand up a reliable and scalable infrastructure for an IoT solution. It is imperative for these types of companies to enter the IoT space not only for the cost savings associated with remote monitoring, but also to improve safety for their workers and the environment.

Fabrikam is one such company that could use a helping hand entering the IoT space. They have recently invested in sensor technology on their rod pumps in the field, and they are ready to implement their cloud-based IoT Solution. Their IT department is small and unfamiliar with cloud-based IoT infrastructure; their IT budget also does not afford them the luxury of hiring a team of contractors to build out a solution for them.

The Fabrikam CIO has recently heard of Azure IoT Central. This online offering will streamline the process of them getting their sensor data to the cloud, where they can monitor their equipment for failures and improve their maintenance practices and not have to worry about the underlying infrastructure. A [predictable cost model](https://azure.microsoft.com/en-us/pricing/details/iot-central/) also ensures that there are no financial surprises.

### Task 1: Model the telemetry data

The first task is to identify the data that the equipment will be sending to the cloud. This data will contain fields that represent the data read from the sensors at a specific instant in time. This data will be used in downstream systems to identify patterns that can lead to cost savings, increased safety and more efficient work processes.

The telemetry being reported by the Fabrikam rod pumps are as follows, we will be using this information later in the lab:

#### Telemetry schema

| Field          | Type     | Description                                                                                                                                                                        |
| -------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| SerialNumber   | String   | Unique serial number identifying the rod pump equipment                                                                                                                            |
| IPAddress      | String   | Current IP Address                                                                                                                                                                 |
| TimeStamp      | DateTime | Timestamp in UTC identifying the point in time the telemetry was created                                                                                                           |
| PumpRate       | Numeric  | Speed calculated over the time duration between the last two times the crank arm has passed the proximity sensor measured in Strokes Per Minute (SPM) - minimum 0.0, maximum 100.0 |
| TimePumpOn     | Numeric  | Number of minutes the pump has been on                                                                                                                                             |
| MotorPowerkW   | Numeric  | Measured in Kilowatts (kW)                                                                                                                                                         |
| MotorSpeed     | Numeric  | including slip (RPM)                                                                                                                                                               |
| CasingFriction | Numeric  | Measured in PSI (psi)                                                                                                                                                              |

### Task 2: Create an IoT Central application

1. In the Azure Portal, open the resource group you created in the **Before the HOL** exercise. From the toolbar, select the _+ Add_ button.

2. In the **Search the Marketplace**, search for and select **IoT Central application**. On the IoT Central application resource overview screen, select _Create_.

    ![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/azure-iot-central-website.png)

3. In the IoT Central Application, fill it out as follows, then select the _Create_ button.

    | Field          | Value     |
    | -------------- | -------- |
    | Resource name   | _enter a globally unique name_   |
    | Application URL | _keep the default_ |
    | Subscription | _select the appropriate subscription_ |
    | Resource group | _select `Fabrikam_Oil`_ |
    | Pricing plan | _select `Standard 2`_ |
    | Template | _select `Custom application`_ |
    | Location | _select the location nearest to you_ |

    ![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/azure-iot-central-new-application.png)

4. Wait for the application to be provisioned.

### Task 3: Create the Device Template

1. In the Azure Portal, open the `Fabrikam_Oil` resource group. Select the IoT Central Application resource from the listing.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/iot-central-application-resource-list.png)

2. From the IoT Central Application Overview screen, select the _IoT Central Application URL_. This opens the IoT Central application in a new tab in your browser.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/iot-central-application-overview.png)

3. We need to define the type of equipment we are using, and the data associated with the equipment. In order to do this, we must define a _Device Template_. Select the _Device Templates_ menu item from the left-hand menu. Then select _+ New_ from the toolbar menu.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/create-device-templates.png)

4. On the `Select type` screen, select _IoT device_ as the custom device template type. Select the _Next: Customize_ button.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/new-template-custom.png)

5. On the `Customize` form, for the device template name, enter _Rod Pump_. Keep the `Gateway device` checkbox unchecked. Select the _Next: Review_ button.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/rod-pump-template-create.png)

6. On the `Review` screen, select the _Create_ button.

7. On the `Rod Pump` device template screen, select _Custom model_ beneath the `Create a model` heading.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/create-a-custom-model.png)

8. On the Rod Pump `Model` screen, select the `Model` parent item in the central navigation pane. Select _Add capability to default component_ from beneath the `Capabilities` heading.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/add-capability-to-default-model.png)

9. The `Capabilities` of the device model describe the _telemetry_ expected from the device, the _commands_ it responds to, and its _properties_ ([device twin properties](https://docs.microsoft.com/en-us/azure/iot-central/core/howto-use-properties)). We'll begin defining the `Telemetry` values, populate the capabilities as described in the following table. Use the _+ Add capability_ button at the bottom of the screen to add additional telemetry values. Select the _Save_ button from the toolbar once complete. You will need to expand the capability form to edit the Schema, Unit, Display Unit and Description fields.

    | Display Name   | Name     | Capability Type | Semantic Type | Schema | Unit   |  Display Unit | Description |
    |----------------|----------|-----------------| --------------|--------|--------|---------------|-------------|
    | Pump Rate       | PumpRate       | Telemetry | None | Double | None |  SPM     | Speed calculated over the time duration between the last two times the crank arm has passed the proximity sensor measured in Strokes Per Minute (SPM)  |
    | Time Pump On    | TimePumpOn     | Telemetry | Time Span | Double | Minute |  Minutes | Number of minutes the pump has been on |
    | Motor Power     | MotorPowerKw   | Telemetry | Power | Double | None |  kW      | Measured in Kilowatts (kW) |
    | Motor Speed     | MotorSpeed     | Telemetry | Angular Velocity | Integer | Revolution per minute |  RPM     | including slip (RPM) |
    | Casing Friction | CasingFriction | Telemetry | None | Double | None |  PSI     | strokes per minute |

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/model-telemetry.png)

10. We also need to define the current state of the pump, whether it is running or not. Remaining on the same screen, Select _+ Add capability_. Add the state with the display name of _Power State_, field name of _PowerState_, capability type of _Telemetry_ and semantic type of _State_. For Value schema, select _String_ then add the values _Unavailable_, _On_, and _Off_ (in each the Display name, Name and Value columns). Select the _Save_ button from the toolbar menu.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/power-state-definition.png)

11. In the device template, Properties are metadata associated with the equipment. For our template, we will expect a property for Serial Number, IP Address, and the geographic location of the pump. Remaining on the same screen, select _+ Add capability_. Define the device properties as follows, then select _Save_ from the toolbar menu:

    | Display Name  | Name         | Capability Type | Semantic Type | Schema | Writable | Description                       |
    | ------------- | ------------ | --------------- | ------------- | ------ |--------- | --------------------------------- |
    | Serial Number | SerialNumber | Property  | None | String      | Off |The Serial Number of the rod pump |
    | IP Address    | IPAddress    | Property  | None | String      | Off |The IP address of the rod pump    |
    | Pump Location | Location     | Property  |  Location | Geopoint  | Off | The geo. location of the rod pump |

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/device-properties-form.png)

12. Operators and field workers will want to be able to turn on and off the pumps remotely. In order to do this, we will define a command. Remaining on the same screen, select _+ Add capability_. Select the _Commands_ tab and select the _New_ button to add a new command. Create a command as follows, and select _Save_ from the toolbar:

    - _Display Name_ - _Toggle Motor Power_
    - _Name_ - _ToggleMotorPower_
    - _Capability Type_ - _Command_
    - _Request_ - _On_
    - (Request) _Display name_ - _Toggle_
    - (Request) _Name_ - _Toggle_
    - (Request) _Schema_ - _Boolean_
    - _Description_ - _Toggle the motor power of the pump on and off_.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/device-template-add-command.png)

13. The `Rod Pump` capabilities should now look similar to the following (collapsed):

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/device-capabilities-summary.png)

14. Now, we can define device specific views to help us visualize telemetry and state of the Rod Pumps. We can also create forms to allow pump operators to execute commands on a device. From the center pane navigation menu of the Rod Pump device template, select the _Views_ item. In the `Select to add a new view` screen, select the _Visualizing the device_ card.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/device-template-add-new-view.png)

15. A View is composed of one or more tiles that display information related to a specific device. In the Edit view form, set the _View name_ to _Dashboard_. Then expand the _Telemetry_ drop down and select each item individually, then choosing the _Add tile_ button to add the chart to the View surface. Feel free to arrange the tiles as desired on the View design surface.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/view-adding-telemetry-tiles-view.png)

16. Remaining on the Dashboard view, expand the `Property` drop down list, and add a tile for `IP Address`, `Serial Number`, and `Pump Location`. Note that the tile for `Pump Location` renders with a map icon, meaning that IoT Central has identified the property as geography-based data, and will render it on a map appropriately.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/view-adding-properties-tiles.png)

17. Spend some time now and investigate the various visualizations and settings you can set on each tile. For instance, you have the ability to customize chart types, colors and axes. You can also resize each tile individually. Select the _Save_ button in the toolbar menu to save the `Dashboard` view.

18. On the `Rod Pump` device template screen, select the `Views` item from the central navigation pane, and choose `Visualizing the device` once again to create a new view. Name this view, `Command` and add a tile for the `Toggle Motor Power` command. Once complete, press the `Save` button in the toolbar. This View will allow pump operators to initiate the toggle power command from the IoT Central application.

19. Finally, we can add a thumbnail image to represent the equipment. Select on the circle icon to left of the template name. This will allow you to select an image file. The image used in this lab can be found on [PixaBay](https://pixabay.com/photos/refinery-pump-oil-pump-industry-514010/). After setting the thumbnail, select the `Publish` button in the device template toolbar.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/device-template-thumbnail.png)

### Task 4: Create and provision real devices

Under the hood, Azure IoT Central uses the [Azure IoT Hub Device Provisioning Service (DPS)](https://docs.microsoft.com/en-us/azure/iot-dps/about-iot-dps). The aim of DPS is to provide a consistent way to connect devices to the Azure Cloud. Devices can utilize Shared Access Signatures, or X.509 certificates to securely connect to IoT Central.

[Multiple options](https://docs.microsoft.com/en-us/azure/iot-central/core/concepts-get-connected) exist to register devices in IoT Central, ranging from individual device registration to [bulk device registration](https://docs.microsoft.com/en-us/azure/iot-central/core/howto-manage-devices#import-devices) via a comma delimited file. In this lab we will register a single device using SAS.

1. In the left-hand menu of your IoT Central application, select _Devices_. Select the _Rod Pump_ template from the `Devices` blade and select the _+ New_ button from the toolbar to add a new device.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/add-real-device-menu.png)

2. A modal window will be displayed with an automatically generated Device ID and Device Name. You are able to overwrite these values with anything that makes sense in your downstream systems. We will be creating three real devices in this lab. Create the following as real devices, ensure `Simulate this device` remains toggled off:

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/real-device-id.png)

    | Device Name          | Device ID |
    | -------------------- | --------- |
    | Rod Pump - DEVICE001 | DEVICE001 |
    | Rod Pump - DEVICE002 | DEVICE002 |
    | Rod Pump - DEVICE003 | DEVICE003 |

3. On the Devices list, notice how all three real devices have the provisioning status of _Registered_.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/new-devices-registered.png)
 ProTip! Use 
