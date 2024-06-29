# Setting Up SOC Automation with Wazuh, TheHive, and Shuffle

I successfully set up and configured a Security Operations Center (SOC) automation project, integrating Wazuh as the security monitoring tool, TheHive for case management, and Shuffle for orchestration and automation. This lab project involved creating a logical diagram of the SOC setup, implementing SOAR integration, and configuring automated response actions. The setup included a network environment comprising a Windows 10 client, Wazuh manager, TheHive, and Shuffle. The project aimed to demonstrate the automation of incident detection, enrichment, and response in a SOC environment.

## Tools

- **Diagram Tool**: draw.io
- **Security Monitoring**: Wazuh
- **Case Management**: TheHive
- **Orchestration & Automation**: Shuffle
- **Client Machine**: Windows 10

## Network Design

The network diagram depicts a logical setup with the following components:
- **Windows 10 Client**: Sends security events to the Wazuh manager.
- **Router**: Facilitates network connections.
- **Internet**: Represents the external network where Wazuh, TheHive, and Shuffle reside.
- **Wazuh Manager**: Receives events from the client, triggers alerts, and sends data to Shuffle.
- **TheHive**: Manages cases and receives alerts from Shuffle.
- **Shuffle**: Handles alerts from Wazuh, enriches Indicators of Compromise (IOCs), and sends notifications and responses.

![topology drawio](https://github.com/FrezsSec/Setting-Up-SOC-Automation-with-Wazuh-TheHive-and-Shuffle/assets/173344802/9242c8fa-bb52-4e5a-a1b7-52da0b982d67)
