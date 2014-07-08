---
title: Introduction
layout: default
---

Arguably, OSGi is arguably one of the best specifications in the Java world. It is a no compromise specification for a component framework that is based on what we today know are the best practices. It extends the type safety first model of Java to hold true over time when the modules morph into sometimes unrecognizable shapes. It provides a solid foundation to build applications that can run anywhere, the original promise of Java; offering these impressive features while remaining as simple as possible, albeit no simpler.

For some, the previous paragraph may come as a surprise because OSGi has had its share of people complaining about it. Surprisingly, the biggest complaint is often the Class Not Found Exception, which is always a perfect sign that people try to push a round peg in a too small square hole, and with all their might. You only see those exceptions when you're not doing engineering but when you are hacking. If you run head on into the walls that OSGi installs and it is giving you a headache, then just look around and find the elegant and easy to use doors: services.

Though this is all true, I do not claim that OSGi is trivial to use; triviality has a way to clash with large system that must evolve over many years. The software profession has a brutal industry that lures us with the siren song of a 'few hours work' to devour us while trying to main gigantic hairballs. As Fred Brooks already said so many years ago in his seminal book 'The Mythical Man Month', there is no silver bullet. Even OSGi will require hard work to build evolvable systems. And though we cannot make building complex systems easy, bnd can at least make it easier (and considerably more fun).

When we started building the OSGi specifications around 1998 it became clear we needed metadata to describe the bundles. We chose the manifest since this was already a well defined resource in the JAR, why invent the wheel? Actually, it was Anselm Baird, a Sun employee at the time, who had come up with this idea in his Java Embedded Server (JES). Basically, the manifest was a property file with benefits. We still consider this choice for embedded metadata an excellent idea. Looking at Maven where the metadata is separated over a surprising number of files. It is clear that the reduction in the number of files and potential errors caused by out-of-sync metadata is quite large. My favorite example of this additional complexity is when the SHA1 files necessary in Maven were also signed, an easy mistake but induced by unnecessary complexity. An OSGi bundle is a completely self described artifact.

So how got bnd started? Well, when we wrote our bundles in the dawn of the new millenium we quickly discovered that transient dependencies, well, eh, sucked. Several of us had great experience in Object Oriented design and though objects clearly had become mainstream, they frustratingly had not provided the level of reusability that some had dreamed of. As Brian Foote shows, we thought we were building shiny castles but in the end built a big ball of mud. Where our predecessors of the structured programming era always hammered on low coupling and high cohesion we had ignored their wise lessons and got ourselves tangled up in a nice mess. Trying to reuse a class too often dragged in more unwanted stuff than the cat does in the whole year.

The beauty of Java was that it had found an elegant solution to the scourge of transient dependencies: Java interfaces. An interface elegantly broke transitive dependencies while maintaining type safety at the cost of an indirection. A cost anyone should gladly bear for hairballs are quite distasteful. Surprisingly, while Java offered us this elegant solution to break transitive dependencies, most people in our industry were blindly committing themselves to the same route, just on another level, which actually would make the consequences even worse.

What we discovered was that we could use the idea of interfaces in an object oriented environment also for modules. An interface is an instance of contract based programming and we found out that we could use the same between modules. The idea behind the OSGi service is that you do not let modules directly interact but that you use reified conduits between the modules so that you keep the modules, which are implementations, unconnected. What better concept in Java to express this contract between modules is there but the concept of a package?

Originally devised as the Java modules, Java packages are heavily underappreciated in the Java world. Even highly expert users sometimes only appreciate the namescoping of the package, brushing over any accessibility issues arising out of the use of multiple classloaders. They are wrong, a package is a perfect module. That said, it is a perfect encapsulation to describe the contract that governs the collaboration between modules. An interface has a too small granularity for such a collaboration contract since it often requires additional interfaces like listeners and also helper classes. Packages, although they are too fine grained for deployment modules, are eminently suitable to define a contract to given the collaboration between deployment modules. 

So we had come up with the surprisingly well fitting dependency model: 'Look Ma, No Coupling!' But then using it became quite painful because it required a lot of tedious house keeping, tracking which packages you were using, of what version. Just bluntly expression your dependency on an implementation sometimes felt as attractive as the next shot of heroin for a junkie. Now, housekeeping and chores in general are quite low on the list of reasons to give meaning to live for most software developers, and we were no exception. Not only are we in general horrible at chores, we tend to make an amazing amount of mistakes doing them. Which, in general is not good software engineering. Fortunately, we largely live in a virtual world where it is easy to let the computer do the chores. And this is the raison d'etre of bnd: take the chores out of OSGi.

The conception of bnd was at Ericsson, where it was used to automate 

 


 
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


