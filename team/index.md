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

Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor
incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis
nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.

{% include section.html %}

{% capture content %}

{% include figure.html image="images/team/1_StockTeamImage.png" %}
{% include figure.html image="images/team/2_StockTeamImage.png" %}
{% include figure.html image="images/team/3_StockTeamImage.png" %}

{% endcapture %}

{% include grid.html style="square" content=content %}
