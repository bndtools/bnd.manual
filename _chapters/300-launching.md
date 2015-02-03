---
order: 300
title: Launching
layout: default
---

bnd integrates an OSGi launcher. This launcher will start a framework and then install and start a list of bundles. The launching environment is described with a number of instructions that start with `-run`.

* `-runfw` — Specify the run framework in repository format
* `-runsystemcapabilities` — 
* `-runbundles` — A list of bundles to install in repository format
* `-runvm` — VM options. Use a comma to separate different options. E.g. `-Dx=4, -Dy=5`.
* `-runproperties` — System properties for the framework to create
* `-runpath` — A list of jars (not required to be bundles) that are put on the classpath and available from the system bundle. Any Export-Package headers are exported.
* `-runsystempackages` — An Export-Package list for packages exported by the system bundle.

## Packaging

Any bndrun file can be packaged by the launcher plugin. This creates an executable JAR that will contain all its dependencies.

    $ bnd package xyz.bndrun

## Exit codes

If the framework stops the launcher will exit. It will set a system exit code that reflects the event type from that the framework returned. The shell script that started the launcher can take the system exit code into account to restart the framework in certain cases.

	OSGi FrameworkEvent             Launcher Constant  Value
	STOPPED                         STOPPED            -9
	WAIT_TIMEDOUT	                TIMEDOUT           -3
	ERROR                           ERROR              -2
	WARNING                         WARNING            -1
	STOPPED_BOOTCLASSPATH_MODIFIED  STOPPED_UPDATE     -4
	STOPPED_UPDATE                  STOPPED_UPDATE     -4

The launcher therefore returns the process exit code UPDATE_NEEDED(-4) when it requires an update. This was chosen over doing an in-process update because it is much safer. So the launching script should look something like:

	do {
	  bnd run app.bndrun
	} while ($?==-4)
