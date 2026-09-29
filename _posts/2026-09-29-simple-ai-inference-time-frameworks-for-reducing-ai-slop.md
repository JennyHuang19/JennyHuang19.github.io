---
layout: post
title: "simple ai - a proposed inference-time framework for reducing ai slop"
date: 2026-09-29
---

<style>
.post-content {
  max-width: 800px;
  margin: 0 auto;
  padding: 0 2.5rem;
}
.post-content a {
  color: #a88bd0;
  text-decoration: none;
  border-bottom: 1px solid #a88bd0;
}
.post-content a:hover {
  color: #a88bd0;
  border-bottom-color: #a88bd0;
}
.post-content a:visited {
  color: #a88bd0;
}
</style>

<div class="post-content" markdown="1">

# simple ai - a proposed inference-time framework for reducing ai slop.

<div style="font-size: 0.95em; color: #666; margin: 1.5rem 0 2rem 0; padding-bottom: 1rem; border-bottom: 1px solid #ddd;">
<span>September 29, 2026</span> • <span>12 min read</span>
</div>

## Introduction

At some point, you may have experienced the feeling of reading AI-generated content, doing a double take, rereading carefully, and realizing that very little was *actually* said. The verbose yet vacuous nature of AI-generated text makes the content needlessly difficult to engage with.

Tasteful writing and coding are central to knowledge work, yet "good taste" – relevance, clarity, and concision – seems to be notoriously difficult for modern large language models (LLMs) to acquire. Taste of this kind is notably non-verifiable and difficult to quantify. Thus, it is poorly served by today's training pipelines, and the result is the now-ubiquitous ["AI slop"](https://arxiv.org/abs/2509.19163): text that is superficially fluent but lacking in style and substance (Shaib et al., 2025; Chakrabarty et al., 2025). For many industries, as workers increasingly delegate writing and coding tasks to AI systems, AI-slop outputs are starting to introduce ambiguity, obscure errors, and erode the accountability that enterprise systems require.

To investigate what user's think of chat–assistant responses, we ran a small-scale exploratory analysis of [ThoughtTrace](https://thoughttrace-project.github.io/), a large-scale dataset of real-world multi-turn human–AI conversations with users' *self-reported reactions* to chat–assistant responses. The dataset comprises 1,058 users and 2,155 conversational turns. Through annotating a random subset of 200 conversations with a human and LLM-judge (Gemini-2.5-pro), the two prominent user complaints we found were both related to style and presentation:

1. **Unnecessary detail.** Users complain that too much unnecessary detail blurs the essence, or core idea, behind the response.
2. **Broad but shallow.** Users complained that the scope of responses is too expansive, but simultaneously reported that models fail to go deeply on what users actually needed.

**How did we end up in a world filled with AI slop?** Machine learning has historically framed model improvement as optimization against a low-dimensional reward, whether learned from [human preferences](https://arxiv.org/abs/1706.03741) ([RLHF](https://arxiv.org/abs/2203.02155)) or derived from [verifiable outcomes](https://arxiv.org/abs/2501.12948) (RLVR). We argue that this paradigm is poorly equipped to address the problem of taste and that it has instead contributed to AI-slop. There is a large body of works on precisely stylistic failures in human-written content: from poor stylistic coding choices (Fowler, 1999; Tufano et al., 2015) to poorly-written prose and how to fix it (Williams, 1990; Pinker, 2014; Sword, 2012). Models are trained on enormous quantities of mediocre human text with exactly such failures, and scalar preference rewards are too coarse to disentangle good style from bad. A marginally better reward function attached to the same single-pass autoregressive generation process will not get us to clean, "de-slopped" writing.

<div style="font-size: 1.2em; font-style: italic; padding: 1.5rem; border-left: 4px solid #5d2f9d; margin: 1rem 0;">
So, we propose a different framework: as models become more capable, we can elevate them from optimizing scalar rewards to following writing or coding <em>best practices</em> directly.
</div>

It would be silly for a human to produce a high-quality piece of prose by practicing under intense scalar feedback until they can churn out a masterpiece in a single pass. Instead, humans write iteratively: they start with an outline; they brainstorm a first draft to get their ideas down; they discard drafts; they follow guidelines or best practices (Williams, 2005; [Strunk, 2000](https://www.gutenberg.org/ebooks/37134)); and they take multiple revision passes, each with a specific goal in mind to fix one principle at a time (e.g., fixing structure, then tone). *We propose to put LLMs in the same setting that humans use to write tasteful prose and clean code.* Concretely, this requires proposing a new inference-time framework:

**LLM-generated output as an object to iterate for multiple iterations.** Rather than generating a response in a single autoregressive pass, the response is constructed and refined over multiple drafting, critique, and revision passes. This form of [inference-time scaling](https://arxiv.org/abs/2408.03314) is now standard for verifiable domains such as [mathematical reasoning](https://arxiv.org/abs/2305.20050) but remains largely unexplored for writing. Under the current paradigm of autoregressive, one-shot writing, we could imagine having a model *think* really hard (chain-of-thought) for two hours and then write the final response in fifteen seconds. What we want instead is an agent that works the way writers actually work: sketch a bad outline and tinker with the draft; edit a *specific* section with respect to a *specific* best practice. Then repeat.

**Self-scaffolding agents organized around codified best practices.** In place of a low-dimensional reward, the agent scaffolds its own work around an expressive, human-interpretable rulebook of domain expertise – a library of writing (Williams, 1990; Pinker, 2014) or coding (Fowler, 1999; Tufano et al., 2015) principles.

This research proposal outlines an approach to operationalizing taste through this agentic, inference-time framework. In order, we propose, (i) a rigorous evaluation methodology for AI slop, (ii) the design of self-scaffolding writer–reader agents that audit their drafts against best-practice rubrics, and (iii) reinforcement learning over the agentic harness itself, so that open, small models can learn how to decompose and orchestrate their revision work. While frontier labs have historically folded everything into RLHF or RLVR, the framework proposed here is qualitatively different: the rubric is not collapsed into a scalar; it is kept *expressive* and guides the model in stages at inference time.

## Related Work

**Inference-time iteration and self-refinement.** A substantial body of work scales inference-time compute through fixed self-correction workflows. [Self-Refine](https://arxiv.org/abs/2303.17651) prompts a model to iteratively critique and revise its own outputs. [Reflexion](https://arxiv.org/abs/2303.11366) converts environment feedback into verbal self-reflections that condition subsequent attempts. Where has even been work proposed on learning correctors have been trained to map outputs to improved outputs (Welleck et al., 2023); however, this work was done for more verifiable domains, such as math. Test-time-compute scaling and process supervision have shown large gains in verifiable domains such as mathematics (Snell et al., 2024; Lightman et al., 2023). Our proposal differs from this previous line of work in two ways. First, prior pipelines are typically *fixed workflows*; we instead propose that the agent should be free to decide – and should *learn via RL to decide* (Stream 3) – how to break up its drafting and revision work. Second, Self-Refine and its successors are largely structured around verifiable tasks; they are not organized around an explicit framework of taste. Instead, we leverage the fact that *agents* are recently at the point where they are capable enough to follow *expressive rulebooks* directly. *The aim of our proposal is to develop a framework for how adherence to best practices should be decomposed across focused passes.*

**Rubric-guided rewards.** A complementary line of work uses structured rubrics in place of preference signals. Rubrics-as-Rewards extends RLVR to non-verifiable domains by aggregating checklist-style rubric judgments into a reward for on-policy post-training (Gunjal et al., 2025); and auto-rubric pipelines are typically constructed with a human in the loop. Critically, these approaches remain *post-training* methods: the expressive rubric is ultimately reduced to a scalar reward used to update weights for single-pass autoregressive generation. A training-centric approach would bake complex rubrics straight into the weights via reinforcement learning. We propose instead to keep the rubric intact and *break it down at inference time*, so that good writing becomes something that is learned as a curriculum: through *one principle* and *one section* at a time.

**Pipelines to Edit AI Writing.** Closest to our project ideas is a small body of work that builds pipelines to edit AI writing. Chakrabarty et al. (2025) define categories of idiosyncrasies common in AI writing (e.g., cliches, redundant exposition) and builds a pipeline that identifies problematic spans in a draft and rewrites each one using few-shot prompts authored with help from professional writers. A follow-up (Chakrabarty et al., 2025) trains this behavior into the model: they fine-tune GPT-4o and Llama-3.1-70B on professional writers' edit traces and train writing-quality reward models to score edited drafts against their originals. Our proposed framework differs in two key ways. First, these methods are rigid: a model identifies spans that violate a category and then makes local edits to each in a single pass, whereas we revise sequentially, letting the agent schedule its own passes and address one principle from the rubric at a time. Second, their automated editing pipeline is trained on expert human edits, whereas ours is based on established style guides at inference time. In short, we spend inference-time compute on guided, iterative revising based on well-known style guides, such as [*The Elements of Style*](https://www.gutenberg.org/ebooks/37134) and *Refactoring: Improving the Design of Existing Code*, rather than local edits based on an editor. We plan to start by building on these works in Stream 1, and on [LLM-as-judge methodology](https://arxiv.org/abs/2306.05685) for scalable evaluation.

## Preliminary Investigation: Separating the Reading Mind from the Writing Mind

> *"Quite often you will discover, on examining the completed work, that there are serious flaws in the arrangement of the material, calling for transpositions."*
>
> — William Strunk Jr., *The Elements of Style*

For human writers, expressing ideas concisely requires careful editing and iterative drafting: a multi-step revision process that LLMs do not natively perform. Revision requires shifting perspectives: generating text as the *writer*, then evaluating it through the eyes of a *reader*. As a feasibility study, we replicated this separation in an LLM by splitting the task: a writer agent creates a first draft, and a reader agent critiques the draft against a style rubric; the writer then revises based on the critique.

We compare this approach to an alternative approach of supplying the rubric in the system prompt during composition. In that baseline, the writer holds the rubric as a guide while drafting: in principle it can plan an active-voice sentence in advance, or suppress a hedging qualifier before it appears. In the writer–reader framework, by contrast, the rubric only enters *after* the draft exists. The question our study asks: is it better to guide the writer with the rubric during composition, or to let a reader discover violations and pass structured feedback back to the writer?

### Methods

We compare the ability of an LLM (`GPT-5-mini`) to conform to [*The Elements of Style*](https://www.gutenberg.org/ebooks/37134) under the two approaches below.

- **Rubric in System Prompt.** A single forward pass with our *Distilled Elements of Style* guide supplied verbatim as the system prompt (no wrapping, prefix, or additional formatting). The user prompt is then given and the model generates its response with the guide in context.
- **Agentic Writing (Writer–Reader Loop).** The writer drafts with *no* rubric in its context. A separate reader call (the same underlying model, with the rubric in context) audits the draft line-by-line and returns a structured critique. The writer receives the critique and revises. Critique and revision may repeat for up to two rounds.

We evaluated responses on a random subset of 10 prompts drawn from a corpus of 1,000 natural-language queries spanning everyday curiosity (cooking, climbing, biology, music, software, etc.).[^1]

**Reader critique format.** The reader returns a JSON object conforming to the schema:

```
{
  "violations": [
    {"principle": "...", "span": "...", "reason": "..."},
    ...
  ]
}
```

Each entry names the violated principle, quotes the offending span verbatim, and gives a one-line reason. The structured output is rendered into a bulleted feedback message for the writer.

**Revision instruction.** The writer is then handed the following system prompt: *"A style reviewer flagged the issues below. Revise your response so each flagged issue is fixed. You may make small edits in service of fixing those issues, but do not rewrite content that wasn't flagged."*

**Scoring.** To score the two approaches, an independent audit call (same model) runs the same *Distilled Elements of Style* rubric over each final output and returns the violations it observes. We report the total count of flagged violations per condition.

### Results

<table style="width: 100%; border-collapse: collapse; margin: 2rem 0; font-size: 0.98em;">
  <thead>
    <tr style="border-bottom: 2px solid #333;">
      <th style="text-align: left; padding: 0.6rem 0.4rem;">Approach</th>
      <th style="text-align: right; padding: 0.6rem 0.4rem;">Style Violations / Response</th>
    </tr>
  </thead>
  <tbody>
    <tr style="border-bottom: 1px solid #ddd;">
      <td style="padding: 0.6rem 0.4rem;">Baseline (No System Prompt)</td>
      <td style="text-align: right; padding: 0.6rem 0.4rem;">1.9</td>
    </tr>
    <tr style="border-bottom: 1px solid #ddd;">
      <td style="padding: 0.6rem 0.4rem;">Rubric in System Prompt</td>
      <td style="text-align: right; padding: 0.6rem 0.4rem;">1.8</td>
    </tr>
    <tr style="border-bottom: 1px solid #333;">
      <td style="padding: 0.6rem 0.4rem;">Agentic Writing (Writer–Reader)</td>
      <td style="text-align: right; padding: 0.6rem 0.4rem;"><strong>0.5</strong></td>
    </tr>
  </tbody>
</table>

<div style="font-size: 0.95em; color: #666; margin: -1rem 0 2rem 0;">
<em>Distilled Elements of Style</em> violations across responses to 10 prompts. The agentic writing approach yields a sharp reduction in per-response style violations.
</div>

Without any guidance (a generator model writes a single draft), the model accumulates 19 violations across 10 responses (1.9 violations per response). With a *Rubric in the System Prompt*, we see the number of violations barely decrease, with 18 violations remaining across 10 responses (1.8 per response). The writer–reader paradigm, by contrast, results in a sharp drop in the number of violation, with 5 violations (0.5 per response) across 10 responses, where 7 of 10 responses have 0 violations.

There were a few common violations that appear after the writing loop. Most prominent examples are `active-voice` (3 of 5 remaining violations) and `parallel-construction` (1 of 5). Converting a passive construction to an active one typically requires reshaping the clause around it, which may brush up against the conservative revise instruction. Violations that can be repaired with local edits, `avoid-qualifiers`, `omit-needless-words`, `positive-form`, are essentially eliminated by the second pass. These results, while preliminary, highlight the substantial headroom available to a principled agentic framework.

## Research Plan

The primary objective of this proposal is to develop, evaluate, and train inference-time agentic frameworks that produce slop-free *writing* and *code* by operationalizing codified best practices. The work is organized into three streams.

### Stream 1: Defining and Evaluating "Tasteful" AI Writing

Our first contribution will be to define and evaluate what it means for writing to be clean and tasteful. To start, we will build on previously-proposed taxonomies of AI slop: both [Shaib et al. (2025)](https://arxiv.org/abs/2509.19163) and Chakrabarty et al. (2025) identify failure modes of LLM-outputted text, through interviews with a small group of experts in NLP, writing, and philosophy. We plan to draw from these identified hallmarks of AI-slop as a starting point to enforce what "good style" in LLM-writing would look like. Notably, two dimensions are important when it comes to measuring AI-slop: one is information density, the amount of substantive content relative to text length, and another is information relevance, how well the content addresses the specific nuances of the prompt (Shaib et al., 2025). The objective then, at a high level, is to produce writing that is both relevant and concise.

The goal of this stream will be to create a benchmark for "taste" in AI models, with the two dimensions of relevance and concision acting as guiding principles. Notably, no benchmark (to the best of our knowledge) currently exists that focuses on enterprise-specific writing tasks: settings where clear, concise writing are increasingly important.

We will propose a benchmark, *StyleBench*, designed to evaluate a model's ability to transform first-draft inputs into polished writing across various domains. We plan to include tasks including: producing cohesive marketing strategies from messy notes, revising preliminary drafts into rigorous technical documentation, and refactoring disorganized code to adhere to established refactoring practices. We propose to collect the "rough draft" data from several online forums: messy marketing notes can be found on the Enron Email corpus. We plan to obtain disorganized code from open source Github repositories.

### Stream 2: Self-Scaffolding Agents for Anti-Slop Writing

The second stream designs self-reflecting agents for anti-slop writing. Our first hypothesis concerns the degree to which slop detection and repair (in writing, code, and documentation) can be automated with good agent scaffolding around known best practices (Strunk, 2000; Williams, 1990). Specifically, we hypothesize the following: models are not very good at outputting "tasteful" content when asked during generation, but particular harness choices will make them much better:

1. Distilling taste from best-practice guides: we hypothesize that, for humans, much progress in taste and style can be achieved by abiding by a checklist-structured handbook (Williams, 1990; Tufano et al., 2015), and that agents are now capable of picking up on good taste through access to such handbooks.
2. Decomposing the rubric: we hypothesize that it is better to have the agent focus on improving one rubric principle per focused pass, instead of attempting to master all principles at once.
3. Decomposing the LLM-generated content (e.g., code or text): we hypothesize that it is hard for a model to attend to all sections of a document at once, and that critics should narrow their concerns to focus on one section at a time.

Another design goal we propose is *legible edits*. Best practices may conflict: omitting needless words can fight clarity; active voice can fight emphasis; parallel construction can fight concision. Rather than resolving these tensions without the user knowing, the agent should surface them. We will have the harness emit, alongside its final output, a report of the conscious trade-offs it chose to make between conflicting best practices. This report serves as an interpretability artifact for the human reader and supervision for the evaluation framework of Stream 1.

### Stream 3: Reinforcement Learning over the Agentic Harness

Finally, we believe that small, open models can also develop such capabilities within the proposed harness, and large models may not be great at this out-of-the-box. We will use reinforcement learning (RL) on the agent to learn how to better utilize expert handbooks. Our plan here is for RL to operate at the level of the *harness*: learning how to decompose the work, which lenses to apply in which order, when to stop revising, and how to allocate a fixed inference budget across passes. We propose to evaluate whether small open models post-trained within this harness can match the writing quality of larger frontier models operating without it. This would help establish the harness itself as a meaningful enabler allowing the model to pick up on "taste."

[^1]: See <https://huggingface.co/datasets/JennyHuang19/cutTheFluff> for the full list of prompts.

</div>
