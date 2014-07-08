---
layout: default
class: Project
title: -consumerpolicy VERSION-MASK
summary: Specify the default version bump policy for a consumer when a binary incompatible change is detected. 
---

	public String getVersionPolicy(boolean implemented) {
		if (implemented) {
			return getProperty(PROVIDER_POLICY, DEFAULT_PROVIDER_POLICY);
		}

		return getProperty(CONSUMER_POLICY, DEFAULT_CONSUMER_POLICY);
	}
	
	final static String	DEFAULT_CONSUMER_POLICY	= "${range;[==,+)}";
	