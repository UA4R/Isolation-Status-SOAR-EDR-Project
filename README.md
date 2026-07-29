# Isolation Status; SOAR EDR Project

## Objectives
Spin up a working **SOAR** playbook that takes an **EDR** detection for credential dumping, enriches it and notifies the right channels automatically, then lets an analyst make the containment call with one click. It's built with an explicit understanding of the detection blind spots. 

This Project is mainly meant to show what happens post exploitation; after an attacker has access to an infrastructure and what analyst are meant to do to mitigate such threats, in this case; Credential Dumping.

### Skills Learnt
- SOAR Automation= Built a multi-stage Tines playbook that chains detection, enrichment, notification and conditional response with a {human in the loop} containment decision.
- Detection Engineering= Adapted an existing community D&R rule in LimaCharlie to match LaZagne's real process telemetry.  
- Threat mapping= Mapped a real credential dumping tool (LaZagne) to MITRE ATT&CK (T1003) and the NIST SP 800-61 incident response lifecycle.
- Cloud Infrastructure= Provisioned and secured an AWS EC2 Windows endpoint with IP restricted access for use as a controlled attack lab.
- API/Tool Integration= Connected four independent tools (EDR, SOAR, Slack, email) via webhooks and REST APIs, including scoped credential handling.

### Tools
| Tool | Purpose |
|---|---|
| ![AWS EC2](https://img.shields.io/badge/AWS-EC2-FF9900?style=flat&logo=amazonaws&logoColor=white) | Cloud infrastructure hosting the Windows victim endpoint |
| ![Windows Server](https://img.shields.io/badge/Windows_Server-2022-0078D6?style=flat&logo=windows&logoColor=white) | Base OS for the target endpoint |
| ![LimaCharlie](https://img.shields.io/badge/LimaCharlie-EDR-1E2A38?style=flat&logoColor=white) | EDR agent, telemetry collection, custom detection rule, sensor isolation |
| ![Tines](https://img.shields.io/badge/Tines-SOAR-000000?style=flat&logoColor=white) | Workflow orchestration; enrichment, notification, (human in the loop) response |
| ![Slack](https://img.shields.io/badge/Slack-Notifications-4A154B?style=flat&logo=slack&logoColor=white) | Analyst facing alert delivery and final isolation status |
| ![xsquare](https://img.shields.io/badge/xsquare-Dummy_Email-6C757D?style=flat&logoColor=white) | Disposable inbox for alert email delivery |
| ![LaZagne](https://img.shields.io/badge/LaZagne-HackTool-8B0000?style=flat&logoColor=white) | Simulated credential dumping tool detonated on the target |

### Frameworks
| Framework | Applied as |
|---|---|
| ![MITRE ATT&CK](https://img.shields.io/badge/MITRE-ATT%26CK-A32638?style=flat&logoColor=white) | Attack technique classification (T1003; OS Credential Dumping) |
| ![NIST SP 800-61](https://img.shields.io/badge/NIST-SP_800--61-1B3A57?style=flat&logoColor=white) | Incident response lifecycle mapping |

### MITRE ATT&CK mapping
| ATT&CK element | What applies here |
|---|---|
| Tactic | Credential Access |
| Technique | T1003; OS Credential Dumping |
| Sub-technique | T1003.001; LSASS Memory, T1003.005; Cached Domain Credentials |
| Evidence | New process creation event for `lazagne.exe` with matching command line, captured by the LimaCharlie sensor and matched by the custom D&R rule |

### NIST SP 800-61 mapping
| NIST 800-61 phase | What applies here |
|---|---|
| Preparation | EC2 provisioning, LimaCharlie sensor deployment, D&R rule creation, Tines story build, Slack/xsquare integration setup |
| Detection & Analysis | LimaCharlie fires the D&R rule on LaZagne execution; Tines enriches the raw detection into the 8 key fields and routes it to Slack and email |
| Containment | Tines prompts the analyst for an isolation decision; on approval, LimaCharlie isolates the sensor via API and confirms status |
| Eradication & Recovery | Intentionally out of scope, isolation buys time for manual re image/credential rotation |
| Post-Incident Activity | Final Slack status message serves as project evidence; README documents lessons learned and known detection gaps |

### Security group / port configuration
| Instance | Port | Application | Reason to keep open |
|---|---|---|---|
| Windows target | 3389 | RDP | Admin access to configure the instance and detonate LaZagne |
| Windows target | 443 (outbound) | LimaCharlie sensor comms | Required for the sensor to reach LimaCharlie's cloud and receive isolation commands |

### Steps
### 1)Preparation
- So for this project we are spinning up a single instance infrastructure on AWS. Principle of least privilege was applied by configuring all ports accordingly. The instance name is soar-edr-victim. The instance was launched in IAM, not Root. The AMI we are using this time is windows server 2022 base. 
![image_alt](https://github.com/UA4R/Isolation-Status-SOAR-EDR-Project/blob/main/Project%203%20Screenshots/1.png)

- For the next step, I logged into my windows Remote Desktop Protocol using my host machine and provided the necessary credential to access my vm windows UI.
![image_alt](https://github.com/UA4R/Isolation-Status-SOAR-EDR-Project/blob/main/Project%203%20Screenshots/2.png)

- So for the next step, I disabled Windows Defender real time protection which is needed so LaZagne (credential dump) doesn't get quarantined the moment I detonate it later on my VM. In real time production environment, you should never do this. 
![image_alt](https://github.com/UA4R/Isolation-Status-SOAR-EDR-Project/blob/main/Project%203%20Screenshots/3.png)

- For the next step, I deployed LimaCharlie on my host machine and connected the windows agent from host LimaCharlie to my VM.
<p align="center">
  <img src="https://raw.githubusercontent.com/UA4R/Isolation-Status-SOAR-EDR-Project/main/Project%203%20Screenshots/4.png" width="49%" />
  <img src="https://raw.githubusercontent.com/UA4R/Isolation-Status-SOAR-EDR-Project/main/Project%203%20Screenshots/5.png" width="49%" />
</p>

- So for the next step, I detonated LaZagne to my VM which is a form of post exploitation technique mimicking real world credential dump.
<p align="center">
  <img src="https://raw.githubusercontent.com/UA4R/Isolation-Status-SOAR-EDR-Project/main/Project%203%20Screenshots/6.png" width="49%" />
  <img src="https://raw.githubusercontent.com/UA4R/Isolation-Status-SOAR-EDR-Project/main/Project%203%20Screenshots/7.png" width="49%" />
</p>
- For the next step, I built a custom D&R rule on LimaCharlie. Then, I re-detonated the attack again on my VM to see in my LimaCharlie if the detection was gotten.
<p align="center">
  <img src="https://raw.githubusercontent.com/UA4R/Isolation-Status-SOAR-EDR-Project/main/Project%203%20Screenshots/8.png" width="49%" />
  <img src="https://raw.githubusercontent.com/UA4R/Isolation-Status-SOAR-EDR-Project/main/Project%203%20Screenshots/9.png" width="49%" />
</p>
- For the next step, I connected LimaCharlie to Tines throught webhook configuration and tested again to see if the results came through to tines
<p align="center">
  <img src="https://raw.githubusercontent.com/UA4R/Isolation-Status-SOAR-EDR-Project/main/Project%203%20Screenshots/10.png" width="49%" />
  <img src="https://raw.githubusercontent.com/UA4R/Isolation-Status-SOAR-EDR-Project/main/Project%203%20Screenshots/11.png" width="49%" />
</p>

- For the next step, I configured Slack to tines and tested the connection to see if it went through. Time may look distorted, but one can use the epoch converter for appropriate time approximation.
<p align="center">
  <img src="https://raw.githubusercontent.com/UA4R/Isolation-Status-SOAR-EDR-Project/main/Project%203%20Screenshots/12.png" width="49%" />
  <img src="https://raw.githubusercontent.com/UA4R/Isolation-Status-SOAR-EDR-Project/main/Project%203%20Screenshots/13.png" width="49%" />
</p>

- For the next step, I configured email to tines and tested if it sent.
![image_alt](https://github.com/UA4R/Isolation-Status-SOAR-EDR-Project/blob/main/Project%203%20Screenshots/14-edit.png)

- For the next step, I added a page and the rest of the configurations.
<p align="center">
  <img src="https://raw.githubusercontent.com/UA4R/Isolation-Status-SOAR-EDR-Project/main/Project%203%20Screenshots/15.png" width="49%" />
  <img src="https://raw.githubusercontent.com/UA4R/Isolation-Status-SOAR-EDR-Project/main/Project%203%20Screenshots/16.png" width="49%" />
</p>

### 2)Detection & Analysis
- So for this, I re-run the whole thing from initiating the attack to detecting and it was a huge success. 
![image_alt](https://github.com/UA4R/Isolation-Status-SOAR-EDR-Project/blob/main/Project%203%20Screenshots/17.png)

### 3)Containment, Eradication & Recovery
- The credential Dump was detected and the machine was successfully isolated.
<p align="center">
  <img src="https://raw.githubusercontent.com/UA4R/Isolation-Status-SOAR-EDR-Project/main/Project%203%20Screenshots/18.png" width="49%" />
  <img src="https://raw.githubusercontent.com/UA4R/Isolation-Status-SOAR-EDR-Project/main/Project%203%20Screenshots/19.png" width="49%" />
</p>

### 4)Post Incident Activity
### Lessons Learnt
-  Credential Dump are a real hazard when it comes to cybersecurity risks and proper defence should be put in place to avoid it like the demonstrated isolation technique
-  Attackers gaining access to a system post-exploit signals a weak security infrastructure, so proper security posture should be put in place to avoid this
-  To avoid the passwords stolen from being used, credit rotation should be put in place as a safety net and the isolated machine should be investigated thoroughly.
-  For D&R rule, attackers could use a different binary meaning signature based rule couldn't detect but a behavioural based rule on the LSASS memory would be more efficient and if paired together would make a stronger D&R rule and overall security posture. 


### Conclusion
This project demonstrates how real life SOC Analyst operations take place and how post exploitation protocols are followed in such incidents. Finally as a project wrap up, the instance was immediately terminated successfully for cost efficiency in terms of the cloud environment.
![image_alt](https://github.com/UA4R/Isolation-Status-SOAR-EDR-Project/blob/main/Project%203%20Screenshots/20.png)
