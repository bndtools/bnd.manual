---
class: Macro
title: uniq (';' LIST )*
summary: Concatenate the lists and then remove any duplicates.
---

	static String	_uniqHelp	= "${uniq;<list> ...}";

	public String _uniq(String args[]) {
		verifyCommand(args, _uniqHelp, null, 1, Integer.MAX_VALUE);
		Set<String> set = new LinkedHashSet<String>();
		for (int i = 1; i < args.length; i++) {
			Processor.split(args[i], set);
		}
		return Processor.join(set, ",");
	}
