---
title: Plugins
layout: default
---
## Plugins
Plugins are objects that can extend the functionality of bnd. They are called from inside bnd when a certain action should take place. For example, bnd uses a repository and plugins provide the actual repository implementations. Or for example, the SpringComponent analyzes the Spring files and adds references found in that XML to the imports.

A plugin is defined as:

	PLUGIN ::= FQN ( ';' \<directive\|attribute\> )*

The following directive is defined for all plugin:

||`path:` ||A path to the jar file that contains the plugin. The directory/jar at that location is placed on your classpath for that plugin.||

bnd current supports the following plugin types:

### aQute.bnd.service.AnalyzerPlugin

	public interface AnalyzerPlugin {
		/**
		 * This plugin is called after analysis. The plugin 
		 * is free to modify the jar and/or change the classpath 
		 * information (see referred, contained).
		 * This plugin is called after analysis of the JAR 
		 * but before manifest generation.
		 * 
		 * @param analyzer
		 * @return true if the classpace has been modified so that the bundle
		 *         classpath must be reanalyzed
		 * @throws Exception
		 */
		boolean analyzeJar(Analyzer analyzer) throws Exception;
	}

### aQute.bnd.service.SignerPlugin

	public interface SignerPlugin {
		/**
		 * Sign the current jar. The alias is the given certificate 
		 * keystore.
		 * 
		 * @param builder   The current builder that contains the 
							jar to sign
		 * @param alias     The keystore certificate alias
		 * @throws Exception When anything goes wrong
		 */
		void sign(Builder builder, String alias) throws Exception;
	}

### aQute.bnd.service.RepositoryPlugin

	public interface RepositoryPlugin {
		/**
		 * Return a URL to a matching version of the given bundle.
		 * 
		 * @param bsn
		 *            Bundle-SymbolicName of the searched bundle
		 * @param range
		 *            Version range for this bundle,"latest" 
		 *            if you only want the
		 *            latest, or null when you want all.
		 * @return    A list of URLs sorted on version, lowest version 
		 *            is at index 0.
		 *            null is returned when no files with the given 
		 *            bsn ould be found.
		 * @throws Exception
		 *             when anything goes wrong
		 */
		File[] get(String bsn, String range) throws Exception;
	
		/**
		 * Answer if this repository can be used to store files.
		 * 
		 * @return true if writable
		 */
		boolean canWrite();
	
		/**
		 * Put a JAR file in the repository.
		 * 
		 * @param jar
		 * @throws Exception
		 */
		File  put(Jar jar) throws Exception;
	
		/**
		 * Return a list of bsns that are present in the repository.
		 * 
		 * @param  regex if not null, match against the bsn and if 
		 *         matches, return otherwise skip
		 * @return A list of bsns that match the regex parameter 
		 *         or all if regex is null
		 */
		List<String> list(String regex);
	
		/**
		 * Return a list of versions.
		 */
		List<Version> versions(String bsn);
	}

### aQute.bnd.service.verifier.VerifierPlugin

	public interface VerifierPlugin {
		/**
		 * This plugin is called after the manifest generation and after the jar is
		 * fully populated with resources. The plugin is intended to verify details
		 * of the jar and manifest.
		 *
		 * @param analyzer
		 * @throws Exception
		 */
		void verify(Analyzer analyzer) throws Exception;
	}

### aQute.bnd.service.Plugin

	/**
	 * An optional interface for plugins. If a plugin implements 
	 * this interface then it can receive the remaining attributes 
	 * and directives given in its clause as
	 * well as the reporter to use.
	 */
	public interface Plugin {
		/**
		 * Give the plugin the remaining properties.
		 * When a plugin is declared, the clause can contain 
		 * extra properties. All the properties and directives 
		 * are given to the plugin to use.
		 * @param map attributes and directives for this plugin's clause
		 */
		void setProperties(Map<String,String> map);
	
		/**
		 * Set the current reporter. This is called at init time. 
		 * This plugin should report all errors and warnings 
		 * to this reporter.
		 * @param processor
		 */
		void setReporter(Reporter processor);
	}
