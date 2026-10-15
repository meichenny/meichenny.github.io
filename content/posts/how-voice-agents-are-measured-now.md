---
title: "How Voice Agents Are Measured Now"
date: 2026-10-15T14:55:00-07:00
description: "Speech-to-speech models used to be judged on whether they could reason over audio and take turns like a person. The newest scoreboards ask whether the agent finished the support call, how often, how fast and at what cost. What each benchmark measures, how it runs, and what it can't tell a contact-center buyer."
tags: [voice-agents, benchmarks, evaluation, contact-centers, speech-to-speech]
ShowToc: true
TocOpen: false
---

xAI published Grok Voice 3.0 on October 14, with a table of scores from Artificial Analysis dated the previous day [[1]](#ref-1). The model sits atop the Artificial Analysis Speech-to-Speech Index at 86.9, ahead of OpenAI's GPT-Live-1 (83.2) and Google's Gemini 3.8 Live (82.6). Below it come rows for speech reasoning, human preference, task success, conversational dynamics, a support-call benchmark called τ-Voice, ServiceNow's EVA-Bench, latency and cost per task.

What interests me here is what isn't in the table. There is no row measuring how natural the voice sounds. The only row based on human judgment comes from people trying to get something done on a call, like booking an appointment, who then picked the conversation they liked more. Even the main pitch is about outcomes: xAI says the model "sets a new record for resolving customer support calls."

That is what this post is about. In December 2024, the big unknown with speech-to-speech models was whether they reasoned worse when the question arrived as audio rather than text. On the test built to measure this, the top models have largely closed the gap. The benchmarks that still separate models place the agent on a simulated support call with tools, a written policy and a caller who has an accent, is in a noisy room or tends to interrupt. Then they check whether the account ends up in the correct state, and how often that holds when the same call runs again.

From here, I take the benchmarks on xAI's scoreboard one at a time, with what a score does and doesn't tell a contact-center buyer, and then cover how to test a voice agent on your own call types.

## The scoreboard at a glance

| Benchmark | Built by | Question it asks | How it runs | Main blind spot |
|---|---|---|---|---|
| Big Bench Audio | Artificial Analysis (2024) | Can the model reason over a spoken question? | 1,000 logic questions read by synthetic voices; an LLM judge grades the spoken answer | No conversation or tools; the top five score 97.7% to 99.7% |
| Full-Duplex-Bench v1 and v1.5 (subset) | Lin et al. (2025) | Does the model handle pauses, turn-taking, interruptions and backchannels? | Pre-recorded clips played to the model; its reply is transcribed and timed | Short static clips with no task |
| τ-Voice | Sierra (Ray et al., 2026) | Can a voice agent resolve a support case under a policy? | Simulated caller with accents, noise and phone audio; the final database state is graded | Simulated callers, English only, three domains |
| Speech Agent Arena | Artificial Analysis | Which hidden model did people prefer, and did it make the right final tool call? | Paid participants complete the same scenario in two live calls | 35 short scenarios with cooperative callers |
| EVA-Bench | ServiceNow (Bogavelli et al., 2026) | Did the agent get it right, and was the call any good? | Bot-to-bot audio calls on 213 enterprise scenarios, five trials each | Simulator doesn't systematically interrupt; tools are mocks |

Two things about this scoreboard matter more than any particular number. First, the older tests are saturated. The top five Big Bench Audio scores on the Artificial Analysis leaderboard range from 97.7% to 99.7%, and the top five conversational-dynamics scores from 96.9% to 98.9% [[2]](#ref-2). τ-Voice scores on the same leaderboard range from 15.1% to 74.5%. If you want to know which model is better at a job, that range is where the useful information is.

Second, the index changed what it measures. In version 1.0 of the Artificial Analysis Speech-to-Speech Index, released in June 2026, conversational dynamics carried a third of the weight. Version 2.0, in use since August 2026, dropped conversational dynamics and replaced it with task success rate [[3]](#ref-3). Half the index now rests on whether a task was completed: τ-Voice and the Arena's task success rate, at 25% each.

## Speech reasoning: Big Bench Audio

Artificial Analysis published Big Bench Audio in December 2024 to answer one specific question: does a model reason worse when it is spoken to? The dataset is 1,000 spoken versions of questions from four Big Bench Hard categories, 250 each: formal fallacies, navigation, object counting and "web of lies" logic puzzles [[5]](#ref-5). The audio uses 23 synthetic voices from top-ranked text-to-speech models. The model hears a question and answers out loud; an LLM judge then compares the transcribed answer with the official one [[3]](#ref-3).

At release, the data showed what Artificial Analysis called a "speech reasoning gap": GPT-4o scored 92% on the text version of the questions and only 66% speech to speech [[5]](#ref-5). In October 2026, Grok Voice 3.0 scores 98.5%, and the highest score on the leaderboard is 99.7% [[2]](#ref-2). For buyers, that is reassuring: on short logic questions, the best models no longer do much worse when they listen instead of read.

The benchmark has real limits, though. Each question is a standalone puzzle read in a single turn by a clean synthetic voice, with no conversation, tools, policy or customer. The scoring methodology has changed, too. Version 1.1 (March 2026) switched the judge to Claude Sonnet 4.6 and began counting unanswered questions as wrong, and version 1.2 (May 2026) reworked the judge and grader so correct answers buried in long responses still get recognized [[3]](#ref-3). A score from 2025 and one from today aren't measuring quite the same thing.

## Conversational dynamics: Full-Duplex-Bench

Since full-duplex models listen and talk at the same time, they have to choose when to speak. Full-Duplex-Bench, from Guan-Ting Lin, Hung-yi Lee and colleagues, evaluates that choice across pause handling, backchanneling, smooth turn-taking and user interruptions [[6]](#ref-6). Version 1.5 adds overlap cases: the user backchannels while the model speaks, talks to someone else in the room, or there is speech in the background [[7]](#ref-7).

Versions 1 and 1.5 are static. The benchmark plays pre-recorded clips to the model and saves what comes back. The v1 clips come from real conversations (the Candor corpus for pauses and turn-taking, the In Conversation Corpus for backchannels) and from synthetic dialogues voiced by a text-to-speech model for interruptions [[6]](#ref-6). The reply is transcribed and timed. Metrics such as the takeover rate show whether the model jumped in when it should have listened, or stayed quiet when it should have spoken. Artificial Analysis uses four of these measures, pause handling, turn-taking, user interruption handling and backchannel handling, each reported as the share of samples handled correctly [[3]](#ref-3).

A high score means the model is less likely to talk over a caller who pauses to find a card number, or to start a new answer when someone says "mm-hmm." Callers hear those mistakes.

What it doesn't tell you is whether anything got done. The clips are short and have no task attached. The top results are also very close: Grok Voice 3.0 scores 97.0%, and the leader, StepAudio 3 Realtime, scores 98.9% [[2]](#ref-2). The same model is the best at speech reasoning. It has no τ-Voice score, so it gets no index value, and Artificial Analysis measures its average time to first audio at 8.83 seconds. A model can top both of the "can it hold a conversation" tests and still not be one you'd put on a phone line.

The benchmark's own authors have followed the same trend. Full-Duplex-Bench v2 runs live multi-turn conversations with an automated examiner that enforces staged goals [[8]](#ref-8), and v3 combines recordings of real human speech, full of fillers, false starts and self-corrections, with tasks that require chained API calls [[9]](#ref-9).

## Task completion: τ-bench, τ²-bench and τ-Voice

This family of tests flipped the scoreboard, and it began with text.

τ-bench (Yao et al., 2024), from Sierra's research team, put a language agent in airline and retail customer service with API tools and a written policy, with a language model playing the customer [[10]](#ref-10). Scoring is mechanical: at the end of the conversation, compare the database state with the annotated goal state. The paper also introduced pass^k, the probability of succeeding on all k attempts at the same task. For support work, it may be the most useful idea in this post. In 2024, GPT-4o succeeded on fewer than half the tasks, and its pass^8 in retail was under 25%. τ²-bench (Barres et al., 2025) added a telecom domain where the customer also has tools and must do things on their own device, just as in a real troubleshooting call [[11]](#ref-11).

τ-Voice (Ray, Dhandhania, Barres and Narasimhan, March 2026) brings 278 of those tasks (114 retail, 50 airline and 114 telecom) onto a voice channel [[12]](#ref-12). The caller is still a language model (GPT-4.1 in the paper), but an ElevenLabs voice speaks its lines with a persona: an impatient suburban caller, an elderly woman calling about something her grandson usually handles, and callers with strong Bengali, Sichuan Mandarin, Senegalese French and Maharashtrian accents. The audio gets mixed with background chatter and traffic, bursts like phone rings and dog barks, muffling, 8 kHz G.711 phone compression and dropped frames. A language model decides when the caller interrupts or says "mm-hmm." For reproducibility, the orchestrator steps simulated audio time forward in 200-millisecond ticks rather than following the wall clock, so the simulator can run a slower, more capable model without making the call feel slow to the agent.

The paper's results were bleak. GPT-5 with reasoning, in text, solved 85% of the tasks. Voice agents from Google, OpenAI and xAI solved 31% to 51% with clean audio and 26% to 38% under realistic conditions [[12]](#ref-12). In the retail ablation, accents were the most harmful single factor on average, and the damage was highly provider-dependent: xAI's model at the time dropped 18 points with accented callers, and Google's dropped one. The authors inspected 91 failed calls and attributed 79% to 90% of the failures to the agent, not the simulator.

The largest failure point was authentication. Agents failed to capture names and emails "even when spelled letter-by-letter," which blocked every step after it. In one call, the agent told the customer "I've updated your shipping address" without making any tool call.

Artificial Analysis now runs its own version of these 278 tasks, with two standard-English caller personas plus five with varied accents and speaker profiles, and takes the mean of three trials [[3]](#ref-3). Grok Voice 3.0 scores 71.6% there, and GPT-Live-1 scores 74.5% [[2]](#ref-2), though GPT-Live-1's score is based on a single trial.

Of everything on the scoreboard, this comes closest to asking whether the agent will solve your callers' problems, and it fails the agent for a wrong change even if the rest of the call sounds fine.

But you shouldn't treat the jump from 38% in March to 71.6% in October as a clean measure of progress. The setups differ. In the paper, the simulator takes in a direct transcript of what the agent says, while in Artificial Analysis's version, the simulated customer hears the agent through Deepgram transcription. Artificial Analysis describes its personas but doesn't say whether its runs include the paper's noise and phone-line effects. Sierra has also posted more than 75 task fixes to its repository since the original releases [[13]](#ref-13). Both versions are English only, with accents produced by text-to-speech, which the authors note makes their accent results "indicative rather than definitive." Their simulated customer is also more patient than a real person, with perfect memory and instant tool responses [[12]](#ref-12). And if your calls are insurance claims or prescription refills, the airline, retail and telecom tasks here are only a proxy.

## Human preference and task success: the Speech Agent Arena

Two more rows on the scoreboard come from Artificial Analysis's Speech Agent Arena [[4]](#ref-4). Paid, screened participants receive a scenario, such as booking a new-patient dental check-up for Tuesday at 9:30 or the first available morning slot, and carry it out in two separate live calls with two anonymous models. Afterward, they pick the conversation they preferred. There are 35 scenarios, 15 with tool use and 20 without.

The preference results are fit into an Elo rating using the Bradley-Terry method, anchored at 1000 for GPT Realtime 1.5 [[3]](#ref-3). Task success rate is calculated separately. For each tool-use conversation, one LLM judge assesses whether the participant followed the scenario, and another judges whether the model made the correct final tool call with the correct arguments. Conversations where the participant strayed from the scenario are dropped first.

These are real people on real calls, something no automated benchmark can offer. Grok Voice 3.0's task success rate of 94.8% is the best on the leaderboard [[2]](#ref-2).

Now place that 94.8% next to the same model's 71.6% on τ-Voice. Same model, same month, both reported as task success, and a gap of 23 points. The Arena's tasks are short, single-goal requests from cooperative people. τ-Voice's are multi-step cases under a policy, from callers designed to be difficult. Neither number is wrong. They answer different questions, and the name alone won't tell you which question you're looking at.

Elo ratings also move as votes arrive. On October 15, the live leaderboard had GPT-Live-1 at 1056 and Gemini 3.8 Live at 993 [[2]](#ref-2), a few points from the 1048 and 990 in xAI's table two days earlier [[1]](#ref-1). The index uses a rating frozen when each model first qualified for publication, so it doesn't reflect the current Elo [[3]](#ref-3).

## Accuracy and experience together: EVA-Bench

EVA-Bench (Bogavelli et al., 2026) from ServiceNow is the only benchmark here designed around enterprise service calls from the ground up [[14]](#ref-14). It has 213 scenarios across three domains: airline rebooking, healthcare HR service delivery and IT service management. In all three, the agent must handle spoken identifiers like confirmation codes, employee IDs and license numbers.

The calls are bot-to-bot. A simulated caller talks to the system under test over a live audio WebSocket, so cascaded pipelines (speech-to-text, then an LLM, then text-to-speech) and speech-to-speech models see the same conditions. Tool calls return deterministic results from a database set up for each scenario. Before scoring, validators check whether the simulated caller stuck to its goal and said what it meant to say, and conversations that fail these checks are regenerated. In the paper's check across four systems, 12% of trials had to be rerun.

Each call receives two pass or fail verdicts. Accuracy (EVA-A) passes only if the final database state matches exactly, the agent didn't stray from the policy or misreport tool results, and its speech got the key entities right. Experience (EVA-X) passes only if turn-taking, conversation progression and conciseness all clear their thresholds. The paper's case for the strict accuracy rule: "An agent can reach the correct end state while fabricating a policy detail, misreading a confirmation code aloud, or proceeding without required confirmations."

Each scenario runs five times, and EVA reports three numbers from those runs: pass@1 (the mean pass rate), pass@k (whether at least one of the runs passed) and pass^k (the chance that all of them pass). This is where xAI's 74% comes from. On ServiceNow's results site, Grok Voice 3.0 has the highest EVA-A pass@1 listed, 0.737 [[15]](#ref-15). Its pass@5 is 0.911, and its pass^5 is 0.553. Across the 12 systems in the paper, the median gap between pass@k and pass^k on accuracy was 0.44 [[14]](#ref-14).

EVA gives the clearest picture here of the trade-off a buyer is making. Among the paper's 12 systems, none cleared 0.5 on both accuracy and experience pass@1. Several newer entries on the site do, including Grok Voice 3.0 at 0.737 for accuracy and 0.712 for experience [[15]](#ref-15). The site also shows how wide the gap between the two can be: Gemini 3.8 Live has the best experience pass rate listed, 0.886, and an accuracy pass rate of just 0.459.

The authors are forthright about the limits [[14]](#ref-14). The published scenarios are English only. The simulator "does not systematically generate interruptions," so barge-in handling is under-tested. Tools are mocks that can't fail or respond slowly the way real APIs do. Systems ran with default turn-detection settings and unoptimized prompts. And the robustness tests cover one accent (French) and one noise condition (a coffee shop).

## Reading the composite: what 86.9 means

The Artificial Analysis index is an equal-weighted average of four components: Big Bench Audio, τ-Voice, an Arena Score that converts Elo into expected preference against an 800-Elo baseline, and the Arena's task success rate [[3]](#ref-3). To get an index score, a model needs results on all four.

Grok Voice 3.0 leads GPT-Live-1 by 3.7 points. In xAI's table, Grok's advantage comes from Big Bench Audio (98.5% vs. 90.1%), Arena Elo (1070 vs. 1048) and Arena task success (94.8% vs. 87.4%) [[1]](#ref-1). On τ-Voice, the hardest support-call test in the index, GPT-Live-1 is slightly ahead on a single trial. So treat the index as a shortlist. Its weights are a choice, and they shifted twice in 2026.

## Latency and cost: check what the task was

xAI says Grok Voice 3.0 matches GPT-Live-1 on τ-Voice "at 40% lower latency" [[1]](#ref-1). That latency is Artificial Analysis's time to first audio, 0.81 vs. 1.34 seconds, which works out to the 40% [[2]](#ref-2). Artificial Analysis measures time to first audio on Big Bench Audio questions [[3]](#ref-3), so the number shows how quickly the model begins answering a logic puzzle. It says nothing about the wait after an agent looks something up mid-call. The τ-Voice authors did track latency during simulated calls: in March, the three providers they tested scored between 0.90 and 1.15 seconds under realistic conditions, averaging response latency and yield latency [[12]](#ref-12).

The cost chart works in a similar way. xAI reports an average cost per task of $0.025 for Grok Voice 3.0, compared with $0.030 for GPT-Live-1 and $0.055 for GPT-Realtime-2.1, labeled "Average cost per task on Artificial Analysis's Big Bench Audio run" [[1]](#ref-1). One task there is a single spoken question. Artificial Analysis's cost-per-hour figure from the same runs also leaves out tool-call charges and cached-token discounts [[3]](#ref-3).

A contact center wants the cost per resolved call. The model's portion is roughly the price per minute multiplied by the average call length, divided by the share of calls the agent resolves. Then add the cost when an unresolved call goes to a person, and a model that's cheaper per minute can cost more per resolved call if it resolves fewer of them. xAI quotes $0.08 per minute for Grok Voice 3.0 [[1]](#ref-1). You'll need to measure the resolution rate yourself.

## Testing a voice agent on your own call types

None of these benchmarks test your callers, your systems or your policies. The only production data in xAI's post comes from xAI itself: Starlink call centers handle 20,000 calls a day with Grok Voice and resolve "roughly 70%" of them with no human in the loop [[1]](#ref-1). That may well be true, and nobody outside xAI can check it. For a vendor evaluation or a pilot, copy the design these benchmarks share.

1. Start with your call mix. Take the call types that make up most of your volume and write scenarios for each: the caller's goal and constraints, the caller's details, a short decision tree for how the caller responds when the agent offers alternatives (as EVA does), and one correct end state.
2. Give the agent a sandboxed copy of your tools. Staging versions of your CRM, order management, billing or ticketing systems let you grade the record after each call, as τ-bench does, rather than judging transcripts by gut feel. Include the failures public benchmarks omit: a slow lookup, an error message, a missing record.
3. Make the callers sound like yours: their accents and languages, the background noise they call from, narrowband phone audio, interruptions, and names spelled out loud. Accent sensitivity in τ-Voice differed sharply by provider, so another company's results won't transfer to you. Give authentication its own test cases, since it was τ-Voice's biggest bottleneck.
4. Put real people on some of the calls. Simulated callers are more patient, and EVA's doesn't interrupt systematically. A few coworkers calling from scenario cards, much like the Arena, will catch issues a synthetic caller wouldn't.
5. Grade more than the end state. Check whether the agent quoted a wrong fee or policy, read the confirmation number back correctly, talked over the caller or went silent, and escalated when your policy requires it.
6. Run each scenario several times and report pass^k next to pass@1. A support line handles the same call types all day, so consistency is what your customers experience. That is exactly the difference between Grok Voice 3.0's 0.737 pass@1 and its 0.553 pass^5 on EVA.
7. Price per resolved call, not per minute.
8. Pin the model version and rerun the suite on every update. xAI's announcement notes that agents using `grok-voice-latest` "have already been switched over automatically" [[1]](#ref-1). That's convenient, and it also means the model behind your agent can change without any change on your side. Your test suite is how you'll find out what changed.

## References

Model scores are as published by xAI, Artificial Analysis and ServiceNow, read on October 15, 2026. Leaderboards change as models and votes are added.

[1] xAI. ["Introducing Grok Voice 3.0."](https://x.ai/news/grok-voice-3) October 14, 2026.
{#ref-1}

[2] Artificial Analysis. ["Speech to Speech AI Model & Provider Leaderboard."](https://artificialanalysis.ai/speech-to-speech) Read October 15, 2026.
{#ref-2}

[3] Artificial Analysis. ["Speech to Speech Benchmarking Methodology."](https://artificialanalysis.ai/methodology/speech-to-speech-benchmarking) Includes the index version history and the Speech Agent Arena judging procedure.
{#ref-3}

[4] Artificial Analysis. ["Speech Agent Arena Overview."](https://artificialanalysis.ai/speech-to-speech/arena/overview)
{#ref-4}

[5] Micah Hill-Smith and George Cameron. ["Big Bench Audio."](https://huggingface.co/blog/big-bench-audio-release) Hugging Face blog, December 20, 2024.
{#ref-5}

[6] Guan-Ting Lin et al. ["Full-Duplex-Bench: A Benchmark to Evaluate Full-duplex Spoken Dialogue Models on Turn-taking Capabilities."](https://arxiv.org/abs/2503.04721) arXiv:2503.04721, 2025.
{#ref-6}

[7] Guan-Ting Lin et al. ["Full-Duplex-Bench v1.5: Evaluating Overlap Handling for Full-Duplex Speech Models."](https://arxiv.org/abs/2507.23159) arXiv:2507.23159, 2025.
{#ref-7}

[8] Guan-Ting Lin et al. ["Full-Duplex-Bench-v2: A Multi-Turn Evaluation Framework for Duplex Dialogue Systems with an Automated Examiner."](https://arxiv.org/abs/2510.07838) arXiv:2510.07838, 2025.
{#ref-8}

[9] Guan-Ting Lin, Chen Chen, Zhehuai Chen and Hung-yi Lee. ["Full-Duplex-Bench-v3: Benchmarking Tool Use for Full-Duplex Voice Agents Under Real-World Disfluency."](https://arxiv.org/abs/2604.04847) arXiv:2604.04847, 2026. Code for all versions: [github.com/DanielLin94144/Full-Duplex-Bench](https://github.com/DanielLin94144/Full-Duplex-Bench).
{#ref-9}

[10] Shunyu Yao, Noah Shinn, Pedram Razavi and Karthik Narasimhan. ["τ-bench: A Benchmark for Tool-Agent-User Interaction in Real-World Domains."](https://arxiv.org/abs/2406.12045) arXiv:2406.12045, 2024.
{#ref-10}

[11] Victor Barres, Honghua Dong, Soham Ray, Xujie Si and Karthik Narasimhan. ["τ²-Bench: Evaluating Conversational Agents in a Dual-Control Environment."](https://arxiv.org/abs/2506.07982) arXiv:2506.07982, 2025.
{#ref-11}

[12] Soham Ray, Keshav Dhandhania, Victor Barres and Karthik Narasimhan. ["τ-Voice: Benchmarking Full-Duplex Voice Agents on Real-World Domains."](https://arxiv.org/abs/2603.13686) arXiv:2603.13686, 2026.
{#ref-12}

[13] Sierra Research. [τ-bench repository (tau2-bench).](https://github.com/sierra-research/tau2-bench) Voice evaluation, task fixes and the current leaderboard.
{#ref-13}

[14] Tara Bogavelli et al. ["EVA-Bench: A New End-to-end Framework for Evaluating Voice Agents."](https://arxiv.org/abs/2605.13841) arXiv:2605.13841, 2026.
{#ref-14}

[15] ServiceNow. [EVA-Bench results.](https://servicenow.github.io/eva/) Read October 15, 2026.
{#ref-15}
