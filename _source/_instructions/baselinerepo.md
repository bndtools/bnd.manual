---
layout: default
class: Macro
title: -baselinerep qname
summary: Define the repository to calculate baselining against
---

	private RepositoryPlugin getBaselineRepo() {
		String repoName = getProperty(Constants.BASELINEREPO);
		if (repoName == null)
			return getReleaseRepo();

		List<RepositoryPlugin> repos = getPlugins(RepositoryPlugin.class);
		for (RepositoryPlugin r : repos) {
			if (r.getName().equals(repoName))
				return r;
		}
		error("Could not find -baselinerepo %s", repoName);
		return null;
	}
