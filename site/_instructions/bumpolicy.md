---
layout: default
class: Macro
title: -bumppolicy 
summary: A mask to bump a version.
---

	public void bump() throws Exception {
		bump(getProperty(BUMPPOLICY, "=+0"));
	}
