# Day 9: Nine o'clock, make GRC fun, tell no one.

### Introduction to GRC
Governance, Risk, and Compliance (GRC) plays a crucial role in any organisation to ensure that their security practices align with their personal, regulatory, and legal obligations. Although in general good security practices help protect a business from suffering a breach, depending on the sector in which an organisation operates, there may be external security regulations that it needs to adhere to.

When you run a large organisation with multiple different teams, how do you stay on top of all these regulations and ensure that good security is applied by all teams? This is where **GRC** comes in. They play a crucial role in understanding external security standards, translating them into internal standards, and then ensuring that they are applied by all teams to help reduce the organisation's risk to an acceptable level. Let's take a quick look at the three functions of GRC.  

#### Governance
Governance is the function that creates the framework that an organisation uses to make decisions regarding information security. Governance is the creation of an organisation's security strategy, policies, standards, and practices in alignment with the organisation's overall goal. Governance also defines the roles and responsibilities that everyone in the organisation has to play to help ensure these security standards are met.  

#### Risk
Risk is the function that helps to identify, assess, quantify, and mitigate risk to the organisation's IT assets. Risk helps the organisation understand potential threats and vulnerabilities and the impact that they could have if a threat actor were to execute or exploit them. By simply turning on a computer, an organisation has some level of risk of a cyber attack. The risk function is important to help reduce the overall risk to an acceptable level and develop contingency plans in the event of a cyber attack where a risk is realised.  

#### Compliance
Compliance is the function that ensures that the organisation adheres to all external legal, regulatory, and industry standards. For example, adhering to the GDPR law or aligning the organisation's security to an industry standard such as NIST or ISO 27001.

### Introduction to Risk Assessments
Before McSkidy and Glitch choose an eDiscovery company to handle their forensic data, they need to figure out which one is the safest choice. This is where a **risk assessment** comes in. It's a process to identify potential problems before they happen. Think of it as checking the weather before going on a hike; if there’s a storm coming, you'd want to know ahead of time so you can either prepare or change your plans.

For McSkidy and Glitch, assessing the risks of each eDiscovery company helps them decide which one is less likely to have a data breach or other issues that could disrupt the investigation.

#### Performing a Risk Assessment
We will work through the process of completing a **risk register**. A risk register tracks the progress of risk mitigation and all open risks. An example of such a risk register is shown below:

![image](https://github.com/user-attachments/assets/e449b873-5e3a-456b-b161-a2183d722db4)

- **Identification of Risks**
  This requires carefully assessing the attack surface of the organisation and identifying areas which might be used to harm the organisation.

- ﻿**Assigning Likelihood to Each Risk**
  In addition to identifying risks, these risks also need to be quantified. To quantify risk, we need to identify how likely or probable it is that the risk will materialise. We can then assign a number (often on a scale of 1 to 5) to quantify this likelihood. Likelihood can also be called the "probability of materialisation of a risk". An example scale for likelihood can be:

1. **Improbable:** So unlikely that it might never happen.
2. **Remote:** Very unlikely to happen, but still, there is a possibility.
3. **Occasional:** Likely to happen once/sometime.
4. **Probable:** Likely to happen several times.
5. **Frequent:** Likely to happen often and regularly.

It might be noticed that while we are trying to quantify the risk, we still don't define exact quantities of what constitutes several times and what constitutes regularly, etc. The reason is that the likelihood for a server which has very high uptime requirements will be different from a server that is used infrequently. Therefore, the likelihood scale will differ from case to case and from asset to asset. On the flip side, we can see that this scale provides us with a very usable scale of differentiating between different probabilities of occurrence of a certain event.

- **Assigning Impact to Each Risk**
  Once we have identified the risks and the likelihood of a risk, the next step is to quantify the impact that the risk's materialisation might have on the organisation. Different organisations calculate impact in different ways. Some organisations might use the **CVSS scoring** to calculate the impact of a risk; others might use their own rating derived from the Confidentiality, Integrity, and Availability of a certain asset, and others might base it on the severity categorisation of the incidents. Similar to likelihood, we also quantify impact, often on a scale of 1 to 5. An example scale of impact can be based on the following definitions:

1. **Informational:** Very low impact, almost non-existent.
2. **Low:** Impacting a limited part of one area of the organisation's operations, with little to no revenue loss.
3. **Medium:** Impacting one part of the organisation's operations completely, with major revenue loss.
4. **High:** Impacting several parts of the organisation's operations, causing significant revenue loss
5. **Critical:** Posing an existential threat to the organisation.

- **Risk Scoring and Ownership**
	- **Assigning a Risk Score:** The last step to performing a risk assessment is to decide what to do with the risks that were found. We can start by performing some calculations on the risk itself. The simplest calculation takes the *likelihood* of the risk and multiplies it with the *impact* of the risk to get a score. Assigning risk scores helps organisations prioritise which risks should be remediated first.
	- **Assigning a Risk Owner:** These team members are responsible for performing an additional investigation into what the cost would be to close the risk vs. what we could lose if the risk is realised. In cases where the cost of security is lower, we can **mitigate** the risk with more security controls. However, if it's higher, we can **accept** the risk. Accepted risks should always be documented and reviewed periodically to ensure that the cost has not changed.

### Internal and Third-Party Risk Assessments
Risk assessments are not just done internally in an organisation, but can also be used to assess the risk that a third party may hold to our organisation. Today, it is very common to make use of third parties to outsource key functions of your business. This changes the risk because a compromise of the third party, where *we* don't have control over *their* security, could result in a compromise of our sensitive assets.

#### Why Do Companies Do Internal Risk Assessments?
Internal risk assessments help companies understand the risks they have within their own walls. They help with:
- Identifying weak spots in security.
- Directing resources to the most important areas.
- Staying compliant with security rules and regulations.

#### Why Do Companies Do Risk Assessments of Third Parties?
This is important because one weak link in the chain can affect everyone. It is important to review if those third-party companies:
- Have good security measures to keep data safe.
- Follow data protection rules.
- Align with the security standards the company.

### Procuring a Partner
Let's put this knowledge to the test!

![image](https://github.com/user-attachments/assets/12f4db42-9d71-49ed-b534-ae7eb0dae5b4)


### Answers
1. What does GRC stand for? **Governance, Risk, and Compliance**.
2. What is the flag you receive after performing the risk assessment? **THM{R15K_M4N4G3D}**.
