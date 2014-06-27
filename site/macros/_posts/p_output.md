___
title: 	Output/Target
class: 	Project
syntax: ${p_output}
summary: 
----

	public String _p_output(String args[]) throws Exception {
		if (args.length != 1)
			throw new IllegalArgumentException("${output} should not have arguments");
		return getOutput().getAbsolutePath();
	}
