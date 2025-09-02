---
title: Team
nav:
  order: 3
  tooltip: Meet our lab!
---

# {% include icon.html icon="fa-solid fa-users" %}Team

Come meet the current members of our lab!

{% include section.html %}

{% include list.html data="members" component="portrait" filter="role == 'principal-investigator'" %}
{% include list.html data="members" component="portrait" filter="role == 'postdoc'" %}
{% include list.html data="members" component="portrait" filter="role == 'phd'" %}
{% include list.html data="members" component="portrait" filter="role == 'undergrad'" %}
{% include list.html data="members" component="portrait" filter="role == 'lab-manager'" %}
{% include list.html data="members" component="portrait" filter="role != 'principal-investigator' and role != 'postdoc' and role != 'phd' and role != 'undergrad' and role != 'lab-manager' and role != 'alumni'" %}

{% include section.html %}

<hr>

# Alumni

These are the past members of our lab, who have contributed so much to our research and community. We wish them all the best in their future endeavors.

{% include list.html data="members" component="portrait" filter="role == 'alumni'" %}


