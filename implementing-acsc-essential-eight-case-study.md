# From Assessment to Remediation: Implementing the ACSC Essential Eight in a Real Environment

---

In August 2024, I initiated and completed a comprehensive assessment against the Australian Cyber Security Centre Essential Eight framework to identify security gaps and improvement opportunities.

Since then, I have been leading in ongoing remediation efforts to strengthen security controls across. This project has provided interesting practical experience for me.

## Environment and Scope of Assessment

- Medium-sized organization
- Target: Maturity level 1
- Microsoft 365 cloud-first
- On-prem network devices
- Third-party applications
- Microsoft Intune endpoint management

The Australian Signals Directorate's Essential Eight is a set of eight baseline mitigation strategies designed to protect Windows environments against cyber threats. However, in this article I am not focusing about Windows platform only because, I wanted to take this opportunity to improve this organisation cybersecurity posture in overall, and wanted to initiate the cybersecurity standard for them and my team members who also were the outsource staffs. I found that their system can be improved. If you follow this article to the end, you will see a lot of Information security policy and documents were created for the client.

Background, at the time, I was both a system administrator and security (outsource), I started with researching information from ASD website below:
https://www.cyber.gov.au/business-government/asds-cyber-security-frameworks/essential-eight

I went through the assessment process guide, accumulate the assessment guidance information to **create a comprehensive checklist (excel file)**. Since, I started looking after this organisation in mid 2024, I came across that they did not have asset and inventory register, therefore, I **initiated the simple asset management process, and created the asset register** using Microsoft List which is an app in MS365. I have spent a couple of months to collect necessary asset details. Additionally, I **create the list of standard user accounts and privilege users** in MS365, Network devices, and third-party web application that the organisation have used.

So, the followings are the list of documents that I developed by myself by researching and utilising resources in the internet.

- Information Security Policy
- Identity Access Review Procedure
- Privilege Access Request Management document
- IT Disaster Recovery Plan
- Cyber Incident Response Plan
- End user Security Policy

I conducted the assessment on all in-scoped systems by

- Reviewing the existence of processes,
- Reviewing the effectiveness of controls, configuration
- Reviewing relevant documents and reports,
- Vulnerability scanning
- Interviewing and surveys.
- etc.

Any assessment evidences, limitataions, screenshots and constraints on testing were documented within the finding summary report. I sent the detaild finding report with executive summary to the client. The followings are the implementation challenges that I want to keep and share as valuable experience. Hope, it might be useful.

> _**Please note that you have to obtain management approval before changing any configuration or implementations below.**_

---

## Case study 1 - Multi-factor authentication issue

I found that MFA is not enforced for all some users. I created conditional access policy to enforce all users to be required the MFA.

| |
|---|
| ![MFA Conditional Access Policy](https://github.com/Penguinsecq/penguinsecq.github.io/raw/main/docs/images/cap-mfa1.png) |

*Figure 1 - Example page of Conditional Access Policy*

However, a couple of users still need to be excluded because of a requirement and a bit technical constraints. To mitigate the risk of not having MFA enabled for the accounts, after excluding them from the MFA policy. The followings condition could be applied depending on your environment:

**Mitigation**

- Device compliant
- Location filtering such as ip address
- Device ID filtering but device registering is required

| |
|---|
| ![Device filter in Conditional access policy](https://github.com/Penguinsecq/penguinsecq.github.io/raw/main/docs/images/cap-device-filter.png) |

*Figure 2 - More filter in conditional access policy*

With the filterings above, risk is mitigated.

---

## Case study 2 - Restrict Admin Privilege issue

There is no documented and approved list of privileged accounts, processes and procedures that outline the requirements for provisioning privileged accounts. Also, a privileged MS365 users have accessed some users' workstation, also there is a usage of privileged user to do daily routine operation.

This is why the asset register, third-party application and user accounts list are required at the beginning of the assessment, you need to know

- How many system (Windows, Network devices, Applications
- How many User accounts both standard and privileged,
- Where the privileged user to be used,
- which systems got accessed by the privileged user (I will talk about this in next article)

**Implementation**

- Create Privilege Access Request Procedure and document.
- I used Microsoft Forms to create a Privilege access request form. Actually, you can utilise the Privilege Access Management in MS365 for this issue if you have a required license (Microsoft 365 E5 (no Teams), Office 365 E5, Microsoft 365 F5).
- Remove all local administrator user.
- After reviewing, remove unnecessary MS365 administrative users.
- Create new user for Intune enrollment only, not Global admin anymore.
- Create policy to prohibit MS365 global admin user to access or logon to any Entra joined workstation

| |
|---|
| ![Privilege Access Management process](https://github.com/Penguinsecq/penguinsecq.github.io/raw/main/docs/images/pam-procedures.png) |

*Figure 3 - Privilege Access Request Procedures flow.*

---

## Case study 3 - Application Control issue (In progress)

Application control is a security approach designed to protect against malicious code executing on systems. When this security approach is implemented, it ensures only approved code such as executables, software libraries, scripts, installers, and drivers is authorized to execute.

This section is the most challenging for me to accomplish the Essential 8 Maturity Level 1, even though I have a implementation plan because client's environment always is different. You have to be very careful about this section because the application control can crash the operating system if necessary "exe", or "dll" files cannot be executed. Let's say your client has many branch in the middle of nowhere, you deploy the application control without solid testing, and end users encounter with the BOD.

**Implementation**

From Microsoft, [https://learn.microsoft.com/en-us/compliance/anz/e8-app-control#application-control-for-windows](https://learn.microsoft.com/en-us/compliance/anz/e8-app-control#application-control-for-windows). While WDAC is preferred, it can be simpler and easier for most organizations to achieve ML1 using just AppLocker as a starting point, both solutions are complimentary.

So, in this article, I am sharing how to implement "AppLocker" with MS Intune. You can find the comparison between WDAC and AppLocker to help you decide which one is appropriate for your environment here. [Application Control in practice](https://github.com/Penguinsecq/penguinsecq.github.io/blob/main/docs/acsc-essential-eight/application-control-in-practice.md)

1. Gathering required applications list as much as possible in your environment.
2. Create your AppLocker control policies based list above. You can utilise the starter policy from NSA cyber here [NSA AppLocker Guidance (coming soon)!!!](https://github.com/nsacyber/AppLocker-Guidance)
3. Deploy the policies in **Audit only** mode. You may need 2 versions for Windows 10 and 11. The deployment details is here. [AppLocker Deployment details (coming soon)!!!](https://github.com/Penguinsecq/penguinsecq.github.io/blob/main/docs/acsc-essential-eight/applocker-deployment-steps.md)
4. Monitor Event Logs for a while, depends on your environment, but I would suggest more than 4 weeks, if your business has application will be running monthly or quarterly only.
Check Event Viewer > Applications and Services Logs > Microsoft > Windows > AppLocker for Event ID 8003. You can find the important event id related to AppLocker here.
[Using Event Viewer with AppLocker](https://learn.microsoft.com/pl-pl/windows/security/application-security/application-control/app-control-for-business/applocker/using-event-viewer-with-applocker)
5. Tuning AppLocker control policies. Update AppLocker rules to include any necessary exceptions or allow rules for trusted apps, vendors, or paths.
6. Test on Pilot Devices: Apply refined policies to a small group of users or machines to confirm no legitimate software is impacted.
7. Repeat Audit Cycle (step 4-6).
8. Inform staffs of the upcoming changes, and provide a process for requesting access to blocked apps.
9. Once confident in the rules, gradually move from audit to enforced mode using staged deployment.
10. Export and save a copy of your final AppLocker policies for rollback or documentation.
11. Enable Enforcement mode.

| |
|---|
| ![AppLocker Implmentation diagram](https://github.com/Penguinsecq/penguinsecq.github.io/raw/main/docs/images/applocker-implement-diagram.png) |

*Figure 4 - AppLocker Implmentation diagram*
