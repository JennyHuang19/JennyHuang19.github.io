---
layout: post
title: "simple ai: inference-time frameworks for reducing ai slop"
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

# simple ai: inference-time frameworks for reducing ai slop.

<div style="font-size: 0.95em; color: #666; margin: 1.5rem 0 2rem 0; padding-bottom: 1rem; border-bottom: 1px solid #ddd;">
<span>September 29, 2026</span> • <span>9 min read</span>
</div>

at some point, you may have read a paragraph of ai-generated text, done a double take, reread it carefully, and realized that very little was *actually* said. the text was fluent. it was organized. it had three bullet points and a closing summary. and it contained almost nothing.

tasteful writing and coding are central to knowledge work, yet "good taste" – relevance, clarity, and concision – seems to be notoriously difficult for language models to acquire. taste of this kind is non-verifiable and hard to quantify, so it is poorly served by today's training pipelines. the result is the now-ubiquitous [*ai slop*](https://arxiv.org/abs/2509.19163): text that is superficially fluent but lacking in style and substance. as more writing and coding gets delegated to ai systems, slop stops being an aesthetic complaint. verbose, vacuous output introduces ambiguity, obscures errors, and erodes the accountability that real work requires.

## what people actually complain about.

to get a sense of what bothers users about assistant responses, we ran a small exploratory analysis of [thoughttrace](https://thoughttrace-project.github.io/), a dataset of real-world multi-turn human-ai conversations paired with users' *self-reported reactions* – 1,058 users and 2,155 conversational turns. annotating a random subset of 200 conversations with a human and an llm judge (gemini-2.5-pro), the two most prominent complaints we found were both about style and presentation rather than correctness:

- **unnecessary detail.** too much padding blurs the essence, or core idea, behind the response.
- **broad but shallow.** the scope of the response is expansive, but the model fails to go deep on the one part the user actually needed.

neither of these is a knowledge failure. the model knew the answer. it just could not decide what to leave out.

## how we ended up here.

machine learning has historically framed model improvement as optimization against a low-dimensional reward, whether learned from [human preferences](https://arxiv.org/abs/1706.03741) ([rlhf](https://arxiv.org/abs/2203.02155)) or derived from [verifiable outcomes](https://arxiv.org/abs/2501.12948) (rlvr). that framing works beautifully when the thing you want is checkable. taste is not checkable. relevance and concision are real properties of a piece of writing, but they resist collapsing into a number.

meanwhile, there is a large body of work on stylistic failure in *human* writing – from poor structural choices in code to muddy prose and how to fix it. models are trained on enormous quantities of mediocre human text containing exactly those failures, and a scalar preference reward is far too coarse to disentangle good style from bad.

<div style="font-size: 1.2em; font-style: italic; padding: 1.5rem; border-left: 4px solid #5d2f9d; margin: 1rem 0;">
a marginally better reward function attached to the same single-pass generation process will not get us to clean writing.
</div>

so i've been interested in a different framing: as models become more capable, we can elevate them from optimizing a scalar reward to following writing and coding *best practices* directly.

## writing is iterative.

it would be strange to teach a person to write by having them practice under intense scalar feedback until they could produce a masterpiece in a single pass. that is roughly what we ask of language models.

humans write badly on purpose first. we sketch an outline, get a bad draft down to see the shape of the thing, throw drafts away, consult [style guides](https://www.gutenberg.org/ebooks/37134), and take several revision passes – each with one goal in mind. fix the structure. then fix the tone. then cut.

putting a model in that same setting requires two changes at inference time.

**treat the output as an object to iterate on.** rather than generating a response in a single autoregressive pass, the response gets constructed and refined over multiple drafting, critique, and revision passes. this form of [inference-time scaling](https://arxiv.org/abs/2408.03314) is now standard for verifiable domains like [mathematical reasoning](https://arxiv.org/abs/2305.20050) but remains largely unexplored for writing. under one-shot writing, we could have a model *think* very hard for two hours and then produce the final response in fifteen seconds. what i want instead is an agent that works the way writers actually work: sketch a bad outline, tinker with the draft, edit a *specific* section against a *specific* principle, then repeat.

**scaffold the work around a codified rulebook.** in place of a low-dimensional reward, the agent organizes its own work around an expressive, human-readable library of domain expertise – writing principles, or refactoring principles for code. the bet is that agents are finally capable enough to follow an expressive rulebook directly, so we no longer need to compress that rulebook into a number.

this is different from existing self-correction pipelines like [self-refine](https://arxiv.org/abs/2303.17651) and [reflexion](https://arxiv.org/abs/2303.11366) in two ways. those are mostly *fixed* workflows, where the sequence of passes is decided in advance; i'd rather the agent decide how to break up its own work. and they are structured around verifiable tasks rather than an explicit framework of taste.

## separating the reading mind from the writing mind.

> *"quite often you will discover, on examining the completed work, that there are serious flaws in the arrangement of the material, calling for transpositions."*
>
> — william strunk jr., *the elements of style*

revision requires shifting perspective: generating text as the *writer*, then evaluating it through the eyes of a *reader*. as a feasibility study, i replicated that separation by splitting the task across two calls to the same model (gpt-5-mini). a writer agent drafts with **no** rubric in its context. a reader agent – same model, rubric in context – audits the draft line by line and returns a structured critique naming each violated principle, the offending span, and a one-line reason. the writer then revises, for up to two rounds.

the comparison i cared about: is it better to hand the writer the rubric *while* it composes, or to let a reader discover violations afterward and pass them back?

the first option seems strictly more informative. a writer holding the rubric can plan an active-voice sentence in advance, or suppress a hedging qualifier before it ever appears. the second option only sees the rubric after the draft exists.

<table style="width: 100%; border-collapse: collapse; margin: 2rem 0; font-size: 0.98em;">
  <thead>
    <tr style="border-bottom: 2px solid #333;">
      <th style="text-align: left; padding: 0.6rem 0.4rem;">approach</th>
      <th style="text-align: right; padding: 0.6rem 0.4rem;">style violations / response</th>
    </tr>
  </thead>
  <tbody>
    <tr style="border-bottom: 1px solid #ddd;">
      <td style="padding: 0.6rem 0.4rem;">baseline (no rubric)</td>
      <td style="text-align: right; padding: 0.6rem 0.4rem;">1.9</td>
    </tr>
    <tr style="border-bottom: 1px solid #ddd;">
      <td style="padding: 0.6rem 0.4rem;">rubric in system prompt</td>
      <td style="text-align: right; padding: 0.6rem 0.4rem;">1.8</td>
    </tr>
    <tr style="border-bottom: 1px solid #333;">
      <td style="padding: 0.6rem 0.4rem;"><strong>writer–reader loop</strong></td>
      <td style="text-align: right; padding: 0.6rem 0.4rem;"><strong>0.5</strong></td>
    </tr>
  </tbody>
</table>

we evaluated on ten prompts drawn from a corpus of 1,000 [natural-language queries](https://huggingface.co/datasets/JennyHuang19/cutTheFluff) spanning everyday curiosity – cooking, climbing, biology, music, software. an independent audit call, running the same rubric, scored each final output.

handing the writer the rubric up front did essentially nothing: 18 violations across 10 responses, against 19 at baseline. the writer–reader loop cut that to 5, with 7 of 10 responses coming back clean.[^1]

i take this to mean the rubric is not hard to *understand*. it is hard to apply to text that does not exist yet. you cannot tell whether a word is needless until you have written it.

what survives the loop is informative too. the remaining violations are concentrated in active voice (3 of 5) and parallel construction (1 of 5) – both of which require reshaping a clause rather than deleting a word, and so brush up against the conservative instruction to leave unflagged content alone. the violations that yield to local edits, like hedging qualifiers and needless words, are essentially eliminated by the second pass.

## legible edits.

best practices conflict. omitting needless words can fight clarity. active voice can fight emphasis. parallel construction can fight concision. a system that resolves those tensions silently is making style decisions on your behalf and hiding them.

so a second design goal: the harness should emit, alongside its final output, a short report of the trade-offs it consciously made. that report is useful twice over – as an interpretability artifact for the reader, and as a supervision signal for evaluating the system itself.

## what i'd like to try next.

**learning the harness, not the prose.** rather than training a model to write well in one pass, train it to decide how to break up the work: which principle to apply in which order, when to stop revising, how to spend a fixed inference budget across passes. the question i find most interesting is whether a small open model inside a good harness can match a much larger model operating without one. if it can, then taste lives in the process, not the weights.

**better evaluation.** there is no benchmark i know of for the kind of writing most people actually do at work. the closest is the writing quality benchmark, which consolidates five writing-preference datasets into roughly 4,700 judgments from professional writers – valuable, but creative-writing adjacent. i'd like something built on messy real inputs: rough notes into a coherent memo, a preliminary draft into rigorous technical documentation, disorganized code refactored into something a colleague can read. rough-draft material is not hard to come by; old email corpora and open source repositories are full of it. scoring would likely combine a small panel of human experts with a collection of strong [llm judges](https://arxiv.org/abs/2306.05685).

## taste as a process.

the framing i keep returning to is that we have been treating taste as a property of a *model* when it may be better treated as a property of a *process*. people do not write well because they have internalized the elements of style so thoroughly that clean prose falls out on the first try. they write well because they wrote something worse first, and then looked at it again.

that is a low bar for a machine to clear. we mostly just haven't been asking it to.

[^1]: ten prompts is a small sample – enough to suggest the effect is not subtle, not enough to put a meaningful error bar on it.

</div>
