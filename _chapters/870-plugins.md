---
title: Plugins Reference
layout: default
---

## Reference

<div>
<dl class="property-index">

{% for c in site.plugins %}<dt><a href="{{c.url}}">{{c.title}}</a></dt><dd>{{c.summary}}</dd>
{% endfor %}

</dl>
</div>

