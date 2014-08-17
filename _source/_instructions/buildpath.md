---
layout: default
class: Macro
title: -buildpath REPO-SPEC ( ',' REPO-SPEC ) *
summary: Provides the class path for building the jar. The entries are references to the repository.
---

The -buildpath instruction is the main mechanism to add build-time dependencies to a project. A dependency is either another project in the workspace, or a bundle in a repository. The -buildpath is only used compile time; it is not used to run projects. Import-Package header is also not directly influenced by the -buildpath. Instead, the Import-Package header is generated based on which packages are actually imported from the code. 

Because -buildpath dependencies are only used compile time it's sufficient to add bundles containing only APIs; you don't need bundles containing (service) implementations.

An example of the -buildpath could be the following, where three dependencies are defined: 

```
-buildpath: some.other.workspace.project,\
   osgi.core,\
   osgi.cmpn
```



					// We have two paths that consists of repo files, projects,
					// or some other stuff. The doPath routine adds them to the
					// path and extracts the projects so we can build them
					// before.

					doPath(buildpath, dependencies, parseBuildpath(), bootclasspath, false, BUILDPATH);

	/**
	 * Parse the list of bundles that are a prerequisite to this project.
	 * Bundles are listed in repo specific names. So we just let our repo
	 * plugins iterate over the list of bundles and we get the highest version
	 * from them.
	 *
	 * @return
	 */

	private List<Container> parseBuildpath() throws Exception {
		List<Container> bundles = getBundles(Strategy.LOWEST, mergeProperties(Constants.BUILDPATH), Constants.BUILDPATH);
		return bundles;
	}

	
		/**
	 * Method to verify that the paths are correct, ie no missing dependencies
	 *
	 * @param test
	 *            for test cases, also adds -testpath
	 * @throws Exception
	 */
	public void verifyDependencies(boolean test) throws Exception {
		verifyDependencies(RUNBUNDLES, getRunbundles());
		verifyDependencies(RUNPATH, getRunpath());
		if (test)
			verifyDependencies(TESTPATH, getTestpath());
		verifyDependencies(BUILDPATH, getBuildpath());
	}

	
