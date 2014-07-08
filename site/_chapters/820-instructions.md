---
title: Instruction Reference
layout: default
---

## Reference

{% for instruction in site.instructions %}| <code><a href="{{instruction.url}}">{{instruction.title}}</a></code> | {{instruction.summary}}|
{% endfor %}

