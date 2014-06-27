___
title: 	
class: 	Project
syntax: ${p_dependson}
summary: 
----

	public String _p_dependson(String args[]) throws Exception {
		return list(args, toFiles(getDependson()));
	}

