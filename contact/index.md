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

<div style="position: relative; width: 100%; padding-bottom: 70%; height: 0; overflow: hidden; border: 1px solid grey;">
  <iframe
    src="https://use.mazemap.com/embed.html#v=1&amp;config=unisg-profile&amp;campusid=710&amp;zlevel=7&amp;center=9.370851,47.425086&amp;zoom=16&amp;sharepoitype=poi&amp;sharepoi=1000760525&amp;utm_medium=iframe"
    style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; border: 0;"
    frameborder="0"
    allow="geolocation"
    scrolling="no">
  </iframe>
</div>

{% include cols.html col1=col1 col2=col2 %}

{% include section.html dark=true %}

{% capture col1 %}
ICS-HSG<br>
Institute of Computer Science<br>
University of St.Gallen<br>
Rosenbergstrasse 30<br>
CH-9000 St.Gallen<br>
{% endcapture %}

{% capture col2 %}
This website is subject to the [HSG](https://www.unisg.ch/) wide policies. See also the pages [Data Protection Declaration](https://www.unisg.ch/en/data-protection-declaration/) and [Copyright & Disclaimer](https://www.unisg.ch/en/copyright-disclaimer/).

{% endcapture %}

{% capture col3 %}
This website was build with the [greenelab jekyll template](https://github.com/greenelab/lab-website-template).

{% endcapture %}

{% include cols.html col1=col1 col2=col2 col3=col3 %}
