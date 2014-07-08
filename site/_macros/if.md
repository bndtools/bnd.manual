---
layout: default
class: Macro
title: if ';' STRING ';' STRING ( ';' STRING )?
summary: Conditional macro that depending on a condition returns either a value for true or optionally for false.
---
layout: default


	static String	_ifHelp	= "${if;<condition>;<iftrue> [;<iffalse>] }";

	public String _if(String args[]) {
		verifyCommand(args, _ifHelp, null, 3, 4);
		String condition = args[1].trim();
		if (!condition.equalsIgnoreCase("false"))
			if (condition.length() != 0)
				return args[2];

		if (args.length > 3)
			return args[3];
		return "";
	}
