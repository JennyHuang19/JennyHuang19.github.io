---
permalink: /
title: ""
author_profile: true
redirect_from: 
  - /about/
  - /about.html
---

I am a third year PhD student at MIT, working in the department of Electrical Engineering and Computer Science with Professor Tamara Broderick.

My background is in statistics and probability, with an interest in the hidden assumptions and fragilities that shape what we present as [data analysis conclusions](https://openreview.net/pdf?id=m6EQ6YdPXV). My recent work has focused on surfacing false certainty in AI: whether in [AI evaluation](https://openreview.net/pdf?id=jNiEMDsRgc), in fluent single answers to open-ended questions, or in [the invisible sediment of long conversation histories](https://arxiv.org/abs/2602.24287). Ultimately, I am interested in designing systems that encourage users to *slow down* and *engage critically* with AI.

I am grateful to have my work supported by the Amazon AI Research Innovation Fellowship, and, previously, the MIT Presidential and Quad Fellowships.

---

<section id="publications">

## Publications

{% if site.author.googlescholar %}
  <div class="wordwrap">You can also find my articles on <a href="{{site.author.googlescholar}}">my Google Scholar profile</a>.</div>
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

<section id="talks">

## Talks and Presentations

{% for post in site.talks reversed %}
  {% include archive-single-talk.html %}
{% endfor %}

</section>

---

<section id="teaching">

## Teaching

{% include base_path %}

{% for post in site.teaching reversed %}
  {% include archive-single.html %}
{% endfor %}

</section>

---

<section id="portfolio">

## Portfolio

{% include base_path %}

{% for post in site.portfolio %}
  {% include archive-single.html %}
{% endfor %}

</section>

---

<section id="blog-posts">

## Blog Posts

{% include base_path %}

{% for post in site.posts reversed %}
  {% include archive-single.html %}
{% endfor %}

</section>

---

<section id="cv">

## CV

Education
------
* Ph.D in Version Control Theory, GitHub University, 2018 (expected)
* M.S. in Jekyll, GitHub University, 2014
* B.S. in GitHub, GitHub University, 2012

Work Experience
------
* Spring 2024: Academic Pages Collaborator
  * GitHub University
  * Duties includes: Updates and improvements to template
  * Supervisor: The Users

* Fall 2015: Research Assistant
  * GitHub University
  * Duties included: Merging pull requests
  * Supervisor: Professor Hub

* Summer 2015: Research Assistant
  * GitHub University
  * Duties included: Tagging issues
  * Supervisor: Professor Git

Skills
------
* Skill 1
* Skill 2
  * Sub-skill 2.1
  * Sub-skill 2.2
* Skill 3

Publications
------
  <ul>{% for post in site.publications reversed %}
    {% include archive-single-cv.html %}
  {% endfor %}</ul>

Talks
------
  <ul>{% for post in site.talks reversed %}
    {% include archive-single-talk-cv.html %}
  {% endfor %}</ul>

Teaching
------
  <ul>{% for post in site.teaching reversed %}
    {% include archive-single-cv.html %}
  {% endfor %}</ul>

</section>
