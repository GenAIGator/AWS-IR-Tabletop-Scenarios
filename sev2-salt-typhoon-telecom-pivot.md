> Community scenario designed for use with [BREACH: A Tabletop Roleplaying Game of AWS Incident Response](https://www.amazon.com/dp/B0F1FQJBP3) by Jonathan Jenkyn.

# The Salt Typhoon Telecom Pivot

**Severity Level:** 2 (Major Incident)

## Situation Brief

Your CloudTrail monitoring fires a critical alert: someone just disabled S3 data event logging on your most sensitive buckets. The unsettling part — the API call came from your trusted telecom partner's cross-account role. A role that should only be reading call records for roaming settlement and number portability. A role that has no business touching CloudTrail configuration.

What your team doesn't know yet: the partner's AWS account was compromised ten days ago by a Chinese state-sponsored intelligence group. For the past week and a half, they've been accessing your environment through the legitimate trust relationship, carefully mimicking normal partner traffic while quietly exfiltrating subscriber records, call metadata, and location data for millions of customers. Disabling the logging is their move to accelerate bulk collection now that they've identified their targets.

This is a nation-state signals intelligence operation running through your most trusted integration point.

## Your Environment

- Multi-account AWS Organisation for a mid-sized mobile carrier
- Production account: S3 buckets with call detail records (18 months of archive plus real-time), DynamoDB tables with subscriber profiles (2.3M records), subscriber location history, and device registry
- Partner Integration account: Cross-account IAM role allowing a trusted telecom partner to assume access for roaming, portability, and billing data exchange
- Shared Services account: Centralised CloudTrail (management + S3 data events for critical buckets), GuardDuty, Config, Security Hub
- VPC peering between your Partner Integration account and the partner's account
- The partner trust relationship is 3 years old with well-established daily access patterns (02:00 UTC batch pulls, business-hours portability lookups, monthly billing reconciliation)
- No IP restriction on the cross-account role assumption
- 12-hour session duration configured on the partner role
- No external ID on the trust relationship (legacy configuration)
- DynamoDB data events NOT logged (cost decision) — only CloudWatch metrics show read capacity consumption
- A CloudTrail configuration monitoring Lambda alerts on event selector changes

## The Adversary

Salt Typhoon is a Chinese state-sponsored group that specialises in telecommunications intelligence collection. Their mission is signals intelligence — not disruption, not extortion, but the quiet bulk collection of communications metadata that reveals who talks to whom, when, how often, and from where. Call detail records and subscriber data let them map social networks, track physical movements via cell tower connections, and identify surveillance targets without ever intercepting call content.

This team includes a telecom domain expert who understands carrier interconnect architecture, an AWS specialist who mapped your cross-account relationships, and a data analyst who will sift through exfiltrated records to identify intelligence targets — government officials, journalists, defence personnel, dissidents. They operate entirely from within the compromised partner's infrastructure, making their traffic indistinguishable from legitimate partner access. They studied the partner's normal patterns for a full week before touching your data, and they will not create new roles, modify trust policies, or install any persistence in your account. Their persistence lives in the partner's compromised environment.

## How It Unfolds

Ten days before your alert fires, Salt Typhoon compromised your telecom partner's AWS account through a spear-phishing campaign targeting their cloud engineering team. They spent the first week doing nothing but watching — learning which role gets assumed, what data gets accessed, when it happens, and what volumes look normal.

Their first actions in your environment perfectly mimic the partner's legitimate daily batch job: assume the role at the expected time, list objects in the expected prefix, download the expected data. Indistinguishable from normal operations. But during those long 12-hour sessions, they begin accessing data outside the documented sharing agreement. A permission drift from 18 months ago — when a developer broadened the role's S3 policy for a one-time migration and never reverted it — means the role can actually read all CDR prefixes, not just the agreed ones. They access domestic call records and lawful intercept metadata.

They scan your DynamoDB subscriber tables to pull complete customer profiles and location histories. Because DynamoDB data events aren't logged, these operations produce no CloudTrail record — only a spike in read capacity visible in CloudWatch metrics that nobody is watching for this pattern.

When they're confident they understand the data landscape and have identified their high-value targets, they make their move to enable sustained bulk exfiltration: they modify CloudTrail's event selectors to remove S3 data event logging on your critical buckets. This is the first action that breaks the pattern of legitimate partner behaviour — and your monitoring Lambda catches it.

## The Trigger

The simulation begins when your automated monitoring detects the CloudTrail modification:

*"Your CloudTrail configuration Lambda fires to #security-ops: 'CRITICAL: CloudTrail event selectors modified — S3 data events REMOVED for buckets prod-cdr-archive and prod-cdr-realtime. Change made by role PartnerTelco-DataExchange-Role (assumed from the partner's account) at 03:17 UTC.'"*

*The CloudTrail event shows the source IP is within the VPC peering range allocated to your partner. The user agent indicates AWS CLI running on Linux. The principal is the same role your partner uses every night for legitimate data exchange.*

Your team must now answer: why is a data exchange role modifying CloudTrail configuration? Was this a mistake by a partner engineer troubleshooting a sync issue, or something far worse?

## Key Decisions

- Do you immediately revoke the CloudTrail permissions from the partner role and restore logging, or investigate further first?
- How quickly do you contact the partner's security team — and what do you do if they're slow to respond or initially deny any compromise?
- When you learn the partner account is compromised, do you sever the trust relationship entirely (breaking roaming, portability, and billing) or restrict it to documented permissions only?
- How do you assess what was accessed during the period when DynamoDB data events weren't logged? Can CloudWatch metrics and VPC Flow Logs reconstruct the picture?
- If you discover that lawful intercept metadata was accessed, do you engage law enforcement and intelligence agencies? This moves beyond a data protection issue into national security territory.
- How do you weigh the security decision (sever the trust) against the business impact (roaming customers lose service, number ports fail, billing reconciliation halts)?

## What This Tests

- **Third-party trust as an attack surface:** Recognising that a compromised partner becomes a trusted insider
- **Permission drift awareness:** Understanding how long-lived cross-account roles accumulate excess permissions over time
- **Forensics in blind spots:** Reconstructing access patterns when DynamoDB data events aren't logged, using CloudWatch and Flow Logs as secondary evidence
- **Cross-organisation coordination:** Working with an external partner who may be unresponsive, in denial, or themselves under active compromise
- **Business continuity trade-offs:** Making risk-based decisions when the secure choice breaks legitimate business operations
- **CloudTrail integrity monitoring:** Detecting and responding to logging configuration changes as a high-priority event
- **Lateral movement via trust:** Understanding how legitimate trust relationships become lateral movement paths

## Director Notes

The central challenge here is cognitive: the access comes from a trusted source. Every API call uses the expected role, from the expected account, within the expected IP range. There are no GuardDuty findings because the access pattern matches the trust configuration. Teams must overcome the instinct to explain away the anomaly as "just our partner doing something odd."

When the team contacts the partner, play their security team as initially sceptical. They haven't seen indicators of compromise. They suggest it might be a misconfiguration. Cooperation comes slowly — hours, not minutes — as evidence mounts. This models the real friction of cross-organisation incident response where you cannot control the other party's urgency.

The business continuity dilemma is real and has no clean answer. Severing the trust relationship is the safest security call, but it disrupts services for real customers immediately. Let the team wrestle with this. There's no single correct answer — the value is in seeing how they reason about the trade-off and communicate it to leadership.

If the team discovers the lawful intercept metadata was accessed, introduce this as a complication that adds a national security dimension. Most teams have never exercised engagement with intelligence agencies or law enforcement on a telecom breach.

Salt Typhoon will not escalate to destructive actions. If access is restricted, they accept the loss and go quiet. Their persistence is in the partner's account, not yours — so full eradication from your environment is achievable by severing or restricting the trust relationship. The deeper problem is ensuring the partner remediates their own compromise before trust is re-established.

Expected duration: 12–14 phases. This is a long scenario with simultaneous workstreams: technical investigation, partner coordination, scope assessment, regulatory notification, and trust relationship decisions running in parallel. Use fatigue mechanics after phase 8.

---

*Disclaimer: This is a entirely fictitious simulation scenario inspired by [BREACH: A Tabletop Roleplaying Game of AWS Incident Response](https://www.amazon.com/dp/B0F1FQJBP3) by Jonathan Jenkyn. All organisations, environments, and events described are fictional and created solely for training purposes. Any resemblance to real incidents or organisations is coincidental. These scenarios are community-created companion content and are not officially affiliated with or endorsed by the author of BREACH.*
