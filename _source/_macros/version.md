---
layout: default
class: Macro
title: version ';' MASK (';' VERSION )
summary: Modify a version to set a version policy. The policy is a mask that is mapped to a version.
---
layout: default


	/**
	 * Modify a version to set a version policy. The policy is a mask that is
	 * mapped to a version.
	 * 
	 * <pre>
	 * +           increment
	 * -           decrement
	 * =           maintain
	 * s			only pos=3 (qualifier). If qualifer == SNAPSHOT, return m.m.m-SNAPSHOT else m.m.m.q
	 * s			only pos=3 (qualifier). If qualifer == SNAPSHOT, return m.m.m-SNAPSHOT else m.m.m
	 * &tilde;           discard
	 * 
	 * ==+      = maintain major, minor, increment micro, discard qualifier
	 * &tilde;&tilde;&tilde;=     = just get the qualifier
	 * version=&quot;[${version;==;${@}},${version;=+;${@}})&quot;
	 * </pre>
	 * 
	 * @param args
	 * @return
	 */
	final static String		MASK_STRING			= "[\\-+=~0123456789]{0,3}[=~]?";
	final static Pattern	MASK				= Pattern.compile(MASK_STRING);
	final static String		_versionHelp		= "${version;<mask>;<version>}, modify a version\n"
														+ "<mask> ::= [ M [ M [ M [ MQ ]]]\n"
														+ "M ::= '+' | '-' | MQ\n" + "MQ ::= '~' | '='";
	final static Pattern	_versionPattern[]	= new Pattern[] {
			null, null, MASK, Verifier.VERSION
												};

	public String _version(String args[]) {
		verifyCommand(args, _versionHelp, null, 2, 3);

		String mask = args[1];

		Version version = null;
		if (args.length >= 3)
			version = new Version(args[2]);

		return version(version, mask);
	}

	String version(Version version, String mask) {
		if (version == null) {
			String v = domain.getProperty("@");
			if (v == null) {
				domain.error(
						"No version specified for ${version} or ${range} and no implicit version ${@} either, mask=%s",
						mask);
				v = "0";
			}
			version = new Version(v);
		}

		StringBuilder sb = new StringBuilder();
		String del = "";

		for (int i = 0; i < mask.length(); i++) {
			char c = mask.charAt(i);
			String result = null;
			if (c != '~') {
				if (i == 3) {
					result = version.getQualifier();
					if (c == 'S') {
						// we have a request for a Maven snapshot
						if ("SNAPSHOT".equals(result))
							return sb.toString() + "-SNAPSHOT";
					} else if (c == 's') {
						// we have a request for a Maven snapshot
						if ("SNAPSHOT".equals(result))
							return sb.toString() + "-SNAPSHOT";
						else
							return sb.toString();
					}
				} else if (Character.isDigit(c)) {
					// Handle masks like +00, =+0
					result = String.valueOf(c);
				} else {
					int x = version.get(i);
					switch (c) {
						case '+' :
							x++;
							break;
						case '-' :
							x--;
							break;
						case '=' :
							break;
					}
					result = Integer.toString(x);
				}
				if (result != null) {
					sb.append(del);
					del = ".";
					sb.append(result);
				}
			}
		}
		return sb.toString();
	}

