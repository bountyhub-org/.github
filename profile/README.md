<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://bountyhub.org/images/logo.png">
  <source media="(prefers-color-scheme: light)" srcset="https://bountyhub.org/logo_whitebg.png">
  <img src="https://bountyhub.org/logo_whitebg.png" alt="BountyHub" width="128" height="128">
</picture>

# BountyHub

### Stop running scripts. Start running workflows.

BountyHub turns the repetitive parts of security research — recon, scanning, collecting,
sorting, and remembering — into reusable workflows that run on infrastructure you control
and keep their results as structured, queryable data.

[![Private beta](https://img.shields.io/badge/status-private_beta-FF6B35?style=for-the-badge)](https://forms.zohopublic.eu/nikolajokicboun1/form/Waitlist/formperma/OSQACzpLzui_gvtoPGSsirKqrz60amMLzJLhkMM_dJU)
[![Join the private beta](https://img.shields.io/badge/Join_the_private_beta-orange?style=for-the-badge&logo=rocket&logoColor=white)](https://forms.zohopublic.eu/nikolajokicboun1/form/Waitlist/formperma/OSQACzpLzui_gvtoPGSsirKqrz60amMLzJLhkMM_dJU)
[![Discord](https://img.shields.io/badge/Discord-Join_the_server-5865F2?style=for-the-badge&logo=discord&logoColor=white)](https://discord.gg/EnPbMJF4dx)
[![Email](https://img.shields.io/badge/Email-nikola.jokic@bountyhub.org-EA4335?style=for-the-badge&logo=gmail&logoColor=white)](mailto:nikola.jokic@bountyhub.org)

[![Website](https://img.shields.io/badge/Website-bountyhub.org-2563EB?style=for-the-badge&logo=googlechrome&logoColor=white)](https://bountyhub.org)
[![Docs](https://img.shields.io/badge/Docs-docs.bountyhub.org-14B8A6?style=for-the-badge&logo=readthedocs&logoColor=white)](https://docs.bountyhub.org)
[![GitHub](https://img.shields.io/badge/GitHub-bountyhub--org-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/bountyhub-org)

</div>

---

> ### BountyHub is entering private beta.
>
> We're looking for hackers, bug bounty hunters, security researchers, pentesters, and
> security teams to run real workflows on the platform and tell us what works, what breaks,
> and what should be built next.
>
> **Active beta testers get one year of Pro (normally $29.50/mo) free.**
>
> **[Join the private beta](https://forms.zohopublic.eu/nikolajokicboun1/form/Waitlist/formperma/OSQACzpLzui_gvtoPGSsirKqrz60amMLzJLhkMM_dJU)** ·
> **[Join the Discord](https://discord.gg/EnPbMJF4dx)** ·
> **[Email us](mailto:nikola.jokic@bountyhub.org)**

---

## Why BountyHub

Most hunting setups are a pile of shell commands, cron jobs, spreadsheets, and notes that only
one person understands. Every target you start from zero, and every result ends up buried in
terminal scrollback.

BountyHub gives that process a home:

- **Workflows you can reuse** — describe your method once, version it, and run it again on every target.
- **Runners you control** — execute on hosted runners or your own machines, with your private tooling.
- **Datasets that remember** — results become structured, queryable data instead of lost output.
- **Real-time OOB monitoring** — catch blind callbacks the moment they fire.
- **Agents with explicit capabilities** — automate the busywork with only the context and tools you grant.

Write it once, run it forever, and spend your time on the findings instead of the plumbing.

<div align="center">

**Same workflow, new target, new findings.**

</div>

---

## Everything, working together

| Capability | What you get |
| --- | --- |
| **Workflows & automation** | Chain tools, commands, transformations, and agents into workflows you can version, schedule, and rerun against any target. BHW is a real Starlark dialect, so it feels like code because it is. |
| **Datasets** | Airtable-style tables for everything you discover: subdomains, hosts, ports, URLs, scan results. Custom schemas, full history, and triggers that fire as your data changes. |
| **Self-hosted runners** | Register your own machines and invoke them from any workflow. Install proprietary scanners and private tooling, and your arsenal never leaves your infrastructure. |
| **Modules** | Package a proven sequence of steps into a reusable module with an explicit input/output contract, then compose it into any workflow. Write once, reuse everywhere. |
| **bhlast: OOB interactions** | A self-hosted Burp Collaborator alternative. Capture DNS and HTTP callbacks, register custom records for SSRF, serve exploits, and get notified on every interaction in real time. |
| **Security agents** | Bring AI agents into your workflows to triage noisy results, investigate leads, and drive tools. You grant the context, tools, and providers explicitly. Nothing runs implicitly. |
| **Artifacts & storage** | Keep command output, files, reports, screenshots, and binaries connected to the run that produced them. Blob storage sits right next to your projects and workflows. |
| **Payload management** | Store payloads in reusable groups and template them with variables. Adapting wordlists and injection strings to each target takes seconds, not copy-paste sessions. |
| **Notifications & integrations** | Scan findings, dataset triggers, and OOB interactions land in one inbox. Connect results to the tools and channels your team already lives in. |
| **Historical results** | Every run is archived. Diff results across versions, spot changes on your targets, and never lose a finding to a rotated log or an overwritten file again. |
| **Reusable templates** | Share and reuse workflows, modules, and proven recon chains. Start a new target from a template that already works instead of a blank file. |
| **Reports & export** | Turn research output into reports instead of manually stitching results together. Query your datasets and hand over something that looks like it took a day, not a week. |

---

## Write it once, run it forever

```starlark
workflow(
    activities = [
        activity(
            name = "discover",
            triggers = [trigger.cron("0 6 * * *")],
            steps = [
                step.exec(command = ["subfinder", "-d", config.var("domain"), "-o", "subs.txt"]),
                step.transform(
                    source = path.file("subs.txt").lines(),
                    pipeline = transform.trim().lower().unique(),
                    destination = path.file("subs-clean.txt"),
                ),
                step.update_dataset(
                    table = "subdomains",
                    source = path.file("subs-clean.txt").lines(),
                    pipeline = transform.unique(),
                    mapping = {"host": row.value},
                    mode = dataset.upsert(),
                ),
            ],
        ),
        activity(
            name = "triage",
            triggers = [trigger.on(discover.succeeded())],
            steps = [
                step.agent(
                    prompt = "Flag exposed admin panels on new hosts.",
                    model = model("default"),
                    provider = provider.key("anthropic/personal"),
                    output = output.json(),
                ),
            ],
        ),
    ],
)
```

A real BHW workflow: scheduled discovery, typed transforms, dataset sync, and an AI triage
step — written once and rerun on every target.

---

## From first probe to actionable data

**Discover** feeds **Transform**, **Transform** feeds **Store**. From there your data fans out:
alerts fire, AI agents pick up leads, and automated scans run in parallel before everything
merges into **Analyze and investigate**. Results loop back in, so analysis triggers agents,
scans hand work to agents, and new leads restart discovery on the next target.

The goal isn't another collection of security tools. It's a system where discovery, execution,
and data feed each other — and your best process runs again on every new target.

---

## Who it's for

| Audience | How BountyHub helps |
| --- | --- |
| **Bug bounty hunters** | Automate recon, keep every finding, and reuse a proven pipeline on every new program. |
| **Security researchers** | Turn a methodology into a repeatable workflow, capture its data, and get back to analysis and exploitation. |
| **Pentesters & red teams** | Build assessment workflows that run against the infrastructure you control, keeping client data and private tooling on your own runners. |
| **Security teams** | Continuously monitor your own attack surface. A new subdomain, port, or callback lands in a dataset, triggers a workflow, and notifies the right people. |

---

## Pricing

| Plan | Price | Best for |
| --- | --- | --- |
| **Free** | $0, forever | Testing BountyHub, or hunting a single target occasionally. |
| **Pro** | $29.50 / month | Hunters who are serious about automation. |
| **Elite** | $49.50 / month | Full-time bounty hunters. |
| **Custom** | Let's talk | Security teams and deeply customized automation. [Get in touch](mailto:nikola.jokic@bountyhub.org?subject=BountyHub%20custom%20plan) |

Every plan includes the core platform: workflows, datasets, artifacts, and hosted runners.
Prices shown are beta pricing and will be locked in for active beta testers.

---

<details>
<summary><strong>What is BountyHub?</strong></summary>

BountyHub is a bug bounty automation platform. It combines CI/CD-style workflow automation,
project management, self-hosted runners, an out-of-band (OOB) interaction server called bhlast,
queryable datasets, and payload management in a single platform for security researchers.
</details>

<details>
<summary><strong>What language are BountyHub workflows written in?</strong></summary>

BHW, a fully valid Starlark dialect — the same language family used by Bazel — extended with a
domain-specific layer for building security automation and recon pipelines.
</details>

<details>
<summary><strong>What is bhlast?</strong></summary>

A managed out-of-band (OOB) interaction server, hosted for you at [bhlast.com](https://bhlast.com).
It captures DNS and HTTP(S) callbacks from blind payloads (SSRF, blind XSS, blind RCE), lets you
register custom DNS records, serves files for blind attacks, and notifies you the moment an
interaction fires. Provision one with a click on a random subdomain under bhlast.com.
</details>

<details>
<summary><strong>Can I run my own tools on BountyHub?</strong></summary>

Yes. Self-hosted runners execute on your own infrastructure, so you can install any tool,
including proprietary and private tooling, and invoke it directly from your workflows.
</details>

<details>
<summary><strong>Is BountyHub free?</strong></summary>

There is a Free plan, and BountyHub is currently in private beta. Active beta testers get one
year of Pro free. See the pricing above for details.
</details>

---

<div align="center">

### Ready to hunt smarter?

Join the private beta and help shape the platform you'll hunt with.

**[Join the private beta](https://forms.zohopublic.eu/nikolajokicboun1/form/Waitlist/formperma/OSQACzpLzui_gvtoPGSsirKqrz60amMLzJLhkMM_dJU)** ·
**[Join the Discord](https://discord.gg/EnPbMJF4dx)** ·
**[Email us](mailto:nikola.jokic@bountyhub.org)** ·
**[Read the docs](https://docs.bountyhub.org)**

<sub>Built by <a href="https://github.com/bountyhub-org">Nikola Jokic</a> - software engineer and part time bounty hunter, bootstrapped and community driven.</sub>

</div>
