___
title: 	p_allsourcepath
class: 	Project
summary: This reads the source and target settings from the IDE
----

	public String _p_allsourcepath(String args[]) throws Exception {
		return list(args, getAllsourcepath());
	}
