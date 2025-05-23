---
---

<!-- # DS-NLP Lab Blog and Research Website -->

The Data Science and Natural Language Processing (DS-NLP) Lab at the University of St. Gallen, led by Professor Siegfried Handschuh, pioneers research at the intersection of artificial intelligence, data science, and Natural Language Processing technologies. Our team focuses on advancing knowledge extraction, semantic representation, Argumentation and the societal impacts of AI-driven language systems. Through innovative projects and scholarly publications, we aim to bridge the gap between complex data and meaningful human understanding. 

{% include section.html background="images/background.jpg" dark=true %}

## Latest Blog Posts

{% assign recent_posts = site.posts | sort: 'date' | reverse | slice: 0, 3 %}
{% for post in recent_posts %}
  {% include post-excerpt.html post=post %}
{% endfor %}

{% include section.html %}


## Highlights

{% capture text %}

The DS NLP Lab has already published various papers and publications. An overview of all publications and a few highlights can be found in the publications tab.

{%
  include button.html
  link="research"
  text="See our publications"
  icon="fa-solid fa-arrow-right"
  flip=true
  style="bare"
%}

{% endcapture %}

{%
  include feature.html
  image="images/HSG_Daten_01.png"
  link="research"
  title="Our Research"
  text=text
%}

{% capture text %}

Explore the books authored by the DS-NLP Lab team, showcasing our contributions to the fields of Artificial Intelligence, Data Science, and their societal impacts. 

{%
  include button.html
  link="books"
  text="See our recent books"
  icon="fa-solid fa-arrow-right"
  flip=true
  style="bare"
%}

{% endcapture %}

{%
  include feature.html
  image="images/cites/book_generativeKünstlicheIntelligenz.jpg"
  link="projects"
  title="Our Projects"
  flip=true
  style="bare"
  text=text
%}

{% capture text %}

We are a steadily growing motivated team that aims to explore the world of data science and natural language processing.

{%
  include button.html
  link="team"
  text="Meet our team"
  icon="fa-solid fa-arrow-right"
  flip=true
  style="bare"
%}

{% endcapture %}

{%
  include feature.html
  image="images/HSG_Illustration_Lecure.png"
  link="team"
  title="Our Team"
  text=text
%}
