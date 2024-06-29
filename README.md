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
