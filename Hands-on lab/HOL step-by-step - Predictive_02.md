## Exercise 2: Run the Rod Pump Simulator

Duration: 30 minutes

Included with this lab is source code that will simulate the connection and telemetry of three real pumps. In the previous exercise, we have defined them as DEVICE001, DEVICE002, and DEVICE003. The purpose of the simulator is to demonstrate real-world scenarios that include a normal healthy rod pump (DEVICE002), a gradually failing pump (DEVICE001), and an immediately failing pump (DEVICE003).

### Task 1: Record device connection information

1. In IoT Central, select _Devices_ from the left-hand menu. Then, from the devices list, select the link for _Rod Pump - DEVICE001_, and select the _Connect_ button located in the upper right corner of the device's page.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/device-connect-menu.png)

2. A Device connection modal is displayed, make note of the ID Scope, Device ID, as well as the primary key value.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/device-connection-info.png)

3. Repeat steps 1 and 2 for DEVICE002 and DEVICE003.

### Task 2: Open the Visual Studio solution, and update connection string values

1. Using Visual Studio Code, open the `C:\MCW-Predictive-Maintenance-for-remote-field-devices-master\Hands-on lab\Resources\FieldDeviceSimulator\Fabrikam.FieldDevice.Generator` folder.

    > If you are prompted by Visual Studio code to install additional components to enable debugging, please select the option to install the components.

2. Open the _appsettings.json_ file, and copy & paste the ID Scope and Device Primary Key values into the file.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/appsettings-updated.png)

3. Open _Program.cs_, go to line 144 and you will find the _SetupDeviceRunTasks_ method. This method is responsible for creating the source code representations of the devices that we have defined earlier in the lab. Each device is initialized based on the values obtained from configuration (appsettings.json). Note that DEVICE001 is defined as the pump that will gradually fail, DEVICE002 as a healthy pump, and DEVICE003 as a pump that will fail immediately after a specific amount of time. Line 167 also adds an event handler that gets fired every time the Power State for a pump changes. The power state of a pump gets changed via a cloud to device command - we will be visiting this concept later on in this lab.

4. Open _PumpDevice.cs_, this class represents a device in the field. It encapsulates the properties (serial number and IP address) that are expected in the properties for the device in the cloud. It also maintains its own power state. Line 73 shows the _RegisterAndConnectDeviceAsync_ method that is responsible for connecting to the global device provisioning endpoint to obtain the underlying IoT Hub connection information for the device to establish a connection to the IoT Central application (through the DeviceClient). Line 110 shows the _SendDevicePropertiesAndInitialState_ method which updates the reported properties from the device to the cloud. This is also referred to as _Device Twins_. Line 154 shows the _SendEvent_ method that sends the generated telemetry data to the cloud.

### Task 3: Run the application

1. Within Visual Studio Code, expand the **.vscode** sub-folder, then open **launch.json**. Update the `console` setting to **externalTerminal**. This will cause the debugger to launch the console window into an external terminal instead of within Visual Studio Code. This is a required step since the internal terminal does not support entering values (ReadLine).

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/vs-code-launch-settings.png)

2. Using Visual Studio Code, Debug the current project by pressing _F5_.

3. Once the menu is displayed, select option 1 to generate and send telemetry to IoT Central.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/generate-and-send-telemetry.png)

4. Allow the simulator to start sending telemetry data to IoT Central, you will see output similar to the following:

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/telemetry-data-generated.png)

5. Allow the simulator to run while continuing with this lab.

6. After some time has passed, in IoT Central select the _Devices_ item in the left-hand menu. Note that the provisioning status of DEVICE001, DEVICE002, and DEVICE003 now indicate _Provisioned_.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/provisioned-devices.png)

### Task 4: Interpret telemetry data

DEVICE001 is the rod pump that will gradually fail. Upon running the simulator for approximately 10 minutes (or 1100 messages), you can take a look at the Motor Power chart in the Device Dashboard, or on the measurements tab and watch the power consumption decrease.

1. In IoT Central, select the _Devices_ menu item, then select the link for _Rod Pump - DEVICE001_ in the Devices list.

2. Ensure the _Dashboard_ tab is selected and observe how the Motor Power usage of DEVICE001 gradually degrades.  **Note**: You may not yet see the degradation at this stage. The motor power will gradually decrease after running for several minutes.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/device001-gradual-failure-power.png)

3. Repeat 1-3 and observe that DEVICE002, the non-failing pump, remains above the 30 kW threshold. DEVICE003 is also a failing pump but displays an immediate failure versus a gradual one.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/device002-normal-operation.png)

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/device003-immediate-failure.png)

### Task 5: Restart a failing pump remotely

After observing the failure of two of the rod pumps, you are able cycle the power state of the pump remotely. The simulator is setup to receive the Toggle Motor Power command from IoT Central and will update the state accordingly and start/stop sending telemetry to the cloud.

1. In IoT Central, select _Devices_ from the left-hand menu, then select _Rod Pump - DEVICE001_ from the device list. Observe that even though the pump has in all purposes failed, that there is still power to the motor. In order to recover DEVICE001, select the _Command_ tab. You will see the _Toggle Motor Power_ command with a Toggle parameter. Set the _Toggle_ parameter to `False`, then select the _Run_ button on the command to turn the pump motor off.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/device001-run-toggle-command.png)

2. The simulator will also indicate that the command has been received from the cloud. Note in the output of the simulator, that DEVICE001 is no longer sending telemetry due to the pump motor being off.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/device001-simulator-power-off.png)

3. After a few moments, return to the _Dashboard_ tab of _DEVICE001_ in IoT Central. Note the receipt of telemetry has stopped, and the state indicates the motor power state is off.

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/device001-stopped-telemetry-power-state-off.png)

4. Return to the _Commands_ tab and toggle the motor power back on again by selecting `True` and pressing the _Run_ button once more. On the _Dashboard_ tab, you will see the Power State switch back to online, and telemetry to start flowing again. Due to the restart of the rod pump - it has now recovered, and telemetry is back into normal ranges!

![alt text](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Predictive-Maintenance-for-remote-field-devices/stage/Hands-on%20lab/media/device001-recovered-1.png)
