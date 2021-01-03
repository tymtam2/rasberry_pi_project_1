- [Introduction](#introduction)
- [Inspiration](#inspiration)
- [Hardware](#hardware)
- [Part A. Pi Setup](#part-a-pi-setup)
- [Part B. Dev environment setup](#part-b-dev-environment-setup)
- [Part C. Cloud setup](#part-c-cloud-setup)
- [Part D. Install IoT Edge on Pi](#part-d-install-iot-edge-on-pi)
- [Part E. Connect to IoT Hub](#part-e-connect-to-iot-hub)
- [(Optional) Part F. Deploy a simulated temperature module](#optional-part-f-deploy-a-simulated-temperature-module)
- [Part G. Build and deploy a custom IoT module](#part-g-build-and-deploy-a-custom-iot-module)
  - [Create a new module - C# project](#create-a-new-module---c-project)
  - [Build, push, deploy, test](#build-push-deploy-test)
- [Part H. Create, train and export Lobe model](#part-h-create-train-and-export-lobe-model)
- [Part I. Run Lobe model in Iot Edge module](#part-i-run-lobe-model-in-iot-edge-module)

# Introduction

This repo documents steps for building and deploying
 * an app written in C# on a Windows laptop
 * with a machine learning image classifier build using Lobe 
 * that detects if a door is left open  
 * running on a Raspberry Pi running Raspberry Pi OS (linux)
 * using Microsoft's Azure IoT Edge. 

 The number of steps required suprprised me, but the steps are simple (i.e. not complex).

# Inspiration

The project was inspired by [Lobe](https://lobe.ai/). Initially I planned to make a simple UWP app and run it on Windows IoT Core. 

Then I found [Custom Vision + Azure IoT Edge on a Raspberry Pi 3](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-install-iot-edge?view=iotedge-2018-06&tabs=windows).

# Hardware 

* Raspberry Pi 4 B 2GB 
* Transparent case by Multicomp
* [Official USB-C Power Supply](https://www.raspberrypi.org/products/type-c-power-supply/) (DC 5.1 3A output)
* Laptop running Windows 10
* Ethernet cable to connect the Pi to the laptop or to the router (not needed if installing headed Raspberry Pi)


# Part A. Pi Setup  

The steps below use a headless (no UI version) of Raspberry PI OS. It's totally OK to use a full version of the Raspberry Pi OS - this makes finding the ip easier but requires a monitor, a keyboard, and a mouse. 

1. Download the Raspberry Pi OS imager, e.g. from https://www.raspberrypi.org/software/   
1. Install and run the imager
1. Flash the card, I chose Raspberry Pi OS Lite.
1. Add an empty file callsed `ssh` (no extension) to be SD card partition called *boot*. 
1. Insert card into the Pi 
1. Boot the Pi
1. Wait a little while 
1. Find ip address of the pi. In Windows Command Prompt: 
   ```
   C:\Users\letss>ping raspberrypi.local
   Pinging raspberrypi.local [fe80::48fd:294e:4b75:31a7%11] with 32 bytes of data:
   Reply from fe80::48fd:294e:4b75:31a7%11: time=1ms
   Reply from fe80::48fd:294e:4b75:31a7%11: time<1ms
   ```
1. SSH into the Pi. The default password is most likely '*raspberry*'.
   ```
   C:\Users\letss>ssh pi@fe80::48fd:294e:4b75:31a7%11
   The authenticity of host 'fe80::48fd:294e:4b75:31a7%11 (fe80::48fd:294e:4b75:31a7%11)' can't be established.
   ECDSA key fingerprint is SHA256:HRQHM1CUnvWPteTLrj1pNeGnH0SZoCfl9eYMTM78Jao.
   Are you sure you want to continue connecting (yes/no)? yes
   Warning: Permanently added 'fe80::48fd:294e:4b75:31a7%11' (ECDSA) to the list of known hosts.
   pi@fe80::48fd:294e:4b75:31a7%11's password:
   Linux raspberrypi 5.4.79-v7l+ #1373 SMP Mon Nov 23 13:27:40 GMT 2020 armv7l

   The programs included with the Debian GNU/Linux system are free software;
   the exact distribution terms for each program are described in the
   individual files in /usr/share/doc/*/copyright.

   Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
   permitted by applicable law.

   SSH is enabled and the default password for the 'pi' user has not been changed.
   This is a security risk - please login as the 'pi' user and type 'passwd' to set a new password.


   Wi-Fi is currently blocked by rfkill.
   Use raspi-config to set the country before use.
   ```
1. Change ssh password 
1. pi@raspberrypi:~ $ sudo raspi-config   
1. Setup WiFi
   1. pi@raspberrypi:~ $ sudo raspi-config
     ![Wifi Setup 1](/help/images/Config-wifi-1.png "Wifi setup - step 1")
     ![Wifi Setup 2](/help/images/Config-wifi-2.png "Wifi setup - step 2")
     ![Wifi Setup 2](/help/images/Config-wifi-3.png "Wifi setup - step 3")
   1. Wait a bit and confirm that the date and time have been synced
      ```
      pi@raspberrypi:~ $ date
      Sat 26 Dec 22:21:27 GMT 2020
      ```
1. Setup timezone:
   ```bash
   pi@raspberrypi:~ $ sudo raspi-config
  

   Current default time zone: 'Australia/Brisbane'
   Local time is now:    Sun Dec 27 08:22:07 AEST 2020.
   Universal Time is now:  Sat Dec 26 22:22:07 UTC 2020.

   pi@raspberrypi:~ $ pi@raspberrypi:~ $ date
   Sun 27 Dec 08:22:16 AEST 2020
   ```
1. *BTW. After the Pi connects to WiFi raspberrypi.local will become IPv4 address:*
   ```
   pi@raspberrypi:~ $ exit
   logout
   Connection to 192.168.1.12 closed.

   C:\Users\letss>ping raspberrypi.local

   Pinging raspberrypi.local [192.168.1.12] with 32 bytes of data:
   Reply from 192.168.1.12: bytes=32 time=3ms TTL=64
   ```

# Part B. Dev environment setup

1. Get VS Code 
1. Get VS Code IoT Edge extension
1. From the left bar select the Azure extension and see if your Azure subscription is selected.
   Please note that it took > 5 minutes and/or VS Code rester for the freshly created subscription to appear. 
1. [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
1. Oher normal dev software:
   1. git
   

# Part C. Cloud setup 

Some steps from [Quickstart: Deploy your first IoT Edge module to a virtual Linux device](https://docs.microsoft.com/en-us/azure/iot-edge/quickstart-linux?view=iotedge-2018-06)

1. Sing up for an Azure subscription. I'm using Pay-As-You-Go Azure subscription, I used up  12 months.
1. Open PowerShell 
1. Login to Azure
   ```
   az login
   ```
   ![Azure CLI login](/help/images/PowerShell1.png "Azure CLI login")
1. Remove legacy iot extension versions and install the latest and greatest version:  
   ```
   az extension remove --name azure-cli-iot-ext
   az extension add --name azure-iot
   ```
1. Create a resource group. 
   ```
   az group create --name group1 --location australiaeast
   ```
1. Create an IoT Hub at the free tier. 
   *Hub name must be globally unique so myhub1 won't work.*
   ```
   az iot hub create --name myhub1 --resource-group group1 --sku F1 --partition-count 2
   ```
   *--partition-count 2* is needed because the default is 4 and is rejected for the F1 tier
1. Create a device registry record
   ```
   az iot hub device-identity create --device-id d1 --edge-enabled --hub-name myhub1
   ```
1. You can peek at the connection string (which will be stored on the device so that it can connect to IoT Hub).
   We'll need it later.
   ```
   az iot hub device-identity connection-string show --device-id d1 --hub-name myhub1
   ```

# Part D. Install IoT Edge on Pi

Following [Install or uninstall the Azure IoT Edge runtime](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-install-iot-edge?view=iotedge-2018-06&tabs=linux)

It's best tot follow the referenced article rather than the steps below because if has explainations and troubleshooting tips.

1. ssh pi@a.b.c.d
1. > Prepare your device to access the Microsoft installation packages.
   > 1.  Install the repository configuration that matches your device operating system.
   >*  Raspberry Pi OS Stretch:
   >```
   >curl https://packages.microsoft.com/config/debian/stretch/multiarch/prod.list > ./microsoft-prod.list
   >```
   >1. Copy the generated list to the sources.list.d directory.
   >```
   >sudo cp ./microsoft-prod.list /etc/apt/sources.list.d/
   >```
   > 1. Install the Microsoft GPG public key.
   >```
   >curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
   >sudo cp ./microsoft.gpg /etc/apt/trusted.gpg.d/
   >```
1. [Install a container engine](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-install-iot-edge?view=iotedge-2018-06&tabs=linux#install-a-container-engine)
   1. `sudo apt-get update`
   1. `sudo apt-get install moby-engine`
1. [Install the IoT Edge security daemon](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-install-iot-edge?view=iotedge-2018-06&tabs=linux#install-the-iot-edge-security-daemon)
1. `sudo apt-get install iotedge`

# Part E. Connect to IoT Hub

Getting inspiration from [Set up an Azure IoT Edge device with symmetric key authentication](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-manual-provision-symmetric-key?view=iotedge-2018-06&tabs=azure-portal%2Clinux)

For a more secure - and the recommended way for production scenarios - you can follow [Set up an Azure IoT Edge device with X.509 certificate authentication](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-manual-provision-x509?view=iotedge-2018-06).

We've created a device using Azure CLI earlier so we don't need to create it. 

1. We need the devices's connection string. This can be done via Azure CLI or in VS Code:
   1. ```powershell
      PS C:\Users\letss> az iot hub device-identity connection-string show --device-id d1 --hub-name myhub1
      {
         "connectionString": "HostName=myhub1.azure-devices.net;DeviceId=d1;SharedAccessKey=t5f/1...MKI="
      }
      ``` 
      or
   1. In VS Code:
      1. Bottom left corner - select IoT Hub 
      1. Select the device 
      1. Rigth click - *Copy Device Connection String*
         ![VS Code - Copy Device Connection String](/help/images/VS_Code_connection_string.png "VS Code - Copy Device Connection String")
1. Back on the device, [Provision an IoT Edge device](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-manual-provision-symmetric-key?view=iotedge-2018-06&tabs=visual-studio-code%2Clinux#provision-an-iot-edge-device)
  1.  `sudo nano /etc/iotedge/config.yaml`
  1. Change the connection string. [Provision an IoT Edge device](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-manual-provision-symmetric-key?view=iotedge-2018-06&tabs=visual-studio-code%2Clinux#provision-an-iot-edge-device) has help how to paste and save in `nano` the text editor that is used here.
  1. `sudo systemctl restart iotedge`
1. [Verify successful setup](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-manual-provision-symmetric-key?view=iotedge-2018-06&tabs=visual-studio-code%2Clinux#verify-successful-setup)  
  1. Check the status of the IoT Edge service. It should be listed as running: `systemctl status iotedge`
      ```bash
      pi@raspberrypi:~ $ systemctl status iotedge
      ● iotedge.service - Azure IoT Edge daemon
         Loaded: loaded (/lib/systemd/system/iotedge.service; enabled; vendor preset: enabled)
         Active: active (running) since Tue 2020-12-29 11:55:45 AEST; 1min 16s ago
         Docs: man:iotedged(8)
      Main PID: 2753 (iotedged)
         Tasks: 14 (limit: 3861)
         CGroup: /system.slice/iotedge.service
               └─2753 /usr/bin/iotedged -c /etc/iotedge/config.yaml

      Dec 29 11:55:45 raspberrypi iotedged[2753]: 2020-12-29T01:55:45Z [INFO] - Edge issuer CA expiration date: 2021-03-29T01:37:57Z
      Dec 29 11:56:18 raspberrypi iotedged[2753]: 2020-12-29T01:56:18Z [INFO] - Starting management API...
      Dec 29 11:56:18 raspberrypi iotedged[2753]: 2020-12-29T01:56:18Z [INFO] - Starting workload API...
      Dec 29 11:56:18 raspberrypi iotedged[2753]: 2020-12-29T01:56:18Z [INFO] - Starting watchdog with 60 second frequency...
      Dec 29 11:56:18 raspberrypi iotedged[2753]: 2020-12-29T01:56:18Z [INFO] - Listening on fd://iotedge.mgmt.socket/ with 1 thread for management API.
      Dec 29 11:56:18 raspberrypi iotedged[2753]: 2020-12-29T01:56:18Z [INFO] - Listening on fd://iotedge.socket/ with 1 thread for workload API.
      Dec 29 11:56:18 raspberrypi iotedged[2753]: 2020-12-29T01:56:18Z [INFO] - Checking edge runtime status
      Dec 29 11:56:18 raspberrypi iotedged[2753]: 2020-12-29T01:56:18Z [INFO] - Creating and starting edge runtime module edgeAgent
      Dec 29 11:56:19 raspberrypi iotedged[2753]: 2020-12-29T01:56:19Z [INFO] - Updating identity for module $edgeAgent
      Dec 29 11:56:19 raspberrypi iotedged[2753]: 2020-12-29T01:56:19Z [INFO] - Pulling image mcr.microsoft.com/azureiotedge-agent:1.0...
      ```
   1. Examine service logs: `journalctl -u iotedge --no-pager --no-full`
   1. Run the checks:
      ```bash
      pi@raspberrypi:~ $ sudo iotedge check --verbose
      Configuration checks
      --------------------
      √ config.yaml is well-formed - OK
      √ config.yaml has well-formed connection string - OK
      √ container engine is installed and functional - OK
      √ config.yaml has correct hostname - OK
      √ config.yaml has correct URIs for daemon mgmt endpoint - OK
      √ latest security daemon - OK
      √ host time is close to real time - OK
      √ container time is close to host time - OK
      ‼ DNS server - Warning
         Container engine is not configured with DNS server setting, which may impact connectivity to IoT Hub.
         Please see https://aka.ms/iotedge-prod-checklist-dns for best practices.
         You can ignore this warning if you are setting DNS server per module in the Edge deployment.
            caused by: Could not open container engine config file /etc/docker/daemon.json
            caused by: No such file or directory (os error 2)
      ‼ production readiness: certificates - Warning
         The Edge device is using self-signed automatically-generated development certificates.
         They will expire in 89 days (at 2021-03-29 01:37:57 UTC) causing module-to-module and downstream device communication to fail on an active deployment.
         After the certs have expired, restarting the IoT Edge daemon will trigger it to generate new development certs.
         Please consider using production certificates instead. See https://aka.ms/iotedge-prod-checklist-certs for best practices.
      √ production readiness: container engine - OK
      ‼ production readiness: logs policy - Warning
         Container engine is not configured to rotate module logs which may cause it run out of disk space.
         Please see https://aka.ms/iotedge-prod-checklist-logs for best practices.
         You can ignore this warning if you are setting log policy per module in the Edge deployment.
            caused by: Could not open container engine config file /etc/docker/daemon.json
            caused by: No such file or directory (os error 2)
      ‼ production readiness: Edge Agent's storage directory is persisted on the host filesystem - Warning
         The edgeAgent module is not configured to persist its /tmp/edgeAgent directory on the host filesystem.
         Data might be lost if the module is deleted or updated.
         Please see https://aka.ms/iotedge-storage-host for best practices.
      × production readiness: Edge Hub's storage directory is persisted on the host filesystem - Error
         Could not check current state of edgeHub container
            caused by: docker returned exit code: 1, stderr = Error: No such object: edgeHub

      Connectivity checks
      -------------------
      √ host can connect to and perform TLS handshake with IoT Hub AMQP port - OK
      √ host can connect to and perform TLS handshake with IoT Hub HTTPS / WebSockets port - OK
      √ host can connect to and perform TLS handshake with IoT Hub MQTT port - OK
      √ container on the default network can connect to IoT Hub AMQP port - OK
      √ container on the default network can connect to IoT Hub HTTPS / WebSockets port - OK
      √ container on the default network can connect to IoT Hub MQTT port - OK
      √ container on the IoT Edge module network can connect to IoT Hub AMQP port - OK
      √ container on the IoT Edge module network can connect to IoT Hub HTTPS / WebSockets port - OK
      √ container on the IoT Edge module network can connect to IoT Hub MQTT port - OK

      18 check(s) succeeded.
      4 check(s) raised warnings.
      1 check(s) raised errors.
      ```
       ![iotedge check - Output](/help/images/iotedge-check-with-errors.png "iotedge check - Output")
   1. Make a (mental) note that we need to address:
      * not rotating logs (this may cause a propblem even a home project)
      * `No such object: edgeHub`

# (Optional) Part F. Deploy a simulated temperature module 

Following [Deploy a module](https://docs.microsoft.com/en-us/azure/iot-edge/quickstart-linux?view=iotedge-2018-06#deploy-a-module) from *[Quickstart: Deploy your first IoT Edge module to a virtual Linux device](https://docs.microsoft.com/en-us/azure/iot-edge/quickstart-linux?view=iotedge-2018-06)*.

**It's best to follow the original article as it has screenshots and explanations!**

1. Install the module
   > 1. Sign in to the Azure portal and navigate to your IoT hub.
   > 1. From the menu on the left pane, under Automatic Device Management, select IoT Edge.
   > 1. Click on the device ID of the target device from the list of devices.
   > 1. On the upper bar, select Set Modules.
   > 1. In the IoT Edge Modules section of the page, click Add and select Marketplace Module from the drop-down menu.
   > 1. Notice that the SimulatedTemperatureSensor module is added to the IoT Edge Modules section, with the desired status running.
   >    Select Next: Routes to continue to the next step of the wizard.
1. Glance at the default routes, and proceed to *Next: Review + create >*
1. Create the manifest
1. Back on the device check if the device obtained the new device and runs the new module. 
   edgeAgent's module logs will confirm that the new configuration is obtained
   ```bash
   pi@raspberrypi:~ $ sudo iotedge logs edgeAgent -f
   <6> 2020-12-30 23:17:40.335 +00:00 [INF] - Edge agent attempting to connect to IoT Hub via Amqp_Tcp_Only...
   <6> 2020-12-30 23:17:40.812 +00:00 [INF] - Edge agent connected to IoT Hub via Amqp_Tcp_Only.
   <6> 2020-12-30 23:17:41.117 +00:00 [INF] - Initialized new module client with subscriptions enabled
   <6> 2020-12-30 23:17:41.412 +00:00 [INF] - Obtained Edge agent twin from IoTHub with desired properties version 2 and reported properties version 2.
   <6> 2020-12-30 23:17:44.422 +00:00 [INF] - Plan execution started for deployment 2
   <6> 2020-12-30 23:17:44.475 +00:00 [INF] - Executing command: "Command Group: (\n  [Create module SimulatedTemperatureSensor]\n  [Start module SimulatedTemperatureSensor]\n)"
   <6> 2020-12-30 23:17:44.487 +00:00 [INF] - Executing command: "Create module SimulatedTemperatureSensor"
   <6> 2020-12-30 23:17:54.377 +00:00 [INF] - Executing command: "Start module SimulatedTemperatureSensor"
   <6> 2020-12-30 23:17:55.359 +00:00 [INF] - Executing command: "Command Group: (\n  [Create module edgeHub]\n  [Start module edgeHub]\n)"
   <6> 2020-12-30 23:17:55.359 +00:00 [INF] - Executing command: "Create module edgeHub"
   <6> 2020-12-30 23:18:17.990 +00:00 [INF] - Executing command: "Start module edgeHub"
   <6> 2020-12-30 23:18:19.178 +00:00 [INF] - Plan execution ended for deployment 2
   <6> 2020-12-30 23:18:19.577 +00:00 [INF] - Updated reported properties
   <6> 2020-12-30 23:18:24.884 +00:00 [INF] - Updated reported properties
   ^C
   pi@raspberrypi:~ $ sudo iotedge list
   NAME                        STATUS           DESCRIPTION      CONFIG
   SimulatedTemperatureSensor  running          Up a minute      mcr.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0
   edgeAgent                   running          Up 2 days        mcr.microsoft.com/azureiotedge-agent:1.0
   edgeHub                     running          Up a minute      mcr.microsoft.com/azureiotedge-hub:1.0
   pi@raspberrypi:~ $
   ```
1. Observe the new module's logs
   ```bash
   pi@raspberrypi:~ $ sudo iotedge logs SimulatedTemperatureSensor --follow
   [2020-12-30 23:17:55 +00:00]: Starting Module
   SimulatedTemperatureSensor Main() started.
   Initializing simulated temperature sensor to send 500 messages, at an interval of 5 seconds.
   To change this, set the environment variable MessageCount to the number of messages that should be sent (set it to -1 to send unlimited messages).
   [Information]: Trying to initialize module client using transport type [Amqp_Tcp_Only].
   [Information]: Successfully initialized module client of transport type [Amqp_Tcp_Only].
         12/30/2020 23:18:36> Sending message: 1, Body: [{"machine":{"temperature":21.361978102294717,"pressure":1.0412380116538285},"ambient":{"temperature":20.75437125761731,"humidity":24},"timeCreated":"2020-12-30T23:18:35.9559149Z"}]
         12/30/2020 23:18:41> Sending message: 2, Body: [{"machine":{"temperature":22.599724394781386,"pressure":1.1822470829497782},"ambient":{"temperature":20.812547497131185,"humidity":24},"timeCreated":"2020-12-30T23:18:41.3406046Z"}]
         12/30/2020 23:18:46> Sending message: 3, Body: [{"machine":{"temperature":22.608422316544885,"pressure":1.1832379854291641},"ambient":{"temperature":20.83138321588346,"humidity":24},"timeCreated":"2020-12-30T23:18:46.4302656Z"}]
   ```
1. Back on the dev machine (not edge device), observe the events in VS Code:
   ![Watching events in VS Code 1](/help/images/VS_Code_monitoring_events_1.png "Watching events in VS Code 1")
   ![Watching events in VS Code 2](/help/images/VS_Code_monitoring_events_2.png "Watching events in VS Code 2") 


# Part G. Build and deploy a custom IoT module

Based on [Tutorial: Develop IoT Edge modules for Linux devices](https://docs.microsoft.com/en-us/azure/iot-edge/tutorial-develop-for-linux?view=iotedge-2018-06)

This module will run the image classifier. The machine learning module will **not** be added; not yet.

1. [Install container engine](https://docs.microsoft.com/en-us/azure/iot-edge/tutorial-develop-for-linux?view=iotedge-2018-06#install-container-engine)
   1. [Install Docker Desktop on Windows](https://docs.docker.com/docker-for-windows/install/) @457MB
   2. (Not sure if required) Restart Windows. ;(
2. [Set up VS Code and tools](https://docs.microsoft.com/en-us/azure/iot-edge/tutorial-develop-for-linux?view=iotedge-2018-06#set-up-vs-code-and-tools) - that's VS Code and the IoT Edge extension - done in prevoius steps
3. [Create a container registry](https://docs.microsoft.com/en-us/azure/iot-edge/tutorial-develop-for-linux?view=iotedge-2018-06#create-a-container-registry)\
   **Pricing details: Basic - price per day $0.167** 
   TODO Switch to Docker Hub (if free/cheaper)
   1. Create at Basic tier.\
      ![Creating Container Registry](/help/images/CreatingContainerRegistry_1.png "Creating Container Registry 1")\
      ![Creating Container Registry](/help/images/CreatingContainerRegistry_2.png "Creating Container Registry 2") 
   1. Once created, go to the resource.
   2. Go to *Access Keys* and enable admin access.\
      ![Container Registry Access Keys](/help/images/CreatingContainerRegistry_3.png "Container Registry Access Keys")\
      ![Container Registry Access Keys - Admin Access](/help/images/CreatingContainerRegistry_4.png "Container Registry Access Keys - Admin Access") 
   3. Note the url, username and a password

## Create a new module - C# project 

Based on [Create a new module project](https://docs.microsoft.com/en-us/azure/iot-edge/tutorial-develop-for-linux?view=iotedge-2018-06#create-a-new-module-project) and [Use Visual Studio Code to develop and debug modules for Azure IoT Edge](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-vs-code-develop-module?view=iotedge-2018-06)

1. Create a new empty folder for the code and open it in VS Code
2. In VS Code F1 (Open command palette) and find *Azure IoT Edge: New IoT Edge Solution*\
   ![VS Code: New IoT Edge Solution 1](/help/images/CreatingModule_1.png "VS Code: New IoT Edge Solution 1") 
   1. Solution name: *ImageClassifierSolution1*
   2. Template: *C# Module*
   3. Module Name: *ImageClassifierModule1*
   4. Image repository (do *not* leave it as *localhost:5000*): *ttregistry1.azurecr.io/ImageClassifierModule1* 
3. After VS Code Restarts, admire the solution structure. It's described in [Create a project template](https://docs.microsoft.com/en-us/azure/iot-edge/tutorial-develop-for-linux?view=iotedge-2018-06#create-a-new-module-project)
4. Insect the .env file and confirm if the username and password is set.\
   ![.env file](/help/images/CreatingModule_env_file.png ".env file") 
5. Set the target architecture for the module:
   1. Visit the Pi's spec page, for example: [Raspberry Pi 4 Tech Specs](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/specifications/):
      Broadcom BCM2711, Quad core Cortex-A72 (ARM v8) 64-bit SoC @ 1.5GHz
   1. ssh to onto the device 
      ```
      pi@raspberrypi:~ $ uname -a
      Linux raspberrypi 5.4.79-v7l+ #1373 SMP Mon Nov 23 13:27:40 GMT 2020 armv7l GNU/Linux
      pi@raspberrypi:~ $ uname -m
      armv7l
      ```
      If it's *armv7l* then the OS is 32-bit
   2. In Command palette find *Azure IoT Edge: Set Default Target Platform for Edge Solution* and set it to, for RPi 4, to *arm32v7*
6. In Dockerfile.arm32v7 change the first line from 
   `FROM mcr.microsoft.com/dotnet/core/sdk:3.1-buster-arm32v7 AS build-env`
   to 
   `FROM mcr.microsoft.com/dotnet/core/sdk:3.1-buster AS build-env`\
   More details in [github](https://github.com/Azure/iotedge/issues/3353). 
7. Open *deployment.template.json* and examine *moduleContent->$edgeAgent->properties.desired->modules*
   There will be to modules there, because *SimulatedTemperatureSensor* is added by default to template solutions.\
   ![Two default modules](/help/images/CreatingModule_2_modules.png "Two default modules") 
8. Delete *SimulatedTemperatureSensor* entries from *deployment.debug.template.json* and from *deployment.template.json*. Delete it from modules and routes - 4 elements need to be deleted.\
   ![Deleting SimulatedTemperatureSenson](/help/images/CreatingModule_deleting_SimulatedTemperatureSensor.png "Deleting SimulatedTemperatureSensor")
9.  Have a look at the C# source code: *raspberry_pi_project_1\module1\ImageClassifierSolution1\modules\ImageClassifierModule1\Program.cs* and follow [Review the sample code](https://docs.microsoft.com/en-us/azure/iot-edge/tutorial-develop-for-linux?view=iotedge-2018-06#review-the-sample-code)
10. Add a direct method for easy testing. At the end of the `Init` method add
   ```c#
      await ioTHubModuleClient.SetMethodHandlerAsync("alive", AliveHandler, ioTHubModuleClient);
   }

   private static Task<MethodResponse> AliveHandler(MethodRequest methodRequest, object userContext)
   {
      var nowUtc = DateTime.UtcNow;
      var o = new { Status = "OK", NowUtc = nowUtc, NowLocal = nowUtc.ToLocalTime() };
      var json = System.Text.Json.JsonSerializer.Serialize(o);
      var bytes = Encoding.UTF8.GetBytes(json);

      var response = new MethodResponse(bytes, (int) System.Net.HttpStatusCode.OK);
      return Task.FromResult(response);
   }
   ``` 
   
## Build, push, deploy, test

1. Sign into Docker
   1. In VS Code, open terminal
   2. `docker login -u <ACR username> <ACR login server>` where the missing pieces are form the step when Azure Container Registry was created\
      ```powershell
      PS C:\Users\letss> docker login -u ttregistry1  ttregistry1.azurecr.io                                   
      Password: 
      Login Succeeded
      PS C:\Users\letss> 
      ```
1. Still in VS Code's terminal, login to Azure Container Registry
      ```powershell
      PS C:\Users\letss> az acr login -n ttregistry1.azurecr.io
      The login server endpoint suffix '.azurecr.io' is automatically omitted.
      Login Succeeded
      PS C:\Users\letss> 
      ```
2. Right-click `deployment.template.json` and select build and push.\
   ![Build and push](/help/images/CreatingModule_builld_and_push.png "Build and push")\
   This process is slow when it runs for the first time.\
   If the process errors out with `#11 1.679 A fatal error occurred, the folder [/usr/share/dotnet/host/fxr] does not contain any version-numbered child folders` then it's likely it is a issue with build machine vs target machien architecture. Check architecture steps above and [github](https://github.com/Azure/iotedge/issues/3353).\
   The terminal output looks like this:
   ```powershell
   PS C:\dev\raspberry_pi_project_1\module1\ImageClassifierSolution1> docker build  --rm -f "c:\dev\raspberry_pi_project_1\module1\ImageClassifierSolution1\modules\ImageClassifierModule1\Dockerfile.arm32v7" -t ttregistry1.azurecr.io/imageclassifiermodule1:0.0.1-arm32v7 "c:\dev\raspberry_pi_project_1\module1\ImageClassifierSolution1\modules\ImageClassifierModule1" ; if ($?) { docker push ttregistry1.azurecr.io/imageclassifiermodule1:0.0.1-arm32v7 }
   [+] Building 318.9s (16/16) FINISHED
   => [internal] load build definition from Dockerfile.arm32v7                                              0.1s 
   => => transferring dockerfile: 445B                                                                      0.1s 
   => [internal] load .dockerignore                                                                         0.1s 
   => => transferring context: 34B                                                                          0.0s 
   => [internal] load metadata for mcr.microsoft.com/dotnet/core/runtime:3.1-buster-slim-arm32v7            1.3s 
   => [internal] load metadata for mcr.microsoft.com/dotnet/core/sdk:3.1-buster                             2.2s 
   => [internal] load build context                                                                         0.1s 
   => => transferring context: 869B                                                                         0.0s 
   => [build-env 1/6] FROM mcr.microsoft.com/dotnet/core/sdk:3.1-buster@sha256:27df774804cbf186cd05d5d997078765aec2f78ca6f63499bd06a698e99ea46a  71.5s 
   => => resolve mcr.microsoft.com/dotnet/core/sdk:3.1-buster@sha256:27df774804cbf186cd05d5d997078765aec2f78ca6f63499bd06a698e99ea46a             0.0s 
   => => sha256:c0afb8e68e0bcdc1b6e05acaa713a6fe0d818086c596bd1ad99133665c4efe63 10.00MB / 10.00MB          7.5s 
   => => sha256:27df774804cbf186cd05d5d997078765aec2f78ca6f63499bd06a698e99ea46a 1.80kB / 1.80kB            0.0s 
   => => sha256:6c33745f49b41daad28b7b192c447938452ea4de9fe8c7cc3edf1433b1366946 50.40MB / 50.40MB         17.7s 
   => => sha256:abac622402fa6b9845058d25fb32d012e9b2ca9b1a90030ca416e85ef01d8635 6.33kB / 6.33kB            0.0s 
   => => sha256:d599c07d28e6c920ef615f4f9b5cd0d52eb106fcd20c3a7daef389f14edd4ef5 51.83MB / 51.83MB         27.5s 
   => => sha256:cd684fb45e33f8fe269a51339ebf07095b27d5f6849ca0b0818466e4c8771333 13.70MB / 13.70MB         17.1s 
   => => sha256:de420d01940e6b081b3eb27161941c265e42eae1a147cd35138cee926a3b249a 13.26MB / 13.26MB         32.8s 
   => => extracting sha256:6c33745f49b41daad28b7b192c447938452ea4de9fe8c7cc3edf1433b1366946                 5.1s 
   => => extracting sha256:ef072fc32a84ef237dd4fcc7dff2c5e2a77565f24d63977d0fa654a6d8512dd8                 1.2s 
   => => extracting sha256:c0afb8e68e0bcdc1b6e05acaa713a6fe0d818086c596bd1ad99133665c4efe63                 0.9s 
   => => extracting sha256:d599c07d28e6c920ef615f4f9b5cd0d52eb106fcd20c3a7daef389f14edd4ef5                 6.8s 
   => => extracting sha256:cd684fb45e33f8fe269a51339ebf07095b27d5f6849ca0b0818466e4c8771333                 2.4s 
   => => extracting sha256:516d2ff1a7c41ff0a6f3ec1f4e0cd9d8afda9e4406c68def2cfa39f28c3a1db3                19.8s 
   => => extracting sha256:de420d01940e6b081b3eb27161941c265e42eae1a147cd35138cee926a3b249a                 1.6s 
   => [stage-1 1/4] FROM mcr.microsoft.com/dotnet/core/runtime:3.1-buster-slim-arm32v7@sha256:cf16e40297f312d0bbcbb82e381366d9f4420709e23a2bf7cfea381f85374411     0.0s 
   => CACHED [stage-1 2/4] WORKDIR /app                                                                     0.0s 
   => [build-env 2/6] WORKDIR /app                                                                          4.7s 
   => [build-env 3/6] COPY *.csproj ./                                                                      0.1s 
   => [build-env 4/6] RUN dotnet restore                                                                  162.0s 
   => [build-env 5/6] COPY . ./                                                                             5.2s 
   => [build-env 6/6] RUN dotnet publish -c Release -o out                                                 71.0s 
   => [stage-1 3/4] COPY --from=build-env /app/out ./                                                       0.2s 
   => [stage-1 4/4] RUN useradd -ms /bin/bash moduleuser                                                    1.4s 
   => exporting to image                                                                                    0.2s 
   => => exporting layers                                                                                   0.2s 
   => => naming to ttregistry1.azurecr.io/imageclassifiermodule1:0.0.1-arm32v7                              
   The push refers to repository [ttregistry1.azurecr.io/imageclassifiermodule1]
   db2fc4848a13: Pushed
   f02421fd1a6d: Pushed
   ef2f16536cd0: Pushed
   c0fe983dfcca: Pushed
   f1b09a481fdc: Pushed
   3c56c0866069: Pushed
   037e397fa13b: Pushed
   0.0.1-arm64v8: digest: sha256:93a1e47f057cabf2f05b1c4c6fb3f3c942c1869e445603f01086df8a440284ec size: 1790
   PS C:\dev\raspberry_pi_project_1\module1\ImageClassifierSolution1> 
   ```  
1. (Optional) Confirm in the Azure portal that the module is there\
   ![Pushed module](/help/images/CreatingModule_in_container_registry.png "Pushed module")
1. Find and open `module.json` and make a mental note that this is where module version is stored. 
1. Again in VS Code, in the IoT extension, find the device, right click and select *Create Deployment for Single Device*\
   ![Creating deployment](/help/images/CreatingModule_creating_deployment.png "Creating deployment")
1. Navigate to *config/deployment.arm64v8.json* file and select it\
   ![Creating deployemnt - choosing file](/help/images/CreatingModule_creating_deployment_2.png "Creating deployment - choosing file")
1. Inspect the device in the IoT extension and check the log messages.\
   ImageClassifierModule1 shoul be running (may need refreshing) and the Output in the terminal should show a sucess.
   ```
   [Edge] Start deployment to device [d1]
   [Edge] Deployment succeeded.
   ```
   ![Creating deployment - confirming](/help/images/CreatingModule_creating_deployment_3.png "Creating deployment - confirming")
2. Inspect device's logs
   1. ssh to the device, in a different terminal: *ssh pi@ip_here*
   2. `journalctl -u iotedge --no-pager --no-full`\
      ```log
      [INFO] - Pulling image ttregistry1.azurecr.io/imageclassifiermodule1:0.0.1-arm32v7...
      [INFO] - Successfully pulled image ttregistry1.azurecr.io/imageclassifiermodule1:0.0.1-arm32v7
      [INFO] - Creating module ImageClassifierModule1...
      [INFO] - Successfully created module ImageClassifierModule1
      [INFO] - [mgmt] - - - [2021-01-03 05:44:44.911078254 UTC] "PUT /modules/ImageClassifierModule1?api-version=2020-07-07 HTTP/1.1" 200 OK 1058 "-" "-" auth_id(-)
      [INFO] - Starting module ImageClassifierModule1...
      [INFO] - Successfully started module ImageClassifierModule1
      ```
3. Test the alive direct method
   1. In the IoT extension, right-click on the module, *NOT* on the device, and select *Invoke Module Direct Method*\
      ![Deployment - testing](/help/images/CreatingModule_testing.png "Deployment - testing")
   2. Type *alive*
   3. No content; [Enter]
   4. The output should be something like this: 
      ```bash
      [DirectMethod] Invoking Direct Method [alive] to [d1/ImageClassifierModule1] ...
      [DirectMethod] Response from [d1/ImageClassifierModule1]:
      {
      "status": 200,
      "payload": {
         "Status": "OK",
         "NowUtc": "2021-01-03T05:57:00.2343891Z",
         "NowLocal": "2021-01-03T05:57:00.2343891+00:00"
      }
      }
      ```
      ![Deployment - testing result](/help/images/CreatingModule_testing_2.png "Deployment - testing result")

# Part H. Create, train and export Lobe model

TODO

Based on [Lobe Tour](https://lobe.ai/tour). Thanks Jake!

1. Download [Lobe](https://lobe.ai/).
2. Train an image classifier
3. 


# Part I. Run Lobe model in Iot Edge module

TODO

Based on [.NET libraries for Lobe](https://github.com/lobe/lobe.NET)

