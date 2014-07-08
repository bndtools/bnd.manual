---
layout: default
class: Project
title: -runbundles* REPO-ENTRY ( ',' REPO-ENTRY )* 
summary:  Add additional bundles, specified with their bsn and version like in -buildpath, that are installed and started before the project is run.
---

	public Collection<Container> getRunbundles() throws Exception {
		prepare();
		justInTime(runbundles, parseRunbundles(), true, RUNBUNDLES);
		return runbundles;
	}

	
		private List<Container> parseRunbundles() throws Exception {
		
		return getBundles(Strategy.HIGHEST, mergeProperties(Constants.RUNBUNDLES), Constants.RUNBUNDLES);
	}
	
	
	
	
						doPath(buildpath, dependencies, parseBuildpath(), bootclasspath, false, BUILDPATH);
					doPath(testpath, dependencies, parseTestpath(), bootclasspath, false, TESTPATH);
					if (!delayRunDependencies) {
						doPath(runfw, dependencies, parseRunFw(), null, false, RUNFW);
						doPath(runpath, dependencies, parseRunpath(), null, false, RUNPATH);
						doPath(runbundles, dependencies, parseRunbundles(), null, true, RUNBUNDLES);
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

	