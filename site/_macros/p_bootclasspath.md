___
title: 	p_bootclasspath
class: 	Project
summary: The project's boot class path
----

	public String _p_bootclasspath(String args[]) throws Exception {
		return list(args, getBootclasspath());
	}

