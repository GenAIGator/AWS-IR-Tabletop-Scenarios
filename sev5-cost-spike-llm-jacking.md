> Community scenario designed for use with [BREACH: A Tabletop Roleplaying Game of AWS Incident Response](https://www.amazon.com/dp/B0F1FQJBP3) by Jonathan Jenkyn.

# The Cost Spike: LLM Jacking

**Severity Level:** 5 (Training Exercise)

## Situation Brief

Your team wakes up to a billing alert that shouldn't exist. Amazon Bedrock charges have blown past $4,000 in under 18 hours — on an account that normally spends a few hundred a month on model invocations. Someone is burning through your Claude and Llama access at industrial scale, and the charges are still climbing. The good news: nobody is after your data. The bad news: every minute you spend figuring this out costs another few dollars.

The root cause is painfully common. A developer committed an IAM access key to a public GitHub repository. Within hours, automated credential scanners found it and plugged it into a proxy service that resells AI model access at a discount. Your key is now powering a grey-market LLM service for hundreds of anonymous users.

## Your Environment

- Single AWS account running a customer-facing web application
- Amazon Bedrock enabled with Claude 3.5 Sonnet, Claude 3 Haiku, Llama 3, and Titan model access
- IAM users with programmatic access keys used for local development and CI/CD
- CloudTrail logging management events
- GuardDuty enabled with default configuration
- Monthly account budget alert set at $10,000 (just fired at 80%)
- No Bedrock-specific spending alarms or service quotas configured
- No automated budget remediation actions in place

## The Adversary

This isn't a sophisticated threat actor — it's an automated pipeline. Credential scanners continuously monitor public repositories for exposed AWS keys. When they find one, they validate it, check what services it can access, and route proxy traffic through it within hours. The operators run a discount LLM service: their customers pay a fraction of the retail cost, and stolen credentials cover the compute. They have zero interest in your data, your infrastructure, or sticking around. The moment you revoke the key, they move to the next one. They process hundreds of leaked keys every week.

## How It Unfolds

The incident begins days before anyone notices. A developer pushes a `.env` file containing their AWS access key to a personal public repository. GitHub's secret scanning detects the exposure and notifies AWS, which attaches a quarantine policy to the IAM user — but that policy restricts resource creation, not model invocation. Bedrock calls slip right through. The developer doesn't notice the GitHub notification or the AWS Support case that opens automatically.

Within a few hours, external scanners independently discover the key. They validate it, enumerate the available Bedrock models, and start routing traffic. Invocation volume ramps quickly — hundreds of calls per minute, primarily targeting the most expensive model available. The usage runs around the clock from globally distributed IP addresses, consistent with a multi-user proxy service.

After roughly 18 hours of sustained abuse, spending crosses $4,000 and your monthly budget alert fires. GuardDuty also generates an anomalous behaviour finding due to the unusual call volume and geographic distribution. The charges accumulate at roughly $200–300 per hour until someone acts.

## The Trigger

The simulation begins when the budget alert lands in your team's shared billing inbox:

*"AWS Budgets: Your account has exceeded 80% of your monthly budget of $10,000. Current forecasted spend: $14,200. The primary cost driver is Amazon Bedrock — $4,127 in charges this billing period (previous month: $340)."*

A GuardDuty finding for anomalous IAM user behaviour is also waiting in the console for anyone who checks.

## Key Decisions

- Do you disable the key immediately, or investigate first to understand the scope? Every phase of delay adds hundreds in charges.
- How do you identify which credential is compromised when you have multiple IAM users with Bedrock access?
- Once contained, do you check whether the attacker attempted to create provisioned throughput (which would commit tens of thousands per month)?
- How do you verify that no customer data was sent to the models during the abuse window?
- Who has the conversation with the developer about how the key was exposed, and how do you frame it constructively?
- What controls do you implement to prevent recurrence — tighter budget alarms, Bedrock service quotas, credential scanning in CI, access key rotation policies?

## What This Tests

- **Credential management:** Understanding the IAM access key lifecycle, rotation, and exposure risks
- **Billing monitoring:** Recognising gaps in cost alerting granularity (a daily Bedrock threshold would have caught this 12+ hours earlier)
- **Containment speed:** Whether the team prioritises stopping the bleeding over building a complete picture first
- **GuardDuty investigation:** Using automated findings alongside billing data to identify the compromised principal
- **AWS quarantine policy awareness:** Understanding what the automatic quarantine does and doesn't block
- **Post-incident hardening:** Identifying preventive controls (model access governance, pre-commit hooks, key rotation enforcement)

## Director Notes

Play the threat actor as absent. There is no human on the other end making decisions — just an automated system consuming your compute. When the key is revoked, invocations stop immediately. No escalation, no adaptation, no negotiation.

This scenario should feel straightforward but slightly uncomfortable. A leaked key on GitHub is a common, preventable mistake. Lean into that tension. The value isn't in the technical difficulty — it's in surfacing the monitoring gaps and process failures that let a 10x cost anomaly go unnoticed for most of a day.

Teams with basic IAM hygiene should contain this within two phases of discovering the alert. If they struggle to identify which key or how to disable it, that's itself a valuable finding about the organisation's credential management maturity.

Keep the pace brisk. This scenario should resolve in 4–6 phases. The real learning happens in the post-incident discussion about what controls were missing.

---

*Disclaimer: This is a entirely fictitious simulation scenario inspired by [BREACH: A Tabletop Roleplaying Game of AWS Incident Response](https://www.amazon.com/dp/B0F1FQJBP3) by Jonathan Jenkyn. All organisations, environments, and events described are fictional and created solely for training purposes. Any resemblance to real incidents or organisations is coincidental. These scenarios are community-created companion content and are not officially affiliated with or endorsed by the author of BREACH.*
