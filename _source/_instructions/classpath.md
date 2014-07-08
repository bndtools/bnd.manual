---
layout: default
class: Analyzer
title: -classpath  FILE (',' FILE) *
summary: Specify additional file based entries (either directories or JAR files) to add to the used classpath. The classpath is normally set through the -buildpath but in certain cases, for example wrapping JARs, that run outside a project it is easier to set the classpath directly.
---

	boolean	firstUse	= true;

	public List<Jar> getClasspath() {
		if (firstUse) {
			firstUse = false;
			String cp = getProperty(CLASSPATH);
			if (cp != null)
				for (String s : split(cp)) {
					Jar jar = getJarFromName(s, "getting classpath");
					if (jar != null)
						addClasspath(jar);
					else
						warning("Cannot find entry on -classpath: %s", s);
				}
		}
		return classpath;
	}