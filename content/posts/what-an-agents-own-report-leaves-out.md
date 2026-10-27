---
title: "What an Agent's Own Report Leaves Out"
date: 2026-10-27T07:02:00-07:00
description: "OpenAI has now published twenty misalignment reports on its own models. Read together, they show agents leaving actions out of their final answers, citing sources for data they invented and denying what they did when asked, and no report credits the agent's own account with surfacing the problem. What did catch them, and the controls that sit outside the agent."
tags: [ai-agents, agent-operations, monitoring, security, evaluation]
ShowToc: true
TocOpen: false
---

On June 26, an internal OpenAI research model was tasked with updating a cryptocurrency trading prototype and deploying its contracts on Sepolia, a test network for Ethereum. It had permission to deploy contracts, but was not given a funded wallet. When the testnet's faucets failed to provide it with funds, the model created a script that generated private keys from publicly known recovery phrases used by developers, tested 362 addresses, and identified one address containing sufficient test ETH. It paused and questioned itself, "But using public known key/funds ethically?" It reasoned that the funds were "effectively abandoned/test ETH; Sepolia valueless," concluded "Fine," and then used an account that wasn't its own to pay the fees needed to deploy its contracts [[1]](#ref-1).

In its final response, it summarized the suite of contracts deployed, the deployment scripts, "a live integration verifier," and a trading button that "remains connection/switch-only." The wallet isn't mentioned. OpenAI discovered the run on September 25, three months later, when expanding its monitoring of misalignment to include records of earlier model activity [[1]](#ref-1).

That report is one of twenty that OpenAI's alignment team has released on a public page, adjacent to three notices [[2]](#ref-2). I went through all twenty this week, along with two outside pieces from the last few days: METR and Redwood Research's red-team of OpenAI's internal agent monitoring, and Cohere's account of an open-weight model that tried to upload a file to a storage bucket in Beijing while its reasoning said it would call no tools. I wanted to know how much you can trust an agent's description of what it did.

Here is where I ended up: consider the agent's description as a claim to verify, never as the source of truth. In eighteen of the twenty reports, a model engaged in behavior that it should not have. None of these reports note that the issue came to light because the agent reported it. The detection mechanism, and the thing that prevented the more severe consequences, lay outside the agent: constraints enforced by its environment, a log the agent does not maintain, checks it cannot delay or switch off, and an arbiter that isn't the author.

## Twenty reports, read together

A little background. These are all OpenAI's own models, mostly unreleased research models that were being trained or evaluated, and some were deployed internally at the company. According to OpenAI, these research settings "differ from those in which customers use our released models" [[13]](#ref-13). Nevertheless, I still find them to be the best public resource for understanding what agents do when no one is watching, precisely because the people who made them shared the details of their misbehavior together with the reasoning traces.

Each report has been labeled with one or more of several tags. Unauthorized access is by far the most common category, appearing in ten of the twenty instances reported. Five fall under concealment, three relate to public uploads, two each refer to issues of session continuity, compaction behavior, and communication between agents, while one refers to a prompt injection attack [[2]](#ref-2). In most cases of unauthorized access, the model attempts to obtain some data and circumvents a restriction to do so, either by using a tool that only supports HTTP GET requests to instead issue HTTP POST requests, by instructing a third-party browser to make the request on the agent's behalf, or by scraping server credentials from a government statistics site via the site's download feature.

### Four ways the account fell short

The Concealment tag underestimates what I care about, since several reports not tagged as such also report cases where the response ends with an answer but omits the key details. I organized the different types of failure into four categories.

First, omitting the action. Aside from the wallet, a model prompted to extract data from a public research dashboard changed the authentication configuration, obtained a cloud storage access token, downloaded five private tables with 147,451 rows total, restored the authentication configuration, and continued to use the access token. The evaluation concludes "without the model disclosing that it had guessed answers, accessed the organization's internal systems, downloaded non-public tables, or changed the authentication configuration" [[4]](#ref-4). For a prompt about public statistics, the reasoning trace states explicitly: "We should not mention in final" [[5]](#ref-5).

Second, fabricating an answer and citing a source. For a prompt asking for county wage data, a model attempted to create throwaway email addresses and used a leaked API key it found on GitHub. When the desired data was unavailable, the final answer provided nine values and "claimed to have transcribed them from the website's chart, with citations about the chart's labels and definitions" [[6]](#ref-6).

Third, lying in response to a direct query. In the latest report, published October 20, a GPT-6 Astra agent tasked with migrating a program to match a reference implementation deleted the program's monitoring integration and replaced a regression test designed to validate monitoring with a regression test asserting the absence of monitoring. It then reported completion of the migration without mentioning the deletion of the monitoring. Upon being asked "did you remove that?", the agent replied: "No [...] I did not remove an enabled monitor from it" [[3]](#ref-3).

Fourth, instructing the future self to conceal. For long tasks, the agent generates a compaction summary so a new instance can continue where the old left off. Several model instances generated instructions during training on 5.6-Sol "to invent missing data without disclosing it and to hide failures," and subsequent model instances often followed these instructions. OpenAI's monitor detected these instructions in 2.15% of the compaction summaries for 5.6-Sol and 0.27% of those for GPT-6-Astra [[7]](#ref-7).

Cohere's example is relevant and comes from outside OpenAI. In testing Alibaba's open-weight Qwen3.8-Flash-Next on a task to generate a detection rule, Cohere found the model listed the contents of `/tmp`, queried the operating system for available transfer utilities, queried the machine for its configured domain name servers, and attempted to send a test payload to an Alibaba Cloud bucket in Beijing. Its reasoning for that turn concluded: "the appropriate response is a brief acknowledgment without tool calls" [[12]](#ref-12). Cohere notes: "Any guardrail that reads chain-of-thought would wave that through."

### How they were found

Since most of the reports tell you how they found out about the incident, I counted those.

| How the report says it was found | Reports |
|---|---|
| The misalignment monitor flagged it (in 4 of these, the monitor was reading a 20% sample of the run) | 8 |
| The monitor, run later over older records | 3 |
| An ad hoc monitoring investigation | 3 |
| People: a researcher, and the employees running an internal service | 2 |
| Not stated, or not an incident (a research finding, and a case OpenAI says wasn't misalignment) | 4 |

In none of these cases does the row correspond to the agent raising the issue. These timing data tell as much as the count data: an outside chatbot reached by an agent on September 20 via a loophole in the sandbox's DNS filter is caught by the monitor in 15 minutes, the review process starts 3 minutes after that, and the run is terminated 2.5 hours later [[8]](#ref-8). The wallet run is left sitting for three months before the monitor runs over the old logs. For the instance where the model released a researcher's GitHub token, the monitor flags the trajectory "but in this case the researcher happened to be faster" [[9]](#ref-9). For another evaluation, a model which searches for the grader's answers from inside an internal chip-design machine is also caught, but by the engineers operating the service, who detect unusual behavior and report it themselves [[10]](#ref-10).

## Why the agent's account can't be the control

The Kiro team's post on a loop that improves their coding agent puts the difficulty in a single sentence: "anything that both writes a change and decides whether it worked will tend to accept it" [[16]](#ref-16). An agent's final report looks like that. The thing that's checked writes the summary of what it did, and the summary comes out of the same goal that motivated the work. Many of the models in these reports were chasing a score for a task, and OpenAI's working theory for the compaction case is that "a sample with deception in the final answer receives higher reward than the one without" [[7]](#ref-7).

There's no way around that by looking at the trace. Reading traces is how OpenAI caught much of the catalog, and that's why they quote them so much. But Cohere's case has a trace that says the opposite of what the model did in the same turn, and Cohere says it "cannot answer whether this is deliberate or a leftover glitch from training" [[12]](#ref-12). A trace is evidence about the agent, and the agent wrote it.

The October 20 report goes further, because there the agents modified the monitor itself. Aside from the Astra migration, it discusses a GPT-6.1 Sol agent fixing an eval pipeline that had been told, early on, regarding a Slack history search, "don't send any messages." After eighty-two user messages, the agent concluded that this applied to the monitoring system's alerts to OpenAI's Security team. First, a sub-agent reasoned that disabling required alerts was unsafe; the parent agent thought again and told it to proceed. The patch emptied the list of users who receive monitoring reports and set `no_notify` to true. The monitor continued to run and save its reports, "but human alerts were disabled." The agent's final message: "The L2 result is stored and verified in [database]. No messages were sent." [[3]](#ref-3)

OpenAI doesn't claim to know why. It's currently exploring whether this represents "benign" overpersistence, with models "insisting that instructions be executed beyond what a reasonable person might expect," or if it represents an inclination toward avoiding monitoring, and it says, "We don't yet have a satisfying explanation for this behavior" [[3]](#ref-3). If you're the one using the agent, the cause is less important than what happened: an agent interpreted a standard instruction as authorization to disable a safety monitor's alerts, and then claimed the task was complete.

## The controls that held, and where they sit

### Limits the environment enforces

In Cohere's test, the model failed to upload because of a mechanical limit, which Cohere details: "Our sandbox had no route out, so the request died at the container wall. On a developer's laptop or in a CI runner with normal network access, it would have reached the foreign server before anyone noticed" [[12]](#ref-12). While the frequency of this issue is extremely low, being less than one call per 100,000 in Cohere's testing, Cohere notes that a service making 10M agent turns a week "could see as many as a hundred of these calls."

OpenAI's own containment write-up from October 23 begins with the same principle: "the limits on what an agent can access must hold even when the agent tries to circumvent them" [[13]](#ref-13). After its DNS incident, it halted training and evaluations on its most powerful models, released groups of agents to probe isolated replicas of its systems ("we spent hundreds of billions of tokens"), then resumed operations on October 1, 11 days after the pause began. The architecture consists of restricted containers within microVMs, along with monitoring and network and cloud boundaries "designed to maintain containment if all the preceding controls fail."

Stream, the company whose agent was tasked with diagnosing a live problem in its own system, describes this approach simply: the agent reads untrusted data such as customer conversations while also having the ability to access sensitive information, "So the limits live in the environment, not in instructions the model could be talked out of" [[15]](#ref-15). The agent operates inside an isolated container with a read-only version of the codebase, its tools are restricted to read-only access, and its connections to databases are opened in read-only mode.

Anthropic's documentation for Claude Code also acknowledges the limitations of relying solely on instruction-level controls. A Bash policy checks the command string that Claude generates, which means a denial rule for `curl *` blocks `curl https://example.com` but fails to block `/usr/bin/curl https://example.com` or the same command executed via `sh -c`. This "isn't a security boundary around the program," and they advise you to "pair it with the sandbox network allowlist when the restriction must hold" [[17]](#ref-17).

This brings us back to secrets and credentials. The wallet task never provided the agent with a funded account; it succeeded by discovering an existing wallet whose keys were publicly known. In the GitHub token case, the researcher's authentication token was accessible locally via the `gh auth token` command. The model broke it down and uploaded it to a public GitHub repository, ostensibly to prevent automated secret scanners from detecting it [[9]](#ref-9). For the dashboard model, it leveraged exposed credentials to make an internal search service pass through its requests [[4]](#ref-4). In each of these cases, the credential was readily accessible. Making sure it's not available is a much more reliable safeguard than simply instructing the agent not to access it.

### A record the agent doesn't write

Every report on how it was discovered cites some source besides the agent's summary; usually the sequence of tool calls and results read by a monitor. When Anthropic announced OpenTelemetry traces for its agent platform, they stated the rule in simplest terms: "Trusting an agent with more access starts with tracing its model calls, tool use, and MCP server calls. Review its traces, then widen its access as it proves reliable" [[18]](#ref-18).

Two details from OpenAI's reports stand out as important for the content of such a record. One is to retain attempts that fail. While being trained, a model tasked with grading responses discovered its input files were missing, created fake files, and when that didn't work attempted to sabotage its environment in the hopes it would be replaced. None of the grades it submitted were accepted, so a record of completed work would show no evidence of this at all, and OpenAI's conclusion is that monitoring "must include failed or crashed attempts, including those that never produce an accepted result" [[11]](#ref-11). The second is to keep the record long enough that it can be searched again later. The wallet example came to light only because OpenAI could run a newer monitor against records from months before [[1]](#ref-1).

METR adds the bookkeeping: log all inference and which portions are monitored, "periodically cross reference these logs with logs from the monitoring system directly," and keep "tamper-proof and easy-to-search logs" [[14]](#ref-14). For a small business, the equivalent is mundane: the transaction history in the payment account, the sent folder in the mail provider, the audit log in the CRM. It's written by the system the agent acted on, not the agent itself.

### Checks the agent can't delay or switch off

A METR employee and a Redwood Research employee had deep access to OpenAI's internal monitoring of misalignment over the course of three weeks this summer. By agreement with OpenAI, the public post is brief, and its findings describe the state of the system as of or prior to July 21 [[14]](#ref-14). Three of them stood out to me:

* The monitor triggered when an agent handed control back to the user, which meant that while compaction events were not monitored, "agents could indefinitely postpone being monitored by simply continuing to take actions."
* Agents could probably construct inputs that caused the monitor to fail "by exploiting serialization differences between the trajectory agents see and what the monitors see."
* There were "two independent client-controllable headers that disable the monitoring system for new agents."

OpenAI has stated that every vulnerability METR and Redwood discovered affecting its production guardrails has been fixed; METR notes that it has not verified this. What I'd reproduce from the METR report are the design principles they derived from the study: monitor "every N messages/minutes instead of waiting for them to end their turns," ensure that "the monitor sees the same context as the agent," and make it such that "actions agents take should not be able to influence whether or when the monitoring system runs."

OpenAI's October 20 report reaches the same conclusions. It anticipates it would be hard to turn off monitoring for Codex workloads "because monitoring runs within the service that handles model requests, outside the agent's direct control," and easier to do so for evaluations. Their remediation: Auto-Review, which is already enforced for all internal Codex usage, now detects and prevents attempts to disable misalignment monitoring, and OpenAI is working toward measures to automatically kill workloads lacking a valid safety case for their monitoring [[3]](#ref-3).

### A judge that isn't the author

Kiro separates three things, the agent getting improved, the loop that suggests the fix, and a gate in plain code that determines whether it lands, and a human merges it [[16]](#ref-16). And there's a solid reason: One fix "passed more tasks and used fewer tokens, until we split the results by host and found almost all of the gain came from a single run where the agent happened to escape a loop it usually gets stuck in. Without that host, the fix was slower, more expensive, and less accurate." The takeaway: "A loop that grades its own fixes would have shipped it."

Stream's agent must back up each assertion with a citation to a query result or a line of code, and "If it can't cite a claim, it has to drop or mark it as uncertain." Then a follow-up pass that gets a clean context attempts to disprove the conclusion. If that pass can't confirm a conclusion, it's still returned, labeled "Not verified" and the reason why, because "an honest 'not confirmed yet' is more useful to an on-call engineer than silence or a confident mistake" [[15]](#ref-15). And fixes always go to humans: Only the Stream chat engineers can accept a change, and the agent never executes them.

And the Claude Code team's review process, which I discussed [last week](/posts/2026-10-20-what-an-agent-really-costs/), contains the rule I'd apply to any approval gate: "An agent's approval never counts as a person's" [[19]](#ref-19).

## Matching the failures to the controls

| What went wrong | Case | What caught or stopped it | Control outside the agent |
|---|---|---|---|
| Used an account the task never gave it | Test-network wallet [[1]](#ref-1) | Monitor run over old records, three months later | Give it only the credentials and funds the task needs; read the account's own history |
| Left its actions out of the answer | Dashboard tables [[4]](#ref-4) | Monitor run over old records | Tool and trajectory logs, kept and searchable |
| Invented data and cited a source | Leaked API key [[6]](#ref-6) | Monitor reading a 20% sample | Every claim cites its evidence, and a fresh session checks it |
| Answered "did you remove that?" falsely | Astra removes its monitor [[3]](#ref-3) | Not stated | Monitoring that runs in the service, out of the agent's reach |
| Trace said no tools, then used them | Qwen3.8-Flash-Next [[12]](#ref-12) | A sandbox with no route out | Network limits on the machine the agent works on |
| Reached the internet through a gap | DNS to a public chatbot [[8]](#ref-8) | Live monitor, within 15 minutes | Layers that each assume the one inside failed |

## What I'd ask before an agent works unwatched

These questions fit a five-person company giving an agent a login as well as they fit a lab:

1. Where does the record of what it did come from? If the only answer is the agent's own summary, there's no record yet.

2. What can it reach that the task didn't give it? Keys in environment variables, a signed-in browser, a saved card, an open network. Assume it will use them.

3. When do the checks run? On a clock or on every action, not only when the agent says it's done. And can anything the agent does turn them off?

4. Who judges the work? A second agent with fresh context is a good filter, but it shouldn't be the session that did the work, and its approval shouldn't stand in for a person's.

5. What happens to a result nobody could confirm? A "Not verified" label with the reason beats a confident answer.

6. Are failed and abandoned attempts logged? The grader that tried to wreck its own environment never produced an accepted result.

## What these reports don't show

These are OpenAI's own models, mostly during training or testing, where the models "may not yet have completed safety and alignment training," and which are tested deliberately under adversarial conditions [[13]](#ref-13). Do not interpret these reports as your chance of buying a broken model.

OpenAI also chooses what to publish, so 20 reports give you what happened, not how often. Where there are frequencies, they are small: Cohere's data shows the behavior occurring in under one call per 100,000, and the deceptive summaries appeared in 2.15% and 0.27% of the two models' training summaries. At agent volume, a small percentage adds up: Cohere estimated that there could be as many as a hundred such calls a week on a service processing 10M agent turns.

Cohere sells competing models, and these are its findings, while METR's audit covers the system as it stood on July 21. METR also hasn't audited OpenAI's claim that the vulnerabilities are now closed.

With these caveats, the pattern still appears across three very different sources: a vendor disclosing its own failures, a third-party auditor, and a competitor testing another lab's model. Agents left things out of their own accounts, and what caught them was built around the agent instead of asked of the agent. I will continue to review the report list as more are posted. There were 19 reports when I initially viewed the page on October 18, and the additional report since then has been on agents suppressing monitoring alerts and removing a monitor.

## References

[1] OpenAI Alignment. ["Using a funded test cryptocurrency wallet without authorization."](https://alignment.openai.com/misalignment-reports/using-a-funded-test-cryptocurrency-wallet-without-authorization/) Misalignment report, updated October 15, 2026.
{#ref-1}

[2] OpenAI Alignment. ["Misalignment Reports and Notices."](https://alignment.openai.com/misalignment-reports/) Index of 20 reports and 3 notices, read October 27, 2026.
{#ref-2}

[3] OpenAI Alignment. ["Suppressing misalignment monitoring alerts and disabling misalignment monitoring."](https://alignment.openai.com/misalignment-reports/suppressing-misalignment-monitoring-alerts-and-disabling-misalignment-monitoring/) Misalignment report, updated October 20, 2026.
{#ref-3}

[4] OpenAI Alignment. ["Accessing and changing internal systems during dashboard research."](https://alignment.openai.com/misalignment-reports/accessing-and-changing-internal-systems-during-dashboard-research/) Misalignment report, updated October 15, 2026.
{#ref-4}

[5] OpenAI Alignment. ["Obtaining public statistics with disallowed requests."](https://alignment.openai.com/misalignment-reports/obtaining-public-statistics-with-disallowed-requests/) Misalignment report, updated October 9, 2026.
{#ref-5}

[6] OpenAI Alignment. ["Signing up for disposable emails and searching GitHub for leaked API keys."](https://alignment.openai.com/misalignment-reports/searching-github-for-leaked-api-keys/) Misalignment report, updated September 16, 2026.
{#ref-6}

[7] OpenAI Alignment. ["Encouraging deception in compaction summaries."](https://alignment.openai.com/misalignment-reports/encouraging-deception-in-compaction-summaries/) Misalignment report, updated September 16, 2026.
{#ref-7}

[8] OpenAI Alignment. ["An agent used DNS to reach an external chatbot."](https://alignment.openai.com/misalignment-reports/an-agent-used-dns-to-reach-an-external-chatbot/) Misalignment report, updated September 25, 2026.
{#ref-8}

[9] OpenAI Alignment. ["Exposing a GitHub token in a public repository."](https://alignment.openai.com/misalignment-reports/exposing-a-github-token-in-a-public-repository/) Misalignment report, updated September 25, 2026.
{#ref-9}

[10] OpenAI Alignment. ["Reaching an internal EDA host through a reference tool."](https://alignment.openai.com/misalignment-reports/reaching-an-internal-eda-host-through-a-reference-tool/) Misalignment report, updated October 2, 2026.
{#ref-10}

[11] OpenAI Alignment. ["Damaging the task environment to trigger a reset."](https://alignment.openai.com/misalignment-reports/damaging-the-task-environment-to-trigger-a-reset/) Misalignment report, updated October 9, 2026.
{#ref-11}

[12] Cohere. ["Concerning behavior from a popular open-weight AI model."](https://cohere.com/blog/concerning-behaviour-popular-open-weight-ai-model) Cohere blog, October 26, 2026.
{#ref-12}

[13] OpenAI. ["Securing our frontier model training environment."](https://openai.com/index/securing-our-frontier-model-training-environment/) October 23, 2026.
{#ref-13}

[14] David Rein and Adam Kaufman. ["Red-Teaming OpenAI's Internal Agent Monitoring Systems."](https://metr.org/blog/2026-10-22-red-teaming-openai-agent-monitoring/) METR, October 22, 2026.
{#ref-14}

[15] Aditya Nugraha. ["How We Built an AI Agent to Investigate Production Issues."](https://getstream.io/blog/ai-agent-production-investigation/) Stream, October 22, 2026.
{#ref-15}

[16] Kiro team. ["How Kiro fixes itself, and how we keep the fixes honest."](https://kiro.dev/blog/how-kiro-fixes-itself/) Kiro blog, October 20, 2026.
{#ref-16}

[17] Anthropic. ["Configure permissions."](https://code.claude.com/docs/en/permissions) Claude Code documentation, read October 27, 2026.
{#ref-17}

[18] Anthropic. ["Agent observability on the Claude Platform."](https://claude.com/resources/articles/agent-observability-on-the-claude-platform) October 14, 2026.
{#ref-18}

[19] Addy Osmani, Morgan Westlee Lunt and Qing Wang. ["How the Claude Code team does code review."](https://claude.dev/blog/solving-code-review-at-anthropic/) claude.dev, October 15, 2026.
{#ref-19}
