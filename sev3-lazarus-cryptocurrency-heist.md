> Community scenario designed for use with [BREACH: A Tabletop Roleplaying Game of AWS Incident Response](https://www.amazon.com/dp/B0F1FQJBP3) by Jonathan Jenkyn.

# The Lazarus Cryptocurrency Heist

**Severity Level:** 3 (Significant Incident)

## Situation Brief

A customer who tracks every basis point of their trades has noticed a $240 discrepancy on a $120,000 swap. Your support team chalked it up to slippage, but the customer pushed back — the numbers don't reconcile. When your engineering team pulls the thread, they'll discover that someone has surgically modified your transaction processing Lambda function, inserting a skimming operation that siphons fractions of a percent from high-value trades. It's been running for two weeks. The total theft is approaching six figures, and some of those funds have already been tumbled through mixing services where recovery becomes nearly impossible.

This is a supply chain attack that started with a fake recruiter on LinkedIn and ended with a North Korean threat group quietly draining your platform's fee margin into a wallet they control.

## Your Environment

- Single AWS account running a DeFi platform for token swaps, liquidity provision, and automated trading
- Lambda functions as the core transaction engine (validates transactions, calculates fees, submits signed transactions to the blockchain)
- The primary function (`tx-processor-prod`) has access to Secrets Manager (hot wallet keys), DynamoDB (transaction ledger), and S3 (audit logs)
- API Gateway fronting the Lambda layer
- DynamoDB tables for transaction records, user balances, and pool state
- CI/CD pipeline via CodeBuild deploys Lambda updates
- CloudTrail logging management events
- GuardDuty enabled with default configuration
- Lambda versioning enabled but previous versions not routinely compared
- No Lambda code signing configured
- Daily automated reconciliation with a 1% fee variance tolerance threshold
- Developer EC2 workstations with deployment-capable IAM roles
- npm packages pulled from the public registry with no private mirror

## The Adversary

This operation is attributed to a North Korean unit specialising in cryptocurrency theft — a branch of the Lazarus Group focused on DeFi platforms. They fund weapons programmes through stolen digital assets, and they've refined their approach over dozens of successful operations. Their hallmark is patience: weeks of social engineering to compromise a single developer, followed by months of low-volume theft designed to stay below every detection threshold.

The team running this operation includes a social engineer who spent three weeks building a relationship with one of your senior developers through a fake recruiter persona, a malware author who weaponised an npm package to harvest credentials from the developer's workstation, and an operator who studied your transaction code before making a surgical modification. They have blockchain expertise, access to mixing services and cross-chain bridges, and the discipline to skim small enough amounts that automated reconciliation passes without flagging.

## How It Unfolds

Three weeks before anyone notices a problem, a convincing recruiter reaches out to a senior developer on your team via LinkedIn. They discuss a "stealth-mode DeFi startup" and an attractive compensation package. Eventually, the recruiter asks the developer to review a technical assessment — a GitHub repository containing a Node.js project. The developer clones it and runs the install.

A malicious dependency executes a postinstall script that silently harvests the developer's IAM session token, environment variables, and SSH keys from the EC2 instance metadata service. The stolen credentials provide access to your deployment pipeline. The threat actor uses them to download your transaction processing function, study its logic, and craft a modified version.

The modification is subtle: a handful of lines inserted into the fee calculation module. For trades above a certain value threshold, the code skims a small randomised percentage and routes it to an external wallet. The amount is taken from the platform's fee margin, not the user's payout — so customers receive the expected amount and the transaction logs record success. The skim hides within normal fee variance caused by gas price fluctuations and slippage.

The deployment happens during a routine release window, using the compromised developer's credentials. It looks like any other Tuesday afternoon push. Over the following two weeks, the modified function processes thousands of transactions normally while quietly diverting funds from high-value trades. The automated daily reconciliation shows fee variance creeping upward — but staying within the 1% tolerance band.

## The Trigger

The simulation begins when a precise customer forces the issue:

*"Your engineering lead forwards a support ticket to #platform-issues: 'User reports a $240 discrepancy on a $120K swap from yesterday. Support couldn't explain it with slippage or gas. The user is a whale — they track every basis point. Can someone pull the transaction logs and verify the fee calculation?'"*

*Your daily reconciliation report also shows today's fee variance at 0.82% — the highest in three months, but still within the 1% tolerance. Nobody has raised it yet.*

The team doesn't know about the compromised developer, the malicious package, or the Lambda modification. They're starting from a single customer complaint about a small number.

## Key Decisions

- Do you treat this as a genuine anomaly or dismiss it as normal variance? How many phases pass before someone examines the actual Lambda code?
- When you discover the code modification, do you immediately revert the function or preserve it as evidence first?
- How do you distinguish between "a developer deployed a bug" and "a developer's credentials were used by an attacker"?
- Once you understand the scope, do you engage blockchain analytics firms immediately to attempt fund recovery, or finish your internal investigation first? (Delay reduces recovery chances as funds move through mixers.)
- How do you calculate the total financial loss across two weeks of variable-rate skimming?
- Do you notify affected users, and if so, how do you explain that their trades executed correctly but the platform was being robbed?
- What do you do about the compromised developer's workstation and the malicious npm package still potentially present?

## What This Tests

- **Lambda code integrity:** Whether anyone compares deployed code against source control as part of incident investigation
- **Supply chain awareness:** Understanding how a social engineering attack on a developer leads to production compromise
- **Financial reconciliation rigour:** Whether variance thresholds are tight enough to catch subtle ongoing theft
- **Blockchain coordination timeliness:** Recognising that cryptocurrency recovery has a shrinking window
- **Credential compromise scoping:** Tracing what else the compromised developer's session token could have accessed
- **Deployment pipeline security:** Whether code signing, mandatory review gates, or deployment anomaly detection would have prevented this
- **Cross-functional communication:** Coordinating between engineering, finance, legal, and external analytics firms simultaneously

## Director Notes

Play the adversary as patient and silent. When the team reverts the Lambda function, the threat actor doesn't respond — they simply stop receiving funds. There's no ransom note, no communication, no escalation. They accept the loss of the ongoing skim and focus on laundering what they've already taken. This silence can be unnerving for teams who expect some kind of confirmation that they've "won."

The critical tension is the time-sensitivity of blockchain recovery. Funds sitting in intermediate wallets can potentially be frozen by exchanges if flagged quickly. Once they hit a mixer, they're effectively gone. Teams that spend too long on internal investigation before engaging external blockchain analytics firms will lose the recovery window.

Guide the team toward discovering the npm supply chain vector. The Lambda modification is the visible symptom, but the root cause is a developer who ran untrusted code on a machine with production-adjacent credentials. This is where the most valuable findings live: workstation isolation, IAM permission boundaries for developer roles, and the trust model for third-party packages.

For financial calculation, make the team work through it: identify the affected transaction window, estimate the variable skim rate across qualifying trades, and produce a concrete dollar figure for leadership. This analytical work is part of the exercise.

Expected duration: 10–12 phases. The scenario has a long tail as teams work through technical investigation, financial impact assessment, and blockchain coordination simultaneously.

---

*Disclaimer: This is a entirely fictitious simulation scenario inspired by [BREACH: A Tabletop Roleplaying Game of AWS Incident Response](https://www.amazon.com/dp/B0F1FQJBP3) by Jonathan Jenkyn. All organisations, environments, and events described are fictional and created solely for training purposes. Any resemblance to real incidents or organisations is coincidental. These scenarios are community-created companion content and are not officially affiliated with or endorsed by the author of BREACH.*
