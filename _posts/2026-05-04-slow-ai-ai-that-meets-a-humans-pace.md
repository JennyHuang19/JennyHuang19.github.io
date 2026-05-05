---
layout: post
title: "slow ai: ai that matches a human's pace"
date: 2026-05-04
image: /images/post-figures/conceptual-multiverse.jpeg
---

<meta property="og:image" content="https://jennyhuang19.github.io/images/post-figures/conceptual-multiverse.jpeg" />
<meta property="og:image:width" content="1200" />
<meta property="og:image:height" content="630" />
<meta property="og:type" content="article" />

<style>
.post-content {
  max-width: 800px;
  margin: 0 auto;
  padding: 0 2.5rem;
}
.post-content a {
  color: #5d2f9d;
  text-decoration: none;
  border-bottom: 1px solid #5d2f9d;
}
.post-content a:hover {
  color: #5d2f9d;
  border-bottom-color: #5d2f9d;
}
.post-content a:visited {
  color: #5d2f9d;
}
</style>

<div class="post-content" markdown="1">

# slow ai: ai that matches a human's pace.

<div style="font-size: 0.95em; color: #666; margin: 1.5rem 0 2rem 0; padding-bottom: 1rem; border-bottom: 1px solid #ddd;">
<span>May 4, 2026</span> • <span>12 min read</span>
</div>

my mind digests information at a much slower pace than i'd like to believe. in college, i could breeze through a math lecture at 2x speed, convinced that i was following everything the professor was saying, only to stare blankly at a problem set not knowing where to begin.

math became very enjoyable for me once i slowed down and acknowledged that absorbing new information takes longer than i typically would anticipate. i would pick out a handful of high-quality problems, learn them inside and out, notice exactly where i got stuck, make a few logical leaps, and return to the same problem the very next day with a fresh pair of eyes. after a certain point, i realized that i didn't actually need to consume that much information at all. understanding a new topic well[^1] had more to do with engaging deeply with a few *core* concepts.

i believe our brains are wired to get stuck on simple problems for extended periods of time. we might toil over the same problem for months – thinking about it in the shower, on the bus, from multiple different angles and perspectives. great scientists and artists often do so for years.

<div style="font-size: 1.2em; font-style: italic; padding: 1.5rem; border-left: 4px solid #5d2f9d; margin: 1rem 0;">
i worry that the way ai has been introduced into our society is antithetical to the slow, non-linear type of thinking necessary for deep engagement with new ideas.
</div>

when information is hurled at us at 200 miles an hour – packaged in a fluent, convincing voice – the path of least resistance becomes to accept that information at face value, without taking the time to question, critique, verify, and make sense of it at our own pace.

## ai can be used for slow thinking.
just as we now have widespread access to a tool for [offloading thinking](https://www.ft.com/content/9c6a1daf-3c36-4035-bf74-1bedbc3e960d?syn-25a6b1a6=1), we have an equally capable one for facilitating deep thinking, the type necessary to reach states of understanding and creativity: ai can follow a [feyman-esque](https://fs.blog/feynman-technique/) trail of questions, generate concrete examples on-the-spot, pull in documents, [rubber-duck](https://en.wikipedia.org/wiki/Rubber_duck_debugging), and play devil’s advocate.

ethicist and cognitive scientist josh may offers a helpful rule of thumb for using ai in [intellectual tasks](https://joshdmay.substack.com/p/why-smart-people-make-weak-arguments): “you should use llms to generate inputs to your thinking, not outputs for others to read.”

## designing slow ai. 
lately, i've been thinking about ways to design ai to be more compatible with slow thinking.

first, an ai system should encourage the user to wrestle with the sequence of decisions that need to be made along the way to producing a final, polished response. just as understanding a mathematical proof requires wrestling with the underlying maze of failed paths along the way - one eventually feels solid about the sequence of logical steps that make a correct path correct - someone who receives an ai-generated response should strive to be familiar with the major conceptual decision nodes that constitute the final response. decomposing the path of conceptual decision nodes that take questions to answers allows the user to engage with alternative responses that could have been generated, reason through why those responses may have been valid (or invalid), and generalize those insights into future questions.

to test out a new mode of human-ai interaction, we created an interface that lays bare the messy [conceptual roadblocks](https://multiverse.csail.mit.edu/) – the conflicting assumptions, interpretations, and frameworks – that shape an ai's final response. the interface was a first attempt at allowing users to work through  a space of possible decisions and resulting outputs, in the form of an interactive decision tree. confronted with a multiplicity of decision points, participants of [our study](https://arxiv.org/abs/2604.17815) felt a stronger sense of ownership over the final llm-generated responses. when compared to a traditional linear chat interface, users were surprised to find how long it took them to work through just one response,[^2] and how much they learned from the different viewpoints along the way. our design draws inspiration from the concept of [multiverse analysis](https://pmc.ncbi.nlm.nih.gov/articles/PMC9636921/), a scientific method that specifies and runs a set of data-analytical choices, reporting results for each.

<iframe src="https://multiverse.csail.mit.edu/" width="100%" height="600px" style="border: 1px solid #ccc;"></iframe>

second, instead of generating standalone responses, an ai system could present solutions in the form of [deliberations](https://knightcolumbia.org/content/representative-ranking-for-deliberation-in-the-public-sphere) between different parties of [generative agents](https://arxiv.org/pdf/2304.03442). a discourse-style interface could surface the hidden assumptions and tradeoffs underlying complex, multiperspective problems. simply reframing the response as a debate may be enough to invite users to read as critics rather than recipients.

third, ai memory should be designed to prevent hidden assumptions from quietly accumulating in the context window. in [recent work](https://arxiv.org/abs/2602.24287), we find that, as chat histories progress, models tend to get caught in old pieces of code (see figure 2\) or vestiges of earlier responses that are no longer relevant. this problem of models becoming [“more repetitive, and sometimes subtly wrong”](https://www.reddit.com/r/ChatGPT/comments/1qngpqa/does_chatgpt_quietly_get_worse_in_long/) as chat histories progress is a familiar headache. rather than linearly accumulating a full conversation transcript in context – tunnel-visioning the model with past lines of reasoning – we can design smarter, more structured ways to condition on the past. one way is to create a wide-angled view of chat history, representing past conversations as [knowledge graphs](https://github.com/MemPalace/mempalace/tree/develop). the model then conditions only on a high-level summary of the past, just enough to guide retrieval, while seeing the full conversation details when they become relevant.

<figure style="text-align: center;">
  <img src="/images/post-figures/cartoon_v3.png" alt="figure 2. a real-world example of gpt-5.2 reusing outdated information found in the context window." width="7%" style="display: block; margin: 0 auto;" />
  <figcaption style="font-size: 1.02em;">figure 2. a real-world example of gpt-5.2 reusing outdated information in its context window. in a previous query, the user requested umap clustering code. in the next turn, the user requests the assistant to "use t-sne instead." left: when the previous assistant response remains in context, the model incorrectly carries over the jaccard metric from umap into the t-sne implementation. right: without the previous response in context, the model generates correct t-sne code with appropriate arguments.</figcaption>
</figure>

finally, the system should respect that not every human problem deserves to be touched by an ai. in the mid-70's, joseph weisenbaum's [*computer power and human reason*](https://archive.org/details/computerpowerhum0000weiz_v0i3) warns against consulting machines on tasks that require deeply human traits like *empathy* and *wisdom*. thus, we can design tools that encourage users to reflect on their [boundaries with ai](https://github.com/sanapandey/ai-boundaries). to test this out, we created a chrome extension that allows users of chat interfaces to define (by placing a pin in a quadrant graph) how involved they’d like an ai to be in different areas of their work and life - from direct, concrete responses to reflective questions thrown back to the user. based on the user's preferred boundaries, the tool produces a *memory* file that users can upload to any chat interface to serve as a guideline for the assistant.

<iframe src="/assets/ai-boundaries/onboarding.html" width="100%" height="600px" style="border: 1px solid #ccc;"></iframe>

## dangers of the growing culture of fast ai.

i worry that the culture surrounding [autonomous ai](https://arxiv.org/pdf/2604.15597) is self-reinforcing: the less we engage, the harder it is to find our ways [back to engaging](https://blog.cosmos-institute.org/p/you-are-not-a-function).

when information is handed to us pre-synthesized on a silver platter, the line between which ideas are our own and which came from an external agent begins to blur. without giving ourselves the time to think critically about what we receive, we risk drowning out our own [voices](https://arxiv.org/pdf/2603.18161). to make matters worse, post-training pipelines have been found to incentivize agents to [steer user behavior](https://arxiv.org/pdf/2405.17713) toward states that are [easier to satisfy](https://arxiv.org/html/2504.03206v2#S1). indeed, [claude user trends](https://arxiv.org/pdf/2601.19062) show that disempowerment patterns in real-world llm usage are growing with time. to date, the human line project has to date documented almost 300 cases of [ai psychosis](https://arxiv.org/pdf/2602.19141).

interestingly, recent work on [self-distillation](https://arxiv.org/abs/2601.20802) has shown that llms learn better and [forget less](https://arxiv.org/abs/2601.19897) when they explain new concepts to themselves. rather than feeding the model content it cannot relate to (e.g., off-policy expert demonstrations), having the model explain the concept in its own words allows it to fold information into its pre-existing knowledge in a more sturdy way. just like machines, humans are also better able to accumulate new knowledge when they are acutely aware of the line that discriminates where their knowledge begins and ends.

so, while it is useful to spawn an agent to speed up work that we wouldn’t gain much from doing ourselves (e.g., writing plotting code), we should be more selective in choices to [outsource our thinking](https://x.com/yacineMTB/status/2018886083120153046) during processes of [knowledge creation](https://ergosphere.blog/posts/the-machines-are-fine/). while ai no doubt gives us incredible "boosts" of speed when used in the right place at the right time, operating at such high speeds also makes steering knowledge work [more difficult](https://www.ft.com/content/9c6a1daf-3c36-4035-bf74-1bedbc3e960d?syn-25a6b1a6=1). without the time to properly digest information at human speeds, it's easy to fall into the trap of spending weeks going down unproductive rabbit holes, circling around but missing the correct solution (or question).

amidst a culture of *fast ai*, it is worth leaning into the slow-thinking mind, the one that was wired to get caught up in simple problems over extended periods of time. indeed, at the current speed of ai progress, our capacity for slow thinking may turn out to be a defining superpower.

[^1]: at least to the extent one was required to over the course of a semester.
[^2]: a 20 minute session was often not enough to explore a single prompt fully.

---

*this post took shape through productive discussions with andre ye, mitchell gordon, marwa abdulhai, andy liu, omar khattab, smitha milli, sana pandey, deb roy, philippe laban, tamara broderick, and other wonderful folks at iclr 2026.*


</div>