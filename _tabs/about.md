---
# the default layout is 'page'
icon: fas fa-info-circle
order: 4
---

{% capture readme_content %}
{{ site.github.repository.readme }}
{% endcapture %}

{{ readme_content | markdownify }}