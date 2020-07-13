# eclipse-tycho

Tutorial how to add [Tycho](https://www.eclipse.org/tycho/ "Tycho")  to an existing Eclipse Plug-in feature...

## Table of content

* [Adding Tycho](#adding-tycho)
* [Main Pointers to the essential files before the explanation](#main-pointers-to-the-essential-files-before-the-explanation)
* [Step by step instructions](#step-by-step-instructions)
    * [Create Parent POM](#create-parent-pom)
    * [Add a Target Definition as a Maven Module](#add-a-target-definition-as-a-maven-module)
    * [Define the Eclipse repository for the build](#define-the-eclipse-repository-for-the-build)
    * [Add POMs for the existing Feature and UI Plug-Ins](#add-poms-for-the-existing-feature-and-ui-plug-ins)
    * [Launch Configurations](#launch-configurations)
    * [Adding Unit TEsts](#adding-unit-tests)
* [Setting up your own project](#setting-up-your-own-project)
    * [Create a Plug-in](#create-a-plug-in)
    * [Create a Feature](#create-a-feature)
    * [Export as p2 repository](#export-as-p2-repository)
    * [Share into a GIT repository](#share-into-a-git-repository)

## Adding Tycho

Below I describe in [Setting up your own Project](#setting-up-your-own-project) the prerequisites of these instruction. 
I start with a basic Eclipse feature project and a basic Eclipse plug-in project.

This tutorial is based on the following information:
* [Tycho Reference Card](https://wiki.eclipse.org/Tycho/Reference_Card "Tycho Reference Card")
* [http://www.vogella.com/tutorials/EclipseTycho/article.html](http://www.vogella.com/tutorials/EclipseTycho/article.html)
* [https://github.com/vogellacompany/tycho-example](https://github.com/vogellacompany/tycho-example) 

## Main Pointers to the essential files before the explanation

As a kind of management summary...

* [Parent POM](https://github.com/holzem/eclipse-tycho/blob/master/pom.xml)
* [Target POM](https://github.com/holzem/eclipse-tycho/blob/master/org.example.eclipse.tycho.target/pom.xml)
* [Target Definition](https://github.com/holzem/eclipse-tycho/blob/master/org.example.eclipse.tycho.target/org.example.eclipse.tycho.target.target)
* [P2 Repository POM](https://github.com/holzem/eclipse-tycho/blob/master/org.example.eclipse.tycho.repository/pom.xml)
* [P2 Repository Manifest](https://github.com/holzem/eclipse-tycho/blob/master/org.example.eclipse.tycho.repository/category.xml)
* [Feature POM](https://github.com/holzem/eclipse-tycho/blob/master/org.example.eclipse.tycho.feature/pom.xml)
* [Plug-in POM](https://github.com/holzem/eclipse-tycho/blob/master/org.example.eclipse.tycho.ui/pom.xml)
* [Unittest POM](https://github.com/holzem/eclipse-tycho/blob/master/org.example.eclipse.tycho.tests/pom.xml)

## Step by step instructions

### Create Parent POM

The spirit and purpose of the parent POM is to make the definition of the children
easier. So most of the basic definition is placed in this file. Define a `groupId` and
an `artifactId` for your project and choose as version 1.0.0-SNAPSHOT 
(since all plugins are automatically created with version `1.0.0.qualifier`.  

	<groupId>org.example.eclipse.tycho</groupId>
	<version>1.0.0-SNAPSHOT</version>
	<artifactId>org.example.eclipse.tycho.parent</artifactId>
	<packaging>POM</packaging>

Choose as properties the current tycho version, the java version and the encoding of the
files in your workspace:

	<properties>
		<tycho-version>1.0.0</tycho-version>
		<project.build.sourceEncoding>windows-1252</project.build.sourceEncoding>
		<Maven.compiler.source>1.8</Maven.compiler.source>
		<Maven.compiler.target>1.8</Maven.compiler.target>
	</properties>

At this time leave the modules section empty.

Now define the plugins that are needed for the build:

* Compile application sources with eclipse plugin dependencies:
  `org.eclipse.tycho` / `tycho-Maven-plugin` / `${tycho-version}`  
* Perform unit tests:
  `org.eclipse.tycho` / `tycho-surefire-plugin` / `${tycho-version}`  
* Create a JAR-package containing all the source files of a osgi project.
  `org.eclipse.tycho` / `tycho-source-plugin` / `${tycho-version}`  
* Generate a source feature for projects of packaging type eclipse-feature
  `org.eclipse.tycho` / `tycho-source-feature-plugin` / `${tycho-version}`  
* Generate p2 metadata for the repository
  `org.eclipse.tycho` / `tycho-p2-plugin` / `${tycho-version}`  
* Specify which environments your software should be built for 
  `org.eclipse.tycho` / `target-platform-configuration` / `${tycho-version}`  

Unfortunately Eclipse partially does not know what to do with the plugins and messages like
_Plugin execution not covered by lifecycle configuration: 
org.apache.Maven.plugins:Maven-clean-plugin:2.5:clean 
(execution: default-clean-1, phase: initialize)_ 
will pop up in your POM editor. To solve this long-standing issue the plugin
`org.eclipse.m2e` / `lifecycle-mapping` was created. This plugin basically
sets the _ignore_ action for the plugin-goal-combination that Eclipse should ignore
in its workspace.

You can find the complete POM in the github repository:
parent [pom.xml](https://github.com/holzem/eclipse-tycho/blob/master/pom.xml "pom.xml") 

### Add a Target Definition as a Maven Module

Having set the basic information how to compile the eclipse feature the target has to
be defined. Create a new module to hold the target definition to be used by the build.
Be sure to set `eclipse-target-definition` as the packaging of the POM.

![New Maven module for target definition page 1](resources/new-maven-module-target-1.png "New Maven module for target definition page 1") 
![New Maven module for target definition page 2](resources/new-maven-module-target-2.png "New Maven module for target definition page 2")

Now create a target definition for the build, using the desired Eclipse Platform version:

The editor for this does not create a target definition I like. So use an text to create the content:

	<?xml version="1.0" encoding="UTF-8"?>
	<?pde version="3.8"?>
	<target name="org.example.eclipse.tycho.target" sequenceNumber="18">
	<locations>
	<location includeAllPlatforms="false" includeConfigurePhase="true" includeMode="planner" includeSource="true" type="InstallableUnit">
	<unit id="org.eclipse.platform.feature.group" version="0.0.0"/>
	<unit id="org.eclipse.jdt.feature.group" version="0.0.0"/>
	<repository location="http://download.eclipse.org/releases/oxygen/201712201001"/>
	</location>
	</locations>
	<environment>
	<os>win32</os>
	<ws>win32</ws>
	<arch>x86_64</arch>
	</environment>
	</target>

This refers to the Eclipse Oxygen.2 release repository and defines dependencies to 
* `org.eclipse.platform.feature.group`
* `org.eclipse.jdt.feature.group`

It defines the desired runtime environment for this target as well.

### Define the Eclipse repository for the build

We are still not there. Now we need to define the p2 repository for the build output and create a category manifest.
Make sure to choose `eclipse-repository` as the packaging of the POM.

![New Maven module for the p2 repository page 1](resources/new-maven-module-repository-1.png "New Maven module for the p2 repository page 1") 
![New Maven module for the p2 repository page 2](resources/new-maven-module-repository-2.png "New Maven module for the p2 repository page 2")
![New category manifest](resources/new-maven-module-repository-category-manifest.png "New category manifest")

Unfortunately I found no way to create the desired output using the manifest editor. So I chose the XML editor instead:

	<?xml version="1.0" encoding="UTF-8"?>
	<site>
	   <feature id="org.example.eclipse.tycho.feature" version="0.0.0">
	      <category name="main"/>
	   </feature>
	   <feature id="org.example.eclipse.tycho.feature.source" version="0.0.0">
	      <category name="main.source"/>
	   </feature>
	   <category-def name="main" label="Eclipse Tycho Demo"/>
	   <category-def name="main.source" label="Eclipse Tycho Demo (Sources)"/>
	</site>

Since the source feature is created by the build it can not be referenced by the editor.

### Add POMs for the existing Feature and UI Plug-Ins

Use *Configure --> Convert to Maven Project*  to convert the feature and plug-in projects.
Then add the following `pom.xml` files to the projects:

*Main content of feature/pom.xml*

	<parent>
		<groupId>org.example.eclipse.tycho</groupId>
		<artifactId>org.example.eclipse.tycho.parent</artifactId>
		<version>1.0.0-SNAPSHOT</version>
	</parent>
	<artifactId>org.example.eclipse.tycho.feature</artifactId>
	<packaging>eclipse-feature</packaging>

*Main content of plugin/pom.xml*

	<parent>
		<groupId>org.example.eclipse.tycho</groupId>
		<artifactId>org.example.eclipse.tycho.parent</artifactId>
		<version>1.0.0-SNAPSHOT</version>
	</parent>
	<artifactId>org.example.eclipse.tycho.ui</artifactId>
	<packaging>eclipse-plugin</packaging>

As you can see there is just the reference to the parent POM, the `artifactId`, and
the special eclipse packaging to set.

### Launch Configurations

I provided the following Eclipse Launch configurations:

![Launch configurations](resources/eclipse-launch-configurations.png "Launch configurations")

* _mvn clean_ cleans up the Maven target directories
* _mvn clean verify_ creates a p2 repository in the target folder of repository project
  (package is not enough since `tycho-surefire-plugin` runs in the Maven integration-test
  life cycle phase)
* _mvn set-version_ is used to set the next version of the plugins whenever it needs a new version
* _Launch Runtime Eclipse_ starts the plugins in a test workspace

### Adding Unit Tests

The next step is to create a new plug-in for unit tests. The standard naming convention to suffix
the plugin name with `.tests`.

![Unit test module page 1](resources/new-test-module-1.png "Unit test module page 1")
![Unit test module page 2](resources/new-test-module-2.png "Unit test module page 2")

At the current time there seems to be no easy way to add this project easily to the GIT project.
I use the following steps:

* Delete the plug-in project (do *not* delete the project contents)
* Move the project into the GIT repository (using some sort of file manager)
* Use EGit Import Projects to import the deleted project again (but now automatically shared 
  in the GIT repository.
* Add the plug-in as module to the parent POM
* Convert the new plug-in to a Maven Project

Create a `pom.xml` in the tests project with the following details. `eclipse-test-plugin`
as packaging defines the plug-in as a unit test plug-in.

	<parent>
		<groupId>org.example.eclipse.tycho</groupId>
		<artifactId>org.example.eclipse.tycho.parent</artifactId>
		<version>1.0.0-SNAPSHOT</version>
	</parent>
	<artifactId>org.example.eclipse.tycho.tests</artifactId>
	<packaging>eclipse-test-plugin</packaging>

Now 


## Setting up your own project

I used Eclipse Oxygen 2 to create the project:

### Create a Plug-in

The following wizard creates a simple plug-in...

![New Plugin-in Project page 1](resources/new-plugin-1.png "New Plug-in Project page 1") 
![New Plugin-in Project page 2](resources/new-plugin-2.png "New Plug-in Project page 2") 
![New Plugin-in Project page 3](resources/new-plugin-3.png "New Plug-in Project page 3") 
![New Plugin-in Project page 4](resources/new-plugin-4.png "New Plug-in Project page 4") 

### Create a Feature

Include the plug-in in the feature... 

![New Feature page 1](resources/new-feature-1.png "New Feature page 1") 
![New Feature page 2](resources/new-feature-2.png "New Feature page 2") 

### Export as p2 repository

Basically Eclipse provides everything to create a p2 repository

![Export Deployable feature page 1](resources/export-deployable-features-1.png "Export Deployable feature page 1") 
![Export Deployable feature page 2](resources/export-deployable-features-2.png "Export Deployable feature page 2") 

This yields a folder structure like:

![Export Deployable feature page 3](resources/export-deployable-features-3.png "Export Deployable feature page 3") 

### Share into a GIT repository

Now you can share the plug-ins into an existing GIT repository with the standard **EGit** share command. 
So far so good.

Now I prepare the next step to add the Maven POMs: I transform the root of the GIT repository to an
Eclipse project. To make it importable I add a `.project` file with the following content. This creates
a Maven parent project, adding the Maven2 builder and nature. We do not need it yet but this gives a nice 
starting point for adding the Maven Tycho build in the next step.

	<?xml version="1.0" encoding="UTF-8"?>
	<projectDescription>
		<name>org.example.eclipse.tycho.parent</name>
		<comment></comment>
		<projects>
		</projects>
		<buildSpec>
			<buildCommand>
				<name>org.eclipse.m2e.core.Maven2Builder</name>
				<arguments>
				</arguments>
			</buildCommand>
		</buildSpec>
		<natures>
			<nature>org.eclipse.m2e.core.Maven2Nature</nature>
		</natures>
	</projectDescription>

 
