___
title: 	List the boot class path (ee files)
class: 	Project
syntax: ${p_bootclasspath}
summary: The boot class path
----

	public String _p_bootclasspath(String args[]) throws Exception {
		return list(args, getBootclasspath());
	}

