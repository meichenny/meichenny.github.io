---
title: "Where Agent Output Gets Lost Before It Ships"
date: 2026-11-03T06:59:00-08:00
description: "Coding agents multiply the work at the start of software production, and two new studies measure how little of it reaches the end: in open source, commits up about 240% and releases up about 30%; inside 655 companies, code waiting longer for review. Where the extra output goes, layer by layer, and what decides how much of it ships."
tags: [ai-agents, agent-operations, productivity, code-review, research]
ShowToc: true
TocOpen: false
---

PostgreSQL 19 is late. It's been roughly ten years that the team delivers each new major version in the window between mid-September and mid-October, but on October 30, Robert Haas wrote that it "most likely won't be released until November" [[4]](#ref-4).

There are only two sentences explaining why. Language models have made finding bugs "vastly easier," and a bunch more people are reporting the bugs they're finding. Language models also make fixing bugs easier, "but not to the same degree," and any fix has to pass through a human committer.

It's the cleanest public example I've seen of a phenomenon two papers measured this year on a larger scale: When an agent speeds up the first stage of a process, the improvement at every stage after is smaller, since they're still moving at human pace.

Demirer, Musolff, and Yang measure in open source that, when you integrate all three generations of AI coding tools, developers see their number of commits rise by about 240%, while their number of releases rises by about 30% [[1]](#ref-1).

Inside 655 companies, Fiona Chen and James Stratton find that integrating coding agents raises the number of pull requests opened by 35.7%, and the number of days from a pull request being submitted to being merged rises by 56.1% [[2]](#ref-2). In their own words, "Code review becomes the bottleneck."

I referenced the NBER abstract in a post two weeks ago on [the cost of an agent](/posts/2026-10-20-what-an-agent-really-costs/). After reading the two papers in full, here's how the specifics would change what I'd do with that information.

In brief, the output that drops off the pipeline early is mostly low-cost drafts that weren't ever going to get shipped, which isn't a big deal. The high-cost drops are finished work sitting in front of the limited number of people authorized to approve it, and work that actually makes it out but is never used.

## Six layers, one shrinking number

Using public GitHub data on over 500,000 developers and Microsoft's internal telemetry about their AI tool use (2022 to 2026), the NBER paper [[1]](#ref-1) compares each developer who adopts a tool to a similarly active developer in the same week one year ago.

It then tracks six dimensions of developer output: changes to code lines, affected code files, commits, pull requests, repositories worked on, and releases.

The analysis focuses on three types of tools: autocomplete (suggesting code while typing, such as GitHub Copilot when it launched), "Sync" agents (co-working on the developer's own computer, such as running Claude Code locally), and "Async" agents (taking a task and executing it independently until completion, like OpenAI's Codex cloud service or GitHub's agent).

| Layer | Autocomplete | Sync agents | Async agents |
|---|---|---|---|
| Lines changed | +234% | +958% | +1,255% |
| Files touched | +50% | +265% | +83% |
| Commits | +30% | +153% | +61% |
| Pull requests | +18% | +86% | +69% |
| Repositories touched | +12% | +47% | +19% |
| Releases | +9% | +20% | not measured |

The table is the paper's Table 6, rounded: the effect of adopting each generation, measured 21 to 30 weeks after adoption. Since developers tend to add new tools on top of old ones, the paper adds columns for its headline numbers (around 240% more commits with all three tools). Releases are measured per repository and can't be divided between the agent and the developer, so there's no release figure for async agents. Read down any column and the number drops. Autocomplete moves from +234% on lines changed to +9% on releases, and sync agents from +958% to +20%. As the authors put it, "Developers write much more code and work on more features after adoption, but ship only modestly more software."

There's one exception in the async column: pull requests increase slightly more than commits do (+69% vs +61%). The authors argue that this means the agent is operating at the pull-request layer directly, because it opens PRs of its own volition rather than simply increasing what's passed up from lower levels. Autocomplete operates solely at the level of code writing, while the newer tools operate at higher levels, which is why the authors suspect the difference between task-level productivity gains and shipped software to close in time: "may narrow over time as tools automate higher layers of the hierarchy."

They've also checked the obvious counter-arguments. A comparison with a design run on non-AI coding tools found no effects. Agents could be filling their code with comments, but the proportion of comment lines in released software didn't move in that direction (it fell from 18.6% in 2021 to 17.4% in early 2026), and that wouldn't even explain increased numbers of repos or releases.

## Where the extra work goes

There are two mechanisms that might generate that shape. One is scope: each release contains more than it used to, so counting releases undercounts the work done. The other is experimentation: there are far more things tried, and most of them don't make it to the end.

Scope accounts for some of it. Lines per commit rise by 67% and files per commit by 22% with sync agents. In the adopters' own repositories, the median release grows from 5.21 thousand lines in 2021 to 13.42 thousand lines in early 2026, a factor of 2.6, and from 48 to 76 files. This is "nowhere near what would be required" to explain the difference between layers, the authors note.

Experimentation explains most of it, and it shows up in three different places. At the level of individual lines, sync agents increase the number of lines added by 7.0 times and the number of lines deleted by 12.2 times, meaning more of what's written gets rewritten or abandoned. At the level of features, the number of created branches goes up by 79%. At the level of entire projects, the share of newly created repositories with no further activity after the first month increases from 60% in the 2021 cohort to 69% in early 2026. It rose by about 1.2 percentage points per year through 2024, and then jumped 3.9 points in 2025, the year agentic tools became available.

The authors don't view the discarded work as wasted work, and I think they're right. Where strong complementarities are involved, "developers simply lack the bandwidth to process every candidate at the later stages, where human input is essential and AI is less effective." People might be comparing two versions that are equally good, or keeping the best of multiple attempts. A draft that costs an agent a few minutes and shows you what not to do is a loss I'll gladly pay every time. Their summary of the mechanism: "AI enables developers to try far more, while human bottlenecks at the later stages filter out most of these attempts."

The paper also fits a model to the data. Each layer takes the work that comes from below and adds its own human effort; the degree to which gains propagate to higher layers depends on the substitutability between the two. Fitted to the autocomplete data, the elasticity of substitution turns out to be about 0.23, meaning that the upstream output and the human input are "strong complements, combined in roughly fixed proportions." A version of the model without any complementarity fits the data more than 100 times worse.

In practical terms: past a certain point, more drafts from below won't translate into more output unless humans contribute their part at the next layer. "In the case of complementarities, even unbounded automation yields only finite gains," the authors say.

There's one additional detail I keep returning to. The model also allows developers to reallocate their own time across layers, e.g., from writing to reviewing, and there is no evidence in the data that they have done that yet. The authors interpret that as an indication that we're still early on a J-curve, where the benefits of a new technology depend on complementary changes.

## Inside companies, the loss is a queue

Most of the open-source developers in the NBER data are solo or work in very small groups, and the authors note that this constraint "should bind more tightly inside firms, where shipping code involves review and approval processes that open-source developers largely avoid." This is precisely the environment that Chen and Stratton examined [[2]](#ref-2). Jellyfish, a software-as-a-service that helps companies keep tabs on software engineering projects, provided them with 445 million work events across GitHub, Jira and Google Calendar for 720,542 employees at 655 companies from January 2021 to August 2026. The authors compare firms that started using coding assistants and agents at various moments in time. My version of the paper is dated Nov. 1.

The gains aren't quite as large as in open source, but they have a similar profile. Following agent adoption, lines of code increased by 44.0%, commits by 36.0% and pull requests opened by 35.7%. Completed work, measured as closed Jira issues, grew by 13.7% (marginally significant) and closed epics by 18.0%; the authors' estimates exclude an increase in output exceeding 30.7%. They also tested for whether issues were simply getting more complicated and didn't find any evidence of that, nor did they find any detectable impact on employment. A major part of why a huge boost to coding translates to such modest growth in output is because coding doesn't constitute the bulk of what engineers do: they're spending 40.8% of their time writing code according to Jellyfish, and 30.8% according to a survey the authors administered to 100 engineers.

Then the authors turn to reviewing. After agents were introduced, the interval between opening a pull request and merging it climbed 56.1%, from 3.56 days on average. The authors divide that into two pieces, which I believe is the best aspect of the paper. For pull requests that received review feedback, the period waiting for a review to start (i.e., the queue) climbed 59.9%, from 2.78 days, and the time under review itself climbed 58.5%, from 3.98 days. There were 38.6% more comments per pull request, while the size of each pull request remained roughly constant. The lengthening of the wait reflects a scarcity of review capacity. The lengthening of review time and higher comment count reflect greater effort needed on a pull request once review starts, which the authors describe as "consistent with a decline in the quality of submitted code." Both of these components rose after agent adoption.

Why don't firms shift more people to reviewing? Because reviewers must understand the codebase. In the data, 82.8% of reviews took place in a codebase where the reviewer had previously committed code or opened a pull request; if reviews were assigned at random to pull requests within the same firm, that figure would be 17.2%. Seniority further concentrates review work: senior engineers complete 75.7% of reviews for pull requests written by senior engineers. A junior engineer explained to the authors:

> "People could make a huge number of commits quickly, but code review was still the bottleneck. On one project, only one senior engineer was allowed to approve changes because he was the expert on the codebase. He was so bandwidth constrained that he could only review my code once a week. After some outages that were caused by AI-generated commits, there were also new rules requiring senior engineer approval for AI-generated code, which created even more review bottlenecks."

So far, AI review tools have not been able to eliminate that bottleneck. As of July 2026, 79.1% of the firms used AI tools for code review, but only 23.3% of comments made in code reviews were made by AI, and just 10.8% of pull requests contained at least one AI comment. A senior engineer told the authors why most teams keep humans in the seat: "The AI tools lack the comprehension of the overview of the entire feature, and what it is supposed to do."

Chen and Stratton also fit their model to the data. According to it, AI agents increased coding productivity by 73.7% and decreased review productivity by 7.0%, causing firms to shift workers' time from coding (down 22.7%) to review (up 43.4%).

Chen and Stratton then project forward to 2031. If review productivity remains constant, the firm's output increases by 57.8%, eventually plateauing when the bottleneck is entirely in code review. But if review productivity improves at the same rate as coding productivity, output increases by 341.4%. In both cases, the increase in coding productivity is the same. But the second one is almost six times higher because of what happens to code review.

## Four cases from the last few weeks

These are aggregate numbers. Since August, we've seen four reports: from a single engineering team, from a single open-source project, from a single large company, and from a single platform. Each one illustrates this same pattern in more detail.

In Will Larson's post on Imprint, the engineer average is 4x more pull requests merged than two years ago, and two teams are at 10x [[3]](#ref-3). Quality also increased: "incidents are down 42% half-over-half, and critical incidents are down 78% half-over-half." But "The p50 pull request review took about 33 minutes to complete 2 years ago, but now takes us about 77 minutes," and "The slope here correlates tightly with the number of pull requests getting merged."

PostgreSQL's numbers are all public [[4]](#ref-4). From the April 8 feature freeze through September 25, commits exclusively going into the main branch went down slightly from 2025 to 2026, while back-patched commits were up more than 60% (from 724 to the release branch in 2025 to 1,169 in 2026). Back-patches that also went into the prior release went up from 248 to 433, more than 74% in a year. Three major features were rolled back between August 27 and September 15. Haas explained: "LLMs can draft fixes, but those fixes aren't always correct or compliant with PostgreSQL standards. So, they still need to be reviewed and committed by humans, and those humans have not gotten any smarter between 2025 and 2026." The review time that would've gone into the release "got diverted into addressing security issues and other bugs in the back-branches." He's not worried about the delay itself, which "emerges directly from the project's concern for quality."

Reuters, reporting in August on internal posts it saw, said code changes to the internal platforms and infrastructure Meta employees use were up 220% year over year, according to an early June post from CTO Andrew Bosworth. "But changes that led to new or upgraded features reaching Meta users were only up 36%" [[6]](#ref-6). The NBER authors point to this as evidence of their attenuation effect happening within a single firm.

Even the middle platform has hit its limit. GitHub's August 17 outage took seven hours and 47 minutes. Its CTO, Vlad Fedorov, wrote that "Both incidents were capacity failures at their core" (the other being an Actions outage on August 6) [[5]](#ref-5). "Since April, monthly commits have grown from 1.4 billion to 2.9 billion." The GitHub blog post does not state what portion of this growth came from agents. To put things in perspective, the NBER paper puts commits attributed to Claude Code alone at more than 7% of all public commits by mid-2026, and writes that this is likely an underestimate of the total for AI-assisted programming.

## The last layer is someone using it

But shipping isn't the end either. The NBER authors also looked at four large software marketplaces [[1]](#ref-1). New iOS apps have been running around 33,000 to 45,000 a month from 2023 through early 2025, then grew to around 108,000 per month by April 2026. New releases on Google Play went from a peak of around 145,000 per month in 2020 to around 42,000 in January 2025, then rebounded to around 99,000 by mid-2026. New Chrome extensions have grown roughly 7-fold since 2023.

Usage hasn't kept up though. Summed across the new apps from each month over the next three months, iOS ratings have remained largely unchanged from 2024 to 2025, Android downloads have increased somewhat, and Chrome downloads have decreased (they were decreasing beforehand anyway). The proportion of new apps that don't reach even a small audience within three months (less than 10 iOS ratings, less than 10 Chrome downloads, and at most 100 Android downloads) has gone up in every marketplace between January 2025 and April 2026: from 78% to 87% on iOS, 19% to 33% on Chrome, and from about 22% to 26% on Android. The authors describe it as "the marginal apps are increasingly invisible to end users."

They provide two interpretations of this result, saying that their data cannot distinguish between them. Perhaps the additional apps are worse. Or perhaps user attention, which includes search results, algorithmic recommendations, and word of mouth, is finite, and is not increasing along with the number of apps that compete for it. The latter interpretation is the one I think about most, since it generalizes to anything a business can now produce more of cheaply: apps, articles, social media posts, sales emails. Producing more just puts more stuff in front of the exact same number of people.

## What I'd change

These are my conclusions drawn from reading the papers and looking at the case studies. They are not what the authors recommend. I wrote these with a software team in mind, but Chen and Stratton write that the distinction between "implementation" and "management" or "review" is "nearly ubiquitous across knowledge work settings." So if a business uses agents to write proposals, generate invoices or respond to customer questions, it will have the same sequence of steps but with different names.

### Count at the layer you get paid for

Commits, pull requests and drafts are inputs. Imprint reports merged pull requests next to incidents; Chen and Stratton count resolved issues.

If your company has an agent on proposals, you want to count proposals sent and won, not drafts made; if your company has a support team, you want to count tickets resolved without a reopen.

If all you measure are the metrics in the upper rows of the table above, every agent is going to look like a 10x improvement.

### Measure the wait separately from the work

The Chen and Stratton split is easy to duplicate: time spent waiting for someone to review something before they look at it, and time from when they start looking until the item is finished.

When the first time increases, you either need more capacity for reviewing, or fewer things going into the queue.

When the second time increases, and the comments pile up, then what arrives needs to have more work done to it, which is something that should be fixed with the agent's instructions and checks, rather than demanding that reviewers turn over items faster.

### Find your one reviewer

In most software projects, there are one or two people capable of approving changes, the ones who know the code. That is what the 82.8% figure and the once-a-week senior engineer are all about. If it's a small company, it's usually the owner. It's the PostgreSQL problem with committers: if you add agents but don't add their time, then you've just made their approval queue longer.

Before hiring a second agent, I would ask who is going to review its work, and how many hours they have available per week to do so.

### Decide the fast lane in code

Imprint's answer is a "small-change" lane [[3]](#ref-3). A CI check that people can't edit decides whether a pull request makes the cut: fewer than 1,000 lines, no p1 or p2 issues from the AI review bot, an author with at least 10 merged pull requests in that service, few services touched, no shared libraries, and per-path exclusions like a service that handles money. A review skill approves these pull requests unless they involve sensitive parts like payments or security. It has "only two options: approve or ignore." A slip that happens is considered "a failure in our system," and will be corrected in the review bot, verification, CI or the harness. So far, "about 10% of changes qualify."

You can apply the same principle outside of code. Write down which types of agent outputs can be sent after a quick glance, reserve money and customer exceptions for humans, and treat mistakes in the fast lane as reasons to narrow the criteria.

### Don't answer AI review with more AI

Larson calls one pattern "meat proxy review." An overwhelmed review team applies their own harness to the pull request, produces a pile of comments with varying quality, has to decide which of those comments the reviewer actually endorses, and then uses another harness to respond to the first harness's comments. He describes it as "a bottleneck-pretending-to-be-a-solution." He suggests putting energy into one shared review process and the loop that corrects the issues it finds, and then simply approving.

### Let drafts die cheaply, before they reach a person

NBER evidence says most of the lost output is in exploration, and agents are good at exploration.

And then it costs once you start sending requests, asking for people's time with every single draft. Let the agent and its checks do the culling, such that you get the one version worth reading instead of the twelve the agent tried.

## What these studies can't tell us yet

The NBER data include open source projects and four marketplaces, little enterprise software, and ends mid-2026, so they're showing you what was around at the time [[1]](#ref-1). The developers in them haven't yet rearranged their schedules. If they do, the gap between levels may narrow.

Chen and Stratton look at adoption per firm, averaging across those who barely use the tool, and their 2031 figures are projections, not data [[2]](#ref-2). In marketplaces, there's less than a year and a half of data post-agentic tools, and no way to tell whether an app is worse or people's attention is thinner.

Imprint is one firm, early in its experiment, and its incidents fell even as volume rose [[3]](#ref-3). That's a good note to keep: it is possible to produce more while checking more, and thus far that extra capacity has been achieved by altering the verification and the review loop, not by having humans read faster.

The biggest unknown in all this is whether review can get quicker. That's where the two scenarios Chen and Stratton envision diverge by almost six-fold, and most firms using AI-assisted review tools already do it largely manually. The NBER paper puts it simply: "In software, the binding constraint appears to be shifting from writing code to reviewing, integrating, and distributing it."

I'm going to watch median review time for teams publishing theirs next year, along with the number of changes they merge.

## References

[1] Mert Demirer, Leon Musolff and Liyuan Yang. ["Writing Code vs. Shipping Code: Productivity Effects Across Generations of AI Coding Tools."](https://www.nber.org/papers/w35275) NBER Working Paper 35275, May 2026, revised September 2026.
{#ref-1}

[2] Fiona Chen and James Stratton. ["Artificial Intelligence in the Firm: Bottlenecks in Software Production."](https://fion.ac/jellyfish.pdf) Working paper, Harvard University, current version November 1, 2026.
{#ref-2}

[3] Will Larson. ["Speeding up code review with small-change."](https://lethain.com/small-changes/) Irrational Exuberance, October 17, 2026.
{#ref-3}

[4] Robert Haas. ["Postgres 19 is late, and it's AI's fault."](https://rhaas.blogspot.com/2026/10/postgres-19-is-late-and-its-ais-fault.html) October 30, 2026.
{#ref-4}

[5] Vlad Fedorov. ["The August 17 outage, and the work ahead."](https://github.blog/news-insights/company-news/the-august-17-outage-and-the-work-ahead/) The GitHub Blog, August 20, 2026.
{#ref-5}

[6] Katie Paul. ["Mark Zuckerberg had a bold plan to replace Meta staff with AI. Here's how it imploded."](https://www.reuters.com/investigations/mark-zuckerberg-had-bold-plan-replace-meta-staff-with-ai-heres-how-it-imploded-2026-08-26/) Reuters, August 26, 2026.
{#ref-6}
