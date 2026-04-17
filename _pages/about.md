---
permalink: /
title: ""
author_profile: true
redirect_from: 
  - /about/
  - /about.html
---

I am a third year PhD student at MIT EECS, advised by Professor Tamara Broderick. My background is in statistics and probability, with an interest in the hidden assumptions and fragilities that shape what we present as [data analysis conclusions](https://openreview.net/pdf?id=m6EQ6YdPXV). 

My recent work has focused on surfacing false certainty in AI: whether in [AI evaluation](https://openreview.net/pdf?id=jNiEMDsRgc), in fluent single answers to open-ended questions, or in the invisible sediment of [long conversation histories](https://arxiv.org/abs/2602.24287). Ultimately, I am interested in designing systems that encourage users to *slow down* and *think critically* when engaging with AI.

I am grateful to have my work supported by the Amazon AI Research Innovation Fellowship, and (previously) the MIT Presidential and Quad Fellowships.

---

<section id="selected-publications" markdown="1">

## Selected Publications

{% if site.author.googlescholar %}
  <div class="wordwrap">You can find a full list of my articles on <a href="{{site.author.googlescholar}}">my Google Scholar profile</a>.</div>
{% endif %}

{% include base_path %}

{% if site.publication_category %}
  {% for category in site.publication_category %}
    {% assign title_shown = false %}
    {% for post in site.publications reversed %}
      {% if post.category != category[0] %}{% continue %}{% endif %}
      {% unless title_shown %}
        <h3>{{ category[1].title }}</h3><hr />
        {% assign title_shown = true %}
      {% endunless %}
      {% include archive-single.html %}
    {% endfor %}
  {% endfor %}
{% else %}
  {% for post in site.publications reversed %}
    {% include archive-single.html %}
  {% endfor %}
{% endif %}

</section>

---

<section id="news" markdown="1">

## News

</section>

---

<section id="thoughts" markdown="1">

## Thoughts


</section>
