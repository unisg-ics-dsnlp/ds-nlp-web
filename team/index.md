---
title: Team
nav:
  order: 4
  tooltip: About our team
---

# {% include icon.html icon="fa-solid fa-users" %}Team


{% include list.html data="members" component="portrait" filter="role == 'professor'" %}
{% include list.html data="members" component="portrait" filter="role != 'professor'" %}

{% include section.html background="images/background.jpg" dark=true %}

<div style="text-align: center;">
  "The best way to predict the future is to invent it" – Alan Kay (1971)
</div>

{% include section.html %}

{% capture content %}

{% include figure.html image="images/team/1_StockTeamImage.png" %}
{% include figure.html image="images/team/2_StockTeamImage.png" %}
{% include figure.html image="images/team/3_StockTeamImage.png" %}

{% endcapture %}

{% include grid.html style="square" content=content %}
