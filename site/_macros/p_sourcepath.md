___
title: 	p_sourcepath
class: 	Project
summary: The path to the project's source directory.
related: src
----

	public String _p_sourcepath(String args[]) throws Exception {
		return list(args, getSourcePath());
	}

