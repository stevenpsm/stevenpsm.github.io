---
title: "How Far Have AI-Enabled Attacks Progressed? From ARTEX in South Korea to Risks Below the Waterline"
date: 2026-10-14T12:00:00+08:00
draft: false
tags: ["English", "AI Security", "Threat Research", "Threat Hunting", "Detection Engineering", "Incident Response"]
categories: ["Research and Analysis"]
description: "What the ARTEX campaign against South Korean financial organizations tells us about visible AI-enabled attacks, less visible risks, and practical defensive priorities."
lang: "en-US"
alternateLang: "zh-CN"
alternateURL: "/posts/ai-attacks-visible-trends-hidden-risks/"
xDefaultURL: "/posts/ai-attacks-visible-trends-hidden-risks/"
ShowToc: true
TocOpen: false
---

> This article is available in two languages: **English** · [中文](/posts/ai-attacks-visible-trends-hidden-risks/)

While reviewing recent cases of AI-enabled cyberattacks, I kept returning to one question: are we looking at a genuinely new offensive capability, or are we giving an old attack chain a more conspicuous label?

The ARTEX campaign against South Korean financial organizations offers a useful vantage point. It involved an AI agent, several models, attacker-controlled infrastructure, and real data exfiltration. It also left behind sessions, configuration files, and memory artifacts. Yet if we remove the word “AI,” the entry points remain familiar: internet-facing peripheral systems, authentication and authorization gaps, and vulnerabilities left unpatched. What changed was how the attacker organized those capabilities and how quickly they could move across targets.

That distinction is the boundary I want to preserve here. One incident is not proof that fully autonomous cyberattacks have arrived. At the same time, poor operational security in several public cases should not lead us to dismiss the longer-term effect AI may have on how offensive work is produced.

The analysis below is based on public material collected through October 10–11, 2026. For disclosed incidents, I have tried to rely on first-party reporting. For activity below the visible surface, I treat the claims as testable hypotheses rather than using invisibility itself as evidence.

The diagrams retain their original Chinese labels; the surrounding text provides the full English analysis.

## 1. The ARTEX campaign: AI first amplified existing weaknesses

![The ARTEX campaign and visible AI-enabled attack trends](/posts/ai-attacks-visible-trends-hidden-risks/01-artex-visible-ai-attack.png)

In October 2026, CrowdStrike disclosed activity targeting South Korean financial organizations. Researchers found Claude Code sessions, ARTEX configuration files, and memory files in open directories controlled by the attacker. These artifacts exposed two environments: one server hosting an ARTEX instance and another that appeared to function as the operator’s main workstation. ARTEX used DeepSeek as its primary model backend, while other sessions involved GLM and Grok. The public report did not establish exactly how the two environments were connected. [CrowdStrike’s original disclosure](https://www.crowdstrike.com/en-us/blog/unknown-threat-actor-uses-artex-to-target-south-korean-finance/) also preserves uncertainty around the total number of affected organizations and the personal clues found in the exposed sessions.

It is tempting to summarize the case as “an AI agent autonomously breached several banks,” but the available evidence does not support such a complete narrative. Public reporting points to peripheral systems used for loan inquiries, mobile work, and branch operations; it does not show that core online banking systems were compromised. Nor does it document, organization by organization, which steps AI actually performed.

What can be stated more confidently is that the campaign encountered several familiar weaknesses: poorly governed internet-facing assets, gaps in authentication or object-level authorization, known vulnerabilities that had not been remediated, and a meaningful interval between initial activity, detection, and containment. AI did not change the nature of those weaknesses. It made reconnaissance, execution attempts, result parsing, and repeated adjustment easier to sustain as a continuous process.

Amazon’s investigation of a separate FortiGate campaign makes the same shift more explicit. The attacker did not rely on new FortiGate vulnerabilities. Instead, they used exposed management interfaces, weak credentials, and single-factor authentication to compromise more than 600 devices across over 55 countries. When hardened environments resisted the standard workflow, the attacker often moved on to softer targets. Amazon’s conclusion was appropriately restrained: AI increased efficiency and scale, not technical depth. [The investigation](https://aws.amazon.com/blogs/security/ai-augmented-threat-actor-accesses-fortigate-devices-at-scale/) points to an uncomfortable consequence of capability diffusion. Its first effect may not be to raise the ceiling of elite operations, but to lower the floor of attackers that poorly defended organizations must withstand.

The ARTEX case also matters for a different reason. The attacker exposed configuration, session, and persistent memory files in open directories, while a cross-server reference gave investigators a path to additional infrastructure. AI tooling can increase operational throughput, but it also creates new, information-dense forensic artifacts. Sessions, project instructions, memory, model configuration, and tool-call records can reveal far more than an isolated file when exposed together.

That investigative success carries strong survivor bias. We can see the activity so clearly because the attacker’s operational security was poor. The evidence shows that this workflow exists; it does not show that more disciplined operators will leave the same opening.

## 2. The operating model is shifting from generation to orchestration and feedback

![The evolution of AI attack orchestration and autonomy](/posts/ai-attacks-visible-trends-hidden-risks/02-ai-attack-orchestration.png)

Generating phishing text, modifying a script, or explaining a vulnerability are not new capabilities. The more consequential change is that models are moving from one-shot advisers into stateful execution loops: plan a task, invoke a tool, interpret the result, correct a failure, and carry the outcome forward in operational memory.

Humans do not disappear from this structure. They move upward into the control plane: choosing targets, preparing infrastructure and credentials, approving consequential actions, judging whether results are credible, and handling exfiltration or monetization. AI increasingly occupies the execution plane, taking on the middle of the workflow where people previously had to switch repeatedly between tools and manually consolidate results.

Public cases broadly show three forms of participation:

- **Orchestration:** models connect reconnaissance, exploitation, credential handling, and data analysis across multiple stages.
- **Runtime integration:** malicious software calls a model during execution to generate or adjust its next action.
- **Social-engineering assistance:** models create multilingual text, audio, or visual material, while people still control delivery, pressure, and the movement of funds.

Anthropic’s disclosure of GTG-1002 is one of the stronger orchestration cases available. The company reported that the actor used Claude Code across reconnaissance, vulnerability discovery, lateral movement, credential collection, and data analysis, while humans retained control over target selection and final extraction decisions. [Anthropic’s report](https://www.anthropic.com/news/disrupting-AI-espionage) demonstrates that long execution chains are operationally plausible. Its autonomy estimates, attribution, and “first documented” framing nevertheless come from the platform provider and should be distinguished from independent validation.

Rather than debating whether a case deserves an L3 or L4 autonomy label, I find four engineering changes more useful:

1. One-shot prompting is becoming a stateful task loop.
2. Tool output can feed directly into the next action.
3. Failure feedback can drive script changes, path adjustment, and retries.
4. A small number of people can maintain throughput that previously required more specialized roles.

These changes can compress an attack timeline, but they do not remove hallucinations, edge cases, or environmental differences. The actor observed by Amazon could generate tools at scale but could not reliably compile custom exploits or adapt to nonstandard environments. An agent may run a familiar path much faster without having learned how to navigate unfamiliar terrain.

## 3. Below the waterline: stronger models are not automatically stealthier

![The visibility boundary and hidden AI-enabled attack hypotheses](/posts/ai-attacks-visible-trends-hidden-risks/03-hidden-ai-attack-iceberg.png)

Public cases pass through a selection filter. A platform saw anomalous use, an attacker exposed infrastructure, a victim detected an intrusion, or a third party happened to capture a sample. What we observe is therefore not the population of AI-enabled attacks; it is the subset that became visible.

This distinction matters. The wealth of readable material in the ARTEX and FortiGate cases does not mean AI users are generally careless. The number of cases disclosed by model providers does not mean little is happening through self-hosted models, open-weight models, or third-party relays. The public distribution reflects both underlying activity and the mechanisms available to discover it, and those effects are difficult to separate.

I would frame the less visible risk as three hypotheses that can be tested independently.

First, model supply is becoming more distributed. Self-hosting, open weights, multiple providers, and API relays reduce the visibility available to any single platform. Less platform visibility does not mean there is no trace inside the target organization. It shifts more responsibility back to endpoint, network, identity, and cloud-control-plane telemetry.

Second, AI may allow a small number of skilled people to cover more specialist functions. We have evidence that some capabilities exist, but not evidence of their prevalence. A successful evaluation can show that a capability is possible; it cannot show how many attackers use it reliably.

Third, high stealth is more likely to come from a combination of capability class and human discipline than from an agent’s default behavior. High concurrency, regular timing, and redundant enumeration are usually noisy. Operators pursuing stealth may place AI earlier in the process—target research, tooling preparation, and offline analysis—then slow the execution phase and keep human control over authorization and exceptions.

This third proposition remains a hypothesis. It at least yields observable counterevidence. If highly regular, high-frequency agent activity repeatedly remains undetected for long periods, the assumption that automation is inherently noisy must be revised. If high-OPSEC cases continue to show human intervention around critical decisions, defenders should not limit their search to obvious signs of model use.

AI also expands the enterprise’s own exposure. It can be a weapon, a target, and an agent: model credentials, API quotas, development frameworks, and plugins become assets worth stealing; enterprise agents connect untrusted input to internal tool permissions. Defending against AI-enabled attacks therefore requires more than identifying the model an adversary used. Organizations also need to know which identities, data, and execution rights their own AI services possess.

## 4. Role compression: the authorization boundary matters more than headcount

![Specialist role coverage, human authorization, and defensive observation](/posts/ai-attacks-visible-trends-hidden-risks/04-human-ai-collaboration.png)

Describing an AI system as equivalent to a fixed number of security specialists is attractive and usually misleading. Tasks differ sharply in how easy they are to verify. Reconnaissance, information classification, script generation, and result normalization are highly patterned. Models can absorb a large share of this repetitive work. Exploit stabilization, adaptation under strong defensive pressure, target selection, and monetization still depend heavily on experience, risk judgment, and access to real-world resources.

A more accurate description is that AI compresses the cost of covering roles; it does not replace a team at a stable ratio. A small number of experienced operators may cover preparation, tooling adaptation, and data analysis with AI, but only under bounded conditions.

| Easier for AI to supplement | Still strongly dependent on people |
| --- | --- |
| Bulk reconnaissance, OSINT processing, routine scripting and tool use | Target valuation and operational boundaries |
| Result parsing, data classification, reporting, and knowledge organization | Deep work in unknown environments |
| Adaptation, retries, and parallel execution within known patterns | Exploit stabilization, result validation, and risk tradeoffs |
| Multilingual content and initial social-engineering material | Live interaction, trust building, and monetization |

The more important defensive question is how authorization changes. Human attention once imposed a natural limit on operational frequency. Now a single approval can release many parallel tasks. Boundaries an engineer once crossed manually may be traversed continuously by an agent through its tool permissions. The permissions may be unchanged while the blast radius is not.

Detection should therefore cover more than whether AI was invoked. It should include task scope, tool calls, identity transitions, stop conditions, and result validation. The same applies to an organization’s own agents: model self-reporting cannot replace system logs; high-risk actions cannot rely on prompt instructions alone; irreversible operations require independent approval; and outcomes should be checked against business facts or deterministic validators.

From the attacker’s side, I treat “a small expert group plus an AI task pool” as a hypothesis about organization, not as a proven template. It predicts stylistic or contextual links across artifacts from different stages, as well as human timing around consequential actions. A future, independently validated campaign that sustains a low-visibility end-to-end chain without human gates would force that hypothesis to be rewritten.

## 5. The defender’s home ground: control terrain and detect paths, not models

![Terrain control and attack-path interruption](/posts/ai-attacks-visible-trends-hidden-risks/05-defender-terrain-control.png)

Defenders can drift toward two unhelpful extremes. One is to wait for new IOCs and a universal “AI attack signature.” The other is to conclude that rapidly improving models have made existing defenses irrelevant. Both positions overlook what an organization still controls: its terrain, identity system, business semantics, and authority to act.

Whatever model an attacker uses, they still need to cross necessary paths: enter through an external service, acquire an identity or token, execute on a host, reach internal tools, and ultimately read, export, or damage data. Paths can change, but they cannot disappear entirely. Defense should not depend on a single control. It should make alternative routes costly and ensure that every boundary crossing leaves facts that can be correlated.

I divide the defensive and response problem into four layers.

**First, reduce and shape the terrain.** Isolate management interfaces, continuously inventory internet exposure, prioritize remediation using both vulnerability and exposure, introduce short-lived credentials and just-in-time access, support session revocation, and preserve object-level authorization around critical data. Both the South Korean campaign and Amazon’s FortiGate investigation show that AI first consumes neglected peripheral systems, reused credentials, and unconstrained management planes.

**Second, build cross-source visibility.** Endpoints should record processes, scripts, and model-related egress. Networks should record connections and data flows. Identity systems should record authentication, token use, and privilege changes. Cloud audit trails should preserve API calls and resource changes. Any one signal may be ordinary. Together they describe a path. Detection needs to move from “this tool appeared” to “this identity acted on this asset, over this interval, and produced this business outcome.”

**Third, return low-severity signals to the chain.** An unusual VPN login, access to a credential store, a remote-management connection, model API egress, or bulk data access may not justify an alert in isolation. When the same identity and assets connect those events within one time window, their meaning changes. Amazon’s FortiGate guidance similarly prioritizes anomalous authentication, DCSync activity, lateral movement from VPN address pools, and access to backup credential stores rather than treating the presence of legitimate open-source tools as proof of compromise.

**Fourth, design response authority in advance.** As attacks accelerate, the bottleneck is often not the absence of a smarter model. It is that nobody is pre-authorized to revoke a session, isolate a device, or suspend a high-risk operation. Reversible actions with a defined blast radius can use deterministic policies. Broad or irreversible actions should require human approval. Every intervention should then be verified independently through system logs and business outcomes. The value of response automation is to remove delay from known actions, not to delegate final judgment to another agent.

Passive deception also belongs in this terrain. Canary credentials and decoy endpoints cannot guarantee a trigger, and they do not replace segmentation or least privilege. They can, however, increase the exposure cost of indiscriminate enumeration. “Counterattack” in this context means controlling one’s own environment, using passive deception, investigating evidence, and taking authorized defensive action—not intruding into attacker infrastructure.

After working through these cases, I am less interested in ending with a call for another “AI-native security product.” The more durable lesson from ARTEX is simpler: offensive production is becoming continuous, parallel, and stateful, while many enterprise defenses remain fragmented across asset inventories, identity logs, EDR, cloud audit, and separate response authorities.

Models will continue to improve, and the specific model used by an attacker may never be visible. What defenders can accumulate is clearer asset ownership, shorter identity chains, more complete local telemetry, and a response system that knows when to act automatically and when to stop for human judgment.

None of this guarantees that every step will be blocked. It does make the attacker’s required path longer, reduces the number of cheap alternatives, and prevents a local failure from automatically becoming a full-chain compromise. That is a more reliable objective than chasing an imagined universal signature for AI-enabled attacks.

---

Primary references:

- [CrowdStrike: Unknown Threat Actor Uses AI-Driven ARTEX to Target South Korean Finance](https://www.crowdstrike.com/en-us/blog/unknown-threat-actor-uses-artex-to-target-south-korean-finance/)
- [Amazon Threat Intelligence: AI-augmented threat actor accesses FortiGate devices at scale](https://aws.amazon.com/blogs/security/ai-augmented-threat-actor-accesses-fortigate-devices-at-scale/)
- [Anthropic: Disrupting the first reported AI-orchestrated cyber espionage campaign](https://www.anthropic.com/news/disrupting-AI-espionage)
