🛡️ Agentic AI Security Automation Lab

## **1. Introduction**

This chapter presents the deployment of the experimental environment developed for the **Agentic AI Security Automation** project. The objective of this environment is to provide a controlled and reproducible platform for testing security monitoring, event collection, attack simulation, and AI-assisted alert analysis.

The proposed lab combines several technologies frequently used in modern cybersecurity environments: **Wazuh**, **auditd**, **Docker**, **Atomic Red Team**, **Postman**, and **n8n** integrated with a Large Language Model (LLM). Together, these components form the initial foundation of an intelligent Security Operations Center (SOC)-oriented laboratory.

At this stage of the project, the deployed environment represents the **first operational version** of the platform. Additional modules and improvements will be integrated in subsequent development phases.

---

## **2. Objectives of the Lab Environment**

The deployment of this lab aims to achieve the following objectives:

- Establish a virtualized and isolated environment for cybersecurity experimentation.
- Deploy a Wazuh-based monitoring infrastructure for alert collection and event visualization.
- Integrate Linux audit logs using auditd to enrich endpoint visibility.
- Simulate adversarial techniques using Atomic Red Team.
- Explore the Wazuh Indexer API through Postman.
- Build an AI-powered agent capable of retrieving and summarizing alerts using n8n and Gemini.

This deployment phase constitutes a key step toward the implementation of a semi-automated AI-assisted SOC workflow.

---

## 3. Tested Versions

The following component versions were used during the development and testing of this lab. Using different versions may result in configuration or compatibility issues.

| Component | Exact Version |
| --- | --- |
| **Ubuntu** | **22.04 LTS** |
| **Docker** | **24.x (Latest Stable)** |
| **Docker Compose** | **v2.x (Latest Stable)** |
| **Wazuh Stack** | **4.12.0** |
| **auditd** | **3.0.x (Latest Stable)** |
| **PowerShell** | **7.4.x (Latest Stable)** |
| **Invoke-AtomicRedTeam** | **v2.3.1** |
| **Postman** | **10.x (Latest Stable)** |
| **n8n** | **1.x (Latest Stable)** |
| **Gemini Model** | **gemini-1.5-pro** |

---

## 4. System Requirements

The implementation of the lab requires the following prerequisites:

- A fresh **Ubuntu 22.04 LTS** machine (physical or virtual)
- Stable internet connectivity
- Administrative (`sudo`) privileges
- At least **8 GB RAM** and **40 GB disk space** to run the Docker containers comfortably

The machine serves as the host for the Wazuh stack, attack simulation tools, and workflow automation components.

---

## **5. Deployment of the Base Environment**

### **5.1 Installation of Git**

The first step consists of updating the operating system and installing **Git**, which is required to retrieve the project files hosted on GitHub.

```
sudo apt update
sudo apt install -y git
git --version
```

This ensures that the host system is ready to clone and manage the project repository.

---

### **5.2 Cloning the Project Repository**

The project files are stored in a GitHub repository. The following commands are used to clone the repository and access the lab setup directory:

```
git clone https://github.com/achref891/soc-homelab/Agentic_AI_Security_Automation/wazuh-n8n-lab-setup.git
cd soc-homelab/Agentic_AI_Security_Automation/wazuh-n8n-lab-setup
```

This repository contains the scripts and resources necessary to initialize the current version of the lab environment.

---

### **5.3 Docker Installation**

To simplify service deployment and ensure modularity, the environment relies on **Docker**. Docker allows the different components of the monitoring platform to be deployed as isolated containers.

The installation is performed using the provided shell script:

```
sudo ./install-docker.sh
docker --version
```

This step prepares the host machine for the deployment of the Wazuh stack.

---

### **5.4 Deployment of the Wazuh Stack**

Once Docker is installed, the Wazuh environment can be launched using the dedicated deployment script:

```
sudo ./docker-spin-up.sh
```

This script initializes the three main containers required by the Wazuh platform:

- the **Wazuh Manager** — responsible for agent management and rule-based alert generation
- the **Wazuh Indexer** — an OpenSearch-based datastore for storing and querying alerts
- the **Wazuh Dashboard** — a web UI for visualizing security events

The use of Docker enables easier deployment, maintenance, and future extension of the platform.

---

## **6. Integration of Linux Audit Logs**

### **6.1 Installation of auditd**

To improve endpoint monitoring capabilities, the Linux auditing framework **auditd** is installed. This component provides detailed system-level event logging and is particularly useful for monitoring authentication, privilege escalation, and process-related activities.

The installation is performed as follows:

```
sudo apt install -y auditd audispd-plugins
sudo systemctl enable --now auditd
sudo systemctl status auditd
```

Once activated, auditd continuously records security-relevant system events.

---

### **6.2 Configuration of Wazuh for Audit Log Collection**

To enable Wazuh to ingest Linux audit logs, the Wazuh agent configuration file must be updated:

```
sudo nano /var/ossec/etc/ossec.conf
```

The following XML block is added inside the `<ossec_config>` section:

```
<localfile>
  <log_format>audit</log_format>
  <location>/var/log/audit/audit.log</location>
</localfile>
```

This configuration instructs the Wazuh agent to continuously monitor the `audit.log` file and forward its content for analysis.

After modifying the configuration, the Wazuh agent must be restarted:

```
sudo systemctl restart wazuh-agent
sudo systemctl status wazuh-agent
```
 
![audit.png](./images/audit.png) 

---

### **6.3 Verification of Audit Log Generation**

To confirm that audit logs are correctly generated, the following command can be used:

```
sudo tail -f /var/log/audit/audit.log
```

The appearance of real-time audit events confirms that the auditing subsystem is operational and ready for Wazuh ingestion.


![auditlog.png](./images/auditlog.png) 

---

## **7. Atomic Red Team Integration**

### **7.1 Purpose of the Integration**

To evaluate the detection and monitoring capabilities of the environment, the lab integrates **Atomic Red Team**, a framework used to simulate adversarial techniques mapped to the **MITRE ATT&CK** framework.

Its integration enables the generation of realistic security events that can later be observed and analyzed by Wazuh — effectively stress-testing the monitoring pipeline

---

### **7.2 Installation of PowerShell**

Since Atomic Red Team relies on PowerShell-based execution modules, **PowerShell** must first be installed on the Ubuntu host:

```
wget -q "https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb"
sudo dpkg -i packages-microsoft-prod.deb
rm packages-microsoft-prod.deb
sudo apt update
sudo apt install -y powershell
pwsh --version
```

![pwsh.png](./images/pwsh.png)

---

### **7.3 Installation of Invoke-AtomicRedTeam**

The required PowerShell modules are then installed using the following command:

```
sudo pwsh-Command 'Install-Module -Name Invoke-AtomicRedTeam,powershell-yaml -Scope AllUsers -Force; Import-Module Invoke-AtomicRedTeam -Force'
```

This module provides the commands necessary to execute Atomic Red Team tests.

---

### **7.4 Cloning the Atomic Red Team Repository**

The repository containing the atomic tests is cloned locally:

```
sudo git clone https://github.com/redcanaryco/atomic-red-team.git /opt/atomic-red-team
```

---

### **7.5 Execution of Atomic Tests**

To generate test events within the lab environment, the following command can be used:

```
sudo pwsh-Command 'Import-Module Invoke-AtomicRedTeam; Invoke-AtomicTest All -PathToAtomicsFolder /opt/atomic-red-team/atomics -Cleanup -Confirm:$false'
```

> ⚠️ **Important:** This command must only be executed in an **isolated and controlled laboratory environment**, as it simulates adversarial behaviors.
> 

The generated activity is expected to produce observable logs and alerts that will be collected by Wazuh.

![Atomic red team.png](./images/Atomic_red_team.png)

---

## **8. Exploration of the Wazuh API Using Postman**

### **8.1 Purpose of the API Exploration**

The Wazuh Indexer stores security alerts in a searchable format. To understand the structure of these alerts and prepare future automation steps, the API is explored using **Postman**.

This phase is particularly important because it allows the identification of:

- accessible alert fields,
- query formats,
- authentication mechanisms,
- and possible integration points for AI-based workflows.

---

### **8.2 Installation of Postman**

The Postman installation is performed using the script included in the lab resources:

```
cd wazuh-n8n-lab-setup
sudo ./install-postman.sh
```

Once installed, Postman is opened and configured by:

- creating or signing into an account,
- importing the provided **Postman collection**,
- importing the associated **environment variables**.

![Postman3.png](./images/Postman3.png)
---

### **8.3 Testing Alert Retrieval Queries**

One of the most important endpoints explored during this phase is:

```
POST /wazuh-alerts-4.x-*/_search
```

This endpoint allows searching and retrieving alerts indexed by Wazuh.

Its use in Postman provides a practical understanding of the structure of Wazuh alert data before integrating it into the AI workflow.

![Postman9.png](./images/Postman9.png)

---

## **9. Initial Integration of an AI Cybersecurity Agent**

### **9.1 Objective**

A central objective of this project is the integration of an **AI-driven cybersecurity assistant** capable of retrieving and summarizing Wazuh alerts using natural language interaction.

At this stage of the project, a first implementation has been initiated using:

- **n8n** for workflow orchestration
- **Gemini** as the LLM
- **HTTP requests** for communication with the Wazuh Indexer

This implementation represents an **initial prototype**, which will be extended in later development stages.

---

### **9.2 Workflow Creation in n8n**

A workflow is created in **n8n** to enable interaction between the user, the AI model, and the Wazuh alert database.

The following nodes are used:

- **Chat Message**
- **AI Agent**
- **Google Gemini**
- **Simple Memory**
- **HTTP Request Tool**

This architecture allows a user to query the environment in natural language and receive summarized cybersecurity insights.

![n8n26.png](./images/n8n26.png)

---

### **9.3 Configuration of Gemini Credentials**

To enable communication with the Gemini model, an API key is generated from:

```
https://aistudio.google.com/app/api-keys
```

The generated key is then configured inside n8n as a new credential.

This step allows the AI model to be used for alert interpretation and summarization.

![image.png](./images/n8n9.png)

---

### **9.4 Configuration of the HTTP Request Tool**

The HTTP Request Tool is configured to retrieve alert data directly from the **Wazuh Indexer**.

The endpoint used is:

```
https://single-node-wazuh.indexer-1:9200/wazuh-alerts-4.x-*/_search
```

Authentication is configured using the **Indexer username and password** obtained from the Postman environment.

![image.png](./images/n8n16.png)

The request method must be set to:

```
POST
```

The JSON body used for querying alerts is copied from Postman and adapted so that the AI can dynamically specify the number of alerts to retrieve.

For example:

```
"size": {{$fromAI("NumAlerts", "The number of alerts to retrieve from the indexer", "number",10)}}
```

Since the environment is deployed internally using containers, SSL verification is disabled for testing purposes.

> **Figure X.17**: Configuration of the HTTP Request node in n8n
> 

![n8nhttp.png](./images/n8nhttp.png)

![image.png](./images/n8n19.png)

---

### **9.5 Configuration of the AI Agent Prompt**

The AI agent is provided with a system prompt defining its role, responsibilities, and severity mapping logic.

The prompt instructs the agent to:

- query Wazuh alerts,
- map `rule.level` values into severity categories,
- group alerts by frequency and type,
- and generate a concise summary using Gemini.

This prompt represents the first logic layer of the intelligent SOC assistant.

![image.png](./images/n8n20.png) 

---

## **10. Initial Functional Validation**

### **10.1 Test Query**

To validate the workflow, a natural language query is sent through the n8n chat interface, such as:

```
Can you retrieve the last 20 alerts and summarize them?
```

If the workflow is correctly configured, the AI agent sends a request to the Wazuh Indexer, retrieves the corresponding alert data, and generates a summarized interpretation.

![image.png](./images/n8n24.png)

---

### **10.2 Example of Retrieved Summary**

During the initial tests, the AI agent successfully generated summaries describing:

- the total number of alerts,
- the associated severity levels,
- the most frequent event types,
- and the observed system activity patterns.

For example, the system identified low-severity events such as:

- PAM session activity
- SELinux permission checks
- successful `sudo` executions

This result demonstrates the feasibility of integrating a conversational AI layer into a SOC-oriented monitoring environment.

---

## **11. Preliminary Security Analysis**

Although the alerts retrieved during the initial tests were mainly classified as **Low severity**, they still provide valuable information about the normal behavior of the monitored endpoint.

At this stage, the analysis focuses on:

- verifying expected user login activity,
- reviewing `sudo` usage,
- observing Linux audit events,
- and correlating low-level alerts with host-level logs.

This phase is important because it establishes a **baseline of normal behavior**, which will later support anomaly detection and more advanced AI-assisted investigations.

---

## **12. Conclusion**

This chapter presented the initial deployment of the **Agentic AI Security Automation Lab**.

The current implementation successfully establishes:

- a **Wazuh-based monitoring environment**,
- **Linux audit log collection** using auditd,
- **attack simulation** using Atomic Red Team,
- **API interaction** through Postman,
- and a **first AI-assisted alert analysis workflow** using n8n and Gemini.

At this stage, the project remains **under active development**. The implemented environment serves as the **foundation** for future enhancements, which may include:

- improved alert classification,
- deeper AI-driven investigation workflows,
- additional automation capabilities,
- and more advanced SOC response scenarios.

Thus, this deployment phase represents an essential milestone in the gradual construction of a more intelligent and automated cybersecurity monitoring platform.
Security Analysis


## Security Analysis

Initial deployment establishes a **baseline of normal system behavior**, enabling future:

* anomaly detection
* automated investigations
* intelligent incident response

---

## Project Status

🚧 **Active Development**

Planned improvements:

* Advanced AI investigation workflows
* Automated response playbooks
* Improved alert classification
* Extended SOC automation scenarios

---

## Author

**Achref Saidani**
Cybersecurity & AI Security Automation
