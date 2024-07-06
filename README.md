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

## Creating a Windows 10 Virtual Machine

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

## Installing Sysmon on Windows 10

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

## Setting Up Wazuh

To set up our Wazuh server, we'll be using DigitalOcean as our cloud provider. You can create an account on DigitalOcean using this [link](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqa01uVG0yV1VvYU5vVUtyeDdnRWY3MXdqbEdhUXxBQ3Jtc0ttX3p2WDI2anZQTGFiaFUxQzR6bXZPOUNLTDFvbUpLdTZWekc2T2xpN2paTkstU2pXVkN0TFJaNG1OMHVHd0Y4YXFtZUdnd1VTZFBoWlZSazJCMGxwYWREbklvUUgwUmllV2NKVGQ0QmZqeThHLVBnZw&q=https%3A%2F%2Fm.do.co%2Fc%2Fe2ce5a05f701&v=YxpUx0czgx4), which provides $200 in free credit for the first 2 months. (Please note, your bank account will be charged a small verification fee, which will be refunded.)

**Steps:**

1. I'll start by building our Wazuh server. To do this, click on the "Create" button in the top right corner on DigitalOcean. Select "Droplet" from the dropdown menu. Next, choose the region closest to your location.

  ![10](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/a7f54132-b13b-495c-ac35-9d93b43c0b9c)

2. Select Ubuntu as the operating system, and choose version 22.04. For the droplet type, select the "Basic" option with the "Premium Intel" CPU. Ensure that the specifications include at least 8 GB of RAM and 50 GB of drive space. I chose the $48 per month plan.

  ![12](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/4a56e394-aece-4876-bd92-b255ad7625d8)

  ![13](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/3a75150a-5cdb-4345-83fc-074efe8a0e2d)

3. Scroll down and choose either to create a password or an SSH key, depending on your preference. I'll opt for creating a password. Remember to use a password manager to keep your credentials secure, especially since they will be exposed to the internet eventually.

  ![14](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/a404569d-0fd2-426c-b5be-97428dad0ef2)


4. Scroll down to change your hostname. I changed mine to "wazuh." After that, click on "Create Droplet."

5. While your droplet is being created, it's crucial to set up a firewall to protect your server from external threats. Follow these steps:

   - Navigate to the "Networking" section on DigitalOcean's dashboard. Click on "Firewalls" and select "Create Firewall."

     ![16](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/5f2164f0-6f1f-4f96-aff1-107fe755a107)



   - Name your firewall (e.g., "firewall").
   - Under "Inbound Rules," set the firewall rules for incoming traffic. Choose "All TCP" and remove all IPs in the "Sources" field. Add only your public IP address. Obtain your public IP address by visiting "what is my IP address" in a new tab.
   - Repeat the same process for UDP.

     ![17](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/1691187b-dcf1-4793-a863-370814813031)


   - Scroll down and click "Create Firewall" to activate it.

6. After creating your firewall, it's essential to add your virtual machine to it to enhance security. Follow these steps:
   
     - Navigate to "Droplets" on the left-hand side of the DigitalOcean dashboard.
     - Select your Wazuh server from the list of droplets.
     - Then Click on "Networking" and scroll down to "Firewalls." Click "Edit" and select the firewall you created earlier.
     - Under "Firewall," click on "Droplets" and then "Add Droplet."

   ![22](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/f26c376c-30a6-41e0-b2f1-25815be36cc1)



6. SSH into Your Wazuh Server: Open PowerShell on your local machine and execute the following command to SSH into your Wazuh server:

  ```ssh root@your_server_ip```
  
7.    Once connected to your virtual machine via SSH, you can start by updating and upgrading the system. Since we are logged in as root, execute the following commands in your terminal:

```apt-get update && apt-get upgrade -y```

![h](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/ce30915e-1ba8-4151-b16b-a831a0a25e3c)

8. Once the update and upgrade process is complete, you can begin the installation of Wazuh. To get started with Wazuh, run the following curl command to download and execute the installation script:

 ```curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh && sudo bash ./wazuh-install.sh -a ```

![24](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/380964ac-2a15-49d3-8675-bf778b358900)

9. Once the installation is complete, it provides a username and password to access the Wazuh Dashboard. Make sure to copy the username and your password, as you will need them to log in to your Wazuh dashboard. The installation summary will look something like this:

```INFO: - - Summary - -
INFO: You can access the web interface https://<wazuh-dashboard-ip>
 User: admin
 Password: <ADMIN_PASSWORD>
INFO: Installation finished.
```

10. Log into Wazuh: Note your server's public IP address from the DigitalOcean dashboard. Open a new browser tab and go to `https://<your-public-ip>` (make sure to use HTTPS). If you see a security certificate warning, click "Advanced" and then "Proceed." On the Wazuh login page, enter admin as the username and paste the password you saved earlier. You should now be logged into the Wazuh dashboard.

![j](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/ca786f4c-7bb9-4cfb-8f70-6f8795277278)

## TheHive Installation

With our client machine and Wazuh up and running, we can proceed to install TheHive on another droplet. We'll use Ubuntu 22.04 for this installation.

**Steps:**

1. **Create and Configure Droplet:**
   - Click on the "Create" button in the top right corner of the DigitalOcean dashboard. Select "Droplet" from the dropdown menu.
   - Choose the region closest to you.
   - Select Ubuntu 22.04 as the operating system.
   - Ensure the droplet configuration includes the necessary resources (e.g., CPU, RAM, storage), similar to Wazuh.
   - use a password or SSH key.
   - Set the hostname to "thehive."
   - Click "Create Droplet."

![26](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/cf7364ea-e517-473d-9848-6f9fe01a39fa)

2. **Add Droplet to Firewall:**
   - Click on "thehive" and Navigate to "Networking" in the left-hand menu.
   - Scroll down to "Firewalls" and click on "Edit."
   - Click on your existing firewall.
   - Go to the "Droplets" tab and click "Add Droplet."
   - Select "TheHive" droplet and click "Add Droplet."
  
Now both TheHive and Wazuh are being protected by the firewall.

![28](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/0f88d93b-b2d0-4b49-ba3a-9eee38b3890a)


3. **SSH into TheHive:**
   - Open a new terminal or PowerShell window.
   - SSH into the new droplet using the command:
     ```sh
     ssh root@<thehive-public-ip>
     ```

4. **Update and Upgrade:**
   - Once logged in, update and upgrade the system:
     ```sh
     apt-get update && apt-get upgrade -y
     ```
5. **Install Necessary Dependencies:**
   - Run the following command to install the necessary dependencies:
     ```sh
     apt install wget gnupg apt-transport-https git ca-certificates ca-certificates-java curl  software-properties-common python3-pip lsb-release
     ```

6. **Install Java:**
   - Run these commands to install Java:
     ```sh
     wget -qO- https://apt.corretto.aws/corretto.key | sudo gpg --dearmor  -o /usr/share/keyrings/corretto.gpg
     echo "deb [signed-by=/usr/share/keyrings/corretto.gpg] https://apt.corretto.aws stable main" |  sudo tee -a /etc/apt/sources.list.d/corretto.sources.list
     sudo apt update
     sudo apt install java-common java-11-amazon-corretto-jdk
     echo JAVA_HOME="/usr/lib/jvm/java-11-amazon-corretto" | sudo tee -a /etc/environment 
     export JAVA_HOME="/usr/lib/jvm/java-11-amazon-corretto"
     ```

7. **Install Cassandra:**
   - Run these commands to install Cassandra:
     ```sh
     wget -qO -  https://downloads.apache.org/cassandra/KEYS | sudo gpg --dearmor  -o /usr/share/keyrings/cassandra-archive.gpg
     echo "deb [signed-by=/usr/share/keyrings/cassandra-archive.gpg] https://debian.cassandra.apache.org 40x main" |  sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
     sudo apt update
     sudo apt install cassandra
     ```

8. **Install Elasticsearch:**
   - Run these commands to install Elasticsearch:
     ```sh
     wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch |  sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
     sudo apt-get install apt-transport-https
     echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" |  sudo tee /etc/apt/sources.list.d/elastic-7.x.list
     sudo apt update
     sudo apt install elasticsearch
     ```

9. **Install TheHive:**
   - Run these commands to install Elasticsearch:
     ```sh
     wget -O- https://archives.strangebee.com/keys/strangebee.gpg | sudo gpg --dearmor -o /usr/share/keyrings/strangebee-archive-keyring.gpg
     echo 'deb [arch=all signed-by=/usr/share/keyrings/strangebee-archive-keyring.gpg] https://deb.strangebee.com thehive-5.3 main' |sudo tee -a /etc/apt/sources.list.d/strangebee.list
     sudo apt-get update
     sudo apt-get install -y thehive
      ```
### TheHive Configuration     
To begin, we need to set up Cassandra, which is the database system that TheHive uses.

1. **Configure Cassandra:**

    - Find the Cassandra settings file at `/etc/cassandra/cassandra.yaml`
    - Open this file using nano (a text editor):

      ```bash
      sudo nano /etc/cassandra/cassandra.yaml
      ```
    - In the file, look for `cluster_name`. Change `cluster_name` to a name you choose.

      ![j](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/eae35060-393b-4547-a832-6897aeed5cf7)


    - Set the `listen_address` parameter to the public IP address of TheHive. **Note:** To find these parameters quickly, just Ctrl + W and write the parameter name.

      ![jj](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/0ad95a9f-8cbd-41ed-9746-25e9a4d5faae)

   - Set the `rpc_address` to the public IP address of TheHive (same IP as `listen_address`).

      ![jjj](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/4471011f-d41b-4bcb-a600-b15db799286e)


   - Set the `seeds` parameter to the public IP address of TheHive.

      ![jjjj](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/2ba7f221-c375-476e-89b4-23657f64ef01)

   These parameters in the Cassandra configuration file (`cassandra.yaml`) are set to the public IP address of TheHive server to ensure that Cassandra can communicate with other nodes in the cluster and with clients that connect to it.

    - After applying the configurations, save the modifications and close the file.
    - Since we've downloaded TheHive packages, we need to remove old files. First, let's stop the Cassandra service to proceed safely:

      ```bash
      sudo systemctl stop cassandra.service
      ```

    - Next, delete the existing Cassandra data:

      ```bash
      sudo rm -rf /var/lib/cassandra/*
      ```

    - Finally, restart the Cassandra service to apply the changes:

      ```bash
      sudo systemctl start cassandra.service
      ```

    - To check if Cassandra is running, use the following command:

      ```bash
      sudo systemctl status cassandra.service
      ```

2. **Configure Elasticsearch:**

   To configure Elasticsearch, follow these steps:

    - Locate the Elasticsearch configuration file at `/etc/elasticsearch/elasticsearch.yml`. Open this file using a text editor
    - Set the `cluster_name` parameter to your desired name. Remove the comment (#) from the line.
    - Set the `node.name` or leave the value as `node-1`. Remove the comment (#).
    - Set the `network.host`. Remove the comment (#) and set the value to the public IP address of TheHive server.
    - By default, the HTTP port is set to `9200`. You can either uncomment this line or leave it as it is. Here, we'll uncomment it.
   
   To start the Elasticsearch service, it requires either a discovery seed or an initial master node. In this case, we'll configure the cluster:

    - Uncomment the `cluster.initial_master_nodes` line and set it to `node-1`:

       ![34](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/b1e0a44a-e6e4-4240-9757-04a3ed06800a)

       ![35](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/0676a15b-4926-445b-8018-b6b0480c645f)

    - Save the changes and close the file
    - Start and enable the Elasticsearch service:
    
       ```bash
       sudo systemctl start elasticsearch
       sudo systemctl enable elasticsearch
       ```
    - To check if Elasticsearch is running, use the following command:

       ```sh
       sudo systemctl status elasticsearch
       ```

       ![36](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/892dca7f-3489-48c2-92ac-b170409715e3)

   
 3. **Configure TheHive:**

    -  First ensure that the users and group associated with TheHive have access to the /opt/thp directory:
       ```sh
       ls -al /opt/thp
       ```
         ![37](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/c00ec534-1aa4-4749-83e4-ce801d86ee33)

      
    - Change the owener to thehive user and thehive group:
       ```sh
       chown -R thehive:thehive /opt/thp
       ```
     
         ![38](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/d9dd5e45-4a2e-447a-be81-c5a1772a086e)

    - Now we're ready to configure TheHive's settings file. Locate the configuration file at `/etc/thehive/application.conf`.
    - Change the hostname to the public IP address of TheHive, the cluster-name to your previously chosen name, and application.baseurl to `http://<public-ip-of-thehive>`.
   
         ![39](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/453fd572-bf84-400d-82e2-e873f3a98117)

    - Now start and enable TheHive:
       ```sh
       sudo systemctl start thehive
       sudo systemctl enable thehive
       sudo systemctl status thehive
       ```

        ![38](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/a2315401-44ec-4bfa-827e-9be906cc850a)

    - Now we can access TheHive by navigating to `http://YOUR_THEHIVE_ADDRESS:9000`. Here are the default credentials for TheHive:
      "Username: admin@thehive.local
       Password: secret"

        ![2024-07-02 11_24_46-Window](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/746e4121-58b2-470f-a11b-0422485f3140)

    - Note that for TheHive to start, all three services—Cassandra, Elasticsearch, and TheHive—should be running.
    - If you encounter an error while logging in to TheHive (authentication failed), check the status of Elasticsearch:

       ```sh
       sudo systemctl status elasticsearch
       ```
    - If Elasticsearch is down, you may need to adjust its JVM options. Open the JVM options file /etc/elasticsearch/jvm.options.d/jvm.options (create it if it doesn't exist) and add the following lines:
       ```sh
      -Dlog4j2.formatMsgNoLookups=true
      -Xms2g
      -Xmx2g
       ```
      ```sh
      sudo systemctl start elasticsearch
      ```
    - And ensure that elasticsearch and cassandra are still running.
      ```sh
      sudo systemctl status elasticsearch
      sudo systemctl status cassandra
      ```

## Configure Wazuh

To begin, log in to Wazuh. You can find admin credentials in the `wazuh-install-files.tar` file. Follow these steps to extract the credentials and access the Wazuh dashboard:

1. Extract the files from the archive:

   ```sh
   tar -xvf wazuh-install-files.tar
   ```
2. Locate and open the wazuh-passwords.txt file to retrieve the login credentials.

3. Log into Wazuh by navigating to `https://<public-ip-of-wazuh>`.
4. Click "Add agent" to begin configuring a new Wazuh agent. The Wazuh agent is a critical component of the Wazuh security platform, responsible for collecting and analyzing security-related data from endpoints.
5. Select "Windows" as the agent type. Assign the server address as the public IP of the Wazuh server. Provide a name for the agent.
 ![42](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/ad58036e-f0ed-4b73-ae5b-766668d16868)

6. Copy the commands provided and paste them into the Windows PowerShell with administrator privileges where you want to install the agent.

   ![43](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/378ef833-abca-4c2d-825c-5f7a34987695)

7. Next, use the command `net start wazuhsvc` to start the Wazuh agent service.

   ![45](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/3a0d425b-afe0-48eb-a3f1-6944fd3783e4)

8. Wazuh agent successfully installed. You can confirm this by navigating to Wazuh Manager → Agents.

   ![46](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/ac811bb2-669a-49c8-a109-0240547eea4f)

9. Now click on "Security Events" and start querying more events.

   ![47](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/1729491c-d2ee-4b2d-8291-d539739db6e4)

## Generate Telemetry 

We will be generating telemetry from our Windows 10 machine and ingesting it into Wazuh.

**Steps:**

#### 1. Locate the Configuration File

Start with our Windows 10 machine. When we install Wazuh, the configuration file is located under `Program Files (x86)`.

1. Open `Program Files (x86)`.
2. Navigate to the `ossec-agent` folder.
3. Locate the file named `ossec.conf`.

#### 2. Edit the Configuration File

1. Right-click `ossec.conf` and open it with Notepad (you may need administrative privileges).
2. This configuration file contains everything related to Wazuh.
3. Scroll down to the section titled `log analysis`.
4. Under the security location, you'll notice some event IDs being excluded by the exclamation mark (`!=`), which means "does not equal to".

#### 4. Add Mimikatz Monitoring

 We want to look for processes that contain Mimikatz. For this, we need either Sysmon installed or Windows Security Event ID 4688 enabled. We will use the Sysmon method since we installed Sysmon previously.
 
1. First back up the `ossec.conf`file.
2. Scroll down to the `log analysis` section.
3. Copy one of the local file tags and paste it underneath the `application`.
4. We need to change the `application` location name to Sysmon’s channel name.
 
```sh
   <!-- Log analysis -->
  <localfile>
    <location>Application</location>
    <log_format>eventchannel</log_format>
  </localfile>

  <localfile>
    <location>Application</location>
    <log_format>eventchannel</log_format>
  </localfile>

```

#### 7. Find Sysmon Channel Name

1. Open Windows Event Viewer.
2. Expand `Applications and Services Logs`.
3. Expand `Microsoft`, then `Windows`.
4. Scroll down to `Sysmon`, expand it, and right-click `Operational`.
5. Select `Properties` to view the full name of the Sysmon channel.
6. Copy this channel name and paste it into the `application` location in the `ossec.conf` file.

![52](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/c48d4859-3d70-4ff9-9fab-e4be3c799aad)


#### 9. Remove Unwanted Log Sources

For the sake of ingestion:
1. Remove the local file tags for `application`, `security`, and `system`.
2. Keep the `active response` tag.
3. This configuration means that only Sysmon logs will be forwarded to the Wazuh manager.
4. Save the modified `ossec.conf` file.
5. Go to Services application and restart Wazuh
   
#### 10. Download and Run Mimikatz

1. **Disable Windows Defender::**
    - Type `security` in the Windows search bar and select **Windows Security**.
    - Dismiss the virus and threat protection prompt.
    - Under **Manage settings** for virus and threat protection settings, scroll down and click on **Add or remove exclusions**.
    - Add an exclusion for your Downloads folder by selecting **Folder** and choosing the Downloads folder. Confirm by selecting **Yes**.

2. **Download and Extract Mimikatz:**
    - Download Mimikatz from the official website and save it in your Downloads folder.
    - If the download is blocked by your web browser, disable the safe browsing feature (e.g., in Google Chrome, go to **Settings** > **Privacy and security** > **Security** > select **No protection**).
    - Once downloaded, right-click the Mimikatz file and select **Extract all**.

3. **Run Mimikatz:**
    - Open an administrative PowerShell session.
    - Navigate to the Mimikatz folder using the `cd` command.
    - Run `mimikatz.exe`.

     ![53](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/1e3b8103-4f13-4418-8744-4f4489543813)


4. **Verify Telemetry**

    - Go to the Wazuh dashboard and search for "mimikatz" to see if any events are logged.
    - Note: You might not see any events because Sysmon events may not trigger alerts or rules in Wazuh by default.

## Configure Wazuh to Log Everything
  1. In the Wazuh manager CLI, create a backup of the `ossec.conf` file located in `/var/ossec/etc/ossec.conf` and save it in your home directory as `ossec-backup.conf`.
  2. Edit the `ossec.conf` file, changing the `logall` and `logall_json` settings from "no" to "yes" under the `<alerts-log>` section. Save and confirm the changes.

     ![54](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/0cc4caff-2641-4a65-8cda-e69d918f59c1)
 
 3. Restart the Wazuh Manager:

    ```sh
    systemctl restart wazuh-manager.service
    ```
4. Enable Archive Logging. The file `archives.log`, `archives.json`, or both will be created in the `/var/ossec/logs/archives/` directory on the Wazuh server. The logs will be placed in these files. To start ingesting these logs, we need to change   our configuration in Filebeat and change the value of `archives: enabled` from `false` to `true`.

    ```sh
    sudo nano /etc/filebeat/filebeat.yml
    ```

     ![55](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/3aa59037-1009-4ab4-b438-51a8207e514e)

5. Restart the filebeat service:

   ```sh
    sudo systemctl restart filebeat
    ```

## Set Up Wazuh Dashboard

  - Go to the Wazuh Dashboard, click the upper-left menu icon, and navigate to **Stack Management** > **Index Patterns** > **Create index pattern**.
  - Use `wazuh-archives-*` as the index pattern name, and set `timestamp` in the **Time field** drop-down list and click "create index pattern".
  - To view the events on the dashboard, click the upper-left menu icon and navigate to **Discover**. Change the index pattern to `wazuh-archives-*`.  

     ![56](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/014cc97d-398a-4c6f-8ae4-710bfeb99597)
    
    - Check the Wazuh dashboard for events related to Mimikatz.
    - Verify the contents of the `archives.log` and `archives.json` files in the `/var/ossec/logs/archives/` directory to ensure that Mimikatz logs are being captured.

    ![57](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/4edb6dbe-db1f-44b0-8810-f9ed7e2ef560)

## Creating Alerts

1. **Access the Rules:**
    - On the Wazuh dashboard, click on the **Home** button.
    - There will be a drop-down next to it, select **Management** > **Rules**.
  
      ![58](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/f2bd77b5-2a94-4e37-8d52-baa6e899d927)

    - Click on **Manage rules files** at the top right.
    - Search for "sysmon" in the search bar.
    - Open `0800-sysmon_id_1.xml`.
    - Select the rule ID section to use as a template for our custom rule:
      ```xml
      <rule id="92000" level="4">
        <if_group>sysmon_event1</if_group>
        <field name="win.eventdata.parentImage" type="pcre2">(?i)\\(c|w)script\.exe</field>
        <options>no_full_log</options>
        <description>Scripting interpreter spawned a new process</description>
        <mitre>
          <id>T1059.005</id>
        </mitre>
      </rule>
      ```

1. **Create a Custom Rule:**
    - Go back and click on **Custom Rules**.
    - You will see one local rule file `local_rules.xml`. Edit that by clicking on the pencil icon.
    - Paste the rule we copied into the local rule file (below the existing rules), paying attention to indentation.
    - Apply the following changes:
      ```xml
      <rule id="100002" level="15">
        <if_group>sysmon_event1</if_group>
        <field name="win.eventdata.originalFileName" type="pcre2">(?i)mimikatz\.exe</field>
        <description>Mimikatz Usage Detected</description>
        <mitre>
          <id>T1003</id>
        </mitre>
      </rule>
      ```

**Note:** Custom rule IDs should begin at 100000. The level indicates the severity of events, with level 15 being the highest and most critical. We set the originalFileName to mimikatz.exe, ensuring that even if an attacker renames the mimikatz.exe file, the original name remains unchanged, allowing us to detect it.

3. **Save and Restart the Manager:**
    - Save the changes and restart the Wazuh manager to apply the new rule.

4. **Test the Custom Rule:**
    
    - Rename the `mimikatz.exe` file to something else and run it.

      ![62](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/b41d0a36-26d0-4015-a80c-521758a5babd)


    - Check the Wazuh dashboard and see if the alerts are triggered.
   
      ![61](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/f4a38b9f-a1c8-472f-95be-77d93050b4cb)

As you can see, the alert was triggered because our custom rule examines the `originalFileName`.
  
## Set Up Shuffle SOAR Integration

This is the final part of our project, where we will implement the Shuffle SOAR configuration and finalize the overall setup. Let’s get started!

### Create an Account on Shuffle

1. **Sign Up**: Go to [shuffler.io](https://shuffler.io) and create an account. After logging in, you will see a page similar to this:

    ![64](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/f3edac56-8cca-4468-b49e-61fe43fc5a7d)


4. **Create a New Workflow**:
   - Click on **New Workflow**.

   ![65](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/bbe7c84c-d5e4-4ca1-9e03-849dc0d1ad26)


   - Enter a name and description for your workflow.
   - Select any option in the use case section.
   - Save changes.

   ![67](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/c6ab67a9-d8b2-4175-afae-800b049dd1c7)


   You should now see a page like this:

    ![68](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/944d0443-ff7f-4558-90d4-0ba0dfb587f0)

### Configure the Webhook

1. **Add a Webhook**:
   - Click on "triggers" section on the left side.
   - Drag and drop the **Webhook** from the left menu.
   - Select it, give it a name on the right hand side, and copy the URI.
   - We will add this URL to the `ossec.conf` file ON Wazuh manager.

      ![70](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/398ed4d0-7162-4537-95f3-0ff71a7f2e0d)


3. **Modify the Webhook**:
   - Click on the **Change Me** icon and make sure the "find actions" is selected as "Repeat back to me"
   - Remove **Hello World** from the **Call** section.
   - Hit the **+** button and select **Execution Argument**.
   - Save it

      ![71](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/826f1be1-dccd-4341-ba65-7f4955605c11)

### Wazuh Integration

To integrate Wazuh with Shuffle, follow these steps:

1. **Edit `ossec.conf`** located in `/var/ossec/etc/`:
   - Add the following integration to the `ossec.conf` file after the `<global>` tag:
     ```xml
     <integration>
       <name>Shuffle Webhook</name>
       <hook_url>YOUR_WEBHOOK_URL_HERE</hook_url>
       <rule_id>100002</rule_id>
       <alert_format>json</alert_format>
     </integration>
     ```
       ![72](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/6a568fd7-0b6e-428e-91fb-a15dee008f67)

   - **Note**: Ensure the indentation aligns with the other lines in the file and `</hook_url>` is not included as the url.
   - We previously assigned the rule ID `100002` to our Mimikatz detection rule, so we'll reference that same number here.
   - The `alert_format` parameter indicates that alerts will be formatted in JSON.

3. **Restart Wazuh Manager**:
   - Apply the changes by restarting the Wazuh manager:
     ```sh
     systemctl restart wazuh-manager.service
     ```
4. **Test Mimikatz**:
   - Run Mimikatz on your Windows client machine.

### Verify and Test in Shuffle

1. **Start the Workflow**:
   - Return to Shuffle and start the process.

     ![73](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/5564d5b3-e413-4b47-846d-62a6fa976d01)

   - Click on the person icon below and press **Test Workflow**.
  
     ![74](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/e148f15b-7f55-45f4-8278-65584ac29f17)

   - You will see the execution details. Click on it.
   - Click **Execution Argument** to see all the information generated from Wazuh.
     
     ![75](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/0cba01c0-824e-44e9-ba9b-e7ed69f26412)

### Integrating Mimikatz Alerts with Shuffle and VirusTotal

We forwarded the Mimikatz alert to Shuffle, and it was successfully received. Next, we'll extract the SHA hash from the file and verify its reputation score on VirusTotal.

#### Parsing the Hash Value

When dealing with hash values, we need to extract the actual hash from the appended type (e.g., "SHA1="). This ensures that we only send the hash value to VirusTotal for checking. Follow these steps:

   - Click on the **Change Me** icon.
   - Instead of "repeat back to me", select **Regex capture group** from Find Actions menu.
   - Set Input Data. Click the **+** button and select **Execution Argument**.
   - Select **hashes** from Execution Argument.
   
      ![76](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/2aa42874-70db-4f2f-a11c-8cb71edb76e1)

   - Create a regex pattern for extracting the SHA-256 value and write it in Regex field
     ```
     (?<=SHA256=)[^,]*(?=,)
     ```
      ![77](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/2e84c6eb-8156-4cc7-aa61-a98859f05ae0)

     
   - Save the workflow.
   - Click on the person icon.
   - Press the refresh button at the top.
     
     ![78](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/40329eb6-57f5-4223-bd85-8b03d23f24fa)

   - Expand the results to see the parsed SHA-256 hash. We can see it parsed out SHA256 hash.

     ![80](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/3da36a26-040a-48af-aa98-33b33385aec9)


   - Rename "Change Me" to "SHA256_Regex.

#### Integrating VirusTotal API

   - Go to the [VirusTotal website](https://www.virustotal.com) and sign up.
   - Copy your API key and go back to Shuffle.
   - In Shuffle, click on **Apps**.
   - Search for virustotal and click on it to activate.
   - Drag VirusTotal into the workflow, and it will automatically connect.

     ![81](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/ed5e1efa-9ae8-4b29-bd67-344f7ce85396)

   - Change the action type  to **Get a hash report**.
   - Click **Authenticate VirusTotal V3**

       ![82](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/2a306f3c-7d75-4b1e-9b14-ec74fd79b63a)

   - Paste your virustotal API key and hit sumbit.
    
      ![83](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/a2dda574-fc23-400d-9eed-ceb086a90f84)
 
   - Make sure to select the Regex output for list for in hash section (or id section).

     ![85](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/0a07001a-881d-4d9d-8796-26273261520d)

   - Save the workflow.
   - Click on the person icon.
   - Rerun the workflow
   - Expand the virustotal output.

     ![87](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/7ac7567a-ddbf-4f29-ae17-e949b119d97f)

     ![88](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/c6c0fe85-81f6-48c4-83a2-db2e190991fd)

66 scanners have detected this file as malicious. 
To quickly recap, we set up our SOAR platform to receive our Wazuh alert. We then performed regex to parse out the SHA-256 hash and used VirusTotal to check its reputation. Next, we will send the details over to The Hive so it can create an alert for case management.

## TheHive Integration

 - Under the application tab at the bottom left corner, search for "The Hive."
 - Select "TheHive" and click on it.
 - Drag it over to your workflow and connect it to VirusTotal.

   ![89](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/f85aa6f1-3cd6-4c9b-bfcd-3b491599440b)


 **Log in to TheHive**:
  - Go to TheHive and log in using the default credentials:
  - **Username:** `admin@thehive.local`
  - **Password:** `secret`
- By default, there is only one organization named "admin."

 **Create a New Organization and User**:
- Click the plus button at the top left corner to create a new organization.
- Name the organization (e.g., `MYSOC`) and provide a description (e.g., `SOC Automation Project`).
- Confirm the creation of the new organization.
  
  ![90](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/a5f60682-1307-43da-a978-ea382f20f344)


**Add Users to the Organization:**
- Click into the new organization. You will see a message indicating no users have been found.
- Add two users:

  ***First User:***
  - Set the type to `Normal`.
  - **Login:** `mysoc@test.com`
  - **Name:** `mysoc`
  - **Profile:** Select `Analyst`.
  - Save and add another user.

  ***Second User:***
  - Set the type to `Service`.
  - **Login:** `shuffle@test.com`
  - **Name:** `SOAR`
  - **Profile:** In a real-world environment, create a new set of permissions based on the principle of least privilege and assign it to the service account. For this demo environment I select `Analyst`.

   ![91](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/54ee9bf4-a457-486a-83ed-e5de399daab7)

   ![92](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/c7e875cc-6260-4572-b1ac-8b05526366b9)

  ***Create password and API key:***

  - Create a password for the normal user account and an API key for the service account.
  - For the normal user account, highlight the user account, select "Preview," scroll down, and click "Set a new password."

    ![93](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/e5e5d10c-6731-40e9-9aac-ca9637145f3b)

  - For the service account, create an API key and, once created, immediately save it in a secure location.
 
     ![94](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/6fc70e36-48a7-4349-8495-19f8536a9596)

**Sign in with newly created account**:

- Log out of the Admin account and log in as the new normal user.
  
  ![95](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/9501712b-2ed7-4f19-aea6-db67d426235a)

  ### Integrate TheHive with Shuffle: ###

  - Go back to your Shuffle instance.
  - On TheHive instance, click on the "Authenticate."
  - Copy your API key from TheHive and paste it in the authentication field.
  - For the URL, enter the public IP address and port number of your Hive instance.
  - Click on "Submit" to save the authentication configuration.

    ![96](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/0e99bfc5-3105-4863-b664-762f85ae47c5)

  - Under "Find Actions," select "Create alert."
  - Fill other fields as follows:

    - **Date:** `$exec.text.win.eventdata.utcTime`
    - **Description:** `"Mimikatz detected on host: $exec.text.win.system.computer from user: $exec.all_fields.data.win.eventdata.user"`
    - **Flag:** `false`
    - **PAP:** `2`
    - **Severity:** `"2"`
    - **Source:** `"Wazuh"`
    - **SourceRef:** `"Rule: 100002"`
    - **Status:** `"New"`
    - **Summary:** `"mimikatz activity detection on host: $exec.text.win.system.computer and The ProcessID is:$exec.text.win.eventdata.processId and the Commandline is $exec.text.win.eventdata.commandLine"`
    - **Tags:** `["T1003"]`
    - **Title:** `$exec.title`
    - **TLP:** `2`
    - **Type:** `"Internal"`
  - Save the workflow.

  #### Modifying Cloud Firewall on DigitalOcean: ####
   1. Log in to DigitalOcean
   2. From the dashboard, click on the "Networking" tab.
   3. Click on "Firewalls" to view the list of firewalls you have created.
   4. Choose the firewall you created for your project.
   5. Click on "Add Rule" to create a new firewall rule for inbound traffic.
   6. **Configure the Rule:**
   - Set the rule type to **Custom**.
   - Set the **Ports** to `9000`.
   - **Remove all IPv6** addresses.
   - **Keep all IPv4** addresses.
  7. Click the "Save" button to apply the new rule.

     ![99](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/d0512d34-8738-416e-bbb0-0b1c93eddd8a)

After completing your tests, make sure to remove this rule or adjust it to ensure the security of your infrastructure.

  - Head back to Shuffle and rerun the workflow by clicking on the person icon.
  - Switch over to TheHive dashboard where we can view our alert.

     ![100](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/94d9f155-510c-45ec-af35-cf3d304aa115)

