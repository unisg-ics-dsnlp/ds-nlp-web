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

<iframe width="600" height="420" frameBorder="0" scrolling="no" marginHeight="0" marginWidth="0" src="https://use.mazemap.com/embed.html#v=1&amp;config=unisg-profile&amp;campusid=710&amp;zlevel=1&amp;center=9.371144,47.425149&amp;zoom=18.9&amp;sharepoitype=poi&amp;sharepoi=1000760525&amp;utm_medium=iframe" style="border:1px solid grey" allow="geolocation" title="Map by MazeMap"></iframe><br/><small><a href="https://www.mazemap.com/">Map by MazeMap</a></small>


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
