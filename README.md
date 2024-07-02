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

## 4. TheHive Installation

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
### 4.1 TheHive Configuration     
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

### 5. Configure Wazuh

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
