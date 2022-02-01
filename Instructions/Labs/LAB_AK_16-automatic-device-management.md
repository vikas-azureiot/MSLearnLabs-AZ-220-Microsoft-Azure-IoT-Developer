---
lab:
    title: 'Lab 16: Automate IoT Device Management with Azure IoT Hub'
    module: 'Module 8: Device Management'
---

# Automate IoT Device Management with Azure IoT Hub

IoT devices often use optimized operating systems or even run code directly on the silicon (without the need for an actual operating system). In order to update the software running on devices like these the most common method is to flash a new version of the entire software package, including the OS as well as the apps running on it (called firmware).

Because each device has a specific purpose, its firmware is also very specific and optimized for the purpose of the device as well as the constrained resources available.

The process for updating firmware can also be specific to the hardware and to the way the hardware manufacturer created the board. This means that a part of the firmware update process is not generic and you will need to work with your device manufacturer to get the details of the firmware update process (unless you are developing your own hardware which means you probably know what the firmware update process).

While firmware updates used to be applied manually on individual devices, this practice no longer makes sense considering the number of devices used in typical IoT solutions. Firmware updates are now more commonly done over-the-air (OTA) with deployments of new firmware managed remotely from the cloud.

There is a set of common denominators to all over-the-air firmware updates for IoT devices:

1. Firmware versions are uniquely identified
1. Firmware comes in a binary file format that the device will need to acquire from an online source
1. Firmware is locally stored is some form of physical storage (ROM memory, hard drive,...)
1. Device manufacturer provide a description of the required operations on the device to update the firmware.

Azure IoT Hub offers advanced support for implementing device management operations on single devices and on collections of devices. The [Automatic Device Management](https://docs.microsoft.com/azure/iot-hub/iot-hub-auto-device-config) feature enables you to simply configure a set of operations, trigger them, and then monitor their progress.

## Lab Scenario

The automated air processing system that you implemented in Contoso's cheese caves has helped the company to raise their already high quality bar. The company has more award-winning cheeses than ever before.

Your base solution consists of IoT devices that are integrated with sensors and a climate control system to provide real-time control of temperature and humidity within a multi-chamber cave system. You also developed a simple back-end app that demonstrated the ability to manage devices using both direct methods and device twin properties.

Contoso has extended the simple back-end app from your initial solution to include an online portal that operators can use to monitor and remotely manage the cave environment. With the new portal, operators can even customize the temperature and humidity within the cave based on the type of cheese or for a specific phase within the cheese aging process. Each chamber or zone within the cave can be controlled separately.

The IT department will be maintaining the back-end portal that they developed for the operators, but your manager has agreed to manage the device-side of the solution.

For you, this means two things:

1. The Operations team at Contoso is always looking for ways to make improvements. These improvements often lead to requests for new features in the device software.

1. The IoT devices that are deployed to cave locations need the latest security patches to ensure privacy and to prevent hackers from taking control of the system. In order to maintain system security, you need to keep the devices up to date by remotely updating their firmware.

You plan to implement features of IoT Hub that enable automatic device management and device management at scale.

The following resources will be created:

![Lab 16 Architecture](media/LAB_AK_16-architecture.png)

## In this lab

In this lab, you will complete the following activities:

* Configure the lab prerequisites (the required Azure resources)
* Write code for a simulated device that will implement a firmware update
* Test the firmware update process on a single device using Azure IoT Hub automatic device management

## Lab Instructions

### Exercise 1: Configure Lab Prerequisites

This lab assumes the following Azure resources are available:

| Resource Type | Resource Name |
| :-- | :-- |
| Resource Group | @lab.CloudResourceGroup(ResourceGroup1).Name |
| IoT Hub | iot-az220-training-{your-id} |
| IoT Device | sensor-th-0155 |

To ensure these resources are available, complete the following steps.

1. In the lab virtual environment, open a Microsoft Edge browser window, and then navigate to the following Web address: 

    > **NOTE**: Whenever you see the green "T" symbol, for example +++enter this text+++, you can click the associated text and the information will be typed into the current field within the virtual machine environment.

    **Web address**: +++https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fMicrosoftLearning%2fMSLearnLabs-AZ-220-Microsoft-Azure-IoT-Developer%2fmaster%2fAllfiles%2FARM%2Flab16.json+++

1. When prompted to Sign in using Azure account credentials, enter the following values at the sign in prompts:

    **Username**: +++@lab.CloudPortalCredential(User1).Username+++

    **Password**: +++@lab.CloudPortalCredential(User1).Password+++

    Once you have signed in, the **Custom deployment** page will be displayed.

1. Under **Project details**, in the **Subscription** dropdown, ensure that the Azure subscription that you intend to use for this lab is selected.

1. In the **Resource group** dropdown, select **@lab.CloudResourceGroup(ResourceGroup1).Name**.

    > **NOTE**: If **@lab.CloudResourceGroup(ResourceGroup1).Name** is not listed:
    >
    > 1. Under the **Resource group** dropdown, click **Create new**.
    > 1. Under **Name**, enter **@lab.CloudResourceGroup(ResourceGroup1).Name**.
    > 1. Click **OK**.

1. Under **Instance details**, in the **Region** dropdown, select the region closest to you.

    > **NOTE**: If the **@lab.CloudResourceGroup(ResourceGroup1).Name** group already exists, the **Region** field is set to the region used by the resource group and is read-only.

1. In the **Your ID** field, enter a unique ID value that includes your initials followed by the current date (using a "YourInitialsYYMMDD" pattern).

    The first part of your unique ID will be your initials in lower-case. The second part will be the last two digits of the current year, the current numeric month, and the current numeric day. For example:

    ccj220101

    During this lab, you could see `{your-id}` listed as part of the suggested resource name whenever you need to enter your unique ID. The `{your-id}` portion of the suggested resource name is a placeholder. You will replace the entire placeholder string (including the `{}`) with your unique value.

1. In the **Course ID** field, enter **az220**.

1. To validate the template, click **Review and create**.

1. If validation passes, click **Create**.

    The deployment will start. It will take several minutes to deploy the required Azure resources.

1. While the Azure resources are being created, open a text editor tool (Notepad is accessible from the **Start** menu, under **Windows Accessories**). 

    You will be using the text editor to store some configuration values associated with the Azure resources.

1. Switch back to the Azure portal window and wait for the deployment to finish.

    You will see a notification when deployment is complete.

    > **WARNING**: Policy settings could cause the deployment to fail during the final "createDevice" operation. If this occurs, you will find instructions below that help you to create the device manually and record the information that is required later in this lab.

1. Once the deployment has completed, in the left navigation area, to review the output values generated during the deployment, click **Outputs**.

1. In your text editor, create a record of the following Outputs for use later:

    * deviceConnectionString

    > **IMPORTANT**: If the deployment failed during the createDevice operation, the Outputs pane may be empty. Complete the following steps to create an IoT device and a record of the information listed above.

    1. On the Azure portal menu, click **Dashboard**.

    1. On the **All resources** tile, to open your IoT hub, click **iot-az220-training-{your-id}**.

    1. On the IoT hub blade, under **Device management**, click **Devices**.

    1. On the Devices page, click **+ Add Device**.

    1. On the Create a device page, under **Device ID**, enter **sensor-th-0155**

        +++sensor-th-0155+++

    1. At the bottom of the page, click **Save**.

    1. On the Devices page, click **Refresh**.

    1. On the Devices page, under **Device ID**, click **sensor-th-0155**.

    1. On the sensor-th-0155 page, to the right of the Primary Connection String value, click **Copy**.

    1. Save the copied value to Notepad and label it as the deviceConnectionString.

    The Azure resources required for this lab are now available.

### Exercise 2: Examine code for a simulated device that implements firmware update

In this exercise, you will review the code of a simulated device app that manages the device twin desired property changes and triggers a local process simulating a firmware update. The process that you implement for launching the firmware update will be similar to the process used for a firmware update on a real device. The process of downloading the new firmware version, installing the firmware update, and restarting the device is simulated.

You will use the Azure Portal to configure and execute a firmware update using the device twin properties. You will configure the device twin properties to transfer the configuration change request to the device and monitor the progress.

#### Task 1: Examine the device simulator app

In this task, you will use Visual Studio Code to review the console app.

1. Open **Visual Studio Code**.

    You may find it helpful to maximize the Visual Studio Code window.

1. On the **File** menu, click **Open Folder**

1. In the Open Folder dialog, navigate to the lab 16 Starter folder.

   On the step before reaching the lab instructions, you downloaded the GitHub repository containing lab resources for this lab. The folder structure includes the following folder path:

    * Allfiles
      * Labs
          * 16-Automate IoT Device Management with Azure IoT Hub
            * Final
              * FWUpdateDevice

    > **NOTE**: By default, the **Allfiles** folder is copied to your Windows Desktop in your virtual machine environment.

1. Click **FWUpdateDevice**, and then click **Select Folder**.

    You should see the following files listed in the EXPLORER pane of Visual Studio Code:

    * FWUpdateDevice.csproj
    * Program.cs

1. In the **EXPLORER** pane, to open the project file, click **FWUpdateDevice.csproj**.

1. Notice the referenced NuGet packages:

    * Microsoft.Azure.Devices.Client - Device SDK for Azure IoT Hub
    * Microsoft.Azure.Devices.Shared - Common code for Azure IoT Device and Service SDKs
    * Newtonsoft.Json - Json.NET is a popular high-performance JSON framework for .NET

#### Task 2: Review the application code

In this task, you will review the code for simulating a firmware update on the device in response to an IoT Hub generated request.

1. Ensure that you have the FWUpdateDevice project folder open in Visual Studio Code.

1. In the **EXPLORER** pane, to open the code file, click **Program.cs**.

1. At the top of the code file, locate the code comment line that begins with **The device connection string**.

    In this simple simulated device app, the device ID and the current firmware version will be tracked during the firmware update process.

    > **NOTE**: You will supply the device connection string value as a parameter when you enter the command to run the app later in this lab.

1. Scroll down to the bottom of the code file to locate the **Main** method.

    > **IMPORTANT**: Since this lab simulates both the device and the firmware update process rather than downloading actual firmware from the cloud to a physical device and rebooting, it can be helpful to review the code used to simulate the process.
  
    Notice that the Main method uses **s_deviceConnectionString** to create a **DeviceClient** instance that connects to IoT Hub. The deviceClient object can be passed between methods of the simulated device app so that the app is able to access and report device twin property updates.  

    The **InitDevice** method simulates the bootup cycle of the device and reports the current firmware by updating the device twin via the **UpdateFWUpdateStatus** method.

    After the device is initialized, the device twin property changed callback is configured.

    The app then enters a loop, where it waits for a device twin update that will trigger the firmware update.

1. Locate the **UpdateFWUpdateStatus** method and review the code:

    This method creates a new **TwinCollection** instance, populates it with the provided values, and then updates the device twin.

1. Locate the **OnDesiredPropertyChanged** method and review the code:

    This method is invoked as the callback when a device twin update is received by the device. If a firmware update is detected, the **UpdateFirmware** method is called. This method simulates the download of the firmware, updating the firmware and then rebooting the device.

### Exercise 3: Test firmware update on a single device

In this exercise, you will use the Azure portal to create a new device management configuration and apply it to your simulated device.

> **NOTE**: In a production environment, your device configuration could apply to hundreds of devices.  

#### Task 1: Start device simulator

1. In your Visual Studio Code window, open a new Terminal pane.

    To create a new Terminal pane, open the **Terminal** menu, and then click **New Terminal**.

    The folder location shown within the command prompt should show the FWUpdateDevice project folder.

1. To run the FWUpdateDevice app, enter the following command:

    ``` bash
    dotnet run "<your device connection string>"
    ```

    > **IMPORTANT**: Remember to replace the placeholder value with your actual device connection string, and be sure to include "" around your connection string.
    >
    > For example: `dotnet run "HostName=iot-az220-training-{your-id}.azure-devices.net;DeviceId=sensor-th-0155;SharedAccessKey={}="`

    After about 5-10 seconds you should see the initial output displayed in the Terminal pane.
 
1. Review the contents of the Terminal pane.

    You should see the following output in the terminal:

    ``` bash
        sensor-th-0155: Device booted
        sensor-th-0155: Current firmware version: 1.0.0
    ```

    Once this information is displayed, the simulated device app enters a holding pattern, waiting for a device twin update that will trigger a firmware update.

#### Task 2: Create the device management configuration

1. Open your Azure portal window.

    > **NOTE**: If you closed the Azure portal browser window, open a new browser windows, navigate to the Azure portal, and use the credentials below to sign in.

    +++http://portal.azure.com+++

    **Username**: +++@lab.CloudPortalCredential(User1).Username+++

    **Password**: +++@lab.CloudPortalCredential(User1).Password+++

1. Navigate to your Azure dashboard.

1. On your Azure portal Dashboard, click **iot-az220-training-{your-id}**.

    Your IoT Hub blade should now be displayed.

1. On the left side navigation menu, under **Device management**, click **Devices**.

1. On the **Devices** page, under **Device ID**, click **sensor-th-0155**.

1. On the sensor-th-0155 page, click **Device twin**.

1. Take a minute to review the contents of the device twin file.

    Notice the values of the desired and reported properties, and the update times listed.

1. Navigate back to your IoT hub blade.

1. On the left side navigation menu, under **Device management**, click **Configurations**.

1. On the **Configurations** pane, click **+ Add Device Configuration**.

1. On the **Create Configuration** page, under **Name**, enter **firmwareupdate**

    +++firmwareupdate+++

    Ensure that you enter **firmwareupdate** under the the required **Name** field for the configuration, not under **Labels**.

1. At the bottom of the blade, click **Next: Twins Settings >**.

1. Under **Device Twin Settings**, in the **Device Twin Property** field, enter **properties.desired.firmware**

    +++properties.desired.firmware+++

1. In the **Device Twin Property Content** field, replace the existing contents with the following:

    ``` json
    {
        "fwVersion":"1.0.1",
        "fwPackageURI":"https://MyPackage.uri",
        "fwPackageCheckValue":"1234"
    }
    ```

    You can right-click in the content field and select **Format Document** to format the JSON if needed.

1. Verify that the JSON content that you entered matches what is displayed above before continuing.

1. At the bottom of the blade, click **Next: Metrics >**.

    You will be using a custom metric to track whether the firmware update was effective.

1. On the **Metrics** tab, under **METRIC NAME**, enter **fwupdated**

    +++fwupdated+++

1. Under **METRIC CRITERIA**, enter the following:

    ``` SQL
    SELECT deviceId FROM devices
        WHERE properties.reported.firmware.currentFwVersion='1.0.1'
    ```

1. At the bottom of the blade, click **Next: Target devices >**.

1. On the **Target Devices** tab, under **Priority**, in the **Priority (higher values ...)** field, enter **10**.

1. Under **Target Condition**, in the **Target Condition** field, enter the following query:

    ``` SQL
    deviceId='sensor-th-0155'
    ```

    > **Note**: If you named your device something different earlier in the labs, be sure to replace the device ID name above with the Device ID that you used to create the device.

1. At the bottom of the blade, click **Next: Review + Create >**

    When the **Review + create** tab opens, you should see a "Validation passed" message for your new configuration.

1. On the **Review + create** tab, if the "Validation passed" message is displayed, click **Create**.

    If the "Validation passed" message is not displayed, you will need to go back and check your work before you can create your configuration.

1. On the **IoT device configuration** pane, under **Configuration Name**, verify that your new **firmwareupdate** configuration is listed.

    Once the new configuration is created, IoT Hub will look for devices matching the configuration's target devices criteria, and will apply the firmware update configuration automatically.

    > **NOTE**: Since both the device and the firmware update process are simulated, rather than downloading firmware to an actual device and rebooting the device, the entire process will be completed quickly and the results will be displayed in the Terminal pane of your simulated device app. 

1. Switch to the Visual Studio Code window, and review the contents of the Terminal pane.

    The Terminal pane should include new output generated by your app that lists the progress of the firmware update process that was triggered.

1. Switch back to your Azure portal window.

1. Open the digital twin file for your sensor-th-0155 device.

1. Take a minute to review the updates recorded within the digital twin during the firmware update process.

    The desired and reported property updates in the digital twin were generated by the Configuration and the simulated device app respectively. You can also see that the status of the Configuration is now set to Applied.

1. Switch to the Visual Studio Code window, stop the simulated app, and close Visual Studio Code.

    You can stop the device simulator by simply pressing the "Enter" key in the terminal.
