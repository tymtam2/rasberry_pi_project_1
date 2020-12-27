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
1. SSH into the Pi. The default password is most likely *raspberry*
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
      Local time is now:      Sun Dec 27 08:22:07 AEST 2020.
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

Now we're following https://docs.microsoft.com/en-us/azure/iot-edge/how-to-install-iot-edge?view=iotedge-2018-06&tabs=linux



