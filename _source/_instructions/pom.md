---
layout: default
class: Processor
title: -pom BOOLEAN | PROPERTIES
summary: Generate a maven pom in the JAR
---

The `-pom` instruction can generate a pom derived from the manifest and store it in the bundle. The default will be splitting up the Bundle Symbolic name in a `groupid` (everything until the last '.') and a `artifactid` (everthing from the last '.' to the end. However, it is also specify overrides in propterties. The following properties are supported:

|Key              |Default          |Description                         |
|`groupid`        |bsn prefix       |The group id used                   |
|`artifactid`     |bsn suffix       |The artifact id to use. If `groupid` is set, the default is the Bundle Symbolic Name|
|`version`        |bundle version   |The version to use.                 |
|`where`          |`pom.xml`        |The location of the pom.xml file.   |

The pom will also attempt to convert the following headers to their POM counterpart:

* `Bundle-Description`
* `Bundle-DocUrl`
* `Bundle-Vendor`
* `Bundle-License`

## Example

The following example bnd file:

	Bundle-SymbolicName: com.example.foo
	Bundle-Version: 1.2.3.qualifier
	-pom: true

Generates the following pom in `pom.xml`:

	<project 	xmlns="http://maven.apache.org/POM/4.0.0" 
			xmlns:xsi="" 
			xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  		<modelVersion>4.0.0</modelVersion>
  		<groupId>com.example</groupId>
  		<artifactId>foo</artifactId>
  		<version>1.2.3.qualifier</version>
  		<name>com.example.foo</name>
	</project>

You can override the different parts of the Maven coordinates:

	Bundle-SymbolicName: com.example.foo
	Bundle-Version: 1.2.3.qualifier
	-pom: groupid=com.example,where=META-INF/maven/pom.xml,version=${version;==;${Bundle-Version}}
	
Generates the following pom in `META-INF/maven/pom.xml`:

	<project 	xmlns="http://maven.apache.org/POM/4.0.0" 
			xmlns:xsi="" 
			xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  		<modelVersion>4.0.0</modelVersion>
  		<groupId>com.example</groupId>
  		<artifactId>com.example.foo</artifactId>
  		<version>1.2</version>
  		<name>com.example.foo</name>
	</project>


