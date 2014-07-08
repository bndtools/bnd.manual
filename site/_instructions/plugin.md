---
layout: default
class: Processor
title: -plugin* PARAMETERS 
summary: Load plugins and their parameters. 
---

	/**
	 * Return a list of plugins. Plugins are defined with the -plugin command.
	 * They are class names, optionally associated with attributes. Plugins can
	 * implement the Plugin interface to see these attributes. Any object can be
	 * a plugin.
	 * 
	 * @return
	 */
	public Set<Object> getPlugins() {
		synchronized (this) {
			if (this.plugins != null)
				return this.plugins;

			plugins = new LinkedHashSet<Object>();
			missingCommand = new HashSet<String>();
		}
		// The owner of the plugin is always in there.
		plugins.add(this);
		setTypeSpecificPlugins(plugins);

		if (parent != null)
			plugins.addAll(parent.getPlugins());

		// We only use plugins now when they are defined on our level
		// and not if it is in our parent. We inherit from our parent
		// through the previous block.

		if (properties.containsKey(PLUGIN)) {
			String spe = getProperty(PLUGIN);
			if (spe.equals(NONE))
				return new LinkedHashSet<Object>();

			spe = mergeProperties(PLUGIN);
			String pluginPath = mergeProperties(PLUGINPATH);
			loadPlugins(plugins, spe, pluginPath);
		}

		addExtensions(plugins);

		for (RegistryDonePlugin rdp : getPlugins(RegistryDonePlugin.class)) {
			try {
				rdp.done();
			}
			catch (Exception e) {
				error("Calling done on %s, gives an exception %s", rdp, e);
			}
		}
		return this.plugins;
	}


	/**
	 * Magic to load the plugins. This is quite tricky actually since we allow
	 * plugins to be downloaded (this is mainly intended for repositories since
	 * in general plugins should use extensions, however to bootstrap the
	 * extensions we need more). Since downloads might need plugins for
	 * passwords and protocols we need to first load the paths specified on the
	 * plugin clause, then check if there are any local plugins (starting with
	 * aQute.bnd and be able to load from our own class loader).
	 * <p>
	 * After that, we load the plugin paths, these can use the built in
	 * connectors.
	 * <p>
	 * Last but not least, we load the remaining plugins.
	 * 
	 * @param instances
	 * @param pluginString
	 */
	protected void loadPlugins(Set<Object> instances, String pluginString, String pluginPathString) {
		Parameters plugins = new Parameters(pluginString);
		CL loader = getLoader();

		// First add the plugin-specific paths from their path: directives
		for (Entry<String,Attrs> entry : plugins.entrySet()) {
			String key = removeDuplicateMarker(entry.getKey());
			String path = entry.getValue().get(PATH_DIRECTIVE);
			if (path != null) {
				String parts[] = path.split("\\s*,\\s*");
				try {
					for (String p : parts) {
						File f = getFile(p).getAbsoluteFile();
						loader.add(f.toURI().toURL());
					}
				}
				catch (Exception e) {
					error("Problem adding path %s to loader for plugin %s. Exception: (%s)", path, key, e);
				}
			}
		}

		//
		// Try to load any plugins that are local
		// these must start with aQute.bnd.* and
		// and be possible to load. The main intention
		// of this code is to load the URL connectors so that
		// any access to remote plugins can use the connector
		// model.
		//

		Set<String> loaded = new HashSet<String>();
		for (Entry<String,Attrs> entry : plugins.entrySet()) {
			String className = removeDuplicateMarker(entry.getKey());
			Attrs attrs = entry.getValue();

			trace("Trying pre-plugin %s", className);

			Object plugin = loadPlugin(getClass().getClassLoader(), attrs, className, true);
			if (plugin != null) {
				// with the marker!!
				loaded.add(entry.getKey());
				instances.add(plugin);
			}
		}

		//
		// Make sure we load each plugin only once
		// by removing the entries that were successfully loaded
		//
		plugins.keySet().removeAll(loaded);

		loadPluginPath(instances, pluginPathString, loader);

		//
		// Load the remaining plugins
		//

		for (Entry<String,Attrs> entry : plugins.entrySet()) {
			String className = removeDuplicateMarker(entry.getKey());
			Attrs attrs = entry.getValue();

			trace("Loading secondary plugin %s", className);

			// We can defer the error if the plugin specifies
			// a command name. In that case, we'll verify that
			// a bnd file does not contain any references to a
			// plugin
			// command. The reason this feature was added was
			// to compile plugin classes with the same build.
			String commands = attrs.get(COMMAND_DIRECTIVE);

			Object plugin = loadPlugin(loader, attrs, className, commands != null);
			if (plugin != null)
				instances.add(plugin);
			else {
				if (commands == null)
					error("Cannot load the plugin %s", className);
				else {
					Collection<String> cs = split(commands);
					missingCommand.addAll(cs);
				}
			}
		}
	}
