---
class: Analyzer
title: classes TODO
summary: A list of class names filtered by a query language
---
 

	static String	_classesHelp	= "${classes;'implementing'|'extending'|'importing'|'named'|'version'|'any';<pattern>}, Return a list of class fully qualified class names that extend/implement/import any of the contained classes matching the pattern\n";

	public String _classes(String... args) throws Exception {
		// Macro.verifyCommand(args, _classesHelp, new
		// Pattern[]{null,Pattern.compile("(implementing|implements|extending|extends|importing|imports|any)"),
		// null}, 3,3);

		Collection<Clazz> matched = getClasses(args);
		if (matched.isEmpty())
			return "";

		return join(matched);
	}
