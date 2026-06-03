> Community scenario designed for use with [BREACH: A Tabletop Roleplaying Game of AWS Incident Response](https://www.amazon.com/BREACH-Resilience-Exercised-Containment-Roleplaying/dp/B0H1JJP7B5/) by Jonathan Jenkyn.

# The Scattered Spider Help Desk Call

**Severity Level:** 4 (Routine Incident)

## Situation Brief

A senior platform engineer messages your security channel: his MFA just stopped working. When he called the help desk, they told him his MFA was reset thirty minutes ago — but he's been in meetings all afternoon and never requested it. Someone socially engineered your help desk, took over a legitimate employee's identity, and is now inside your AWS environment using their credentials. The attacker is fast, technically competent, and already exploring what they can reach across your accounts.

This is identity compromise that starts with a phone call, not an exploit. Your verification procedures were followed. They just weren't strong enough.

## Your Environment

- Multi-account AWS Organisation (Production, Sandbox, Shared Services)
- IAM Identity Center for all workforce authentication with MFA enforced
- Production account is tightly locked down — specific roles accessible via Identity Center only
- Sandbox account has broader permissions (developers can create IAM users and roles for testing)
- CloudTrail enabled for management events across all accounts
- GuardDuty with cross-account aggregation
- Identity Center audit logs capturing authentication events, MFA changes, and permission assignments
- Help desk uses ServiceNow for MFA reset requests with a basic verification procedure (employee ID, manager name, ticket reference)
- No callback verification or manager approval required for MFA resets
- Cross-account roles exist between sandbox and production (some created by developers for testing)

## The Adversary

Scattered Spider is a young, English-speaking threat group whose primary weapon is social manipulation. Before picking up the phone, they spend days building a profile of their target — scraping LinkedIn for reporting structures, combing corporate directories, pulling employee ID formats from leaked documents. They time their calls for shift changes when analysts are distracted, impersonate senior technical staff, and create artificial urgency to pressure help desk workers into cutting corners.

Once inside, they move with purpose. They know their window is limited — the real employee will eventually notice something is wrong. Their goal is to establish independent persistence within 20 minutes of gaining access, typically by creating new IAM credentials in less-monitored accounts. From there, they look for cross-account trust relationships they can exploit to reach valuable data. They're financially motivated: steal data, sell access, or extort the target.

## How It Unfolds

The groundwork is laid days before the call. The threat actor researches a senior engineer through LinkedIn and social media, gathering their role, manager's name, team structure, and professional history. They obtain the company's employee ID format from a previously leaked HR document. They even plant a ServiceNow ticket using credentials harvested from an earlier infostealer infection on a help desk analyst's personal device.

The call comes during shift change. The impersonator is confident, references the pre-staged ticket, provides the correct employee ID and manager name, and pressures the analyst with urgency about a production incident. The analyst follows the documented procedure — verifies the ticket, confirms the details — and processes the reset. Within four minutes, a new MFA device is enrolled. Within five, the attacker is logged into the AWS Console.

From there, they move methodically: enumerate available accounts and permission sets, discover the sandbox account offers broad access, identify cross-account roles that trust the sandbox, and begin creating independent access credentials. If they find a role that bridges to production, they'll use it to reach customer data. The entire sequence from MFA reset to established persistence takes under twenty minutes.

## The Trigger

The simulation begins when the legitimate employee notices something is wrong:

*"David Chen, senior platform engineer, messages #security-ops at 17:12: 'Something weird — I just tried to log into the Console and my MFA isn't working. Help desk says it was reset at 16:52 but I didn't request that. I've been in meetings since 15:00. Can someone look into this?'"*

*You check the Identity Center audit logs: MFA deregistered at 16:52, new MFA device registered at 16:56, ConsoleLogin at 16:57 from an unfamiliar IP address. David confirms it wasn't him.*

The attacker has been active for approximately 20 minutes by the time you see this message.

## Key Decisions

- Do you immediately revoke all active sessions for the compromised identity, or reset the MFA first? (Resetting MFA alone doesn't terminate existing sessions.)
- How do you determine what the attacker did during the 20-minute head start? Which accounts did they touch?
- Do you check for newly created IAM users or roles in less-monitored accounts like the sandbox?
- How do you audit cross-account role assumptions that originated from the compromised session?
- When you discover the help desk was socially engineered, how do you communicate that finding without assigning blame to the analyst who followed their procedure?
- What verification improvements do you recommend — callback to registered numbers, manager approval, time delays on MFA resets?

## What This Tests

- **Session management vs. credential management:** Understanding that revoking credentials doesn't kill active sessions
- **Identity Center investigation:** Correlating Identity Center audit logs with CloudTrail across multiple accounts
- **Cross-account lateral movement:** Tracing role assumptions from a compromised identity through sandbox into production
- **Persistence hunting:** Finding IAM users or access keys created in less-monitored accounts
- **Social engineering awareness:** Recognising that the initial access vector was human, not technical
- **Communication under pressure:** Coordinating between security, the help desk, and the affected employee in real time
- **Process improvement:** Identifying that the verification procedure itself was the vulnerability, not the person who followed it

## Director Notes

Speed matters in this scenario. Scattered Spider moves fast and the outcome depends heavily on how quickly the team acts. If they revoke sessions within 10 minutes of David's message, they may catch the attacker mid-operation. If they take 30 minutes to begin investigating, persistence is likely already established.

Play the attacker as opportunistic and fast. They don't linger once they have what they need. If the team revokes the compromised session before persistence is created, the attacker is locked out entirely — a clean win. If the team is slow, the attacker goes quiet and relies on their independently created credentials for future access.

The critical teaching moment is the distinction between resetting an identity (MFA, password) and revoking active sessions. Many teams instinctively reset credentials without terminating sessions — which leaves the attacker's existing Console session active for hours.

Avoid letting the team blame the help desk analyst. The documented procedure was followed. The finding is systemic: the procedure lacked out-of-band verification. Guide the post-incident discussion toward process improvement rather than individual failure.

Expected duration: 6–8 phases. Can resolve quickly with decisive action or extend if the team misses the persistence mechanism in the sandbox.

---

*Disclaimer: This is a entirely fictitious simulation scenario inspired by [BREACH: A Tabletop Roleplaying Game of AWS Incident Response](https://www.amazon.com/dp/B0F1FQJBP3) by Jonathan Jenkyn. All organisations, environments, and events described are fictional and created solely for training purposes. Any resemblance to real incidents or organisations is coincidental. These scenarios are community-created companion content and are not officially affiliated with or endorsed by the author of BREACH.*
