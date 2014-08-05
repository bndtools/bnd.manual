---
title: Quick Start
layout: default
---

Since bnd is by design headless, the best way to get start is to use one of the IDEs like [bndtools][1]. They have tutorials and an IDE is a more pleasant place than a command line. that said, these tutorials tend to focus on the higher layers.
Therefore, this section uses the command line bnd application to demonstrate much of the inner details of bndlib. bnd has lots of commands to try out aspects of bndlib. The [install section](120-install.md) shows how to install bnd.

## Wrapping

A common use of bnd is to wrap a JAR file; there are still too many files out there that have no OSGi metadata.This lack of metadata makes them kind of useless as a bundle since it will be fully isolated from the other bundles. In OSGi, you must explicitly indicate what packages should be imported and what packages should be exported, by default everything is private. Though this can be quite frustrating sometimes, a rather strong argument can be made for the long term benefits of this strategy.

The OSGi metadata is stored in the _manifest_ of the JAR file. The manifest is a like a properties file stored in `META-INF/MANIFEST.MF` in the JAR file. The manifest format is [specified by Sun/Oracle][4]. Though the manifest is a text file and human readable it has a number of quirks that makes it hard to edit for humans. So even though it is text, the intention of the manifest was never to be done by hand, bnd like tools were always the intention. Tools can provide many useful defaults, handle expansions, and verify consistency,

So let's use bnd to wrap the [javax.activation.jar][2] file even though it is already wrapped for us in the [Enterprise Bundle Repository][3]. Let's download it:

	$ mkdir jar
	$ curl http://repo1.maven.org/maven2/javax/activation/activation/1.1.1/activation-1.1.1.jar >jar/javax.activation-1.1.1.jar
	
bnd provides a convenience function to print out the contents of a JAR. The --uses option shows the packages and what other packages they use. 

	$ bnd print --uses jar/javax.activation.jar 
	[USES]
	com.sun.activation.registries            []
	com.sun.activation.viewers               [javax.activation]
	javax.activation                         [com.sun.activation.registries]

### Simplistic

In the simplistic case our goal is to provide a manifest that makes all the packages in the source JAR available to the other bundles (exports) and that imports anything the classes in the JAR require from the external world (imports). bnd is controlled by a _bnd_ file. This file is a standard Java properties file. In this properties file we can provide headers, instructions, and macros to bnd. In general, a header starts with an upper case, and an instruction with a minus sign ('-'). The format of the headers and instructions is defined by the OSGi specifications. The headers specifically try to follow the structure of an OSGi header. Since exports are defined with the `Export-Package` header and imports are defined by the `Import-Package` header we can create a bnd file that looks very much like the desired manifest, except for the instruction that defines our class path:

	javax.activation.bnd:
	
		-classpath:		jar/javax.activation-1.1.1.jar
		Export-Package: *
	
The casing of the headers is important, even though OSGi headers are case insensitive. That is Export-Package is not the same for bnd as Export-package.

The properties format we use in bnd is darned flexibile. A header is recognized when you start the line with a header name (a word without spaces) and then:

* Any white space (space, tab)
* A  colon (':'), white space around the colon is ignored
* An equal sign ('='), white space around the equal sign is ignored
 
Though lines can be as long as you want, it is better for your brains to keep them short. You can break a line with a backslash ('\') followed by a newline (\n). Be careful, the newline *must* follow the backslash or you will get whatever comes after the backslash in the header. Some examples that all set the variable `foo` to 3:

	foo				= 	3
	foo:3
	foo 3
	foo \
	3

We can now wrap the source JAR with the following command:

	$ bnd javax.activation.bnd
	
bnd is prepared to show the manifest of any bundle:

	$ bnd javax.activation.jar
	[MANIFEST javax.activation]
	Bnd-LastModified                         1407229057278                           
	Bundle-ManifestVersion                   2                                       
	Bundle-Name                              javax.activation                   
	Bundle-SymbolicName                      javax.activation                   
	Bundle-Version                           0                                       
	Created-By                               1.8.0 (Oracle Corporation)              
	Export-Package                           com.sun.activation.registries;version="0.0.0",
	                                         com.sun.activation.viewers;uses:="javax.activation";version="0.0.0",
	                                         javax.activation;version="0.0.0"
	Manifest-Version                         1.0                                     
	Require-Capability                       osgi.ee;filter:="(&(osgi.ee=JavaSE)(version=1.4))"
	Tool                                     Bnd-2.4.0.201408050759                  


### The Manifest

It is clear that bnd has been busy. It detected 3 packages (com.sun.activation.registries, com.sun.activation.viewers, and javax.activation) and since we exported all, they appear in the manifest as exports. Observant readers will have noticed that we used _wildcards_ ('*'). The OSGi specifications are not so generous, every package must be explicitly listed so that the manifest contains all the information required to process a bundle. This, again, has many long term macro advantages. It's however bnd's goal in life to provide the convenience in this model and that is the reason that bnd supports wildcards in virtual any situation where they are usable. A wildcard requires a _domain_ to be effective, you can only select something out of a set, its domain. In the case of `Export-Package`, its domain is the _class path_. This class path is set by the `-classpath` instruction. In this case the activation jar.


### Export Package

We now have a number of issues. First, we should not export the `com.sun.*` packages since they are private. The other issue is that we need to version the exported package. Why? Well, an export means you're making a technical debt. Other developers will use your package and you need to signal them these changes when you modify the package one day. The sole purpose of a version is provide that signal. For this reason, the versions have a strictly defined syntax:

* major – For breaking change
* minor – For changes that break the provider of the package but are backward compatible for the consumer of the package. See [semantic versions][4].
* micro – Backward compatible changes, e.g. small bug fixes or documentation changes
* qualifier – Identifying the build 

We can add the version to the export-package by decorating the exported packages. 
  
		-classpath:		jar/javax.activation-1.1.1.jar
		Export-Package: javax.activation;version=1.1.1

Though we replaced the wildcard with the only public package, you could also decorate the wildcard.

### Headers

bnd also added a number of other headers. Let's discuss them one by one.

The `Bnd-LastModified` header contains the last modification time of the newest source that was used to construct the JAR. This field is used by build tools to track if the JAR must be renewed.

The `Bundle-ManifestVersion` is defined by the OSGi specifications as the semantics of the manifest headers since release 3, the versions that bnd is tracking. This header is always added automatically. The `Manifest-Version` is also standard JAR, it defines the overall syntax of manifests.

The `Bundle-Name` and `Bundle-SymbolicName` headers are derived from the file name in this case. In general, bnd is quite creative in coming up with these names unless you override them with a specific header. For example:

		-classpath:		jar/javax.activation-1.1.1.jar
		Export-Package: javax.activation;version=1.1.1
		Bundle-SymbolicName: bnd.osgi.javax.activation

### Bundle Version

The `Bundle-Version` header obviously provides the version. Default it will use version 0, so we should override it. We could just add the 1.1.1 version from our source JAR, however, this would make it impossible to to distinguish the different variations we might make over time. A good tool is to use the ${tstamp} macro for the _qualifier_. The qualifier is the last part of the version. A macro provides a textual replacement, bnd has hundreds of macros (including access to Javascript) and you can define your own. The ${tstamp} macro provides a time stamp.

		-classpath:		jar/javax.activation-1.1.1.jar
		Export-Package: javax.activation;version=1.1.1
		Bundle-Version:	1.1.1.${tstamp}

The `Created-By` header is standard for JARs, it defines the Java VM that ran the tool, it is kind of useless. It is a nice way to demonstrate how one can remove headers from the output. The `-removeheaders` instruction can be used to remove headers from the manifest. 

		-classpath:		jar/javax.activation-1.1.1.jar
		-removeheaders: Created-By, Tool*
		
		Export-Package: javax.activation;version=1.1.1
		Bundle-Version:	1.1.1.${tstamp}

### Remove Headers

The `removeheaders` header uses a _selector_ expression. A selector expression consists of a number of comma separated clauses. A clause contains a name (which can be wildcarded) and some optional instructions. A selector is always applied to a _domain_, in the case of `-removeheaders` the domain is all the headers in the manifest. The clauses in the selector are applied one by one in their declaration order to the domain. If a clause matches a member of the domain then the member is removed from the set, and the next member is again matched against the first clause. A suffix of ':i' indicates a case insensitive match.

This ordering is important because selector clauses can use negation. When a clause starts with an exclamation mark ('!') then a matching member is further ignored. Therefore, the following selector will never match the Bundle-Name, the first clause already threw any header that starts with `Bundle-` out:

	-removeheaders: !Bundle-*, Bundle-Name:i

The `Require-Capability` header ensures that the runtime has at least a Java 1.4 VM by requiring an OSGi execution environment. bnd calculates this requirement based on the compiler version used to compile the classes in the JAR. We can override this requirement or disable it with the `-noee=true` instruction.


### Output

We can also specify the output file. By default, bnd will store JAR under the same name as the bnd file, but then with a `.jar` extension. You can override the default from the command line:

	$ bnd do --output bundle javax.activation.bnd

You can either specify a directory or a file name. However, you can also specify the output in the bnd file itself:

		-classpath:		jar/javax.activation-1.1.1.jar
		-removeheaders: Created-By, Tool*
		-output: 		bundle/java.activation-1.1.1.jar

		Export-Package: javax.activation;version=1.1.1
		Bundle-Version:	1.1.1.${tstamp}

### DNRY

The 'Do Not Repeat Yourself' mantra encodes one of the more important lessons of software engineering. You should always strive to define a 'thing' or a piece of knowledge in only one place. In our little example, we actually repeat ourselves with the version 1.1.1:

* The classpath
* The export package
* The version
* The output

bnd has a macro processor on board that is a life saver if you are addicted to DNRY (we are). In this case we can create a macro that captures the 1.1.1 and reuse it whenever we need it.

		v:				1.1.1
		
		-classpath:		jar/javax.activation-${v}.jar
		-removeheaders: Created-By, Tool*
		-output: 		bundle/java.activation-${v}.jar

		Export-Package: javax.activation;version=${v}
		Bundle-Version:	${v}.${tstamp}
   
Variables are always lower case (if they started with an upper case, they would end up in the manifest). However, any instruction and header is also a macro and we could refer to that one. For example, we could refer to the Bundle-Version instead of using a separate macro:

		-classpath:		jar/javax.activation-${Bundle-Version}.jar
		-removeheaders: Created-By, Tool*
		-output: 		bundle/java.activation-${Bundle-Version}.jar

		Export-Package: javax.activation;version=${Bundle-Version}
		Bundle-Version:	1.1.1.${tstamp}

However, the timestamp in the output is kind of awkward ... Fortunately there is a special builtin macro that can handle this, the ${version} macro. This macro can pick out parts of a version as well as bump versions. In this case we use it to ignore the qualifier.

		-classpath:		jar/javax.activation-${version;===;${Bundle-Version}}.jar
		-removeheaders: Created-By, Tool*
		-output: 		bundle/java.activation-${version;===;${Bundle-Version}}.jar

		Export-Package: javax.activation;version=${version;===;${Bundle-Version}}
		Bundle-Version:	1.1.1.${tstamp}

This is actually starting to look ridiculously convoluted and contrived, obviously the first 'version' was the best:

		v:				1.1.1
		
		-classpath:		jar/javax.activation-${v}.jar
		-removeheaders: Created-By, Tool*
		-output: 		bundle/java.activation-${v}.jar

		Export-Package: javax.activation;version=${v}
		Bundle-Version:	${v}.${tstamp}






bnd is the Swiss army knife of OSGi, it is used for creating and working with OSGi bundles. Its primary goal is take the pain out of developing bundles. With OSGi you are forced to provide additional metadata in the JAR's manifest to verify the consistency of your "class path". This metadata must be closely aligned with the class files in the bundle and the policies that a company has about versioning. Maintaining this metdata is an error prone chore because many aspects are redundant.

bnd's raison d'etre is therefore to remove the chores and use the redundancy to create the manifest from the class files instead of maintaining it by hand. The core task is therefore to analyze the class files and find any dependencies. These dependencies are then merged with ''instructions'' supplied by the user. For example, adding a version to all imported packages from a specific library can be specified as:

    Import-Package: com.library.*; version = 1.21

The OSGi manifest must explicitly mention a package, bnd allows the use of wildcards. bnd contains many more such conveniences. bnd roots are about 10 years old and bnd has therefore a large number of functions that remove such chores. These range from simplifying the use of OSGi Declarative Services, working with Spring and Blueprint, WAR and WAB files, version analysis, project dependencies, and much more.

Over time bnd started to appear in many different incarnations. It is an an ant task, a command line utility, and a bundle for Eclipse. Other projects have used bndlib to create a maven plugin, bndtools and Sigil both Eclipse IDEs, and others. By keeping the core library small and uncoupled (bnd has no external connections except Java 5), it is easy to embed the functionality in other projects.

## Workflow
Traditionally, JAR files were made with the JDK jar tool, the jar ant task, or the Maven packager. All these tools share the same concept. The developer creates a directory image of the jar by copying files to a directory; this directory is then jarred. This model can be called the ''push'' model. Obviously this method works well.

bnd works differently, it uses the ''pull'' model. Instructions in the bnd file describe the contents of the desired JAR file without writing this structure to disk. The contents from the output can come from the class path or from anywhere in the file system. For example, the following instruction includes the designated packages in the JAR:

  Private-Package: com.example.*
 
bnd can create a JAR from packages the sources, directories or other JAR files. You never have to copy files around, the instructions that Bnd receives are sufficient to retrieve the files from their original location, preprocessing or filtering when required.

The Jar is constructed from 3 different arguments:

    Export-Package
    Private-Package
    Include-Resource

Private-Package and Export-Package contain ''instructions''. Instructions are patterns + attributes and directives, looking like normal OSGi attributes and directives. For example:

    Export-Package: com.acme.*;version=1.2

Each instruction is applied to each package on the classpath in the definition order. That is, if an earlier instruction matches, the later instruction never gets a chance to do its work. If an instruction matches its attributes and properties are applied to the packages. The difference between the Private-Package argument and the Export-Package arguments is that the export version selects the packages for export. If the packages overlap between the two, the export wins.

An instruction can also be negative when it starts with a '!'. In that case the package is excluded from the selection. For example:

    Export-Package: !com.acme.impl, com.acme.*;version=1.2

Note that the instructions are applied in order. If the ! instruction was at the end in the previous example, it would not have done its work because the com.acme.* would already have matched.

The Include-Resource argument can be used to copy resources from the file system in the JAR. This is useful for licenses, images, etc. The instructions in the argument can be a directory, a file, or an inline JAR. The default JAR path is the the root for a directory or the filename for a file. The path can be overridden. Instructions that are enclosed in curly braces, like {license.txt}, are pre-processed, expanding any macros in the file.

Once the JAR is created, the bnd program analyzes the classes and creates an import list with all the packages that are not contained in the jar but which are referred to. This import list is matched against the Import-Package instructions. Normally, the Import-Package argument is *; all referred packages will be imported. However, sometimes it is necessary to ignore an import or provide attributes on the import statement. For example, make the import optional or discard the import:

    Import-Package: !com.acme.*, *;resolution:=optional

The arguments to bnd are normal given as a set of properties. Properties that begin with an upper case are copied to the manifest (possibly after processing). Lower case properties are used for macro variables but are not set as headers in the manifest.

After the JAR is created, the bnd program will verify the result. This will check the resulting manifest in painstaking detail.

The bnd program works on a higher level then traditional jarring; this might take some getting used to. However, it is much more elegant to think in packages than that it is to think in files. The fact that bnd understand the semantics of a bundle allows it to detect many errors and allows bundles to be created with almost no special information. 

bnd will not create an output file if none of the resources is newer than an existing output file.

The program is available in several forms: command line, ant task, maven plugin, and an Eclipse plugin.

##Tips
There are some common pitfalls that can be prevented by following the tips:

* Keep it simple. bnd's defaults are pretty good and not specifying is usually the best solution. KISS!
* Think packages ... yes it feels redundant to specify the packages that are in your source directory but your artifact will get a life of its own over time. Many IDEs and build tools restricted us to one artifact per project but bnd allows many artifacts, allowing the choice of granularity to you. As OSGi's packages can be easily refactored you can design the contents of your artifacts depending on the deployment needs. Think packages!
* Private is always better than export, only use export when you absolute need it.
* Not versioning an exported package is at your own peril. Sorry, that is false, it is at the peril of your users.
* Do not use the Bundle-ClassPath, if you need to include whole JARs, see the @ option at Include-Resource
* If you do not understand a header, remove it
* If you have a problem, make an example that is as small as possible and send it to [me][mailto:Peter.Kriens@aQute.biz|me].


[1]: http://www.bndtools.org
[2]: http://repo1.maven.org/maven2/javax/activation/activation/1.1.1/activation-1.1.1.jar
[3]: http://ebr.springsource.com/repository/app/
[4]: http://docs.oracle.com/javase/8/docs/technotes/guides/jar/jar.html#JARManifest
[5]: 170-versioning.html