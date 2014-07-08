---
layout: default
class: Project
title: -snapshot STRING 
summary: String to replace in version when it ends in SNAPSHOT. Intended for maven builds. Default is to leave the word as is, but it can be used to replace this for example with a timestamp (${tstamp}).
---



	@Override
	public void analyze() throws Exception {
		super.analyze();
		cleanupVersion(getImports(), null);
		cleanupVersion(getExports(), getVersion());
		String version = getProperty(BUNDLE_VERSION);
		if (version != null) {
			version = cleanupVersion(version);
			if (version.endsWith(".SNAPSHOT")) {
				version = version.replaceAll("SNAPSHOT$", getProperty(SNAPSHOT, "SNAPSHOT"));
			}
			setProperty(BUNDLE_VERSION, version);
		}
	}

