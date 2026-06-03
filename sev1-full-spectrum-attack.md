> Community scenario designed for use with [BREACH: A Tabletop Roleplaying Game of AWS Incident Response](https://www.amazon.com/BREACH-Resilience-Exercised-Containment-Roleplaying/dp/B0H1JJP7B5/) by Jonathan Jenkyn.

# The Full-Spectrum Attack

**Severity Level:** 1 (Crisis)

## Situation Brief

Everything is happening at once. A customer reports a suspicious login page. GuardDuty fires an alert showing a compromised IAM credential making high-privilege API calls from an unknown IP. When your team starts pulling the thread, they discover not one compromise but six — DNS hijacking, WAF rules disabled, backdoored compute launch templates, a Lambda function silently exfiltrating documents, an EventBridge rule maintaining persistent access, and an active credential harvesting operation. All of it pre-positioned and activated in a coordinated sequence designed to overwhelm your capacity to respond.

This is a nation-state operation targeting your clients' sensitive financial documents. The adversary has deep knowledge of your architecture, weeks of preparation behind them, and the resources to operate across every layer of your stack simultaneously. Your team is outnumbered and will have to make hard choices about what to address first while data actively leaves your environment.

## Your Environment

- Multi-account AWS Organisation running a B2B SaaS platform for financial document processing (contracts, invoices, tax filings)
- Production account: CloudFront distributions, Application Load Balancers, EC2 Auto Scaling groups, RDS PostgreSQL databases, S3 document storage, Lambda functions for OCR and document classification
- Shared Services account: CI/CD pipelines, centralised CloudTrail, GuardDuty, Security Hub, Route 53 hosted zones for all public DNS
- Development account: Testing and staging with IAM users and broader permissions
- AWS WAF on CloudFront and ALB with rate limiting and managed rule groups (Core Rule Set, Known Bad Inputs, SQL Injection)
- GuardDuty enabled in all accounts
- CloudTrail logging management and data events
- VPC Flow Logs enabled
- Shield Standard on public-facing resources
- Application data path: Route 53 → CloudFront → ALB → EC2 ASG → RDS
- Lambda document processing pipeline triggered by S3 events
- EventBridge rules for scheduled maintenance

## The Adversary

This is a top-tier nation-state team operating at the highest level of capability, stealth, and resourcing. They've spent weeks inside your environment conducting quiet reconnaissance — mapping accounts, cataloguing Lambda functions, reviewing launch templates, understanding your WAF configuration, and identifying your DNS architecture. They have pre-positioned every attack vector and are now executing them in sequence to create maximum confusion for defenders.

Their objective is espionage: exfiltrating sensitive financial documents belonging to your clients, which include government contractors and defence industry firms. But they also want lasting access for ongoing intelligence collection. The initial data grab is the primary goal; the persistence mechanisms ensure they can return later for future documents. They have dedicated operational support, external infrastructure staged and ready (credential harvesting sites, command-and-control servers, exfiltration endpoints), and they will adapt intelligently to your containment actions — shifting focus to whichever attack vectors you haven't addressed yet.

## How It Unfolds

The operation began three days ago when the adversary compromised a senior developer's access key through targeted phishing. Since then, they've moved quietly: mapping your infrastructure, identifying trust relationships, and staging their attack. By the time your team gets its first indicator, four of six attack vectors are already in place.

The DNS attack is subtle — a weighted routing policy sends only 10% of traffic for your login page to a credential harvesting server. Ninety percent of users still reach the legitimate site, making the hijack difficult to notice through casual monitoring. The cloned login page is nearly pixel-perfect, differing only in the TLS certificate organisation name.

The WAF modifications are blunt but effective: managed rule groups disabled, rate limits increased by two orders of magnitude. Your application layer protections are functionally gone.

The compute compromise is a time bomb: the ASG launch template now includes a reverse shell in its user data. The next instance that scales in will phone home to the attacker on port 443, blending with normal HTTPS traffic.

The Lambda backdoor is elegant: four lines added to your document classification function copy every processed document to an external bucket before returning the normal result. The function still works perfectly — documents classify and index correctly — but a copy of everything goes to the adversary.

The EventBridge persistence is their insurance: a new rule triggers a health-check Lambda every six hours that validates access and reports back to command infrastructure. The function's execution role has broad cross-account permissions.

Once an instance scales with the compromised template, the attacker has command-line access to production, direct database connectivity, and the ability to accelerate exfiltration through multiple channels simultaneously.

## The Trigger

The simulation begins with two indicators arriving simultaneously:

*"Customer success forwards an urgent client message: 'One of our users was asked to re-enter their password today and the page looked slightly different — the footer was missing and the certificate showed a different organisation name. They entered their credentials before noticing. Should we be concerned?'"*

*"Three minutes later, #security-alerts shows a GuardDuty finding: UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration.OutsideAWS — IAM user dev-senior-01 making API calls from an unfamiliar IP. Calls include route53:ChangeResourceRecordSets, wafv2:UpdateWebACL, and ec2:CreateLaunchTemplateVersion."*

Your team is now looking at a compromised credential that has already modified DNS, WAF, and compute infrastructure. The Lambda and EventBridge modifications may already be in place. The clock is running.

## Key Decisions

- With six simultaneous attack vectors, what do you address first? DNS hijacking (customers actively being phished), WAF disablement (application exposed), Lambda exfiltration (data actively leaving), or the IAM credential (source of ongoing modifications)?
- If you disable the compromised IAM credential, do you understand that all existing modifications remain active and operating independently?
- How do you handle the DNS situation when 10% of users may have been redirected to a harvesting site? Force password resets for all users? Just the affected window?
- How do you identify and terminate the reverse shell connection if a compromised instance has already launched? Can you isolate it without causing a customer-facing outage?
- Can your team split effectively across multiple workstreams, or do you need to serialise your response?
- How do you verify you've found everything? The adversary had days of access — what else might they have modified that you haven't discovered yet?
- When leadership demands an update 20 minutes in, can you articulate the scope clearly while still executing containment?

## What This Tests

- **Coordination under extreme pressure:** Whether the team can divide responsibilities and work multiple vectors simultaneously without losing coherence
- **Prioritisation:** Making rational triage decisions when everything feels urgent — what's causing active harm right now versus what can wait 15 minutes?
- **Multi-service investigation:** Correlating indicators across Route 53, WAF, EC2, Lambda, EventBridge, and IAM simultaneously
- **Containment vs. availability:** Understanding which response actions will cause service disruption and managing that trade-off
- **Evidence preservation under time pressure:** Capturing forensic artifacts while also trying to stop active data exfiltration
- **Communication under crisis:** Briefing leadership coherently when the situation is still developing and scope is unclear
- **Persistence hunting:** Not stopping at the obvious findings — understanding that a sophisticated actor may have mechanisms you haven't found yet
- **Stress management:** Whether the team can use support mechanics and coordination to avoid being overwhelmed

## Director Notes

This scenario is designed to overwhelm. Do not apologise for the pace. The purpose is to surface what the organisation would actually need — in staffing, tooling, automation, and process — to handle a coordinated nation-state attack. Most teams will not achieve full containment, and that's the finding.

Run this at speed. Advance the clock aggressively. Introduce complications frequently: leadership wanting updates, customer escalations, a media inquiry if the DNS hijack becomes public, the threat actor accelerating exfiltration through vectors the team hasn't closed yet.

Play the adversary as intelligent and adaptive. When the team closes one vector, the attacker shifts focus to the remaining ones. Only sustained containment across all vectors simultaneously will shut them down. They have maximum persistence — early escalation checks should succeed easily, requiring the team to hit high difficulty thresholds to make real progress.

Expect stress to accumulate fast. Multiple team members will likely hit high stress levels. At least one may become overwhelmed. The team must use communication support and coordination mechanics, or they will lose effectiveness at exactly the wrong moment. If the team doesn't self-manage their stress, let it impact their rolls.

Only run this scenario with experienced teams who have completed several lower-severity simulations. It should feel like a genuine crisis — the kind of event that would trigger an all-hands war room in a real organisation. The learning is in the gap between what the team can do and what they would need to do. That gap becomes the roadmap for investment in detection, automation, and response capabilities.

Expected duration: 12–16 phases. Most teams won't achieve full containment. The debrief conversation about what would need to be different is where the real value lives.

---

*Disclaimer: This is a entirely fictitious simulation scenario inspired by [BREACH: A Tabletop Roleplaying Game of AWS Incident Response](https://www.amazon.com/dp/B0F1FQJBP3) by Jonathan Jenkyn. All organisations, environments, and events described are fictional and created solely for training purposes. Any resemblance to real incidents or organisations is coincidental. These scenarios are community-created companion content and are not officially affiliated with or endorsed by the author of BREACH.*
