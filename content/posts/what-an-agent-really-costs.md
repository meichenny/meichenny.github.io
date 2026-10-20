---
title: "What an Agent Really Costs"
date: 2026-10-20T07:05:00-07:00
description: "Token prices keep falling, and they were never the expensive part. An AI agent's real cost is the human time around it: briefing, waiting, reading, reviewing and rescuing. What the newest data measures, what it only assumes, and how to count it per accepted task."
tags: [ai-agents, agent-operations, productivity, evaluation, human-in-the-loop]
ShowToc: true
TocOpen: false
---

One metric in a report Anthropic put out on October 16 on how work has changed internally has been sitting with me since then. In interactive mode, when someone guides an agent through a task, the sessions are "around 80% idly waiting for human input" [[1]](#ref-1). The agents move quickly; they wait for us.

In this post I'm looking at what an AI agent actually costs when it's getting stuff done. My quick answer: the token bill is the smallest piece and shrinking the quickest. What makes or breaks an agent is the human time surrounding it, drafting the brief, fielding questions, reviewing output, editing, fixing mistakes, even picking up the slack where needed. Very few people ever publish this metric. One of the most impressive agent efficiency savings I've seen published this year relies on a guess at this figure, rather than a real measurement.

I'd start by measuring both human minutes and agent spend per accepted task (counting abandoned and rescued tasks), along with a quality bar to ensure that time saved in review isn't just traded for time spent in cleanup later. Then the big question is how to allocate our time. The best publicly available case study I know of is how the Claude Code team uses agents to review code, which I'll detail below.

## The part of the bill that keeps shrinking

Vercel releases a monthly index based on traffic passing through its AI Gateway, which is what teams use to send requests to various model providers. In the October issue, they note that September's gateway spend was about 13x what it was in December, but that the average price per token was 47% lower [[2]](#ref-2). The median paid customer who was active in both months more than doubled their spend. Teams also swap models quite regularly: in September, nearly 1 in 2 teams changed their top used model, and almost 1 in 3 teams changed their top used lab. As Vercel writes, "The model is becoming a replaceable part of a much larger investment."

Two important caveats: first, this data reflects Vercel's traffic, which represents teams building production applications using Vercel, rather than all companies. Second, the fact that tokens became cheaper did not result in lower bills; instead, the volume of tokens increased at a faster rate than their costs declined. However, for any given task, the price of the model keeps going down, and it has become commonplace to move standard workloads to a more cost-effective model. If the cost of an agent were largely determined by token usage, this article would be much shorter.

## Where the time goes

The Anthropic report, by economists Nathan Wilmers, Regina Catipon, Maxim Massenkoff and Peter McCrory, is based on internal company data on employment, AI use, output, surveys, Slack, calendars, and hiring, plus 17 interviews with employees [[1]](#ref-1). They're the firm's stats on itself, at a firm that, in the report's words, "has thoroughly molded its workflows around AI." I'm reading them as an early glimpse of what high usage looks like, not as a prediction of how other people will run their businesses.

The output side is substantial. By August, the agents that individual employees were directing had logged six times as many hours of runtime as the humans giving them directions. (Here agent time is runtime, which includes inference and active tool use but not any wait time over 30 minutes; human hours assume a 40-hour week.) Code commits per engineer were 5 times higher than they had been six months prior. Lawyers were processing more contracts, accounts payable was moving more invoices, and salespeople were closing more deals.

The attention side is where the cost lies. First, waiting. Subtracting the agents' active working time from the wall-clock time of each session produced the 80% figure above; one engineer said people are "bottlenecking the agents." Then, reading. Translating the output tokens into reading time (0.75 words per token, 250 words a minute), the interactive sessions of even non-technical groups generate enough text to require "over half of the work week" to read.

Then, the management work. Meta-work, writing specifications for multi-agent runs, having one agent review another agent's output, monitoring pipelines, grew from 10% of the average employee's interactive sessions in fall 2025 to 22% in spring 2026. Written standing instructions for agents went "from a few hundred in July 2025 to 8,500 a year later." One engineer "set up an agent chief of staff to triage agent requests." Meetings also grew: the median employee spends 20% more time in meetings than a year ago, and among technical individual contributors meeting time is up 16%, although the report notes that some of this is due to the company growing.

Another engineer noted that dealing with agents feels like babysitting because every new request comes "with an equal demand for your attention." The report concludes that "human attention is a constant bottleneck."

A separate paper finds the same shape far beyond one company. Mert Demirer, Leon Musolff and Liyuan Yang combined data on over 500,000 GitHub developers and their AI usage telemetry [[3]](#ref-3). Autocomplete, interactive coding agents and autonomous coding agents increased commits by 30%, 180% and 240% respectively. But the 240% fell to 80% for the number of projects, and to 30% for actual releases. The authors interpret this as a weak-link pattern, with AI's gains "attenuated by human bottlenecks in the production chain." Agents do much more work at the task level, and less of it makes it to the end, because a person still needs to decide, check, and ship.

## The ten-minute assumption

If human time is the cost we care about, we would expect agent-saving studies to account for it. A recent study I've read closely doesn't. 

In June, researchers including Perplexity employees shared a preprint covering Perplexity's Search and Computer products, using production data [[4]](#ref-4). In 10,000 pairs of almost identical queries inputted into each, "Computer performs 26 minutes of autonomous work per user session, versus 33 seconds for Search." It's a measurement, and a noteworthy one. The study then estimates that Computer reduces the time needed to finish a comparable job from 269 minutes to 36 minutes, which it states as 87% of time and 94% of cost saved.

The 269 minutes come from pricing each "do" step in a Computer session (running a command, writing a file, editing one, working in a web app) at an estimated human time. The Computer half of the analysis receives only a brief description: "Computer's human time is fixed at 10 minutes of oversight per task (e.g., writing the prompt and reviewing the output)." The authors acknowledge it. A footnote on an alternate proxy states that it "does not capture the verification cost for Computer outputs so its fixed cost might be underestimated," and a sensitivity analysis shows the time benefit persists if the oversight assumption is multiplied by 24.

Michael Cutler looked at that sensitivity calculation from the other direction [[5]](#ref-5). Twenty-six minutes of automated time plus 240 minutes of manual oversight yields approximately 266 minutes, compared to the 269-minute baseline. According to his calculations, the time advantage "survives" at the point where it's effectively zero, and the savings are "anywhere between 87% and nothing." His concluding comment is that the paper "has measured how much the machine does and assumed how much the human still has to."

I don't believe the study is dishonest; it's explicit about its caveats. What I'm arguing is that this is what most agent ROI figures resemble: the machine portion has been tracked, and the human portion is a static number chosen by an author. The bias direction is also relevant. Verifying a 26-minute Computer session that created files and executed commands will likely take more time than verifying a 33-second Search response, even when the subsequent human questions are similar.

Even our own intuition of the savings may be wrong, and we have substantial data to suggest so. In METR's randomized trial involving veteran open-source software engineers, conducted in early 2025, developers took 19% longer when permitted to use AI, yet afterward still thought they were 20% faster [[6]](#ref-6). METR now claims those findings no longer reflect the impact of present-day systems. Its February 2026 revision is helpful in another respect: measuring the time taken has become more difficult. Some participants "would often work an unrelated task while waiting for the agent to complete its work," and METR reported it could not reliably measure time taken for developers running multiple agents simultaneously [[7]](#ref-7). The wait time Anthropic calculated shows up again as a measurement issue.

## A unit that counts failures

Next, Cutler provides an idea of what should be measured, which he explicitly calls "a proposed method, not a report of an experiment I have completed" [[8]](#ref-8). It's the most useful advice I've seen on the topic, so let me summarize it.

Choose one task class, like bounded bug fixes, sourced research notes, or standard data reconciliation. Document the criteria that classify a task as belonging to that class and record every instance prior to beginning work, otherwise you'll only be quantifying the best cases.

Each business task is the unit you count; all agent efforts on it are categorized by it. If the agent's first draft is rejected and it provides two subsequent drafts, that is a single task, three attempts; opening a new conversation does not reset the price tag.

Next, track the amount of human effort spent on active work in five buckets:

| Category | What it covers |
|---|---|
| Briefing | Defining the request, assembling context, explaining constraints |
| Supervision | Reading progress, answering questions, redirecting attempts |
| Review | Checking the result against its acceptance criteria |
| Rework | Correcting the result, or replacing it after a failed attempt |
| Verification | Running and inspecting the checks acceptance needs |

While the agent is left running in the background, pause the timer. Include all charges related to that job, including retries, tool usage, and automated checks. "An evaluator agent also costs money," says Cutler.

Don't reset the ID when a job fails, or you'll lose the count of how many times you tried. And if you forget to log the timer for a run, that's missing information, not 0 seconds.

Then, to get your unit costs, divide everything by the tasks that were accepted:

```text
human minutes per accepted task = all cohort human minutes / accepted tasks
agent spend per accepted task = all cohort agent spend / accepted tasks
acceptance rate = accepted tasks / started tasks
```

"All" also means time and money spent on tasks that were either rejected or abandoned. A task a person completed manually is classified as a manual takeover, because "Otherwise a successful rescue can look like successful delegation."

His synthetic example: three tasks cost 45 human minutes and $6 in agent and tool fees, two are accepted and one is abandoned. That's 22.5 human minutes and $3 per accepted task. If he only divided the cost of the two successes by two, the abandoned work wouldn't show up.

The step I would emphasize precedes all of this: decide beforehand what conditions a task class needs to meet in order for the task class to be worth delegating, which include a human-time budget, a quality threshold, and the number of takeovers you're willing to accept. Only then does the data give you enough information to know whether to keep going, modify your workflow, or stop delegating that class.

He is also clear about what the ledger won't capture. It can show you where review time is being spent, but establishing a saving requires a proper comparison with completing the same type of task without the agent, at the same quality bar.

## One good run proves very little

A second reason cost-per-accepted-task will likely exceed the demo estimate is that as Nikola Balić writes for Steel, which provides browser infrastructure for agents (thus, this is a vendor's argument), an agent run is what reliability engineers call a series system: it works only if each step works [[9]](#ref-9). In his example, if there are seven steps in the workflow, each of which typically succeeds, the overall success rate is 61.8%, meaning 38% of runs fail. This is a simplified scenario, with independent steps and only one attempt at each, he points out; real-world workflows will have added dependencies, retries and fallbacks. But the general conclusion remains the same. That a run succeeded in front of you doesn't tell you much about the overall failure rate. His recommendation mirrors Cutler's: evaluate solutions based on "cost per accepted result, not cost per second of compute," and account for retry costs and review effort when calculating.

## A worked case: code review at agent volume

The best public description I have found of a team actively doing this work is by Addy Osmani, Morgan Westlee Lunt and Qing Wang of the Claude Code team, describing their approach to reviewing code [[10]](#ref-10). While these are internal stats from Anthropic, and they're operating in the software domain, the principles carry over.

It was volume that pushed them to make the switch. The daily number of changes landing in Anthropic's monorepo grew from around 2,500 in May to more than 30,000 in September, and Claude generates around 90 percent of Claude Code's own code. No one can review all of that manually. Thus they took a metric of their own that is close to the unit Cutler suggests. They aim for most PRs to "merge with at most one human in the loop," without increasing revert rates or incident counts.

Human time is tracked via human approvals per PR (median and 95th percentile), human comments, and wait times. The limit is set by the revert/incident rates: "if they rise, we haven't saved human time, we've only moved it." That line does a much better job than me of summarizing this post.

Machines take the first pass. Before a PR is created, an agent runs tests and builds, and another agent reviews the diff in a fresh context. Each PR then receives an automated review by multiple agents, with a different set voting on findings before publishing. A separate security review also runs, and no PR can merge if any open findings remain from either.

Finally, human approval requirements depend on risk. A classifier looks at the content of a change, not the author, and assigns a level:

| Risk | Examples | Human approvals |
|---|---|---|
| Low | Docs, copy, internal-only features, additive changes behind a flag | None, once automated review and security checks are green |
| Medium | Ordinary product changes; sensitive areas such as caching, the auto-updater or system prompts | One person |
| High | The highest-risk code, plus a small set of paths that always need two people | Two people |

There are a few rules on top. "An agent's approval never counts as a person's." The person who directs the agent counts as one of the reviewers required by the PR. Approval applies to a single commit, so any new commits that the system cannot vouch for require a new approval.

Errors are designed to be cheap to reverse. There's a merge queue, feature flags, internal use first, staged releases, and fast rollback. Rather than slow everything down, the team would "rather get faster at noticing and fixing problems," they wrote.

The section I found most useful was the one on noise. "Noise costs more when the author is an agent." When a person gets a nitpick, it takes a few seconds to wave it off. With an agent, the nitpick has to be fixed, meaning a commit and another review round. Agent authors and reviewers "can keep each other busy indefinitely." One PR went through over fifty review-and-fix cycles overnight before its author finally halted it.

When the team looked at what the reviewer kept flagging, about 59% of it was code that had been present since the reviewer's first pass. It wasn't finding any new bugs, just a few more old ones every cycle. They came up with a set of rules to stop the cycle. After three cycles, the reviewer should ask the author to take a step back and reconsider the whole approach. After the first few cycles, it should only re-review the new diff. It should flag the category of bug so that all instances are fixed at once, and it should never repeat a finding.

These are the results on the team's internal pull requests:

| Measure | Before | After |
|---|---|---|
| Pass after one review and fix round | 65% | 78% |
| Need five or more rounds | 8.9% | 1.5% |
| Mean reviews to pass | 2.08 | 1.32 |

Between June and September, merged PRs to Claude Code more than doubled, and the portion of those that had to be reverted dropped by more than 40 percent. However, the team is clear-eyed about one missing piece: it doesn't have circuit breakers that make approvals stricter when the number of reverts or incidents goes up. It wants these tools in place before more of its code goes straight to zero-human merges.

The key things humans should still do, the team lists:

- Decide if we're going to build it in the first place.
- Review high-risk code.
- Review architecture that other people will follow.
- Look at the proof.
- Be responsible for whatever you shipped.

"A person is accountable for every change that ships, whoever typed it."

## Making the human part smaller

If we lay these out next to each other, there are several categories in which to reduce the human cost per accepted task, and none of them involves hiring more agents.

### Spend attention by risk

Claude Code's three-tier setup works well: no human approval when the cost of a mistake is low and easy to correct, one person for regular tasks, and two for the most dangerous paths.

The valuable thing is having those tiers decided in advance, by looking at the change itself, rather than having to review everything or nothing.

### Give every agent loop a stopping rule

Another agent who checks an agent is also a cost, and two agents could go on forever in cycles of rounds where no one profits. It's cheap to have a limit to the number of rounds allowed, to forbid repeating a finding, and to pass on to a human after some number of rounds.

### Let a separate checker and a memory take the first pass

Cognition's Devin Oncall, a product for triaging production outages, includes a separate reviewer agent that verifies the primary agent's analysis against mistakes "such as treating correlation as causation, dismissing contradictory telemetry, or confusing a trigger with the underlying problem" [[11]](#ref-11). Cognition further says that its responders bypass roughly 60% of investigations by reusing prior investigations and clustering relevant alerts, which reduces the cost of alert response by more than half. Those are marketing claims from the vendor, but the design is worth emulating: A separate reviewer from the worker, and a memory bank of known incidents so you don't re-discover the same solutions.

### Keep a record from outside the agent, then widen access

For example, Anthropic's announcement about the availability of OpenTelemetry traces for agents reads: "Trusting an agent with more access starts with tracing its model calls, tool use, and MCP server calls. Review its traces, then widen its access as it proves reliable" [[12]](#ref-12). A Ramp engineer quoted in that post claims that a system that scores the traces of their agents has reduced token usage by 10 to 20% and improved session speed by 30% for some agents. Traces also help keep the human-time ledger clear, because they show you what the agent was doing in between your reviews.

### Write the correction down once

That's the same idea as the 8,500 standing instructions at Anthropic, and why the Claude Code team often fixes repeatable review comments "at the source" (e.g. in a skill, a lint rule or an instruction file). Any correction that a human has to make twice should be turned into an instruction that an agent reads each time.

## How I would start with one agent

This is about what I'd take away from everything I've written here, if I were a small-business owner bringing my first agent onto an actual job. Again, this isn't a tested recipe, just my synthesis.

1. Find a specific, recurring task, like coding invoices, first response in the support inbox, or a weekly report, and define what done right means, and who accepts it.
2. Before the experiment begins, decide how much of your time you're willing to spend per task, and how many takeovers you'll count as a failed experiment.
3. Let it run for a few weeks, keeping a quick log in the five categories Cutler mentioned. A quick log is sufficient; a missing log is a problem.
4. Tally everything against the tasks you've approved, including the ones you've either given up on or finished yourself.
5. Categorize the task's actions according to risk. Actions that send, pay, delete or publish require a person's approval; lower-risk items may only need occasional spot checks.
6. If you find yourself fixing the same issue twice, codify it as a standing rule.
7. Read through the log, and then decide whether to continue, adjust the system, or discontinue the process.

The goal here is to have less time spent per task approved, without any additional cleanup work.

## What nobody has measured yet

I looked for any published, measured human time to oversee an accepted agent task for general office tasks, by task category. I did not find one.

Anthropic measured wait time and estimated reading time for their own internal tasks. The Claude Code team measured approvals and wait time for code. Perplexity's paper just assumed 10 min. METR showed that it's difficult to measure time when you're running multiple agents at once. Cutler published a protocol, not results.

I think that is the most important open question on how much agents cost. We know token price, and it drops every month. But we don't have much data on the human side of the equation and it's what buyers are basing their decisions on. If you actually run agents on real tasks and keep track of how much human time was spent on each task, that data would be much more useful than another benchmark.

## References

[1] Nathan Wilmers, Regina Catipon, Maxim Massenkoff and Peter McCrory. ["How employment is changing at Anthropic."](https://www.anthropic.com/research/employment-at-anthropic) Anthropic, October 16, 2026.
{#ref-1}

[2] Vercel. ["AI Gateway Production Index, October 2026."](https://vercel.com/blog/ai-gateway-production-index-october-2026) Vercel blog, October 2026.
{#ref-2}

[3] Mert Demirer, Leon Musolff and Liyuan Yang. ["Writing Code vs. Shipping Code: Productivity Effects Across Generations of AI Coding Tools."](https://www.nber.org/papers/w35275) NBER Working Paper 35275, May 2026, revised September 2026.
{#ref-3}

[4] Jeremy Yang, Kate Zyskowski, Noah Yonack and Jerry Ma. ["How AI Agents Reshape Knowledge Work: Autonomy, Efficiency, and Scope."](https://arxiv.org/abs/2606.07489) arXiv:2606.07489, 2026.
{#ref-4}

[5] Michael Cutler. ["26 Minutes vs 33 Seconds: What Production Data Tells Us About Agentic AI."](https://cutler.sg/blog/2026-09-agentic-ai-vs-conversational-search-26-minutes) September 25, 2026.
{#ref-5}

[6] Joel Becker, Nate Rush, Beth Barnes and David Rein. ["Measuring the Impact of Early-2025 AI on Experienced Open-Source Developer Productivity."](https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/) METR, July 10, 2025.
{#ref-6}

[7] Joel Becker, Nate Rush, Tom Cunningham, David Rein and Khalid Mahamud. ["We are Changing our Developer Productivity Experiment Design."](https://metr.org/blog/2026-02-24-uplift-update/) METR, February 24, 2026.
{#ref-7}

[8] Michael Cutler. ["How to Measure the Human Cost of Agent Work."](https://cutler.sg/blog/2026-10-measure-agent-review-time) October 16, 2026.
{#ref-8}

[9] Nikola Balić. ["Build the agent. Don't build its computer."](https://www.nibzard.com/agent-computer) First published on the Steel blog, October 7, 2026.
{#ref-9}

[10] Addy Osmani, Morgan Westlee Lunt and Qing Wang. ["How the Claude Code team does code review."](https://claude.dev/blog/solving-code-review-at-anthropic/) claude.dev, October 15, 2026.
{#ref-10}

[11] Cognition. ["Introducing Devin Oncall."](https://devin.ai/blog/introducing-devin-oncall) Devin blog. Read October 20, 2026.
{#ref-11}

[12] Anthropic. ["Agent observability on the Claude Platform."](https://claude.com/resources/articles/agent-observability-on-the-claude-platform) October 14, 2026.
{#ref-12}
