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
.post-content pre {
  font-size: 0.72em;
  line-height: 1.45;
}
.post-content pre code {
  font-size: inherit;
}
</style>

<div class="post-content" markdown="1">

# simple ai - a proposed inference-time framework for reducing ai slop.

<div style="font-size: 0.95em; color: #666; margin: 1.5rem 0 2rem 0; padding-bottom: 1rem; border-bottom: 1px solid #ddd;">
<span>september 29, 2026</span> • <span>12 min read</span>
</div>

<div style="font-size: 1.1em; font-style: italic; margin: 0 0 2rem 0;">
this post was written with extensive discussions and fun chats with omar khattab, tamara broderick, and dennis wei.
</div>

## introduction

at some point, you may have experienced the feeling of reading ai-generated content, doing a double take, rereading carefully, and realizing that very little was *actually* said. the verbose yet vacuous nature of ai-generated text makes the content needlessly hard to engage with.

tasteful writing and coding are central to knowledge work, yet "good taste" – relevance, clarity, and concision – seems to be notoriously hard for modern llms to acquire. taste of this kind is notably non-verifiable and difficult to quantify, making it poorly served by today's training pipelines. the result is the now-ubiquitous ["ai slop"](https://arxiv.org/abs/2509.19163): text that is superficially fluent but lacking in style and substance ([shaib et al., 2025](https://arxiv.org/abs/2509.19163); [chakrabarty et al., 2025](https://arxiv.org/abs/2409.14509)).

to investigate what user's think of chat–assistant responses, we ran a small-scale exploratory analysis of [thoughttrace](https://thoughttrace-project.github.io/), a large-scale dataset of real-world multi-turn human–ai conversations with users' *self-reported reactions* to chat–assistant responses. the dataset comprises 1,058 users and 2,155 conversational turns. through annotating a random subset of 200 conversations with a human and llm-judge (gemini-2.5-pro), the two prominent user complaints we found were both related to style and presentation:

1. **unnecessary detail.** users complain that too much unnecessary detail blurs the essence, or core idea, behind the response.
2. **broad but shallow.** users complained that the scope of responses is too expansive, but simultaneously reported that models fail to go deeply on what users actually needed.

**how did we end up here?** machine learning has historically framed model improvement as optimization against a low-dimensional reward, whether learned from human preferences (rlhf) or derived from verifiable outcomes (rlvr). we argue that this paradigm is poorly equipped to address the problem of taste and that it has instead contributed to ai-slop. there is a large body of works on precisely stylistic failures in human-written content: from poor stylistic coding choices (fowler, 1999; tufano et al., 2015) to poorly-written prose and how to fix it (williams, 1990; pinker, 2014; sword, 2012). models are trained on enormous quantities of mediocre human text with exactly such failures, and scalar preference rewards are too coarse to disentangle good style from bad. a marginally better reward function attached to the same single-pass autoregressive generation process will not get us to clean, "de-slopped" writing.

<div style="font-size: 1.2em; font-style: italic; padding: 1.5rem; border-left: 4px solid #5d2f9d; margin: 1rem 0;">
as models become more capable, we can elevate them from optimizing scalar rewards to following writing or coding <em>best practices</em> directly.
</div>

it would be silly for a human to produce a high-quality piece of prose by practicing under intense scalar feedback until they can churn out a masterpiece in a single pass. instead, humans write iteratively: they start with an outline; they brainstorm a first draft to get their ideas down; they discard drafts; they follow guidelines or best practices ([williams, 2005](https://en.wikipedia.org/wiki/Style:_Lessons_in_Clarity_and_Grace); [strunk, 2000](https://www.gutenberg.org/ebooks/37134)); and they take multiple revision passes, each with a specific goal in mind to fix one principle at a time (e.g., fixing structure, then tone). *we propose to put llms in the same setting that humans use to write tasteful prose and clean code.* concretely, this requires proposing a new inference-time framework:

**llm-generated output as an object to iterate for multiple iterations.** rather than generating a response in a single autoregressive pass, the response is constructed and refined over multiple drafting, critique, and revision passes. this form of inference-time scaling is now standard for verifiable domains such as mathematical reasoning but remains largely unexplored for writing. under the current paradigm of autoregressive, one-shot writing, we could imagine having a model *think* really hard (chain-of-thought) for two hours and then write the final response in fifteen seconds. what we want instead is an agent that works the way writers actually work: sketch a bad outline and tinker with the draft; edit a *specific* section with respect to a *specific* best practice. then repeat.

**self-scaffolding agents organized around codified best practices.** in place of a low-dimensional reward, the agent scaffolds its own work around an expressive, human-interpretable rulebook of domain expertise – a library of writing (williams, 1990; pinker, 2014) or coding (fowler, 1999; tufano et al., 2015) principles.

this research proposal outlines an approach to operationalizing taste through this agentic, inference-time framework. in order, we propose, (i) a rigorous evaluation methodology for ai slop, (ii) the design of self-scaffolding writer–reader agents that audit their drafts against best-practice rubrics, and (iii) reinforcement learning over the agentic harness itself, so that open, small models can learn how to decompose and orchestrate their revision work. while frontier labs have historically folded everything into rlhf or rlvr, the framework proposed here is qualitatively different: the rubric is not collapsed into a scalar; it is kept *expressive* and guides the model in stages at inference time.

## related work

**inference-time iteration and self-refinement.** a substantial body of work scales inference-time compute through fixed self-correction workflows. [self-refine](https://arxiv.org/abs/2303.17651) prompts a model to iteratively critique and revise its own outputs. [reflexion](https://arxiv.org/abs/2303.11366) converts environment feedback into verbal self-reflections that condition subsequent attempts. where has even been work proposed on learning correctors have been trained to map outputs to improved outputs (welleck et al., 2023); however, this work was done for more verifiable domains, such as math. test-time-compute scaling and process supervision have shown large gains in verifiable domains such as mathematics (snell et al., 2024; lightman et al., 2023). our proposal differs from this previous line of work in two ways. first, prior pipelines are typically *fixed workflows*; we instead propose that the agent should be free to decide – and should *learn via rl to decide* (stream 3) – how to break up its drafting and revision work. second, self-refine and its successors are largely structured around verifiable tasks; they are not organized around an explicit framework of taste. instead, we leverage the fact that *agents* are recently at the point where they are capable enough to follow *expressive rulebooks* directly. *the aim of our proposal is to develop a framework for how adherence to best practices should be decomposed across focused passes.*

**rubric-guided rewards.** a complementary line of work uses structured rubrics in place of preference signals. rubrics-as-rewards extends rlvr to non-verifiable domains by aggregating checklist-style rubric judgments into a reward for on-policy post-training (gunjal et al., 2025); and auto-rubric pipelines are typically constructed with a human in the loop. critically, these approaches remain *post-training* methods: the expressive rubric is ultimately reduced to a scalar reward used to update weights for single-pass autoregressive generation. a training-centric approach would bake complex rubrics straight into the weights via reinforcement learning. we propose instead to keep the rubric intact and *break it down at inference time*, so that good writing becomes something that is learned as a curriculum: through *one principle* and *one section* at a time.

**pipelines to edit ai writing.** closest to our project ideas is a small body of work that builds pipelines to edit ai writing. chakrabarty et al. (2025) define categories of idiosyncrasies common in ai writing (e.g., cliches, redundant exposition) and builds a pipeline that identifies problematic spans in a draft and rewrites each one using few-shot prompts authored with help from professional writers. a follow-up (chakrabarty et al., 2025) trains this behavior into the model: they fine-tune gpt-4o and llama-3.1-70b on professional writers' edit traces and train writing-quality reward models to score edited drafts against their originals. our proposed framework differs in two key ways. first, these methods are rigid: a model identifies spans that violate a category and then makes local edits to each in a single pass, whereas we revise sequentially, letting the agent schedule its own passes and address one principle from the rubric at a time. second, their automated editing pipeline is trained on expert human edits, whereas ours is based on established style guides at inference time. in short, we spend inference-time compute on guided, iterative revising based on well-known style guides, such as [*the elements of style*](https://www.gutenberg.org/ebooks/37134) and *refactoring: improving the design of existing code*, rather than local edits based on an editor.

## separating the reading mind from the writing mind

> *"quite often you will discover, on examining the completed work, that there are serious flaws in the arrangement of the material, calling for transpositions."*
>
> — william strunk jr., *the elements of style*

for human writers, expressing ideas concisely requires careful editing and iterative drafting: a multi-step revision process that llms do not natively perform. revision requires shifting perspectives: generating text as the *writer*, then evaluating it through the eyes of a *reader*. as a feasibility study, we replicated this separation in an llm by splitting the task: a writer agent creates a first draft, and a reader agent critiques the draft against a style rubric; the writer then revises based on the critique.

we compare this approach to an alternative approach of supplying the rubric in the system prompt during composition. in that baseline, the writer holds the rubric as a guide while drafting: in principle it can plan an active-voice sentence in advance, or suppress a hedging qualifier before it appears. in the writer–reader framework, by contrast, the rubric only enters *after* the draft exists. the question our study asks: is it better to guide the writer with the rubric during composition, or to let a reader discover violations and pass structured feedback back to the writer?

### methods

we compare the ability of an llm (`gpt-5-mini`) to conform to [*the elements of style*](https://www.gutenberg.org/ebooks/37134) under the two approaches below.

- **rubric in system prompt.** a single forward pass with our *distilled elements of style* guide supplied verbatim as the system prompt (no wrapping, prefix, or additional formatting). the user prompt is then given and the model generates its response with the guide in context.
- **agentic writing (writer–reader loop).** the writer drafts with *no* rubric in its context. a separate reader call (the same underlying model, with the rubric in context) audits the draft line-by-line and returns a structured critique. the writer receives the critique and revises. critique and revision may repeat for up to two rounds.

we evaluated responses on a random subset of 10 prompts drawn from a corpus of 1,000 natural-language queries spanning everyday curiosity (cooking, climbing, biology, music, software, etc.).[^1]

**reader critique format.** the reader returns a json object conforming to the schema:

```
{
  "violations": [
    {"principle": "...", "span": "...", "reason": "..."},
    ...
  ]
}
```

each entry names the violated principle, quotes the offending span verbatim, and gives a one-line reason. the structured output is rendered into a bulleted feedback message for the writer.

**revision instruction.** the writer is then handed the following system prompt: *"a style reviewer flagged the issues below. revise your response so each flagged issue is fixed. you may make small edits in service of fixing those issues, but do not rewrite content that wasn't flagged."*

**scoring.** to score the two approaches, an independent audit call (same model) runs the same *distilled elements of style* rubric over each final output and returns the violations it observes. we report the total count of flagged violations per condition.

### results

<table style="width: 100%; border-collapse: collapse; margin: 2rem 0; font-size: 0.98em;">
  <thead>
    <tr style="border-bottom: 2px solid #333;">
      <th style="text-align: left; padding: 0.6rem 0.4rem;">approach</th>
      <th style="text-align: right; padding: 0.6rem 0.4rem;">style violations / response</th>
    </tr>
  </thead>
  <tbody>
    <tr style="border-bottom: 1px solid #ddd;">
      <td style="padding: 0.6rem 0.4rem;">baseline (no system prompt)</td>
      <td style="text-align: right; padding: 0.6rem 0.4rem;">1.9</td>
    </tr>
    <tr style="border-bottom: 1px solid #ddd;">
      <td style="padding: 0.6rem 0.4rem;">rubric in system prompt</td>
      <td style="text-align: right; padding: 0.6rem 0.4rem;">1.8</td>
    </tr>
    <tr style="border-bottom: 1px solid #333;">
      <td style="padding: 0.6rem 0.4rem;">agentic writing (writer–reader)</td>
      <td style="text-align: right; padding: 0.6rem 0.4rem;"><strong>0.5</strong></td>
    </tr>
  </tbody>
</table>

<div style="font-size: 0.95em; color: #666; margin: -1rem 0 2rem 0;">
<em>distilled elements of style</em> violations across responses to 10 prompts. the agentic writing approach yields a sharp reduction in per-response style violations.
</div>

without any guidance (a generator model writes a single draft), the model accumulates 19 violations across 10 responses (1.9 violations per response). with a *rubric in the system prompt*, we see the number of violations barely decrease, with 18 violations remaining across 10 responses (1.8 per response). the writer–reader paradigm, by contrast, results in a sharp drop in the number of violation, with 5 violations (0.5 per response) across 10 responses, where 7 of 10 responses have 0 violations.

there were a few common violations that appear after the writing loop. most prominent examples are `active-voice` (3 of 5 remaining violations) and `parallel-construction` (1 of 5). converting a passive construction to an active one typically requires reshaping the clause around it, which may brush up against the conservative revise instruction. violations that can be repaired with local edits, `avoid-qualifiers`, `omit-needless-words`, `positive-form`, are essentially eliminated by the second pass. these results, while preliminary, highlight the substantial headroom available to a principled agentic framework.

## research plan

### stream 1: defining and evaluating "tasteful" ai writing

our first contribution will be to define and evaluate what it means for writing to be clean and tasteful. to start, we will build on previously-proposed taxonomies of ai slop: both [shaib et al. (2025)](https://arxiv.org/abs/2509.19163) and chakrabarty et al. (2025) identify failure modes of llm-outputted text, through interviews with a small group of experts in nlp, writing, and philosophy. we plan to draw from these identified hallmarks of ai-slop as a starting point to enforce what "good style" in llm-writing would look like. notably, two dimensions are important when it comes to measuring ai-slop: one is information density, the amount of substantive content relative to text length, and another is information relevance, how well the content addresses the specific nuances of the prompt (shaib et al., 2025). the objective then, at a high level, is to produce writing that is both relevant and concise.

the goal of this stream will be to create a benchmark for "taste" in ai models, with the two dimensions of relevance and concision acting as guiding principles. notably, no benchmark (to the best of our knowledge) currently exists that focuses on enterprise-specific writing tasks: settings where clear, concise writing are increasingly important.

we will propose a benchmark, *stylebench*, designed to evaluate a model's ability to transform first-draft inputs into polished writing across various domains. we plan to include tasks including: producing cohesive marketing strategies from messy notes, revising preliminary drafts into rigorous technical documentation, and refactoring disorganized code to adhere to established refactoring practices. we propose to collect the "rough draft" data from several online forums: messy marketing notes can be found on the enron email corpus. we plan to obtain disorganized code from open source github repositories.

### stream 2: self-scaffolding agents for anti-slop writing

the second stream designs self-reflecting agents for anti-slop writing. our first hypothesis concerns the degree to which slop detection and repair (in writing, code, and documentation) can be automated with good agent scaffolding around known best practices (strunk, 2000; williams, 1990). specifically, we hypothesize the following: models are not very good at outputting "tasteful" content when asked during generation, but particular harness choices will make them much better:

1. distilling taste from best-practice guides: we hypothesize that, for humans, much progress in taste and style can be achieved by abiding by a checklist-structured handbook (williams, 1990; tufano et al., 2015), and that agents are now capable of picking up on good taste through access to such handbooks.
2. decomposing the rubric: we hypothesize that it is better to have the agent focus on improving one rubric principle per focused pass, instead of attempting to master all principles at once.
3. decomposing the llm-generated content (e.g., code or text): we hypothesize that it is hard for a model to attend to all sections of a document at once, and that critics should narrow their concerns to focus on one section at a time.

another design goal we propose is *legible edits*. best practices may conflict: omitting needless words can fight clarity; active voice can fight emphasis; parallel construction can fight concision. rather than resolving these tensions without the user knowing, the agent should surface them. we will have the harness emit, alongside its final output, a report of the conscious trade-offs it chose to make between conflicting best practices. this report serves as an interpretability artifact for the human reader and supervision for the evaluation framework of stream 1.

### stream 3: reinforcement learning over the agentic harness

finally, we believe that small, open models can also develop such capabilities within the proposed harness, and large models may not be great at this out-of-the-box. we will use reinforcement learning (rl) on the agent to learn how to better utilize expert handbooks. our plan here is for rl to operate at the level of the *harness*: learning how to decompose the work, which lenses to apply in which order, when to stop revising, and how to allocate a fixed inference budget across passes. we propose to evaluate whether small open models post-trained within this harness can match the writing quality of larger frontier models operating without it. this would help establish the harness itself as a meaningful enabler allowing the model to pick up on "taste."

[^1]: see <https://huggingface.co/datasets/JennyHuang19/cutTheFluff> for the full list of prompts.

</div>
