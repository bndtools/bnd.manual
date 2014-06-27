---
class: Macro
title: isdir ( ';' FILE )+
summary: True if all given files are directories
---

	public String _isdir(String args[]) {
		if (args.length < 2) {
			domain.warning("Need at least one file name for ${isdir;...}");
			return null;
		}
		boolean isdir = true;
		for (int i = 1; i < args.length; i++) {
			File f = new File(args[i]).getAbsoluteFile();
			isdir &= f.isDirectory();
		}
		return isdir ? "true" : "false";

	}
