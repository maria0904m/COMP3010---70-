# COMP3010---70-
## 1.Introduction
Security Operations Centres (SOCs) are responsible for continuously monitoring, detecting, and responding to threats across an organisation’s infrastructure. “SOCs are tasked with monitoring and detecting threats across an organization's technology ecosystem“(Sysdig, 2024). This reflects that, SOC analysts must work with high volume, heterogeneous telemetry (network flows, endpoint logs, cloud audit trails) and turn this raw data into actionable incident investigations. 

Splunk serves as a common SIEM platform in such an environment. It enables various log sources to be searched and correlated with the use of its Search Processing Language (SPL). “A SIEM collects and organizes all the data coming from various sources within your network infrastructure … to find unusual behavior and malicious activity and stop it automatically” (Splunk, 2024).

This research will utilize the BOTSv3 dataset, provided by Splunk, which simulates various attacks against Frothly, a fictional brewing company. The BOTSv3 provides pre-indexed logs from AWS, Windows endpoints, S3 access logs, and various other supporting technologies that enable realistic SOC workflows without the overhead required to build a live lab.

**The objectives for this coursework are to:**
- Deploy Splunk Enterprise in an Ubuntu virtual machine and ingest the BOTSv3 dataset.
- Validate the availability of AWS and endpoint logs related to the investigation.
- Reflect on how these investigative steps map onto SOC roles, incident handling methodologies, and the wider cyber kill chain.
- Explain how each investigation in SPL represents a real world process for SOC: detect, escalate, and analyze.

**Scope and assumptions:**\
This exercise will focus only on the botsv3 index, using only the historical logs included in the dataset. No live data ingestion, forwarding, or external integrations were configured. All activities are performed from the perspective of a Tier 1 and 2 SOC analyst responding to alerts relating to AWS and endpoint behaviour. While BOTSv3 is an extremely realistic environment it remains a controlled dataset. Thus, findings should be interpreted with the understanding that the attacks are of predefined patterns, without the noise found in a real world setting, and do not represent evolving attacker behavior as in live environments.

This report documents the full investigative workflow, from setup through guided questions, and links findings back to real-world SOC practices.

---

## 2. SOC Roles and Incident Handling 
A SOC operates within several layers, each balancing alerts coming in with the need for deeper investigation. As stated by SANS Institute (2024), “tiering allows analysts to focus on the right level of analysis, reducing burnout while improving event accuracy.” (SANS Institute, 2024). Understanding what each tier does and how they support detection, escalation and response helps place the BOTSv3 analysis in a realistic SOC workflow.

However, tiering introduces operational challenges:
- Alert fatigue: Tier 1 analysts may ignore low-fidelity alerts, potentially missing early stages of sophisticated attacks
- Escalation gaps: If Tier 1 fails to escalate, incidents may go unresolved, delaying containment and increasing risk exposure.
- Coverage limitations: Resource constraints mean not all telemetry can be actively monitored, necessitating risk-based prioritization.

### Tier 1: Monitoring, alert validation and escalation
These analysts are continuously monitoring SIEM dashboards and checking alerts. Their focus is on rapid triage, not deep investigation. They confirm whether an alert is genuine, look at the basic user and device context, identify suspicious IAM or S3 activity, and compare events to determine if anything unusual needs to be escalated.
In the BOTSv3 scenario, Tier 1 would escalate:
- CloudTrail API calls executed without MFA (Q2), increased credential risk
- PutBucketAcl events indicating potential S3 misconfiguration (Q4-6)
- Endpoint anomalies such as a Windows host running unexpected OS edition (Q8)
This would include the Tier 1’s role in ensuring that meaningful events are detected early and accurately escalated. If escalation fails, SOC maturity suffers, as Tier 2 cannot perform timely root cause analysis.

---

### Tier 2: Investigation, multi source correlation and response
These analysts do deep investigations, whose work include correlation across identity logs, S3 access data, configuration changes and endpoint telemetry to find the root cause, impact and scope. “Analysts review the higher-priority security incidents escalated by triage specialists and do a more in-depth assessment using threat intelligence.”(Palo Alto Networks, 2024) 

In the BOTSv3 scenario Tier 2 responsibilities are:
- Enumerating IAM user activity to build identity baselines (Q1)
- Determine MFA enforcement gaps
- Tracing the lifecycle of an S3 public exposure incident

Tier 2 would also implement containment measures, revoke IAM keys, lock compromised roles, remove public S3 ACLs, or isolate non-compliant endpoints

---

### Tier 3: Threat hunting, SIEM engineering and strategic improvement
Tier 3 focuses on proactive defense and long term optimization. It tunes detection, builds automation and strengthens the cloud security posture. Tier 3 transforms incident knowledge into durable and enterprise wide security countermeasures.

“Proactively searches for suspicious behavior and tests and assesses network security to detect advanced threats and identify areas of vulnerability or insufficiently protected assets.” (CrowdStrike, 2024)

---

### NIST SP 800-61 Alignment
The BOTSv3 workflow aligns with the NIST incident handling model:

**Preparation:** installing Splunk, loading BOTSv3 and validating sourcetypes

**Detection and analysis:** IAM enumeration, MFA validation, misconfiguration investigation, endpoint deviation detection

**Containment and recovery:** Removing public ACLs, enforcing MFA, correcting OS inconsistencies

**Post incident activity:** better correlation searches, stronger governance, revised baselines

This investigation has shown how SOC analysts need to pivot across AWS identity logs, configuration changes, S3 access patterns, and endpoint telemetry to build a coherent incident narrative. It underlines the importance of SPL proficiency, cloud misconfiguration awareness, and understanding attacker behaviour, core competencies required in a professional cloud-centric SOC operating at an outstanding level.

---

## 3. Installation and Data Preparation
### 3.1. Splunk Setup
Splunk Enterprise was installed on the Ubuntu virtual machine using the official .deb package, found on their website. Installation was completed vid dpkg, followed by acctepting the licence agreement and creating an account./>
<img width="1126" height="631" alt="image" src="https://github.com/user-attachments/assets/2cb720e9-e0ea-4199-b4a4-4feb6c7b7908" /> 
After successful installation, Splunk is started using sudo/opt/splunk/bin/splunk start. The Splunk web interface became available at the link in the terminal, in which I needed to create an account. I then open the section ‘Search and Reporting’ /> 
<img width="1117" height="631" alt="image" src="https://github.com/user-attachments/assets/04e172d2-c732-4aa1-aadb-0b4bcb0eaf77" />

---

### 3.2. BOTSv3 Dataset Ingestion
The BOTSv3 dataset was downloaded from the BOTSv3 GitHub repository. Sine the dataset is packaged as pre-indexed application, ingestion required only the following steps:
- Moving the btosv3.tgz into the splunk directory
- Extracting the archive using tar to unpack
- Restarting Splunk so the application and its indexes can load

After the restart the index botsv3 and its sourcetypes were available right away. There was no manual parsing required, no setup and no data configuration. That is reflective of the fact that most enterprise SOC actually deploy log ingestion via Splunk apps rather through manually engineered ingestion, which let the analysts focus on investigation, not engineering. />
<img width="1122" height="632" alt="image" src="https://github.com/user-attachments/assets/a8887d26-6578-4da9-ba99-bf79d6c4b24a" />

"Effective data ingestion ensures that all relevant data from disparate sources is collected, centralized, and made available for analysis in near real-time. This is critical for organizations that need to act quickly on insights, detect anomalies, and respond to incidents efficiently" (Airbyte, 2025). 

---

### 3.3. Data Validation
To make sure the dataset had loaded successfully, I did a verification search: index=botsv3 earliest=0. Splunk returned over 100,000 events, confirming the index was active and populated. Next, I opened several of these results in Raw view to confirm that critical fields like host, index, sourcetype, were properly parsed. The sourcetype list in the left panel indicated the availability of major telemetry sources expected to be relevant to this investigation.  “Data integrity is the assurance that an organization’s data is accurate, complete and consistent at any point in its lifecycle. Organizations rely on clean data for decision‑making, predicting consumer behavior, assessing market trends and securing against data breaches.” (IBM,2024) />

<img width="1123" height="632" alt="image" src="https://github.com/user-attachments/assets/f8ca5c3a-10a7-418d-beda-080a6face47f" />

From the perspective of a SOC, this validation shows the ‘preparation’ stage of incident handling, making sure the telemetry is complete, correctly indexed before any investigative work is done. A SOC analyst cannot draw any reliable conclusions if the data pipeline is incomplete or misconfigured, so confirmation of data quality is an essential step in real world investigations.

---

### 3.4. Justification of Setup
Using a Splunk VM has several advantages regarding real SOC investigation practices: 
- Full administrative control to carry out restarts, configuration changes and troubleshooting
- Isolation from production systems, this provides a safe area for conducting analysis
- Realistic SIEM workflow, normalised logs, and the analyst will focus on detection, investigation, interpretation only.

This setup reflects accurately how mature SOC environments work: logs come centralized, structured and ready for analysis, so the analysts can focus their resources on detection, triage and investigation.

---

## 4. Guided Questions
### 4.1. Question 1
I examined the CloudTrail logs to find which IAM users had interacted with the AWS services. I extracted all unique IAM usernames present in the API activity. I sorted all the username fields alphabetically. This gave a list of all unique identities, successful and failed calls included. This is the foundation for detecting impersonation, privilege misuse or anomalous behaviour. />

<img width="1122" height="632" alt="image" src="https://github.com/user-attachments/assets/eb148dfa-2abf-4aee-8440-2d40cd9c4622" />

<img width="1122" height="631" alt="image" src="https://github.com/user-attachments/assets/c133dba7-c469-40ef-8912-1ea3bb94aa24" />

**SOC Analysis**\
IAM user activity enumeration is a crucial part of cloud investigations, establishing which identities are active, distinguishing human users from service accounts, and whether any unusual accounts appear. Usually, IAM enumeration is the first step in cloud incident analysis, because attackers more often exploit weak identity hygiene,inactive accounts or excessive permissions.

---

### 4.2. Question 2 - MFA 
Firstly, to find out how MFA is represented in the dataset, I needed to review CloudTrail’s identity context fields. I checked all the fields on the left hand side of Splunk. By scrolling through the fields, I found: userIdentity.sessionContext.attributes.mfaAuthenticated. This is a boolean field that directly indicates if MFA was used during the API call.  />

<img width="1123" height="632" alt="image" src="https://github.com/user-attachments/assets/a0da3764-c10c-457d-a556-ed576dc2845f" />

**SOC Analysis**\
Monitoring the use of MFA is critical in detecting compromised accounts and ensuring cloud security best practices. Considering AWS credentials can easily be stolen or reused, MFA acts as a barrier preventing unauthorised access.

---

### 4.3. Question 3 - Processor models
To identify the type of the processor model used, I investigated the dataset for sourcetypes that contain hardware information. I found: hardware. I ran the query and this returned three events. I located the CPU model field, which enabled me to identify the type of the processor used. />

<img width="1121" height="631" alt="image" src="https://github.com/user-attachments/assets/1bebcba8-4a75-430f-9e73-dd8d75bbb73f" />

**SOC Analysis**\
Hardware profiling is often an overlooked, but very strategic task in a SOC. CPU families and models can impact vulnerability exposure, performance and attack surface. Unexpected hardware configurations may signal misconfigured infrastructure or compromised hosts.

---

### 4.4. Questions 4-6
I analyzed the CloudTrail events to determine how an S3 bucket became publicly accessible. Sorting these chronologically allowed me to find the first PutBucketAcl event, which represented the misconfiguration. From the raw JSON details, I looked for three key elements: event ID (taken directly from the eventID field), IAM User (taken from the userIdentity.userName field), bucket name (taken from requestParameters.bucketName). />

<img width="1123" height="632" alt="image" src="https://github.com/user-attachments/assets/af5122bf-dd20-400a-bb6a-021d1cfaec0b" />

<img width="1123" height="633" alt="image" src="https://github.com/user-attachments/assets/5fff297d-13b7-4856-83df-de344c681568" />

<img width="1123" height="632" alt="image" src="https://github.com/user-attachments/assets/87964083-2ad2-4795-a2c5-a58188104d55" />

**SOC Analysis**\
Misconfigured S3 ACLs are one of the most common cloud security failures, often leads to accidental data exposure. CloudTrail logs can provide evidence of:
- Who made the IAM permission changes
- What was changed
- Event timestamp
- How the configuration was altered

---

### 4.5. Question 7
To assess what activity happened during the time the bucket had been expose, I inspected S3 access logs. The initial wide search  has returned a large number of events, I refined the analysis to focus on .txt uploads. This has reduced the dataset to a small number of events, from which I found the uploaded file by looking at the request path. />

<img width="1126" height="632" alt="image" src="https://github.com/user-attachments/assets/79ed32e6-d848-411a-a8ca-05b78640b4c1" />

**SOC Analysis**\
This reflects standard SOC procedure for determining the impact of storage misconfigurations. Analysts must verify whether the attacker uploaded malicious files, exfiltrated data, or probed the exposed bucket. Filtering for specific file types reflects targeted forensics that would be used in cloud breach investigations.

---

### 4.6. Question 8 
I used endpoint telemetry from the sourcetype ‘winhostmon’ to create a deduplicated list of the operating system version per host. That showed the existence of a single host running on a different version of Windows, and for the user I was requested, identifying the anomalous machine within the environment. /> 

<img width="1126" height="632" alt="image" src="https://github.com/user-attachments/assets/a04ed474-7954-42ff-b095-34e29f810d69" />


**SOC Analysis**\
Operating systems inconsistencies may indicate build drift, unmanaged devices or compromised endpoints. Sometimes, malicious actors drop rogue systems or change the OS configuration to maintain persistence. The detection of deviations from an expected baseline is, therefore, important for endpoint security.

---

## 5. Conclusion
These exercises have ensured comprehensive SOC level analysis in Splunk using the BOTSv3 dataset regarding AWS identity behaviour, cloud misconfiguration, S3 access patterns and endpoint telemetry. 

**Key lessons learned:**\
- CloudTrail is main forensic source in understanding identity centred security problems
- MFA enforcement for API calls is critical to mitigate credential attacks
- S3 bucket misconfigurations are a leading category of cloud security failures
- Cross dataset correlation is important for determining attack scope and impact.
- Baseline deviations in endpoint OS can indicate early misconfigurations or usage of unauthorized systems.

**Forward looking recommandations:**\
- Implement automated detection using Splunk for events that grant public access
- MFA to be enforced for all IAM users and API call level
- Apply AWS block public access settings
- Standardise endpoint OS build and enable deviation alerts
- Integrate regular IAM privilege audits

This investigation demonstrates how SOC analysts transform raw telemetry into actionable security intelligence, blending technical evidence with operational context. Structured analysis supported by evidence and SOC aligned reflection develop both technical and operational competencies required in real world incident handling

---

## Appendix
### 1. Question 1
**SOC Relevance**\
Understanding the IAM identities is a foundation for:
- Attribution of suspicious API calls
- Privilege management
- Identifying inactive or suspicious accounts
- Detecting compromised credentials 
This further reinforces IAM as a core element for cloud incident detection and response.

### 2. Question 2
**SOC Relevance**\
Failure to monitor MFA status is one of the common root causes of major cloud breaches. “There are over 300 million fraudulent sign-in attempts to our cloud services every day. 
Cyberattacks aren’t slowing down, and it’s worth noting that many attacks have been successful without the use of advanced technology.” (Microsoft, 2019)

MFA status allows SOC to:
- Build alerts for API calls executed without MFA
- Identify violations of company MFA policies
- Improve IAM baselines, using enforced MFA
- Stolen credential detection

### 3. Question 3
**SOC Relevance**\
Understanding the hardware baselines supports:
- Asset compliance and provisioning policies
- Vulnerability management for hardware specific flows
- Detection of unauthorised hardware.

### 4. Questions 4 - 6
**SOC Relevance**\
“Every leaky S3 bucket is a potential compliance violation, source of sensitive data for attackers, gateway to a larger cloud data breach” (CyberlAngel, 2024) 

Understanding and detecting S3 ACL changes is important for:
- Detection of misconfigurations and rapid containment
- Preventing accidental leaks
- IAM governance improvement (reviewing user access management)
- Maintaining strong cloud security posture
Precise visibility into cloud misconfiguration events helps to avoid unintended data exposure and to respond when risky permissions are introduced. 

### 5. Question 7
**SOC Relevance**\
Understanding what files were uploaded is crucial for:
- Assessing whether the incident is benign or malicious
- Determine whether attackers planted scripts, malware or phishing files
- Initiating containment or clean up actions
- Preserving forensic evidence for escalation
Access logs in S3 are a big part of cloud threat analysis, especially when public exposure is suspected

6. Question 8
**SOC Relevance**\
Monitoring of endpoint baseline is important because:
- Compliance, making sure that all hosts are adhering to approved build standards
- Detection of rogue devices
- Vulnerability management prioritization
- Maintaining consistent hardening positions
Identification of the single host running on a different OS indicates how small variances can point to larger security problems.
