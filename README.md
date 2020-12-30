# Introduction

This repo documents the steps I took to setup a Rasberry Pi 4 2GB to run Microsoft's IoT Edge.

# Inspiration

The project was inspired by [Lobe.ai](https://lobe.ai/). Initially I planned to make a simple UWP app on Windows IoT Core. 

Then I found [Custom Vision + Azure IoT Edge on a Raspberry Pi 3](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-install-iot-edge?view=iotedge-2018-06&tabs=windows)


# Hardware 

* Raspberry Pi 4 B 2GB 
* Multicomp seethrough case
* [Official USB-C Power Supply](https://www.raspberrypi.org/products/type-c-power-supply/) (DC 5.1 3A output)


# Part A. Pi Setup  

[comment]: # 4. Update your Windows 10, if needed. 
[comment]: # 5. Dead end - newest windows is rejected by the "Check" script.

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
    ```
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
1. Azure CLI
1. Oher normal dev software:
   1. git
   1. git config --global credential.helper wincred
   

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
   1. ```
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
      ```
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
      ```
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

Following [Deploy a module](https://docs.microsoft.com/en-us/azure/iot-edge/quickstart-linux?view=iotedge-2018-06#deploy-a-module) from *Quickstart: Deploy your first IoT Edge module to a virtual Linux device*.

**It's best to follow the original article as it has screenshots and explanations!**

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
   ```
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
   ```
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