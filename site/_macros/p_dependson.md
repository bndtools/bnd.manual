___
title: 	p_dependson
class: 	Project
summary: Provides a list of project names this project depends on 
----

	public String _p_dependson(String args[]) throws Exception {
		return list(args, toFiles(getDependson()));
	}

