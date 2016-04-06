---
title: Maven Bnd Repository Plugin
layout: default
summary: A plugin to use and release to Maven repositories 
---

The Maven Bnd Repository plugin provides a full interface to the Maven local repository in `~/.m2/repository` and remote repositories like [Nexus] or [Artifactory]. It implements the standard bnd Repository Plugin and can provide an OSGi Repository for resolving. A typical configuration looks like:

	-plugin.central = \
		aQute.bnd.repository.maven.provider.MavenBndRepository; \
			releaseUrl=https://repo.maven.apache.org/maven2/; \
			index=${.}/central.maven; \
			name="Central"
		
	-plugin.wrapper = \
	        aQute.bnd.deployer.repository.wrapper.Plugin; \
	            location            =	"${build}/cache/wrapper"; \
	            reindex				=	true

	-connection-settings: ${.}/settings.txt
 

The class name of the plugin is `aQute.bnd.repository.maven.provider.MavenBndRepository`. It can take the following configuration properties:

| Property         | Type  | Default | Description |
|------------------|-------|---------|-------------|
| `releaseUrl`     | `URI` |       |The URL to the repositories of released artifacts.| 
| `snapshotUrl`    | `URI` |       |The URL to the repositories of snapshot artifacts. If this is not specified then snapshots will also be attempted to be released to the `releaseUrl`.|
| `local`          | `PATH`| `~/.m2/repository` | The file path to the local Maven repository. This is by  |
|                  |       |                    | default the `~/.m2/repository` defined by maven. If specified, it should use forward 
|                  |       |                    | slashes. If the directory does not exist, the plugin will attempt to create it.|
| `readOnly`       |`true|false` | If set to _truthy_ then this repository is read only.|
| `name`           | `NAME`|                    | The name of the repository is 'Maven'|
| `index`          | `PATH`|                    | The path to the _index_ file. The index file is a list if Maven _coordinates_.| 

If no `releaseUrl` nor a `snapshotUrl` are specified then the repository is _local only_. For finding archives, both URLs are used, first `releaseUrl`.

The `index` file specifies a view on the remote repository, it _scopes_ it. Since we use the bnd repositories to resolve against, it is impossible to resolve against the world. The index file falls under source control, it is stored in the source control management system. This guarantees that at any time the project is checked out it has the same views on its repository. This is paramount to prevent build breackages due to changes in repositories.

## Coordinates & Terminology

The index file contains a list of _coordinates_. A coordinate specifies an _archive_ in a Maven _revison_. An archive is a ZIP, POM, JAR, or any other type of file. In Maven, these files are addressed within a revision with an _extension_ and a _classifier_. The extension indicates the type of the file and the classifier is a modifier that makes the name unique in the project. A _revision_ is the combination of a _program_ and a _version_, where the program is the combination of _groupId_ and _artifactId_.

For an archive, the extension has a default of `jar` and the classifier is by default not set (either `null` or empty). The syntax for the coordinates is therefore:

	group ':' artifact ( ':' extension ( ':' classifier )? )? ':' version ( '-SNAPSHOT` )?

Valid coordinates are:

	group:artifact:1.0-SNAPSHOT
	commons-cli:commons-cli:jar:sources:1.3.1
	commons-lang:commons-lang:2.6
	commons-logging:commons-logging:1.2
	org.osgi:org.osgi.core:6.0.0
	org.osgi:osgi.annotation:6.0.1

## Releasing

In the release cycle, a JAR is `put` to  _release_ repository. The project of the released JAR is provided as _context_, this means it can inherit from the project and workspace. This plugin will read the [-maven-release] instruction from the context. This can result in generating source and javadoc jars. By default, a local only release only installs the actual binary. 

To properly release the revision we need to know Maven specific information that is normally specified in the Maven _pom_. The bnd build can construct a pom using the [-pom] instruction. This instruction creates a pom in the JAR at `META-INF/group/artifact/pom.xml`. This plugin requires such a pom to be present. If it is not present, the [-maven-release] instruction must provide a PATH to an alternative pom. If no pom can be found then this is aborts the release. 

In general, the plugin will first _install_ the JARs and its constituents into the `local` revision directory, using the Maven naming rules. If this is successful, it uploads the files to the remote repository using the remote naming rules. It will then update the `maven-metadata` to reflect the new release.

In Maven, revisions that end in `-SNAPSHOT` are treated special in many places. In bnd, we support this model with the [-snapshot] and [-pom] instructions. If a `snapshotUrl` is specified, then versions that end in `SNAPSHOT` are released to that URL.

## Authentication

The Maven Bnd Repository uses the bnd Http Client. See the [-connection-settings] instruction for how to set the proxy and authentication information.

## IDEs

The repository view in the IDE will show detailed information when you hover the mouse over the the repository entry, the program entry, or the revision entry. 

You can add new entries by:

* Editing the `index` file. The repository will be updated immediately
* Using the menus on the revision or program entries to delete entries
* Dropping a URL to a POM. This pom will be parsed and then its coordinates are added. POMs from `search.maven.org` are also supported.
* Using the menus to add all runtime or compile time dependencies of another entry.

[Nexus]: http://www.sonatype.com/nexus-repository-sonatype
[Artifactory]: https://www.jfrog.com/open-source/
[-maven-release]: /instructions/maven-release
[-snapshot]: /instructions/snapshot
[-pom]: /instructions/pom
[-connection-settings]: /instructions/connection-settings
