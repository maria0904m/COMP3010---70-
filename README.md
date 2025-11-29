# COMP3010---70-

## 4. Guided Questions
### 4.1. Question 1
I examined the CloudTrail logs to find which IAM users had interacted with the AWS services. I extracted all unique IAM usernames present in the API activity. I sorted all the username fields alphabetically. This gave a list of all unique identities, successful and failed calls included. This is the foundation for detecting impersonation, privilege misuse or anomalous behaviour

**SOC Analysis**\
IAM user activity enumeration is a crucial part of cloud investigations, establishing which identities are active, distinguishing human users from service accounts, and whether any unusual accounts appear. Usually, IAM enumeration is the first step in cloud incident analysis, because attackers more often exploit weak identity hygiene,inactive accounts or excessive permissions.

**SOC Relevance**\
Understanding the IAM identities is a foundation for:
- Attribution of suspicious API calls
- Privileage management
- Identifying inactive or suspicious accounts
- Detecting compromised credentials 
This further reinforces IAM as a core element for cloud incident detection and response.

---

### 4.2. Question 2 - MFA 
Firstly, to find out how MFA is represented in the dataset, I needed to review CloudTrail’s identity context fields. I checked all the fields on the left hand side of Splunk. By scrolling through the fields, I found: userIdentity.sessionContext.attributes.mfaAuthenticated. This is a boolean field that directly indicates if MFA was used during the API call.

**SOC Analysis**\
Monitoring the use of MFA is critical in detecting compromised accounts and ensuring cloud security best practices. Considering AWS credentials can easily be stolen or reused, MFA acts as a barrier preventing unauthorised access.

MFA status allows SOC to:
- Build alerts for API calls executed without MFA
- Identify violations of company MFA policies
- Improve IAM baselines, using enforced MFA
- Stolen credential detection


**SOC Relevance**\
Failure to monitor MFA status is one of the common root causes of major cloud breaches.

---

### 4.3. Question 3 - Processor models
To identify the type of the processor model used, I investigated the dataset for sourcetypes that contain hardware information. I found: hardware. I ran the query and this returned three events. I located the CPU model field, which enabled me to identify the type of the processor used.

**SOC Analysis**\
Hardware profiling is often an overlooked, but very strategic task in a SOC. CPU families and models can impact vulnerability exposure, performance and attack surface. Unexpected hardware configurations may signal misconfigured infrastructure or compromised hosts.

**SOC Relevance**\
Understanding the hardware baselines supports:
- Asset compliance and provisioning policies
- Vulnerability management for hardware specific flows
- Detection of unauthorised hardware.

---

### 4.4. Questions 4-6
I analyzed the CloudTrail events to determine how an S3 bucket became publicly accessible. Sorting these chronologically allowed me to find the first PutBucketAcl event, which represented the misconfiguration. From the raw JSON details, I looked for three key elements: event ID (taken directly from the eventID field), IAM User (taken from the userIdentity.userName field), bucket name (taken from requestParameters.bucketName).

**SOC Analysis**\
Misconfigured S3 ACLs are one of the most common cloud security failures, often leads to accidental data exposure. CloudTrail logs can provide evidence of:
- Who made the IAM permission changes
- What was changed
- Event timestamp
- How the configuration was altered

**SOC Relevance**\
Understanding and detecting S3 ACL changes is important for:
- Detection of misconfigurations and rapid containment
- Preventing accidental leaks
- IAM governance improvement (reviewing user access management)
- Maintaining strong cloud security posture
Precise visibility into cloud misconfiguration events helps to avoid unintended data exposure and to respond when risky permissions are introduced.

---

### 4.5. Question 7
To assess what activity happened during the time the bucket had been expose, I inspected S3 access logs. The initial wide search  has returned a large number of events, I refined the analysis to focus on .txt uploads. This has reduced the dataset to a small number of events, from which I found the uploaded file by looking at the request path.

**SOC Analysis**\
This reflects standard SOC procedure for determining the impact of storage misconfigurations. Analysts must verify whether the attacker uploaded malicious files, exfiltrated data, or probed the exposed bucket. Filtering for specific file types reflects targeted forensics that would be used in cloud breach investigations.

**SOC Relevance**\
Understanding what files were uploaded is crucial for:
- Assessing whether the incident is benign or malicious
- Determine whether attackers planted scripts, malware or phishing files
- Initiating containment or clean up actions
- Preserving forensic evidence for escalation
Access logs in S3 are a big part of cloud threat analysis, especially when public exposure is suspected.

---

### 4.6. Question 8 - 
I used endpoint telemetry from the sourcetype ‘winhostmon’ to create a deduplicated list of the operating system version per host. That showed the existence of a single host running on a different version of Windows, and for the user I was requested, identifying the anomalous machine within the environment

**SOC Analysis**\
Operating systems inconsistencies may indicate build drift, unmanaged devices or compromised endpoints. Sometimes, malicious actors drop rogue systems or change the OS configuration to maintain persistence. The detection of deviations from an expected baseline is, therefore, important for endpoint security.

**SOC Relevance**\
Monitoring of endpoint baseline is important because:
- Compliance, making sure that all hosts are adhering to approved build standards
- Detection of rogue devices
- Vulnerability management prioritization
- Maintaining consistent hardening positions
Identification of the single host running on a different OS indicates how small variances can point to larger security problems.

