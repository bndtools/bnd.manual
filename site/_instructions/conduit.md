---
layout: default
class: Project
title: -conduit 
summary: This project is a front to one or more JARs 
---

	public Jar build() throws Exception {
		trace("build");
		init();
		if (isTrue(getProperty(NOBUNDLES)))
			return null;

		if (getProperty(CONDUIT) != null)
			error("Specified " + CONDUIT + " but calls build() instead of builds() (might be a programmer error");



	/**
	 * Build Multiple jars. If the -sub command is set, we filter the file with
	 * the given patterns.
	 *
	 * @return
	 * @throws Exception
	 */
	public Jar[] builds() throws Exception {
		begin();

		// Are we acting as a conduit for another JAR?
		String conduit = getProperty(CONDUIT);
		if (conduit != null) {
			Parameters map = parseHeader(conduit);
			Jar[] result = new Jar[map.size()];
			int n = 0;
			for (String file : map.keySet()) {
				Jar c = new Jar(getFile(file));
				addClose(c);
				String name = map.get(file).get("name");
				if (name != null)
					c.setName(name);

				result[n++] = c;
			}
			return result;
		}

		List<Jar> result = new ArrayList<Jar>();
		List<Builder> builders;

		builders = getSubBuilders();

		for (Builder builder : builders) {
			try {
				Jar jar = builder.build();
				jar.setName(builder.getBsn());
				result.add(jar);
			}
			catch (Exception e) {
				e.printStackTrace();
				error("Sub Building " + builder.getBsn(), e);
			}
			if (builder != this)
				getInfo(builder, builder.getBsn() + ": ");
		}
		return result.toArray(new Jar[result.size()]);
	}
