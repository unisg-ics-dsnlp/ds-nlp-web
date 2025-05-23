---
---

# unisg-ics-dsnlp's Website

An engaging 1-3 sentence description of your lab.

{% include section.html %}

## Highlights

{% capture text %}

Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.

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
  image="images/photo.jpg"
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
