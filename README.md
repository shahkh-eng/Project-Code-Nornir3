# Project-Code-Nornir3 

Network Automation Framework “NORNIR” V3, Connect and extract output based on NAPALM Getters and parse output/save in Excel file “GNS Lab”

LinkedIn Article: https://www.linkedin.com/pulse/network-automation-framework-nornir-connect-extract-output-javed/

**(Lab Setup and the instructions are the same as previous implementation with Nornir V2, We have minor changes as Nornir is updates its version to 3) 
**

For this tutorial we will be using network automation framework “Nornir” V3. First we need to understand what is a framework; a framework gives us a basic structure around which we can write our code in a standardized manner. Frameworks are more application specific e.g. flask or django are python frameworks for developing web applications. Similarly we have Nornir to interact with networking devices and writing automation code in python. Automation framework is also important to connect to multiple devices at the same time (concurrently), which greatly reduce response time when it comes to many devices in our inventory.   

We have other frameworks options like Ansible but Nornir is different in the way that we write our own python code to control the automation. Ansible is written in python but to use it we have to write the instructions in its DSL format (YAML). There are also speed concerns, Nornir is faster when it comes to thousands of devices in our inventory. Nornir provides us an interface that does a lot of heavy lifting for us.

Nornir Components:

·       Config file

·       Hosts file

·       Group file

·       Default file

Please check the caveats related to your environment before using any of the below scripts/libraries or configurations in your working environment.

Prerequisites:

1.      GNS3 GUI “v 2.1.21” & GNS3 VM installed and integrated “Please check my previous post regarding these steps”.

2.      Install venv “python3 and nornir3” on your Linux or MAC environment. In my case I will be using PyCharm (venv “python3 + Nornir3”) on Windows 10. For communication with the GNS3 Lab setup I installed “Microsoft KM-TEST Loopback adopter”. You can find the process on this link.

3.      Cisco “IOS” image for Cisco emulated devices.


1. Configure the Microsoft KM-TEST Loopback adapter with the ip address in “192.168.122.100/24” subnet.


2. Configure the IP Address on R1-R5 “IOS” Router interfaces connected to “Ether Hub” from the same subnet as the appliance, below is one example from R1

·        R1(config)#interface ethernet 2/0

·        R1(config-if)#ip address 192.168.122.101 255.255.255.0

·        R1(config-if)#no shutdown

3. You have to configure R1-R5 for SSHv2

(config)#hostname (R1-R5)
(config)#ip domain-name python.com
(config)#crypto key generate rsa (use 1024 bit)
(config)#enable password cisco
(config)#username gnslab password cisco
(config)#username gnslab privilege 15
(config)#ip scp server enable
(config)#line vty 0 4
(config-line)#login local
(config-line)#transport input ssh
(config-line)#exit
4. Try to ping and SSH the R1-e2/0 ip addresse from your IDE (Should work if above steps are correct).

Nornir config File: https://github.com/shahkh-eng/Project-Code-Nornir2
---
inventory:
  plugin: SimpleInventory
  options:
    host_file: "hosts.yml"
    group_file: "group.yml"
runner:
  plugin: threaded
  options:
    num_workers: 20

Nornir group.yml file:
---
IOS_Routers:

  platform: ios

  username: gnslab

  password: cisco

Nornir hosts.yml file: 
---
R1:
  hostname: 192.168.122.1
  groups:
    - IOS_Routers

R2:
  hostname: 192.168.122.2
  groups:
    - IOS_Routers

R3:
  hostname: 192.168.122.3
  groups:
    - IOS_Routers

R4:
  hostname: 192.168.122.4
  groups:
    - IOS_Routers

R5:
  hostname: 192.168.122.5
  groups:
    - IOS_Routers


Python Script to retrieve device Facts and store them into an Excel Workbook

Below script will connect to the relevant inventory hosts and run the napalm Getters “Facts”, it will retrieve the data in a structured format, parse the output and save it in an excel workbook for further analysis and reporting purposes. 


**Script with comments:**

'''
Requirements to run this code
 Python 3.7 or above
 Nornir 3.0
 nornir_napalm
 nornir_utils
 openpyxl
 Download the above libraries with pip3
'''

# Import Statements
from nornir import InitNornir
from nornir_napalm.plugins.tasks import napalm_get
from nornir_utils.plugins.functions import print_result
from openpyxl import Workbook

# Initialize Nornir with a configuration file
nr = InitNornir("config.yml")

# Create a new Workbook s wb
wb = Workbook()

# Take Active Worksheet as ws
device_ws = wb.active

# Change the title of active workseet to Device Facts
device_ws.title = "Devices Details"

# Statically assign headers to Device Facts ws
device_headers = ["Device Name", "Vendor",
                  "Model", "OS", "Serial", "Up Time", ]

# Write headers on the top line of the file
device_ws.append(device_headers)

# Nornir to run napalm getters e.g. facts
getter_output = nr.run(task=napalm_get, getters=["facts"])

# For loop to get interusting values from the output
for host, task_results in getter_output.items():
    try:
        # Get the device facts result
        device_output = task_results[0].result
        # From Dictionery get vendor name
        print(device_output)
        vendor = device_output["facts"]["vendor"]
        print(vendor)
        # From Dictionery get model
        model = device_output["facts"]["model"]
        # From Dictionery get version
        version = device_output["facts"]["os_version"]
        # From Dictionery get serial
        ser_num = device_output["facts"]["serial_number"]
        # From Dictionery get uptime
        uptime = device_output["facts"]["uptime"]
        # Append results to a line to be saved to the worksheet
        line = [host, vendor, model, str(version), str(ser_num), str(uptime), ]
        # Save values to row in worksheet
        device_ws.append(line)

    except:
        #    print(host)
        line = [host, 'Not Accessable', 'N/A', 'N/A', 'N/A', 'N/A', ]
        # Save values to row in worksheet
        device_ws.append(line)
        continue

# Save workbook

wb.save(filename="device information.xlsx")

