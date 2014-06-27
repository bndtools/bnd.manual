___
title: 	p_testpath
class: 	Project
summary: The path of JARs placed on the remote VM's classpath for testing
----

	public String _p_testpath(String args[]) throws Exception {
		return list(args, getRunpath());
	}

