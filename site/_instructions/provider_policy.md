---
layout: default
class: Project
title: -providerpolicy VERSION-MASK
summary: Specify the default version bump policy for a provider when a binary incompatible change is detected. 
---

	final static String	DEFAULT_PROVIDER_POLICY	= "${range;[==,=+)}";
	public String getVersionPolicy(boolean implemented) {
		if (implemented) {
			return getProperty(PROVIDER_POLICY, DEFAULT_PROVIDER_POLICY);
		}

		return getProperty(CONSUMER_POLICY, DEFAULT_CONSUMER_POLICY);
	}
	
	