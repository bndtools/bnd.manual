___
title: 	p_buildpath
class: 	Project
summary: The project's buildpath
----

	public String _p_buildpath(String args[]) throws Exception {
		return list(args, getBuildpath());
	}
