# Security Portfolio: Aaron Saye

Documented cloud security investigations, built in a live Azure tenant
(Mad Hat Labs, a multi-user training environment).

Target role: SOC Analyst / Security Analyst
Currently: IT Support Specialist | Dallas, TX
Contact: the.asaye.email@gmail.com · www.linkedin.com/in/aaron-saye

## Investigations
| # | Title | Focus | Write-up |
|---|-------|-------|----------|
| 1 | Operation Dead Deploy | Governance forensics, deployment audit trail | [Operation Dead Deploy, "Investigating a Governence Bypass in Azure"] |
| 2 | The Stolen Identity | App registration attack kill chain (Entra ID) | coming, week 2 |
| 3 | Privilege Audit | RBAC and least privilege | coming, week 3 |
| 4 | Spin Up and Lock Down | Compute attack surface | coming, week 4 |
| 5 | Network the Operative | Network segmentation | coming, week 5 |
| 6 | Bucket Looting | Storage exposure hunting | coming, week 6 |
| 7 | Find the Anomaly | Log analysis and KQL | coming, week 7 |
| 8 | Hunt the Threat | SIEM operations (Sentinel) | coming, week 8 |
| 9 | Score the Tenant | Cloud security posture | coming, week 9 |
| 10 | The Breach (capstone) | Full incident investigation | coming, week 10 |

# [Operation Dead Deploy, "Investigating a Governence Bypass in Azure"]

## Scenario
A temporary test deployment appeared in the Azure training subscription with signs that it did not follow the organization’s operational standards. I investigated how the resource was organized, what metadata and deployment history revealed, and why the governance controls allowed the noncompliant deployment to remain.

## Environment
Live multi-user Azure training tenant, Microsoft Azure portal, Azure subscription and Resource Groups, Azure resource tags, Azure Resource Manager deployment history, Azure Policy compliance and assignment views, Reader access

## Investigation
The first step was to find any resource groups that had any titles that were named outside of the organizations standard naming patterns or policies. The standarized naming convention was RG- for each resource group and I was eventually able to find one with a title that started as "test": 

<img width="3812" height="1810" alt="photo 1 bad RG name" src="https://github.com/user-attachments/assets/60a08209-0e52-4010-a6e4-dd32c178eeaf" />

I then proceeded to investigate the resource found within:

<img width="3819" height="1818" alt="photo 2 found a resource" src="https://github.com/user-attachments/assets/9cc74f1b-815e-4df8-b20d-b239b0e6b62a" />

The details of the resource weren't able to help much with determining what may have caused this resource group to be named incorrectly:

<img width="3803" height="1816" alt="photo 3 looked at details of the resource" src="https://github.com/user-attachments/assets/e2a50a01-838b-4ca4-829e-dfd7f098a350" />

However upon inspecting the resource I could see that something was successfully deployed:

<img width="3810" height="1803" alt="photo 4 deployment found within the resource" src="https://github.com/user-attachments/assets/6acaa3b2-fc79-49d9-8962-1cc87cd7515a" />

I determined it was worth taking a look at the tags of the parent resource to see what kind of info I could find:

<img width="3817" height="1821" alt="photo 5 the tags of the resource" src="https://github.com/user-attachments/assets/2371120d-081d-4af1-a695-252e360e770d" />

Looked at the policies in place with the parent resource group:

<img width="3807" height="1827" alt="photo 6 compliance issues found with the deployment" src="https://github.com/user-attachments/assets/1f9b207b-1ef2-4093-8d33-c6832b3cf30e" />

Then when further inspecting the naming convention that had the compliance issue I was able to see that it was set to an audit instead of deny policy:

<img width="3796" height="1800" alt="photo 7 found the policy was set to audit and should have been deny" src="https://github.com/user-attachments/assets/7cd50141-86af-4251-9173-cbbdf6b0f860" />

## What broke / what surprised me
After piecing together the different information found within the tags and policies found pertaining to the offending resource group I was able to determine that the RG was created by an intern who incorrectly named it, however it was still able to make a deployment regardless of the naming policy being incorrect.

## Findings and recommendations
The naming policy for resource groups was set to audit instead of deny which allowed the hastily created RG to allow a deployment which cause a compliance alert. Changing the policy to one of "deny" would instead cause any incorrectly named resources to be denied and in turn stop any uneeded alerts within the org.

## What I learned
- Resource Groups, names, and tags are security evidence, not merely administrative details.
- Azure Resource Manager deployment history can reconstruct when and how a resource was created.
- Azure Policy must be evaluated by both scope and effect; reporting noncompliance differs from preventing it.
- Strong investigations connect multiple evidence sources instead of relying on a single portal view.
- Next time, I would document the expected standard first, making it easier to distinguish an intentional exception from a governance failure.
