---
layout: default
class: Project
title: -include PATH-SPEC ( ',' PATH-SPEC ) * 
summary:  Include a number of files from the file system
---

	/**
	 * Inspect the properties and if you find -includes parse the line included
	 * manifest files or properties files. The files are relative from the given
	 * base, this is normally the base for the analyzer.
	 * 
	 * @param ubase
	 * @param p
	 * @param done
	 * @throws IOException
	 * @throws IOException
	 */

	private void doIncludes(File ubase, Properties p) {
		String includes = p.getProperty(INCLUDE);
		if (includes != null) {
			includes = getReplacer().process(includes);
			p.remove(INCLUDE);
			Collection<String> clauses = new Parameters(includes).keySet();

			for (String value : clauses) {
				boolean fileMustExist = true;
				boolean overwrite = true;
				while (true) {
					if (value.startsWith("-")) {
						fileMustExist = false;
						value = value.substring(1).trim();
					} else if (value.startsWith("~")) {
						// Overwrite properties!
						overwrite = false;
						value = value.substring(1).trim();
					} else
						break;
				}
				try {
					File file = getFile(ubase, value).getAbsoluteFile();
					if (!file.isFile() && fileMustExist) {
						error("Included file " + file + (file.exists() ? " does not exist" : " is directory"));
					} else
						doIncludeFile(file, overwrite, p);
				}
				catch (Exception e) {
					if (fileMustExist)
						error("Error in processing included file: " + value, e);
				}
			}
		}
	}

