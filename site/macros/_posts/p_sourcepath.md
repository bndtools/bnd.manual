___
title: 	Source path
class: 	Project
syntax: ${p_sourcepath}
summary: This reads the source and target settings from the IDE
related: src
----

	public String _p_sourcepath(String args[]) throws Exception {
		return list(args, getSourcePath());
	}

