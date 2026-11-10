---
title: "Where the Text Goes in a Voice Agent"
date: 2026-11-10T07:05:00-08:00
description: "In four days in October, ElevenLabs, Mistral and Block shipped what looked like three opposite ways to build a voice agent. Two of them keep text between hearing the caller and answering, and the third throws it out. What that choice buys, what it costs, what the benchmarks say by design, and what to ask a vendor."
tags: [voice-agents, speech-to-speech, architecture, contact-centers, evaluation]
ShowToc: true
TocOpen: false
---

On October 20, ElevenLabs released Eleven Realtime, a duo of models it markets as "a new class of Speech to Speech models", where each one is "a complete voice agent" [[1]](#ref-1). Its landing page claims that speech recognition, turn-taking, the LLM, and the voice "are all tuned end-to-end" [[2]](#ref-2). On the same day, Mistral released Voxtral 3, with a voice agent SDK where "each stage is a separate model, so you can change the LLM, the voice, or the language without rewriting the agent" [[3]](#ref-3). Three days later, there was a demonstration by a small research lab at Block, showing off a new speech model they called Dessa, trained "on nothing but human conversation. Sounds! No text, no pretrained models" [[4]](#ref-4).

When I originally wrote about the first two, I contrasted them: one model vs. a pipeline. Then I went back to the middle of the ElevenLabs post, under the heading "Built on cascaded architecture," which says that Eleven Realtime "is built on a cascaded architecture of the components described above, tuned end to end," and notes that the company calls it a speech-to-speech model "because that is what you get with audio in, audio out, one model ID" [[1]](#ref-1). Thus two out of the three launches are pipelines. One is presented such that the user only sees one model name. This changes the nature of the question.

For someone who wants to put a voice agent on their support line, the relevant distinction is whether text is present between the moment of hearing the caller and the moment of responding, and who tunes the transitions between stages. Text in the middle lets you log what the agent heard and chose, apply rules before it answers, and swap components. It deprives you of prosody from the caller, and historically it's been expensive in terms of latency.

The public measurements show full-duplex research models can have the right timing and poor reasoning, and the best score on a support benchmark I was able to verify for these approaches is from a full-duplex voice which passes its decisions to a text agent. If you only keep text in the middle, you won't reach those numbers either: the tuned pipeline scores roughly the same as the one-model approach. There's no public, clean benchmark of pipelines vs. audio-native models on support calls; that's the one I'd most like to see produced.

## Why voice agents were built as pipelines

The standard voice agent is a pipeline. First, a voice activity detector determines when someone starts speaking, then a speech-to-text model transcribes their speech, next a language model produces a response as text, and finally a text-to-speech model speaks that response. Each step can be built by different vendors and evaluated separately.

The clearest case against the chain I know of is Moshi, the full-duplex model introduced by Kyutai in 2024 [[11]](#ref-11). The authors cite three issues with pipeline models [[11]](#ref-11). Pipeline steps cause "a latency of several seconds between interactions." Text is the intermediary format, so "non-linguistic information that modifies meaning," e.g. emotion or non-verbal vocalizations, "is lost in the interaction." Pipelines require detecting speaker turns, ignoring "overlapping speech, interruptions and interjections."

Human timing shows why the first problem matters. Stivers et al. examined conversation times for yes/no questions in 10 languages [[15]](#ref-15). Across the entire dataset, the average delay before a response was 208 ms. The mode delay was 0 ms. In every single language, the mode occurred between 0 and 200 ms. A system that takes a second or more to begin answering sits well outside the rhythm callers are used to.

Full-duplex models solve all three issues. There are no speaker turns to detect, since the model listens and talks at the same time. They process audio directly, so prosody and tone aren't discarded. Moshi had a 200 ms practical latency [[11]](#ref-11).

## Five designs in use now

Reading the 3 launch examples next to OpenAI's current voice agent guide (which also lists 3 architectures [[5]](#ref-5)), I count 5 designs in use or in research today. In each row, the question is, what goes in between the caller's voice and the agent's response?

| Design | Recent example | Between hearing and replying | What you can read afterward | Who tunes the handoffs |
|---|---|---|---|---|
| Pipeline you assemble | Mistral's Voxtral 3 SDK; OpenAI's chained pipeline | A transcript, then the LLM's reply in text | What speech-to-text heard and what the LLM wrote | You or your platform |
| Pipeline tuned as one product | Eleven Realtime | The same stages, with text at each step | What it heard and what it decided to say | The vendor |
| One audio-native model | OpenAI's Realtime API | Audio inside one model | The reply's transcript; the caller's side comes from a separate transcription model | The vendor |
| Full-duplex talker with a text backend | OpenAI's GPT-Live | A live voice model that hands reasoning and tools to a backend you choose | Transcript events, plus the backend's requests and results | The vendor for talk, you for the backend |
| Speech only | Block's Dessa (research demo) | Audio only | Whatever you transcribe yourself (Block added subtitles to its demo clips afterward) | The vendor |

But then there's the 4th row which I wasn't anticipating. OpenAI describes their GPT-Live as a model that "handles a spoken conversation while a backend agent looks up information, uses tools, and completes tasks," and say that "you choose the backend model or agent independently of the voice model" [[6]](#ref-6). You send detailed instructions, business rules and tool workflows to the backend. The live model receives "a short prompt for conversation style and when to delegate." It's duplex on the front, a text agent in the back.

Viewed this way, the launches are less opposed than they looked. Both Mistral and ElevenLabs have text at the center and disagree on who does the tuning. OpenAI offers a design that keeps the speaking in audio, and the choosing in text. Only Block bets on no text.

## What the text buys you

ElevenLabs put the argument for text forward much better than I ever could, and that's quite a statement considering they sell a single model ID. "Cascaded also has practical advantages in production, because every step produces text. You can see what the agent heard and what it decided to say, so you can tell a misheard word from a wrong decision" [[1]](#ref-1). It's the same text, it says, that allows its platform to add guardrails and PII redaction to these agents.

That split between hearing wrong and deciding wrong is the most helpful part of any of the three launch posts for a contact center. When things go wrong, you first need to know which one happened, because the fix will be different. A misheard account number could mean an audio issue, a spell-back step, or a bad STT model. A bad refund could mean a bad prompt, a bad policy, or a bad tool.

OpenAI's guide also mentions this benefit of its chained path: Use it "when each stage needs to be visible or replaceable," for example to "store the transcript, run policy checks before the text agent responds, call internal systems, then generate speech only after the workflow reaches an approved answer" [[5]](#ref-5). That last part is important for compliance calls. If you have a policy that runs before the agent speaks, you can prevent a wrong answer from being spoken to the caller. If the policy runs after the fact, it can still catch mistakes, but it won't prevent them.

An audio-native model doesn't offer this level of detail. From OpenAI's Realtime API reference: "Input audio transcription is not native to the model, since the model consumes audio directly. Transcription runs asynchronously through the /audio/transcriptions endpoint and should be treated as guidance of input audio content rather than precisely what the model heard" [[7]](#ref-7). This means the transcript in your logs comes from a second model making its own guess. So if an agent messes up a name, you can't be sure if the agent or the logging model was at fault.

Finally, text opens up the possibility of swapping components. Mistral's pitch is that you can swap the LLM, the voice, or the language without changing the agent, and deploy the entire stack using Mistral's API or your own infrastructure [[3]](#ref-3). Kyutai, which developed Moshi, has also released Unmute, which wraps "text LLMs" in its speech-to-text and text-to-speech models and "works with any text LLM you like" [[12]](#ref-12). If a new, better reasoning model comes out, you can switch to it without waiting on anyone if you control your own pipeline. If you rely on a single product, you're stuck waiting for your vendor to upgrade. And the same holds true for a tuned pipeline sold by itself under one model ID.

## What the text costs

The first cost is speed, which mostly has been engineered out. ElevenLabs lays out the old compromise explicitly: fused models "initially set the bar for how natural a voice agent can feel, while a cascaded setup meant increased latency at every handoff and components never tuned against each other" [[1]](#ref-1). Mistral's solution is streaming. The company explains that in its SDK, "no stage waits for the one before it, so each turn completes in under one second, and the agent stops talking when the caller cuts in" [[3]](#ref-3). Its real-time transcription model operates with "sub-500ms latency," and its end-of-turn prediction model uses "both acoustic and semantic information" to determine whether the caller is done speaking.

On Artificial Analysis's speech-to-speech leaderboard, Eleven Realtime v1 Max begins responding in 0.57 seconds on average, and v1 Mini in 0.62 seconds [[8]](#ref-8). That's quicker than Grok Voice 3.0 (0.81), Gemini 3.8 Live (1.18) and both GPT-Live-1 settings that AA tested (1.24 and 1.34). Two caveats, though. Those are averages for spoken logic problems from Big Bench Audio, not phone conversations (ElevenLabs makes the same note about its own number [[2]](#ref-2)) and they're still more than double the 208-ms human average. But the notion that a pipeline must be the slower approach doesn't hold true anymore here.

The second cost is tone, and nobody's figured out how to engineer that down yet. On the Dessa page, Block points out that today's voice AI "turns your voice into text and throws away an incredible amount of information text alone can't carry," and that Dessa "natively hears and speaks emotion, sarcasm, laughter, a frustrated sigh, a voice cracking with sadness" [[4]](#ref-4). The company argues that this is a safety concern: "Imagine being in serious pain while a voice AI has no idea and just keeps going."

If you're calling customer service, you might be the person saying "fine" in a tone that conveys the opposite, or the person who's about to cry because her claim has been denied. The transcript will say "fine." Mistral's new voices change their emotion, pace and emphasis "based on conversational context" [[3]](#ref-3), but that's the agent's tone, determined by the words. Nothing is said about a pipeline that can hear the caller's tone; there isn't even a mention of this feature in any of the three launch posts.

## What the scoreboard says by design

Artificial Analysis rates speech-to-speech systems with benchmarks in speech reasoning (Big Bench Audio), conversational dynamics (part of Full-Duplex-Bench), a customer support call benchmark (τ-Voice), and a human preference arena that additionally verifies that the correct final tool call was executed. I detailed how each of these functioned in an [earlier post](/posts/2026-10-15-how-voice-agents-are-measured-now/). Here are rows from the leaderboard as of my reading on November 10, selected because I could verify each one's design from its maker [[8]](#ref-8):

| AA entry | Design | Speech reasoning | Conversational dynamics | τ-Voice | Arena task success | Time to first audio |
|---|---|---|---|---|---|---|
| Moshi (Kyutai) | Full duplex | 4.4% | 61.0% | n/a | n/a | n/a |
| PersonaPlex (NVIDIA) | Full duplex, built on Moshi | 19.1% | 91.0% | n/a | n/a | n/a |
| Eleven Realtime v1 Mini | Tuned pipeline | 75.9% | 95.8% | 46.8% | 60.3% | 0.62 s |
| Eleven Realtime v1 Max | Tuned pipeline | 98.8% | 94.1% | 47.6% | 87.1% | 0.57 s |
| GPT-Realtime-2.1 High | One audio-native model | 95.8% | 95.7% | 45.7% | 91.5% | 1.21 s |
| GPT-Live-1 (Astra, medium) | Full-duplex talker with a backend | 90.1% | 94.9% | 74.5% | 87.4% | 1.34 s |

Four points to note here.

First, the full-duplex research models reason very poorly, regardless of timing. PersonaPlex, a system by NVIDIA built upon the Moshi framework [[13]](#ref-13), achieves 91.0% on conversational dynamics and 19.1% in speech reasoning, while Moshi gets 61.0% and 4.4%. Even if a system can pause, back-channel and yield in a human-like manner, it might still fail to answer a logic question posed verbally.

This was a familiar pattern when Artificial Analysis first released the Big Bench Audio benchmark back in December 2024. GPT-4o scored 92% on the text version, while OpenAI's realtime preview scored 66% on speech-to-speech, although the Whisper → GPT-4o → TTS-1 pipeline showed "minimal performance degradation compared to pure text processing" [[10]](#ref-10). At the time, Artificial Analysis concluded that "for applications where reasoning accuracy is critical, pipeline approaches currently offer the optimal balance." OpenAI's one-model Realtime model now scores 95.8%, and Eleven Realtime v1 Max 98.8%, so short logic questions posed verbally do not distinguish between the two designs much anymore.

They still do, however, when it comes to support. On τ-Voice, the difference between the two OpenAI systems is almost 29 points: GPT-Live with a backend scores 74.5%, whereas the one-model Realtime scores 45.7% (AA also lists another GPT-Live configuration scoring 59.3%). Eleven Realtime v1 Max scores 47.6%, which is close to the one-model model. The fact that the model kept text in the middle did not, by itself, boost the score. ElevenLabs' blog post does not mention that score, only mentioning the 87.1% arena task success rate [[1]](#ref-1). I'm only taking the two OpenAI scores as illustrative, though. AA notes that GPT-Live was evaluated on only one trial and GPT-Realtime-2.1 High on two, whereas most models were tested with three trials, and AA's methodology page does not specify how GPT-Live with a backend was set up [[9]](#ref-9). But these are models from the same provider, tested on the same benchmark. And the design with the reasoning part offloaded to a text backend outperforms by a large margin.

Lastly, there is no proper comparison between cascaded and native systems on support calls. The authors of the τ-Voice paper have only tested three audio-native models, and adding "cascaded ASR→LLM→TTS baselines" as additional experiments is listed as future work because it "would help isolate voice modality effects from architecture choices" [[14]](#ref-14). There are several cascade systems available in the AA arena: ElevenLabs Agents, Deepgram, Cartesia, Inworld, each with a specific speech-to-text, LLM, and voice, but they do not show up under τ-Voice [[8]](#ref-8). Out of the entries I could check, only Eleven Realtime is explicitly referred to as a cascade by its maker, so a single system is standing in for an entire class of models on τ-Voice.

## Dessa up close

The one among the three that I find most interesting but also least useful is Dessa. Block writes that Dessa was trained from scratch on human conversation with no text, is full duplex and "knows when to cut in and when to backchannel," its inference "runs faster than real time" and it "runs locally on a MacBook Pro M4 Max or better" [[4]](#ref-4). It learned its language "from hundreds of times fewer words" than an LLM, and according to Block it trained Dessa on one NVIDIA GB200 NVL72.

None of the things a contact center needs has been demonstrated on it. There's a page with live recordings, including one of Dessa talking to GPT-Live-1, a waitlist and early access for "a small group of AI safety researchers." There's no mention of an API, price, benchmark or test on phone audio. The clips are casual conversation, and the page states that "nobody steers it": Dessa "picks what to say, when to say it and how, down to its own voice and personality." This is attractive for a research demo and less so for a support line, where the agent has to follow a policy, call a tool and say the correct refund amount.

I should also take another look at Moshi. NVIDIA gives credit to it for first introducing full duplex listening and speaking [[13]](#ref-13). It's not text-free. It begins "from a text language model backbone," predicting "time-aligned text tokens as a prefix to audio tokens" in an approach its authors dub "Inner Monologue" that "significantly improves the linguistic quality of generated speech" [[11]](#ref-11). Even the model leading the charge against text included text internally. With Dessa, Block is betting that we can drop it altogether. If this works out, it will be a genuinely interesting research finding. At this point it is just a demo, and I'll revisit my opinion on the day it scores on a support task.

## What I'd ask a vendor

Here are the questions I would like answers to, regardless of what a vendor calls their solution:

1. Is the text in my logs actually the text an agent used to take action? In a pipeline, yes. In an audio-native model, the caller's portion might be coming from a different transcription model, which OpenAI suggests treating as guidance [[7]](#ref-7). You'll want to know which one you are receiving.
2. Can a rule execute before the agent begins speaking? Policy validation and PII masking, for example, may only apply to the log, which means a bad answer will be recorded after the caller already heard it.
3. Whose judgment is it, and can I override it? With a delegating architecture or your own pipeline, the judgment model and the business logic belong to you. With a tuned all-in-one solution, the vendor's settings take precedence.
4. How do we account for sentiment? The inability to detect a sigh should lead us to specify what cues trigger a transfer to a human and encode that logic directly.
5. What metrics are being reported, measured on what? AA measures its time to first audio on spoken logic questions. The difference between Arena task success and τ-Voice for the Eleven Realtime v1 Max is roughly 40 points, and even greater for OpenAI's Realtime solution. Request data on your specific use cases, using phone audio and repeated testing, as I outlined in my [measurement post](/posts/2026-10-15-how-voice-agents-are-measured-now/).
6. What information is provided to the caller while a tool is executing? OpenAI advises that acknowledgments like "I'm checking" should be evaluated independently from the answer the user expects [[5]](#ref-5). A brief "one moment" does not reduce the time until the actual answer is ready.

## Where I land

My conclusion is a division between the layer that speaks and the layer that reasons. Speech requires speed, turn-taking and, eventually, tone. In the AA benchmarks, the optimized pipeline is now matching the audio-native architectures in speed and turn-taking, leaving tone as the only feature unique to audio, and so far the evidence for that is Block's demo.

Reasoning requires logic, policy, and an audit trail, which remains text-based. ElevenLabs keeps text and optimizes the joints to hide them. Mistral keeps text and exposes the joints. GPT-Live adds full-duplex voice over any text agent you select. On the only published support benchmark, that latter design scores best of the three I could compare, and the tuned pipeline, for all its text, lands next to the one-model design.

I believe that text renders an agent's decision-making process transparent, but the underlying model, tooling, and instructions determine whether those decisions are correct. Block's Dessa is the clean counterexample, and its point about tone is right. An agent unable to recognize user frustration is missing key context a human would use. But until a text-free model can follow a refund policy and say it back correctly, I'd want text at the point where a voice agent makes its decisions, and I'd test the timing on my own calls.

## References

[1] Min Kim. ["Introducing Eleven Realtime: a simpler way to build natural voice agents."](https://elevenlabs.io/blog/realtime) ElevenLabs, October 20, 2026.
{#ref-1}

[2] ElevenLabs. ["Eleven Realtime."](https://elevenlabs.io/realtime) Product page, read November 10, 2026.
{#ref-2}

[3] Mistral AI. ["Voxtral 3."](https://mistral.ai/news/voxtral-3/) October 20, 2026.
{#ref-3}

[4] Block. ["Dessa."](https://block.xyz/dessa) Research demo page (Ragavan Thurairatnam, Dessa lead), October 2026, read November 10, 2026.
{#ref-4}

[5] OpenAI. ["Voice agents."](https://developers.openai.com/api/docs/guides/voice-agents) API documentation, read November 10, 2026.
{#ref-5}

[6] OpenAI. ["Getting started with GPT-Live."](https://developers.openai.com/api/docs/guides/live) API documentation, read November 10, 2026.
{#ref-6}

[7] OpenAI. ["Realtime."](https://developers.openai.com/api/reference/resources/realtime) API reference, input audio transcription configuration, read November 10, 2026.
{#ref-7}

[8] Artificial Analysis. ["Speech to Speech."](https://artificialanalysis.ai/speech-to-speech) Leaderboard and summary table, read November 10, 2026.
{#ref-8}

[9] Artificial Analysis. ["Speech to Speech Benchmarking Methodology."](https://artificialanalysis.ai/methodology/speech-to-speech-benchmarking) Read November 10, 2026.
{#ref-9}

[10] Micah Hill-Smith and George Cameron. ["Evaluating Audio Reasoning with Big Bench Audio."](https://huggingface.co/blog/big-bench-audio-release) Artificial Analysis, Hugging Face blog, December 20, 2024.
{#ref-10}

[11] Alexandre Défossez et al. ["Moshi: a speech-text foundation model for real-time dialogue."](https://arxiv.org/abs/2410.00037) arXiv:2410.00037, 2024.
{#ref-11}

[12] Kyutai. ["Unmute."](https://github.com/kyutai-labs/unmute) GitHub repository README, read November 10, 2026.
{#ref-12}

[13] NVIDIA. ["PersonaPlex: Natural Conversational AI With Any Role and Voice."](https://research.nvidia.com/labs/adlr/personaplex/) NVIDIA ADLR, January 2026.
{#ref-13}

[14] Soham Ray, Keshav Dhandhania, Victor Barres and Karthik Narasimhan. ["τ-Voice: Benchmarking Full-Duplex Voice Agents on Real-World Domains."](https://arxiv.org/abs/2603.13686) arXiv:2603.13686, 2026.
{#ref-14}

[15] Tanya Stivers et al. ["Universals and cultural variation in turn-taking in conversation."](https://www.pnas.org/doi/10.1073/pnas.0903616106) PNAS 106(26): 10587-10592, 2009.
{#ref-15}
