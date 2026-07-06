---
title: "Local AI Meeting Minutes with Docker Model Runner and Docker Agent: No Cloud, No Leaks"
date: 2026-07-03T12:00:00Z
tags: [
  "docker", "docker-agent", "docker-model-runner", "local-llm", "ai-agents",
  "meeting-minutes", "speech-to-text", "diarization", "privacy", "devsecops",
  "apple-silicon", "gguf", "qwen"
]
keywords: [
  "local AI meeting minutes", "Docker Model Runner", "Docker Agent",
  "private meeting transcription", "on-device speech to text", "local LLM Apple Silicon",
  "speaker diarization", "GGUF", "Qwen2.5", "unexpected end of JSON input"
]
author: "Matteo Bisi"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Local AI meeting minutes on Apple Silicon: on-device transcription with speaker diarization, then minutes from a 14B model through Docker Model Runner and Docker Agent. No cloud, no data flow to third parties, plus the story of a bug Docker fixed the same day I reported it."
canonicalURL: "https://www.msbiro.net/posts/local-ai-meeting-minutes-docker-agent/"
disableShare: true
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "https://www.msbiro.net/social-image.png"
    alt: "Local AI meeting minutes with Docker Model Runner and Docker Agent on Apple Silicon"
    caption: ""
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/matteobisi/msbiro.net/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

Like most people in this industry, I spend a good part of my week in meetings: vendor evaluations, customer calls, technical deep dives on cloud-native projects. And like most people, I want minutes out of them without spending an hour writing them myself.

The obvious answer in 2026 is "send the recording to an AI service". For me, that answer is wrong by definition. Those recordings contain customer names, security architectures, commercial terms, gap analyses. As a security team leader, I cannot be the person who enforces data-handling policies on everyone else and then ships a customer's security posture to a third-party API for convenience.

So I set a constraint and built around it: **no audio, no transcript, no prompt ever leaves my machine**. A MacBook Pro M4 with 32 GB of RAM, native speech-to-text, and a 14B model running through Docker Model Runner, orchestrated by Docker Agent. It works, it is fast enough to use daily, and getting there involved one interesting bug that Docker fixed the same day I reported it.

This is what I built and what I learned.

---

## The Problem It Solves

Meeting transcription services are excellent and I have nothing against them, until the content is sensitive. A one-hour technical meeting with a customer easily contains: who the customer is, what their infrastructure looks like, which security gaps they have, what they are planning to buy, and how much they are willing to spend. That is precisely the data an attacker or a competitor would want, concentrated in a single audio file.

Sending it to a cloud service means trusting someone else's retention policy, someone else's breach response, and someone else's subprocessors, for every single meeting. Under frameworks like NIS2 and DORA the burden is on you to justify that flow. The simplest justification is not needing one: keep the whole pipeline local.

The good news is that in 2026 the local stack is good enough for real work:

- **Apple Silicon speech-to-text** is fast and accurate, including speaker diarization, with no container and no GPU setup.
- **Docker Model Runner (DMR)** pulls GGUF models straight from Hugging Face and serves them behind an OpenAI-compatible API on `localhost`.
- **Docker Agent** gives you a declarative agent definition (a YAML file) on top of DMR: system prompt, model binding, sampling parameters, output caps, all versioned in Git.

The result: a 47-minute meeting becomes a structured, speaker-attributed Markdown minute in about 6 minutes of local compute, in Italian or English, with an action-items table and a decisions section that says so when nothing was decided. And the MacBook stays usable while it runs.

---

## The Pipeline: From Recording to Minutes, Fully On-Device

Two stages, two scripts, plus a small interactive wizard that chains them. All the code (scripts, agent definition, prompt) is published in [my technical_notebook repository](https://github.com/matteobisi/technical_notebook/tree/main/projects/speech):

{{< mermaid >}}
flowchart TD
    subgraph row1 [" "]
        direction LR
        A["meeting.mp4"] --> B["ffmpeg\naudio extraction"] --> C["speech diarize\npyannote, who spoke when"]
    end
    subgraph row2 [" "]
        direction LR
        D["speech transcribe\nQwen3-ASR 1.7B, per turn"] --> E["speaker-attributed\ntranscript .txt"] --> F["docker agent run\nminute_agent.yml"]
    end
    subgraph row3 [" "]
        direction LR
        G["Docker Model Runner\nQwen2.5-14B GGUF, localhost"] --> H["meeting_minutes.md"]
    end
    C --> D
    F --> G
    style row1 fill:none,stroke:none
    style row2 fill:none,stroke:none
    style row3 fill:none,stroke:none
{{< /mermaid >}}

Stage 1 is transcription. I use the native `speech` CLI (Homebrew, Apple Silicon) instead of a containerized Whisper, for reasons I will detail in a moment. The output is a transcript where every line carries a timestamp and a `speaker_N` label, plus a small header where you map labels to real names before generating minutes.

Stage 2 is the minutes. The transcript goes to a local **Qwen2.5-14B-Instruct** (Q4_K_M quantization, ~8.4 GB), pulled from Hugging Face by Docker Model Runner and driven by **Docker Agent** with a checked-in agent definition. One command:

```bash
./minutes.sh meeting-transcript.txt --type meeting
```

A clarification on the plumbing, because precision matters when the whole point is privacy. Docker Agent never touches the TCP endpoint you might expect: the agent YAML only declares `provider: dmr`, and the DMR provider reaches Model Runner through Docker Desktop's internal socket endpoint (`model-runner.docker.internal`). The `localhost:12434` OpenAI-compatible endpoint is used by my script only for a model pre-warm call and for an optional direct-API mode. Two different local channels, same local engine; the model traffic never leaves the machine on either path.

One thing does phone home by default, though: docker agent sends usage telemetry (command and session metadata, not your content) to Docker. In a pipeline built so that nothing leaves the machine, I turn it off explicitly:

```bash
export TELEMETRY_ENABLED=false
```

### Why Two Stacks? Why Is the Transcription Not an Agent Too?

Fair question, since the second half of this pipeline is all Docker.

The first reason is plain: **there is nothing to point an agent at**. Docker Model Runner's OpenAI-compatible surface is [`chat/completions`, `completions`, and `embeddings`](https://docs.docker.com/ai/model-runner/api-reference/); it serves language models, not speech models. There is no audio-transcription endpoint, so a "transcription agent" would just be an agent shelling out to some ASR binary, which is the tool minus the agent.

The second reason is the accelerator, and Docker's own architecture makes the argument for me. On macOS there is no Metal GPU passthrough into containers or VMs; that is exactly why [Docker Model Runner runs llama.cpp as a native host process](https://www.docker.com/blog/run-llms-locally/) on Apple Silicon instead of inside a container. Docker looked at compute-bound inference on a Mac and chose native execution. Speech-to-text and diarization are the same class of workload with no DMR equivalent, so I made the same choice Docker made: the `speech` CLI runs natively on MLX and CoreML, using the GPU and the Neural Engine directly. My previous containerized Whisper setup ran CPU-only at roughly real-time speed; the native CLI transcribes the same meetings about 5x faster.

The last reason is a design opinion I will defend: **agents belong where language lives, not where signal processing lives**. Transcription is deterministic: same audio in, same text out. There are no instructions to interpret, no formatting rules, no judgment calls; an agent runtime adds ceremony and nothing else. The minutes stage is the opposite. It is entirely instructions: output structure, language selection, what counts as a decision, what to ignore. That is where a declarative agent definition earns its place. One optimized tool per job, composed through a plain text file: the transcript is the interface between the two stacks, and either half can be swapped without touching the other.

---

## Stage 1: Transcription, and the Day My Meeting Had 22 Speakers

Getting the transcript right took more iteration than expected. Documenting the problems is worth the detour, because anyone building on pyannote will hit them.

**Problem one: over-diarization.** A 47-minute call with three participants came back with **10 speaker labels**. Most of the extras were crosstalk fragments, half-second "sì, esatto" backchannels promoted to full speakers.

The `speech diarize` CLI has no "expected number of speakers" flag, but it has a clustering threshold. The help text says *lower = fewer speakers*. I measured it on the same audio file:

| `--cluster-threshold` | Speakers detected (3 real) |
|---|---|
| 0.40 | 22 |
| 0.55 | 13 |
| 0.715 (default) | 10 |
| **0.85** | **7** |

The direction is exactly the opposite of what the help claims: **higher threshold = fewer speakers**. Measure, do not trust the docs. 0.85 became my default.

**Problem two: the phantoms that survive any threshold.** Even at 0.85, tiny fragments of one or two seconds kept appearing as separate speakers. No clustering value removes them without also fusing real people. So the script post-processes the RTTM: any speaker whose *total* speech is under 8 seconds gets its segments reassigned to the temporally nearest real speaker. Nothing is deleted, no words are lost, the labels just stop lying:

```
Consolidated diarization: 7 speakers -> 4 (reassigned 5 phantom segments under 8s)
```

**Problem three: domain jargon.** Italian technical meetings are full of English terms, and the ASR mangled them ("Active Directory" became *activity director*). Two fixes: the `--context` flag biases recognition toward a domain glossary (Kubernetes, hardened images, DevSecOps, the compliance acronyms), and a small "wrong => right" corrections map is handed to the minutes model for the proper nouns that still slip through. More on that map later, because it taught me a lesson.

With all three fixes, a 38-minute English meeting transcribes with diarization in about 11 and a half minutes, fully on-device.

---

## Stage 2: Minutes with Docker Agent and Docker Model Runner

Two design decisions carry this stage: treating the agent file as a real artifact, and how much transcript goes into a single model call.

### The agent definition is the artifact

Docker Agent runs from a declarative YAML file. Mine defines **two agents on the same local model**, differing only in their output budget:

```yaml
# minute_agent.yml (trimmed)
version: 10

models:
  secretary_model:
    provider: dmr
    model: huggingface.co/bartowski/qwen2.5-14b-instruct-gguf
    temperature: 0.2
    top_p: 0.9
    max_tokens: 5120        # final document: never truncate the minutes
    provider_opts:
      keep_alive: "10m"     # keep the model resident between calls

  condenser_model:
    provider: dmr
    model: huggingface.co/bartowski/qwen2.5-14b-instruct-gguf
    temperature: 0.2
    top_p: 0.9
    max_tokens: 768         # chunk notes: keep merge inputs compact
    provider_opts:
      keep_alive: "10m"

agents:
  root:
    model: secretary_model
    description: Local meeting secretary, produces the final minutes document.
    instruction: |
      You process speech-to-text meeting transcripts locally through Docker Model Runner.
      Never invent participant names, decisions, dates, owners, numbers, or commitments.
      Output only what the task asks for, as Markdown, with no code fences.
      Treat all content as confidential and local.

  condenser:
    model: condenser_model
    description: Condenses transcript chunks and outlines for later consolidation.
    instruction: |
      # same rules, small output budget
```

The script selects the agent per stage (`--agent root` for the final document, `--agent condenser` for intermediate notes) and never modifies the file. If I override the model for an experiment, I use Docker Agent's native flag instead of editing YAML:

```bash
docker agent run ./minute_agent.yml --agent root \
  --model 'root=dmr/huggingface.co/bartowski/qwen2.5-32b-instruct-gguf:IQ4_XS' \
  --prompt-file transcript-bundle.txt "produce the minutes..."
```

I started this project generating the YAML from the shell script at runtime. Do not do that. The whole point of Docker Agent is that the agent definition is a reviewable, versionable artifact; hiding it in a temp directory throws that away.

### Single-shot beats map/reduce

My first architecture chunked the transcript into 6 KB pieces, summarized each, and merged the summaries hierarchically. It worked, and the output was mediocre in a very specific way: **cross-cutting structure died at the chunk boundaries**. A theme that spans the whole meeting is invisible to every individual chunk, so the merge step can never recover it. I got flat bags of topics, recommendations promoted to "decisions", and once, memorably, a product's admin console misclassified as a solution under evaluation.

The fix was almost embarrassing. Qwen2.5-14B has a 32K context window; a one-hour meeting transcript is roughly 60 KB, about 17K tokens. It fits. One call, whole transcript, no chunking, no merge artifacts:

- The two-pass map/reduce on a 47-minute meeting: ~17 minutes, shallow output.
- Single-shot on the same meeting: **6 minutes 7 seconds**, and the minutes correctly said *"no formal decision was taken"* instead of inventing five decisions.

The chunked path still exists as an automatic fallback for meetings above ~90 minutes. Everything else goes single-shot.

---

## The Bug: `unexpected end of JSON input`

Here is the part of the journey I promised. When I first pushed the whole 48 KB transcript through Docker Agent, it failed after ~30 seconds, every time:

```
model failed: error receiving from stream: unexpected end of JSON input
```

The same request against the same DMR endpoint via plain `curl` worked perfectly, streaming or not, up to 60 KB. So the backend was healthy and something between Docker Agent and the model was giving up. My working theory was a fixed ~30-second first-token timeout in the client: a 17K-token prompt legitimately prefills for longer than that on a 14B local model, in complete silence, before the first token streams out.

I did what you should always do at this point: measured it (cold model: fails at 34.3s; pre-warmed model: fails at 30.2s), wrote it up, and **filed an issue**: [docker/docker-agent#3298](https://github.com/docker/docker-agent/issues/3298).

What happened next is the best-case scenario for open development:

1. A maintainer investigated **the same day** and corrected my theory: there is no fixed 30s deadline in docker-agent. The real cause is an **idle connection drop in the proxy path between docker-agent and Docker Model Runner** while the model prefills silently. `curl` succeeded because it talks to DMR directly, skipping that hop.
2. The actual defect was error handling: the resulting truncation errors were classified as *non-retryable*, so one transient drop killed the whole run with a cryptic decoder message.
3. [PR #3302](https://github.com/docker/docker-agent/pull/3302) made those errors retryable and shipped in **v1.90.0, the same day**. The retry works because the backend's KV cache still holds the just-attempted prefill, so the identical retry reaches first-token fast enough to survive.

The satisfying detail: while waiting, I had already worked around it in my script with a retry loop that re-sends the *identical* prompt after a drop, deliberately not touching the model in between so the KV cache stays warm. The official fix is the same idea, implemented where it belongs. When your workaround and the maintainer's patch converge on the same mechanism, you know the diagnosis is right.

```bash
# the workaround, still useful on older docker-agent builds
while true; do
    if docker agent run "$AGENT_FILE" --agent "$ACTIVE_AGENT" --exec ... ; then
        break
    fi
    echo "stream dropped; retrying same prompt (warm KV cache)..."
    sleep 2   # identical retry (do NOT send a different prompt in between),
              # it would evict the KV cache that makes the retry fast
done
```

One practical note: **Docker Desktop bundles docker-agent and lags the GitHub releases**. At the time of writing, Docker Desktop bundles v1.88 while brew is at v1.97; the fix landed in v1.90.0, so the bundled build does not have it yet. My script resolves the brew binary explicitly (`brew --prefix docker-agent`), because Desktop also puts its older `docker-agent` on the PATH, shadowing brew's:

```bash
brew install docker-agent
"$(brew --prefix docker-agent)/bin/docker-agent" version
```

---

## Bigger Is Not Better: The 32B Experiment

With single-shot working, the obvious question: would a bigger model produce better minutes? DMR makes the experiment trivial, quantization tags included:

```bash
docker model pull hf.co/bartowski/Qwen2.5-32B-Instruct-GGUF:IQ4_XS   # 17.7 GB
```

The result surprised me. The 32B (at that aggressive quantization) took twice as long (~12 minutes vs 6), used three times the memory, caught a couple of commercial details the 14B had missed, and **got the core structure of the meeting wrong**, collapsing the vendor's two-product architecture into one and misattributing the whole discussion to the wrong product line.

Meanwhile the detail gap turned out to be a prompt problem, not a model problem. One added instruction ("always capture pricing tiers, trial terms, user limits, SLAs, deployment options") and the 14B captured everything the 32B had caught, at the original speed, with the structure intact.

My rule after this test: **on local hardware, a well-prompted smaller model beats a heavily-quantized bigger one**, and it leaves your machine usable while it runs. Spend the effort on the prompt before spending it on VRAM.

---

## Three Prompt Lessons from a Local 14B

Running a 14B as a meeting secretary is an exercise in precise instructions. Three failures worth sharing:

**1. The corrections map became meeting topics.** I pass the model a spelling map for proper nouns the ASR garbles (product names, vendor names). One run, the model turned those names into *fabricated agenda sections*, complete with invented open questions and risks about them. The fix was explicit scoping: *"These are spelling fixes, NOT meeting topics. Never create a section, bullet, decision, or risk about these names. Never introduce a name that did not come up in the meeting."* Anything extra you hand a small model can turn into content unless you state exactly what it is for.

**2. Instruction position matters at long context.** With the directive placed *before* a 48 KB transcript, the model drifted: it answered in English instead of Italian and over-summarized. The instruction was 17K tokens away from the generation point. Moving the directive *after* the transcript, adjacent to where generation starts, fixed the language and the depth in one move.

**3. Decisions are not recommendations.** Small models love to promote things: a product feature becomes "a decision was made to adopt X". The instruction that finally stuck: *"list ONLY what participants explicitly agreed; if the meeting was exploratory, write exactly that no formal decision was taken."* The most trustworthy line in my minutes is now the one that says nothing was decided.

---

## Results

Real meetings, both languages, measured on the M4 / 32 GB machine:

| Meeting | Transcription + diarization | Minutes (single-shot, 14B) |
|---|---|---|
| 47 min, Italian, 3 speakers | — | 6 min 07 s |
| 38 min, English, 3 speakers | 11 min 35 s | 4 min 59 s |
| 1 h 07 min, Italian, 6 speakers | — | 7 min 30 s (2 m 21 s with the model already warm) |

The minutes come out with correct speaker attribution (using the manual name mapping), an action-items table with real owners, commercial terms when they were mentioned, and decision sections that reflect what actually happened. Run-to-run there is some variance in which fine details make the cut, which is inherent to a single pass of a 14B; as an attendee, topping up a detail takes seconds. As a non-attendee, you get a document that actually tells you what happened.

And the property that motivated all of this holds by construction: the recording, the transcript, and every prompt stayed on the laptop.

---

## Takeaways

**Local AI is ready for this job.** A year ago I would not have said it. Today, a quantized 14B through Docker Model Runner produces minutes I actually distribute, and Apple Silicon transcribes faster than real time with diarization. The privacy argument no longer costs you the outcome.

**Docker Desktop has become a complete AI suite.** This is the part that changed how I think about the platform. I pulled a 14B model the way I pull an image. I defined my agents in a YAML file that lives in Git next to my scripts, reviewed like any other configuration. The endpoint is OpenAI-compatible, so nothing in the workflow would care if I swapped the model tomorrow. At no point did I create a Python virtualenv, compile llama.cpp, or fight a Metal flag: Model Runner runs the engine natively on the host, and that is simply someone else's solved problem. For a developer, this is what "integrated" is supposed to feel like: the AI stack behaves like the container stack you already know. For a business environment it matters even more. The whole solution lives inside a platform that has already passed procurement, security review, and update management in thousands of companies, so you can build and ship an advanced AI solution without introducing a single new vendor, runtime, or approval cycle.

**The ecosystem responds like a product, not a side project.** When I hit a real failure on a long prompt, I filed a measured report, and the docker-agent maintainers investigated it, corrected my own wrong theory, and shipped the fix the same day. That turnaround did more for my confidence in the stack than any feature list could. Precise evidence in, fix out: the loop works, but only if you write the report.

**And the small native tool holds up its half.** The `speech` CLI is the opposite philosophy (one open-source binary, one job) and it is the right philosophy for that job: a brew install, models cached on first run, and a 38-minute meeting diarized and transcribed on the Neural Engine in under 12 minutes. A focused tool that does its job excellently needs no further justification. The two philosophies compose because the interface between them is a text file.

The outcome, day to day: I record the meeting, run one command, and a few minutes later I have distributable minutes with named speakers, in Italian or English. For a security team, the pipeline turns a real data-governance problem into a non-problem: there is no third party, no DPA to review, no retention policy to audit, because there is no flow. The meeting never leaves the room.

---

## Quick Reference

```bash
# --- Stage 0: prerequisites (macOS, Apple Silicon) ---
brew install ffmpeg
brew install soniqo/tap/speech          # native ASR + diarization
brew install docker-agent               # newer than the Docker Desktop bundle
docker model pull hf.co/bartowski/qwen2.5-14b-instruct-gguf   # via Docker Model Runner

# --- Stage 1: transcribe with diarization ---
./transcribe_video.sh meeting.mp4 it --diarize
#   -> meeting.txt with [timestamps] speaker_N labels
#   edit the header to map speaker_N to real names

# --- Stage 2: minutes, fully local ---
export TELEMETRY_ENABLED=false          # docker agent usage telemetry off
./minutes.sh meeting.txt --type meeting
#   -> meeting_minutes.md

# or the guided wizard for the whole flow:
./speech.sh

# diarization tuning that mattered:
#   --cluster-threshold 0.85       (higher = FEWER speakers, despite the help text)
#   min-speaker floor 8s           (post-filter reassigns phantom speakers)

# model experiments via Docker Model Runner quant tags:
docker model pull hf.co/bartowski/Qwen2.5-32B-Instruct-GGUF:IQ4_XS
```

---

*Tested with docker-agent v1.90 to v1.97 (brew, while Docker Desktop bundled v1.79 to v1.88), Docker Model Runner with Qwen2.5-14B-Instruct Q4_K_M, and the `speech` CLI on a MacBook Pro M4, 32 GB. The stream-truncation fix landed in docker-agent v1.90.0.*

---

## References

- [The full project code (scripts, agent YAML, prompt) in my technical_notebook](https://github.com/matteobisi/technical_notebook/tree/main/projects/speech)
- [Docker Model Runner documentation](https://docs.docker.com/ai/model-runner/)
- [Docker Model Runner API reference (chat/completions, completions, embeddings)](https://docs.docker.com/ai/model-runner/api-reference/)
- [Docker: Run LLMs locally (Model Runner runs llama.cpp as a host process on Apple Silicon)](https://www.docker.com/blog/run-llms-locally/)
- [Docker Agent (docker-agent) on GitHub](https://github.com/docker/docker-agent)
- [Issue #3298 (stream truncation on long local-model prefill)](https://github.com/docker/docker-agent/issues/3298)
- [PR #3302 (the fix: retry stream truncation, clarify the error)](https://github.com/docker/docker-agent/pull/3302)
- [docker-agent v1.90.0 release](https://github.com/docker/docker-agent/releases/tag/v1.90.0)
- [Qwen2.5-14B-Instruct GGUF by bartowski on Hugging Face](https://huggingface.co/bartowski/Qwen2.5-14B-Instruct-GGUF)
- [speech CLI (native Apple Silicon ASR)](https://github.com/soniqo/speech-swift)
- [pyannote speaker diarization](https://github.com/pyannote/pyannote-audio)
