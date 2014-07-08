---
layout: default
class: Macro
title: subst STRING ';' REGEX ';' STRING (';' NUMBER )?
summary: Substitute all the regex matches in the target for the given value; if a count is specified, limit the number of replacements to that count.
---

	static String	_subst	= "${subst;<target>;<regex>;<replace>[;count]}";

	public StringBuffer _subst(String args[]) throws Exception {
		verifyCommand(args, _subst, null, 4, 5);

		Pattern p = Pattern.compile(args[2]);
		Matcher matcher = p.matcher(args[1]);
		String replace = args[3];
		int count = Integer.MAX_VALUE;

		if (args.length > 4) {
			count = Integer.parseInt(args[4]);
		}

		StringBuffer sb = new StringBuffer();

		for (int i = 0; i < count; i++) {
			if (matcher.find()) {
				matcher.appendReplacement(sb, replace);
			} else
				break;
		}
		matcher.appendTail(sb);
		return sb;
	}
