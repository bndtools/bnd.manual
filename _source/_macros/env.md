---
layout: default
class: Macro
title: env ';' KEY
summary: The given environment variable
---
layout: default

	public String _env(String args[]) {
		verifyCommand(args, "${env;<name>}, get the environmet variable", null, 2, 2);

		try {
			return System.getenv(args[1]);
		}
		catch (Throwable t) {
			return null;
		}
	}
