___
title: 	List the all source paths
class: 	Project
syntax: ${p_allsourcepath}
summary: This reads the source and target settings from the IDE
----

	public String _p_allsourcepath(String args[]) throws Exception {
		return list(args, getAllsourcepath());
	}
