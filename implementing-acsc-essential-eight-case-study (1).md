# From Assessment to Remediation: Implementing the ACSC Essential Eight in a Real Environment

---

In August 2024, I initiated and completed a comprehensive assessment against the Australian Cyber Security Centre Essential Eight framework to identify security gaps and improvement opportunities.

Since then, I have been leading ongoing remediation efforts to strengthen security controls across the environment. This project has provided interesting practical experience for me.

## Environment and Scope of Assessment

- Medium-sized organization
- Target: Maturity Level 1
- Microsoft 365 cloud-first
- On-prem network devices
- Third-party applications
- Microsoft Intune endpoint management

## Background

The Australian Signals Directorate's Essential Eight is a set of eight baseline mitigation strategies designed to protect Windows environments against cyber threats. However, in this article I am not focusing on the Windows platform only, because I wanted to take this opportunity to improve this organisation's cybersecurity posture overall, and to initiate a cybersecurity standard for them and my team members who were also outsourced staff. I found that their systems could be improved. If you follow this article to the end, you will see that a lot of information security policies and documents were created for the client.

At the time, I was both a system administrator and security consultant (outsource). I started by researching information from the ASD website:

https://www.cyber.gov.au/business-government/asds-cyber-security-frameworks/essential-eight

I went through the assessment process guide and accumulated the assessment guidance information to **create a comprehensive checklist (Excel file)**. Since I started looking after this organisation in mid-2024, I found that they did not have an asset and inventory register. Therefore, I **initiated a simple asset management process and created an asset register** using Microsoft Lists, an app in MS365. I spent a couple of months collecting the necessary asset details. Additionally, I **created a list of standard user accounts and privileged users** in MS365, network devices, and third-party web applications that the organisation used.

The following documents were developed by myself through research and utilising resources on the internet:

- Information Security Policy
- Identity Access Review Procedure
- Privilege Access Request Management document
- IT Disaster Recovery Plan
- Cyber Incident Response Plan
- End User Security Policy

## Assessment Approach

I conducted the assessment on all in-scope systems by:

- Reviewing the existence of processes
- Reviewing the effectiveness of controls and configurations
- Reviewing relevant documents and reports
- Vulnerability scanning
- Interviewing and surveys

All assessment evidence, limitations, screenshots, and constraints on testing were documented within the finding summary report. I sent the detailed finding report with an executive summary to the client.

The following are the implementation challenges that I want to keep and share as valuable experience. I hope it might be useful.

> **Please note that you must obtain management approval before changing any configuration or implementations below.**

---

## Case Study 1 — Multi-Factor Authentication Issue

I found that MFA was not enforced for some users. I created a Conditional Access policy to require MFA for all users.

| |
|---|
| ![MFA Conditional Access Policy](https://github.com/Penguinsecq/penguinsecq.github.io/raw/main/docs/images/cap-mfa1.png) |

*Figure 1 - Example page of Conditional Access Policy*

However, a couple of users still needed to be excluded due to requirements and technical constraints. To mitigate the risk of not having MFA enabled for those accounts, the following conditions could be applied depending on your environment:

**Mitigation**

- Device compliant
- Location filtering such as IP address
- Device ID filtering (device registration required)

| |
|---|
| ![Device filter in Conditional Access Policy](https://github.com/Penguinsecq/penguinsecq.github.io/raw/main/docs/images/cap-device-filter.png) |

*Figure 2 - More filter in Conditional Access Policy*

With the filterings above, risk is mitigated.

---

## Case Study 2 — Restrict Admin Privilege Issue

There was no documented and approved list of privileged accounts, processes, or procedures that outlined the requirements for provisioning privileged accounts. Additionally, privileged MS365 users had accessed some users' workstations, and privileged accounts were being used for daily routine operations.

This is why the asset register, third-party application list, and user accounts list are required at the beginning of the assessment. You need to know:

- How many systems (Windows, network devices, applications)
- How many user accounts, both standard and privileged
- Where privileged users are used
- Which systems are accessed by privileged users *(I will talk about this in the next article)*

**Implementation**

- Create a Privilege Access Request Procedure and document
- Use Microsoft Forms to create a Privilege Access Request form. You can also utilise Privilege Access Management in MS365 if you have the required license (Microsoft 365 E5, Office 365 E5, or Microsoft 365 F5)
- Remove all local administrator users
- After reviewing, remove unnecessary MS365 administrative users
- Create a new user for Intune enrollment only — not Global Admin
- Create a policy to prohibit MS365 Global Admin users from accessing or logging on to any Entra joined workstation

| |
|---|
| ![Privilege Access Management process](https://github.com/Penguinsecq/penguinsecq.github.io/raw/main/docs/images/pam-procedures.png) |

*Figure 3 - Privilege Access Request Procedures flow*

---

## Case Study 3 — Application Control Issue

Application control is a security approach designed to protect against malicious code executing on systems. When implemented, it ensures only approved code — such as executables, software libraries, scripts, installers, and drivers — is authorized to execute.

This section was the most challenging to accomplish for Essential Eight Maturity Level 1. You have to be very careful here because application control can crash the operating system if necessary `.exe` or `.dll` files cannot be executed. For example, if your client has many branches in remote locations and you deploy application control without solid testing, end users may encounter a broken operating system.

**Implementation**

From Microsoft: [Application Control for Windows](https://learn.microsoft.com/en-us/compliance/anz/e8-app-control#application-control-for-windows)

While WDAC is preferred, it can be simpler and easier for most organizations to achieve ML1 using just AppLocker as a starting point — both solutions are complementary.

In this article, I am sharing how to implement AppLocker with Microsoft Intune. You can find a comparison between WDAC and AppLocker to help decide which is appropriate for your environment here: [Application Control in Practice](https://github.com/Penguinsecq/penguinsecq.github.io/blob/main/docs/acsc-essential-eight/application-control-in-practice.md)

1. Gather the required applications list as comprehensively as possible for your environment.
2. Create your AppLocker control policies based on the list above. You can utilise the starter policy from NSA Cyber here: [NSA AppLocker Guidance](https://github.com/nsacyber/AppLocker-Guidance)
3. Deploy the policies in **Audit Only** mode. You may need 2 versions for Windows 10 and 11. Deployment details here: [AppLocker Deployment Details (coming soon)](https://github.com/Penguinsecq/penguinsecq.github.io/blob/main/docs/acsc-essential-eight/applocker-deployment-steps.md)
4. Monitor Event Logs for a while — I suggest more than 4 weeks if your business has applications that run only monthly or quarterly.
   Check: `Event Viewer > Applications and Services Logs > Microsoft > Windows > AppLocker` for Event ID 8003.
   Reference: [Using Event Viewer with AppLocker](https://learn.microsoft.com/pl-pl/windows/security/application-security/application-control/app-control-for-business/applocker/using-event-viewer-with-applocker)
5. Tune AppLocker control policies. Update rules to include necessary exceptions or allow rules for trusted apps, vendors, or paths.
6. Test on pilot devices: Apply refined policies to a small group of users or machines to confirm no legitimate software is impacted.
7. Repeat the audit cycle (steps 4–6).
8. Inform staff of upcoming changes and provide a process for requesting access to blocked apps.
9. Once confident in the rules, gradually move from audit to enforced mode using staged deployment.
10. Export and save a copy of your final AppLocker policies for rollback or documentation.
11. Enable Enforcement mode.

| |
|---|
| ![AppLocker Implementation diagram](https://github.com/Penguinsecq/penguinsecq.github.io/raw/main/docs/images/applocker-implement-diagram.png) |

*Figure 4 - AppLocker Implementation diagram*
