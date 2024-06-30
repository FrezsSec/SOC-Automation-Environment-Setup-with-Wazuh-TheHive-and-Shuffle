# Setting Up SOC Automation with Wazuh, TheHive, and Shuffle

I successfully set up and configured a Security Operations Center (SOC) automation project, integrating Wazuh as the security monitoring tool, TheHive for case management, and Shuffle for orchestration and automation. This lab project involved creating a logical diagram of the SOC setup, implementing SOAR integration, and configuring automated response actions. The setup included a network environment comprising a Windows 10 client, Wazuh server, and TheHive server. The project aimed to demonstrate the automation of incident detection, enrichment, and response in a SOC environment.

## Tools

- **Virtualization**: VirtualBox for Windows 10 
- **Client Machine**: Windows 10 with Sysmon
- **Security Monitoring**: Wazuh (Installed on Ubuntu in the cloud)
- **Case Management**: TheHive (Installed on Ubuntu in the cloud)
- **Orchestration & Automation**: Shuffle


## Network Design

The network diagram depicts a logical setup with the following components:
- **Windows 10 Client**: Sends security events to the Wazuh server.
- **Router**: Facilitates network connections.
- **Internet**: Represents the external network where Wazuh, TheHive, and Shuffle reside.
- **Wazuh Server**: Receives events from the client, triggers alerts, and sends data to Shuffle.
- **TheHive Server**: Manages cases and receives alerts from Shuffle.
- **Shuffle**: Handles alerts from Wazuh, enriches Indicators of Compromise (IOCs), and sends notifications and responses.

![2 drawio](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/d4f8cc7e-7c85-4db6-8153-b1f98212e7ac)

# Installation Guide

In this section, we will cover the installation steps for each component of our SOC automation setup.

## 1. Creating a Windows 10 Virtual Machine

### Prerequisites
- VirtualBox 
- Download Windows 10 ISO

### Steps
1. Download VirtualBox from the [official website](https://www.virtualbox.org/) and install it on your host machine.
2. Navigate to this [link](https://www.microsoft.com/en-us/software-download/windows10) and download the Media Creation Tool to generate the Windows 10 ISO image.
3. Open VirtualBox and click "New" to create a new virtual machine. Select a name for the virtual machine (e.g., Windows 10). Then browse to the ISO image folder, double-click it, and tick "Skip unattended installation." Finally, click "Next."

![3](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/c4cba61b-0cbc-4ed3-8e1f-24ce6ba04608)

4. Assign 4 GB of RAM to your virtual machine, or adjust this value based on your available resources, and then click "Next."
5. For the virtual disk, leave the size as 50 GB and click "Next."
6. Now you will see a summary of your settings. If everything looks good, click "Finish."

![6](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/f89a831c-f26b-4866-975d-ceb05f7b76ab)

7. Start the virtual machine and then click "Install Now". When presented with the "Activate Windows" screen, select "I don't have a product key" and choose "Windows 10 Pro" for the edition. Next, accept the license terms. Then, select "Custom: Install Windows only (advanced)" and click "Next". Windows will now begin the installation process.

## 2. Installing Sysmon on Windows 10

**Steps:**

1. On your Windows virtual machine, open a web browser and navigate to the official Microsoft Sysinternals [website](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon).
2. Download Sysmon from the website.
3. Navigate to the [GitHub page](https://github.com/olafhartong/sysmon-modular) where `sysmonconfig.xml` is available and download it to your Windows virtual machine.
4. Locate the downloaded Sysmon ZIP file and extract its contents.
5. Open Powershell with administrative privileges.
6. Navigate to the directory where Sysmon is extracted.
7. Place the `sysmonconfig.xml` file in the same directory where Sysmon is extracted.
8. Run the command to install Sysmon with a basic configuration:

``` .\Sysmon64.exe -i .\sysmonconfig.xml ```

![9](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/37251597-a518-4c87-849a-5f4a5795a9d4)


7. Verify that Sysmon has been installed correctly by checking the Services application.

## 3. Setting Up Wazuh

To set up our Wazuh server, we'll be using DigitalOcean as our cloud provider. You can create an account on DigitalOcean using this [link](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqa01uVG0yV1VvYU5vVUtyeDdnRWY3MXdqbEdhUXxBQ3Jtc0ttX3p2WDI2anZQTGFiaFUxQzR6bXZPOUNLTDFvbUpLdTZWekc2T2xpN2paTkstU2pXVkN0TFJaNG1OMHVHd0Y4YXFtZUdnd1VTZFBoWlZSazJCMGxwYWREbklvUUgwUmllV2NKVGQ0QmZqeThHLVBnZw&q=https%3A%2F%2Fm.do.co%2Fc%2Fe2ce5a05f701&v=YxpUx0czgx4), which provides $200 in free credit for the first 2 months. (Please note, your bank account will be charged a small verification fee, which will be refunded.)

**Steps:**

1. I'll start by building our Wazuh server. To do this, click on the "Create" button in the top right corner on DigitalOcean. Select "Droplet" from the dropdown menu. Next, choose the region closest to your location.

![10](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/8f50caa0-e4e1-451a-aff6-3c44704f7193)

![11](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/f461e4fc-18e4-41c2-8b8b-a9e26bf6dec2)

2. Select Ubuntu as the operating system, and choose version 22.04. For the droplet type, select the "Basic" option with the "Premium Intel" CPU. Ensure that the specifications include at least 8 GB of RAM and 50 GB of drive space. I chose the $48 per month plan.

![12](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/e1d5b458-efc8-4044-81c5-5ed516c9c562)

![13](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/cce15ccf-f21f-4017-ac52-57e3e4d79e62)

3. Scroll down and choose either to create a password or an SSH key, depending on your preference. I'll opt for creating a password. Remember to use a password manager to keep your credentials secure, especially since they will be exposed to the internet eventually.

![14](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/a9f7b022-a75a-4172-b883-d44b2e687752)

4. Scroll down to change your hostname. I changed mine to "wazuh." After that, click on "Create Droplet."

![15](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/775764aa-6baa-48e9-a88d-2267f0fcb6f6)

5. While your droplet is being created, it's crucial to set up a firewall to protect your server from external threats. Follow these steps:

- Navigate to the "Networking" section on DigitalOcean's dashboard. Click on "Firewalls" and select "Create Firewall."

![16](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/8532b385-f490-4747-866c-efd59f7551f8)

- Name your firewall (e.g., "firewall").
- Under "Inbound Rules," set the firewall rules for incoming traffic. Choose "All TCP" and remove all IPs in the "Sources" field. Add only your public IP address. Obtain your public IP address by visiting "what is my IP address" in a new tab.
- Repeat the same process for UDP.

![17](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/fead8bb6-9129-439b-a839-54f973a0ab1b)

- Scroll down and click "Create Firewall" to activate it.

6. After creating your firewall, it's essential to add your virtual machine to it to enhance security. Follow these steps:
- Navigate to "Droplets" on the left-hand side of the DigitalOcean dashboard.
- Select your Wazuh server from the list of droplets.

![18](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/8c3546b3-f9dc-418d-8327-fc6558424ef9)

- Then Click on "Networking" and scroll down to "Firewalls." Click "Edit" and select the firewall you created earlier.
- Under "Firewall," click on "Droplets" and then "Add Droplet."

![19](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/74b5059e-7043-4909-a170-9d2ab07dd5c2)

![22](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/4fc45d7f-bf5b-46e7-9264-2e503852567b)



