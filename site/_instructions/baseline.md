---
layout: default
class: Macro
title: -baseline BSN-SPEC (',' BSN-SPEC )*
summary: Control what bundles are enabled for baselining
---

	@Override
	public void doBaseline(Jar dot) throws Exception {

		String diffignore = project.getProperty(Constants.DIFFIGNORE);
		trace("ignore headers %s", diffignore);
		differ.setIgnore(diffignore);

		Jar fromRepo = getBaselineJar();
		if (fromRepo == null) {
			trace("No baseline jar %s", getProperty(Constants.BASELINE));
			return;
		}

		Version newer = new Version(getVersion());
		Version older = new Version(fromRepo.getVersion());

		if (!getBsn().equals(fromRepo.getBsn())) {
			error("The symbolic name of this project (%s) is not the same as the baseline: %s", getBsn(),
					fromRepo.getBsn());
			return;
		}

		//
		// Check if we want to overwrite an equal version that is not staging
		//

		if (newer.getWithoutQualifier().equals(older.getWithoutQualifier())) {
			RepositoryPlugin rr = getBaselineRepo();
			if (rr instanceof InfoRepository) {
				ResourceDescriptor descriptor = ((InfoRepository) rr).getDescriptor(getBsn(), older);
				if (descriptor != null && descriptor.phase != Phase.STAGING) {
					error("Baselining %s against same version %s but the repository says the older repository version is not the required %s but is instead %s",
							getBsn(), getVersion(), Phase.STAGING, descriptor.phase);
					return;
				}
			}
		}

		trace("baseline %s-%s against: %s", getBsn(), getVersion(), fromRepo.getName());
		try {
			Baseline baseliner = new Baseline(this, differ);

			Set<Info> infos = baseliner.baseline(dot, fromRepo, null);
			if (infos.isEmpty())
				trace("no deltas");

			for (Info info : infos) {
				if (info.mismatch) {
					SetLocation l = error(
							"Baseline mismatch for package %s, %s change. Current is %s, repo is %s, suggest %s or %s\n",
							info.packageName, info.packageDiff.getDelta(), info.newerVersion, info.olderVersion,
							info.suggestedVersion, info.suggestedIfProviders == null ? "-" : info.suggestedIfProviders);
					l.header(Constants.BASELINE);

					fillInLocationForPackageInfo(l.location(), info.packageName);
					if (getPropertiesFile() != null)
						l.file(getPropertiesFile().getAbsolutePath());
					l.details(info);
				}
			}
			aQute.bnd.differ.Baseline.BundleInfo binfo = baseliner.getBundleInfo();
			if (binfo.mismatch) {
				SetLocation error = error("The bundle version (%s/%s) is too low, must be at least %s",
						binfo.olderVersion, binfo.newerVersion, binfo.suggestedVersion);
				error.context("Baselining");
				error.header(Constants.BUNDLE_VERSION);
				error.details(binfo);
				FileLine fl = getHeader(Pattern.compile("^" + Constants.BUNDLE_VERSION, Pattern.MULTILINE));
				if (fl != null) {
					error.file(fl.file.getAbsolutePath());
					error.line(fl.line);
					error.length(fl.length);
				}
			}
		}
		finally {
			fromRepo.close();
		}
	}
	
	
		/**
	 * This method attempts to find the baseline jar for the current project. It
	 * reads the -baseline property and treats it as instructions. These
	 * instructions are matched against the bsns of the jars (think sub
	 * builders!). If they match, the sub builder is selected.
	 * <p>
	 * The instruction can then specify the following options:
	 * 
	 * <pre>
	 * 	version : baseline version from repository
	 * 	file    : a file path
	 * </pre>
	 * 
	 * If neither is specified, the current version is used to find the highest
	 * version (without qualifier) that is below the current version. If a
	 * version is specified, we take the highest version with the same base
	 * version.
	 * <p>
	 * Since baselining is expensive and easily generates errors you must enable
	 * it. The easiest solution is to {@code -baseline: *}. This will match all
	 * sub builders and will calculate the version.
	 * 
	 * @return a Jar or null
	 */
	public Jar getBaselineJar() throws Exception {
		String bl = getProperty(Constants.BASELINE);
		if (bl == null || Constants.NONE.equals(bl))
			return null;

		Instructions baselines = new Instructions(getProperty(Constants.BASELINE));
		if (baselines.isEmpty())
			return null; // no baselining

		RepositoryPlugin repo = getBaselineRepo();
		if (repo == null)
			return null; // errors reported already

		String bsn = getBsn();
		Version version = new Version(getVersion());
		SortedSet<Version> versions = removeStagedAndFilter(repo.versions(bsn), repo, bsn);

		if (versions.isEmpty()) {
			// We have a repo
			Version v = new Version(getVersion());
			if (v.getWithoutQualifier().compareTo(Version.ONE) > 0) {
				warning("There is no baseline for %s in the baseline repo %s. The build is for version %s, which is <= 1.0.0 which suggests that there should be a prior version.",
						getBsn(), repo, v);
			}
			return null;
		}

		//
		// Loop over the instructions, first match commits.
		//

		for (Entry<Instruction,Attrs> e : baselines.entrySet()) {
			if (e.getKey().matches(bsn)) {
				Attrs attrs = e.getValue();
				Version target;

				if (attrs.containsKey("version")) {

					// Specified version!

					String v = attrs.get("version");
					if (!Verifier.isVersion(v)) {
						error("Not a valid version in %s %s", Constants.BASELINE, v);
						return null;
					}

					Version base = new Version(v);
					SortedSet<Version> later = versions.tailSet(base);
					if (later.isEmpty()) {
						error("For baselineing %s-%s, specified version %s not found", bsn, version, base);
						return null;
					}

					// First element is equal or next to the base we desire

					target = later.first();

					// Now, we could end up with a higher version than our
					// current
					// project

				} else if (attrs.containsKey("file")) {

					// Can be useful to specify a file
					// for example when copying a bundle with a public api

					File f = getProject().getFile(attrs.get("file"));
					if (f != null && f.isFile()) {
						Jar jar = new Jar(f);
						addClose(jar);
						return jar;
					}
					error("Specified file for baseline but could not find it %s", f);
					return null;
				} else {
					target = versions.last();
				}

				// Fetch the revision

				if (target.getWithoutQualifier().compareTo(version.getWithoutQualifier()) > 0) {
					error("The baseline version %s is higher than the current version %s for %s in %s", target,
							version, bsn, repo);
					return null;
				}
				if (target.getWithoutQualifier().compareTo(version.getWithoutQualifier()) == 0) {
					if (isPedantic()) {
						warning("Baselining against jar");
					}
				}
				File file = repo.get(bsn, target, attrs);
				if (file == null || !file.isFile()) {
					error("Decided on version %s-%s but cannot get file from repo %s", bsn, version, repo);
					return null;
				}
				Jar jar = new Jar(file);
				addClose(jar);
				return jar;
			}
		}

		// Ignore, nothing matched
		return null;
	}
	
	
	
		@Description("Compare a newer bundle to a baselined bundle and provide versioning advice")
	@Arguments(arg = {
			"[newer jar]", "[older jar]"
	})
	interface baseLineOptions extends Options {
		@Description("Output file with fixup info")
		String fixup();

		@Description("Show any differences")
		boolean diff();

		@Description("Be quiet, only report errors")
		boolean quiet();

		@Description("Show all, also unchanged")
		boolean all();
	}


	interface diffOptions extends Options {
		@Config(description = "Print the API")
		boolean api();

		@Config(description = "Print the Resources")
		boolean resources();

		@Config(description = "Print the Manifest")
		boolean manifest();

		@Config(description = "Show full tree")
		boolean full();

		@Config(description = "Print difference as valid XML")
		boolean xml();

		@Config(description = "Where to send the output")
		File output();

		@Config(description = "Limit to these packages (can have wildcards)")
		Collection<String> pack();

		@Config(description = "Ignore headers")
		Collection<String> ignore();
	}


	
	
