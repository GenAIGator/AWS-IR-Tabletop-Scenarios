# AWS IR Tabletop Scenarios

Community-created incident response simulation scenarios for security teams running on AWS. Designed for use with [BREACH: A Tabletop Roleplaying Game of AWS Incident Response](https://www.amazon.com/dp/B0F1FQJBP3) by Jonathan Jenkyn.

---

## What Is This?

Five ready-to-run cybersecurity incident scenarios that turn your tabletop exercises from flat walkthroughs into high-energy, dice-driven simulations. Each scenario drops a team into a realistic AWS incident — from a leaked credential burning through Bedrock credits to a nation-state executing six simultaneous attack vectors across your entire stack.

These scenarios are built for teams who use AWS and want to practice incident response in a way that's engaging, realistic, and produces actionable findings.

---

## About the Book

These scenarios are designed as companion content for **BREACH** by Jonathan Jenkyn — a tabletop roleplaying game purpose-built for AWS incident response training. The book provides the complete game system:

- **Dice-based skill checks** that add real uncertainty to incident response actions (d10 pools with exploding criticals and botch mechanics)
- **Participant Capability Sheets** where players rate their real skills — creating dice pools that reflect actual team strengths and gaps
- **A phase clock** that creates time pressure and forces prioritisation
- **Layered rules** that let teams start with basic mechanics and gradually add complexity (stress, fatigue, fog of war, opposed rolls, collaborative actions)
- **Threat actor personas** with ratings for Capability, Stealth, Persistence, and Resources
- **Scalable severity** from training exercises (Severity 5) to full crisis simulations (Severity 1)

The genius of the system is that it makes tabletop exercises genuinely fun while surfacing real gaps in skills, access, coordination, and tooling. When someone fails a roll because their capability sheet honestly reflects limited experience with a service — that's a finding that maps directly to a training investment.

👉 **[Get the book on Amazon](https://www.amazon.com/dp/B0F1FQJBP3)**

---

## Scenarios

| File | Severity | Title | Threat Actor | Duration |
|------|:---:|-------|-------------|:---:|
| [sev5-cost-spike-llm-jacking.md](sev5-cost-spike-llm-jacking.md) | 5 (Training) | The Cost Spike: LLM Jacking | Opportunistic credential abuser | 4–6 phases |
| [sev4-scattered-spider-help-desk.md](sev4-scattered-spider-help-desk.md) | 4 (Routine) | The Scattered Spider Help Desk Call | Social engineering group | 6–8 phases |
| [sev3-lazarus-cryptocurrency-heist.md](sev3-lazarus-cryptocurrency-heist.md) | 3 (Significant) | The Lazarus Cryptocurrency Heist | North Korean state-sponsored | 10–12 phases |
| [sev2-salt-typhoon-telecom-pivot.md](sev2-salt-typhoon-telecom-pivot.md) | 2 (Major) | The Salt Typhoon Telecom Pivot | Chinese state-sponsored | 12–14 phases |
| [sev1-full-spectrum-attack.md](sev1-full-spectrum-attack.md) | 1 (Crisis) | The Full-Spectrum Attack | Top-tier nation-state | 12–16 phases |

### Severity Scale

| Level | What to Expect | Best For |
|:---:|---|---|
| 5 | Training exercise. Learn the mechanics. | First-time teams, new players |
| 4 | Routine incident. Coordinate and prioritise. | Building confidence |
| 3 | Significant incident. Teams will need to stretch. | Experienced teams wanting a challenge |
| 2 | Major incident. Full containment may not be achievable. | Pressure-testing coordination |
| 1 | Crisis. Everything happens at once. | Advanced teams only |

---

## How to Use These

1. **Get the book** — You need the BREACH rule system for dice mechanics, capability sheets, and game flow
2. **Pick a scenario** — Start with Severity 5 for new teams, work up from there
3. **Assign a Game Director** — One person runs the scenario (reads the Director Notes section)
4. **Players fill out capability sheets** — Honest self-assessment of skills and access
5. **Run the session** — Director narrates, players respond, dice determine outcomes
6. **Debrief** — Discuss findings, identify gaps, pick 3 things to improve

Each scenario includes everything the Director needs: situation brief, environment description, adversary profile, timeline, trigger event, key decision points, and pacing notes.

---

## Creating Your Own Scenarios

These five cover a range of threat actors and severity levels, but the format is straightforward enough to create more. The key ingredients:

- A realistic AWS environment (specific services, configurations, logging state)
- A threat actor with clear motivation and capability ratings
- A timeline of what happens if the team doesn't intervene
- A trigger event that kicks off the simulation
- Key decision points that force interesting choices
- Skills and areas the scenario is designed to exercise

AI tools are particularly good at generating new scenarios in this format — give them a threat intelligence report and ask for a tabletop scenario using this structure.

---

## License

This work is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). You're free to share and adapt these scenarios for any purpose, including commercial, as long as you provide attribution.

---

## Disclaimer

All scenarios in this repository are entirely fictitious and created solely for training purposes. Organisations, environments, and events described are fictional. Any resemblance to real incidents or organisations is coincidental. These scenarios are community-created companion content and are not officially affiliated with or endorsed by the author of BREACH.

---

*Created by [GenAI Gator](https://github.com/GenAIGator)*
