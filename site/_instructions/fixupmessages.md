---
layout: default
class: Project
title: -fixupmessages MATCHER ( ';' directive )* 
summary: Fixup errors and warnings. 
---


	/**
	 * Move errors and warnings to their proper place by scanning the fixup
	 * messages property.
	 */
	private void fixupMessages() {
		if (fixupMessages)
			return;
		fixupMessages = true;
		Parameters fixup = new Parameters(getProperty(Constants.FIXUPMESSAGES));
		if (fixup.isEmpty())
			return;

		Instructions instrs = new Instructions(fixup);

		doFixup(instrs, errors, warnings, FIXUPMESSAGES_IS_ERROR);
		doFixup(instrs, warnings, errors, FIXUPMESSAGES_IS_WARNING);
	}

	private void doFixup(Instructions instrs, List<String> messages, List<String> other, String type) {
		for (int i = 0; i < messages.size(); i++) {
			String message = messages.get(i);
			Instruction matcher = instrs.finder(message);
			if (matcher == null || matcher.isNegated())
				continue;

			Attrs attrs = instrs.get(matcher);

			//
			// Default the pattern applies to the errors and warnings
			// but we can restrict it: e.g. restrict:=error
			//

			String restrict = attrs.get(FIXUPMESSAGES_RESTRICT_DIRECTIVE);
			if (restrict != null && !FIXUPMESSAGES_IS_ERROR.equals(restrict))
				continue;

			//
			// We can optionally replace the message with another text. E.g.
			// replace:"hello world". This can use macro expansion, the ${@}
			// macro is set to the old message.
			//
			String replace = attrs.get(FIXUPMESSAGES_REPLACE_DIRECTIVE);
			if (replace != null) {
				trace("replacing %s with %s", message, replace);
				setProperty("@", message);
				message = getReplacer().process(replace);
				messages.set(i, message);
				unsetProperty("@");
			}

			//
			//
			String is = attrs.get(FIXUPMESSAGES_IS_DIRECTIVE);

			if (attrs.isEmpty() || FIXUPMESSAGES_IS_IGNORE.equals(is)) {
				messages.remove(i--);
			} else {
				if (is != null && !type.equals(is)) {
					messages.remove(i--);
					other.add(message);
				}
			}
		}
	}
