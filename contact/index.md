---
title: Contact
nav:
  order: 6
  tooltip: Email, address, and location
---

# {% include icon.html icon="fa-regular fa-envelope" %}Contact

We welcome inquiries from students, researchers, and industry partners interested in Natural Language Processing and Data Science. Whether you’re looking to collaborate, explore academic opportunities, or learn more about our work, feel free to reach out.

{%
  include button.html
  type="email"
  text="E-Mail"
  link="siegfried.handschuh@unisg.ch"
%}
<!-- {%
  include button.html
  type="phone"
  text="(555) 867-5309"
  link="+1-555-867-5309"
%} -->
{%
  include button.html
  type="address"
  tooltip="Our location on Google Maps for easy navigation"
  link="https://maps.app.goo.gl/xtVrtMF7oMNENwac7"
%}

{% include section.html %}


{%
  include figure.html
  image="images/HSG_Bürosituation_05_RGB.png"
  caption="contact template"
%}


{% include cols.html col1=col1 col2=col2 %}

{% include section.html dark=true %}

{% capture col1 %}
ICS-HSG

Institute of Computer Science

University of St.Gallen

Rosenbergstrasse 30

CH-9000 St.Gallen

{% endcapture %}

{% capture col2 %}
This website is subject to the [HSG](https://www.unisg.ch/) wide policies. See also the pages [Data Protection Declaration](https://www.unisg.ch/en/data-protection-declaration/) and [Copyright & Disclaimer](https://www.unisg.ch/en/copyright-disclaimer/).

{% endcapture %}

{% capture col3 %}
This website was build with the [greenelab jekyll template](https://github.com/greenelab/lab-website-template).

{% endcapture %}

{% include cols.html col1=col1 col2=col2 col3=col3 %}
