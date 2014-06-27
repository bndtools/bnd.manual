---
class: Macro
title: range ';' MASK ( ';' VERSION )
summary: Create a semantic version range out of a version using a mask to control the bump
---
 

	/**
	 * Schortcut for version policy
	 * 
	 * <pre>
	 * -provide-policy : ${policy;[==,=+)}
	 * -consume-policy : ${policy;[==,+)}
	 * </pre>
	 * 
	 * @param args
	 * @return
	 */

	static Pattern	RANGE_MASK		= Pattern.compile("(\\[|\\()(" + MASK_STRING + "),(" + MASK_STRING + ")(\\]|\\))");
	static String	_rangeHelp		= "${range;<mask>[;<version>]}, range for version, if version not specified lookyp ${@}\n"
											+ "<mask> ::= [ M [ M [ M [ MQ ]]]\n"
											+ "M ::= '+' | '-' | MQ\n"
											+ "MQ ::= '~' | '='";
	static Pattern	_rangePattern[]	= new Pattern[] {
			null, RANGE_MASK
									};

	public String _range(String args[]) {
		verifyCommand(args, _rangeHelp, _rangePattern, 2, 3);
		Version version = null;
		if (args.length >= 3)
			version = new Version(args[2]);
		else {
			String v = domain.getProperty("@");
			if (v == null)
				return null;
			version = new Version(v);
		}
		String spec = args[1];

		Matcher m = RANGE_MASK.matcher(spec);
		m.matches();
		String floor = m.group(1);
		String floorMask = m.group(2);
		String ceilingMask = m.group(3);
		String ceiling = m.group(4);

		String left = version(version, floorMask);
		String right = version(version, ceilingMask);
		StringBuilder sb = new StringBuilder();
		sb.append(floor);
		sb.append(left);
		sb.append(",");
		sb.append(right);
		sb.append(ceiling);

		String s = sb.toString();
		VersionRange vr = new VersionRange(s);
		if (!(vr.includes(vr.getHigh()) || vr.includes(vr.getLow()))) {
			domain.error("${range} macro created an invalid range %s from %s and mask %s", s, version, spec);
		}
		return sb.toString();
	}

