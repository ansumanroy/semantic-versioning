# OSS style project
Tony Mkhael <tony.mkhael@gmail.com>

## What is this about?

if you're unfamiliar with semantic versioning, please start reading [here](http://semver.org/).
The rest of the document will focus on how to effectively use semantic versioning for your OSS projects.
It will guide you in detail through all the steps needed to

1. Setup a project
2. Building it using gradle
3. Documenting using javadoc and asciidoctor
4. Publishing to a nexus repository
5. Handling versioning using nebula

This is not about general guidelines or so, but merely a simple tutorial on how to put all the available bricks together in order
to have a OSS compliant project. It contains useful snippets you will/should probably use variants of in your OSS project.

## Detailed guide

In this tutorial, we will interact with the below tools/libraries.
Although no specific knowledge is required, it is always better to have a quick look around to see what's behind each one.

* [Gradle](http://www.gradle.org): Highly flexible build tool
* [Asciidoctor](http://www.asciidoctor.org): Toolset to generate cool documentation from pure text format
* [Nebula](https://nebula-plugins.github.io/): Gradle plugins provided by NetFlix
* [Jenkins CI](http://www.jenkins-ci.org/): Continuous delivery server
* [Nexus](http://www.sonatype.org/nexus/): Maven repository server

### Setting up your project

First, let's start by setting up the gradle project. For this you would need the following structure.

**Project structure**
```
.
+-- build.gradle <1>
+++ gradle
|   +++ wrapper
|       +-- gradle-wrapper.properties <2>
|       +-- gradle-wrapper.jar
+-- gradlew <3>
+-- gradlew.bat

1. The main gradle script for your project
2. The gradle wrapper files to specify the gradle version to use
3. The gradle command line scripts
```

> Except for the build.gradle, you can use the exact copy of those files in this r   epo, as those files do not change much and they only exist to bootstrap your gradle setup.

> It is necessary to have a gradle.properties file in the project root to be able to run some gradle tasks i.e. publish. You can also make it work if you're behind a proxy
You will not find this file in this repo, since it is supposed to be injected by the build environment (CI/Dev environment..)
However, here is a sample content that you should adapt in case you want to try out with this repo..

**gradle.properties**
```properties
#control daemon mode
org.gradle.daemon#true

#used for gradle JVM
systemProp.https.proxyHost#proxy
systemProp.https.proxyPort#3128
systemProp.http.proxyHost#proxy
systemProp.http.proxyPort#3128
systemProp.http.nonProxyHosts#*.domain.com|localhost

#use that specific JDK
org.gradle.java.home#C:\\myfavoritejdk\\

#used for the publishing
nexusGroup#com.acme
nexusHost#nexus-dev
nexusRepo#myrepo
nexusUser#user
nexusPassword#password
```

Now let's make this a java project. we'll add the following java class under src/main/java/com/acme/example/SemanticVersioning.java

***SemanticVersioning.java***
```java
package com.acme.example;

public final class SemanticVersioning {

  /**
   * A sample for the javadoc
   *
   * @param args Tha arguments to this magnificient application
   * @since 0.0.1
   */
  public static void main(String[] args) {
    System.out.println("Hello Semantic versioning");
  }

}
```

This also has to be reflected in the gradle script as below

***build.gradle***
```gradle
project.group = nexusGroup // <1>
project.description = 'Semantic versioning'

apply plugin: 'java' // <2>

repositories { // <3>
    jcenter()
}

<1> Assign a group id and a description for the project
<2> Apply the gradle java plugin
<3> Add jcenter as an artifact repository in case we need some useful libraries
```

### Building the project

To build the project, all we need to done is invoke the gradle build task using

```bash
+> ./gradlew build
Starting a new Gradle Daemon for this build (subsequent builds will be faster).
Inferred project: semantic-versioning, version: 0.0.2-SNAPSHOT
:compileJava
:processResources UP-TO-DATE
:classes
:jar
:assemble
:compileTestJava UP-TO-DATE
:processTestResources UP-TO-DATE
:testClasses UP-TO-DATE
:test UP-TO-DATE
:check UP-TO-DATE
:build

BUILD SUCCESSFUL

Total time: 11.917 secs
```

The build artifacts are generated under the build directory.

> Instead of trying to build your .gitignore from scratch, you can use [Netflix's template](https://github.com/Netflix/gradle-template/blob/master/.gitignore) which covers pretty much everything

If you need, at any time to look at the available tasks on your project you can use the tasks task.
Here's a sample usage with the output.


***List gradle tasks***
```bash
+> ./gradlew tasks
Inferred project: semantic-versioning, version: 0.0.2-SNAPSHOT
:tasks

------------------------------------------------------------
All tasks runnable from root project - Semantic versioning
------------------------------------------------------------

Build tasks
-----------
assemble - Assembles the outputs of this project.
build - Assembles and tests this project.
buildDependents - Assembles and tests this project and all projects that depend on it.
buildNeeded - Assembles and tests this project and all projects it depends on.
classes - Assembles classes 'main'.
clean - Deletes the build directory.
jar - Assembles a jar archive containing the main classes.
testClasses - Assembles classes 'test'.

Build Setup tasks
-----------------
init - Initializes a new Gradle build. [incubating]

Documentation tasks
-------------------
asciidoctor - Converts AsciiDoc files and copies the output files and related resources to the build directory.
javadoc - Generates Javadoc API documentation for the main source code.

Help tasks
----------
components - Displays the components produced by root project 'semantic-versioning'. [incubating]
dependencies - Displays all dependencies declared in root project 'semantic-versioning'.
dependencyInsight - Displays the insight into a specific dependency in root project 'semantic-versioning'.
help - Displays a help message.
model - Displays the configuration model of root project 'semantic-versioning'. [incubating]
projects - Displays the sub-projects of root project 'semantic-versioning'.
properties - Displays the properties of root project 'semantic-versioning'.
tasks - Displays the tasks runnable from root project 'semantic-versioning'.

JRuby tasks
-----------
jrubyGenerateGradleRb - Generate a gradle.rb stub for executing Ruby binstubs
jrubyPrepare - Pre-cache and prepare all dependencies (jars and gems)
jrubyPrepareGems - Prepare the gems from the `gem` dependencies, extracts into jruby.installGemDir
jrubyPrepareJars - Prepare the jar dependencies from the `gem` dependencies, collect them into

Nebula Release tasks
--------------------
candidate
devSnapshot
final
releaseCheck
snapshot

Upload tasks
------------
uploadArchives - Uploads all artifacts belonging to configuration ':archives'

Verification tasks
------------------
check - Runs all checks.
test - Runs the unit tests.

Other tasks
-----------
generateHtml5
install - Installs the 'archives' artifacts into the local Maven repository.
release - Releases this project.
wrapper

Rules
-----
Pattern: clean<TaskName>: Cleans the output files of a task.
Pattern: build<ConfigurationName>: Assembles the artifacts of a configuration.
Pattern: upload<ConfigurationName>: Assembles and uploads the artifacts belonging to a configuration.

To see all tasks and more detail, run gradlew tasks --all

To see more detail about a task, run gradlew help --task <task>

BUILD SUCCESSFUL
```

### Adding javadoc and source jars

There is nothing simpler than that when using Nebula tools (provided by NetFlix).
Nebula consists of a set of plugins that are opinionated with the Netflix style, but should work really well for you when you follow their standard approach

***build.gradle***
```gradle
plugins {
    id 'nebula.project' version '3.2.1' // <1>
}

apply plugin: 'nebula.project' // <2>

<1> This instructs gradle to use the nebula project plugin
<2> Applies the nebula project plugin
```

The nebula project plugin provides the following

* Builds Javadoc and Sources jars
* Record information about the build and stores it in the .jar, via gradle-info-plugin
* Easy specification of people involved in a project via gradle-contacts-plugin
* Doesn't fail javadoc if there are none found

If you run gradlew tasks, you can now see the additional tasks in the build category:

***Newly added tasks***
```
Build tasks
-----------
assemble - Assembles the outputs of this project.
build - Assembles and tests this project.
buildDependents - Assembles and tests this project and all projects that depend on it.
buildNeeded - Assembles and tests this project and all projects it depends on.
classes - Assembles classes 'main'.
clean - Deletes the build directory.
jar - Assembles a jar archive containing the main classes.
javadocJar // <1>
sourceJar // <2>
testClasses - Assembles classes 'test'.

<1> The java doc jar tasks which creates the artifact-javadoc.jar
<2> The source jar tasks which creates the artifact-sources.jar
```

Those new artifacts can be built by invoking gradlew javadocJar sourceJar build

### Adding some cool documentation

Asciidoctor is a really useful tool when it comes to technical documentation in all its forms: Tutorials, User guides, HowTos, FAQs etc..
It is yet another markdown format, enriched by many plugins that you are encouraged to explore, such as diagrams, math equations, tree view etc..
Many of these asciidoctor extensions are used to generate the documentation you are actually reading

Gradle integrates nicely with Asciidoctor with the asciidoctor-gradle-plugin.
Here's a snippet on how to use it..

***build.gradle***
```gradle

plugins {
    id 'org.asciidoctor.convert' version '1.5.2' // <1>
    id 'com.github.jruby-gradle.base' version '0.3.0'
}

dependencies {
    gems 'rubygems:asciidoctor-diagram:1.3.0' // <2>
    asciidoctor 'com.github.allati.asciidoctor.monotree:asciidoctor-extension-monotree:0.0.1'
    asciidoctor 'org.asciidoctor:asciidoctorj-pdf:1.5.0-alpha.8'
}

apply plugin: 'com.github.jruby-gradle.base' // <3>
apply plugin: 'org.asciidoctor.convert'

class AsciiDoctorDefault extends org.asciidoctor.gradle.AsciidoctorTask { // <4>

    AsciiDoctorDefault() {

        resources {
            from(sourceDir) {
                include 'css/**'
                include 'images/**'
            }
        }

        requires 'asciidoctor-diagram'

        attributes 'build-gradle': new File('build.gradle'),
                'source-highlighter': 'highlightjs',
                'highlightjs-theme': 'github',
                'sourceDir': '../../main/java',
                'rootDir': '../../../',
                'imagesdir': 'images',
                'imagesoutdir': 'images',
                'setanchors': 'true',
                'idprefix': '',
                'idseparator': '-',
                'docinfo1': 'true',
                'docVersion': project.version.toString()
    }

}

task(generateHtml5, type: AsciiDoctorDefault) { // <5>
    dependsOn jrubyPrepareGems
    gemPath jrubyPrepareGems.outputDir
    backends 'html5'
}

task(generatePdf, type: AsciiDoctorDefault) { // <6>
    dependsOn jrubyPrepareGems
    gemPath jrubyPrepareGems.outputDir
    backends 'pdf'
}

<1> Add plugins asciidoctor and jruby (to allow using asciidoctor extensions written in ruby on the gradle jvm)
<2> Add some asciidoctor extensions, such as diagrams and monotree
<3> Apply the plugins
<4> Define a base task extending the asciidoctor provided tasks to fill in some defaults
<5> Add a task to generate documentation with html5 backend
<6> Add a task to generate documentation with pdf backend
```

### Publishing the artifacts

One of the useful things nebula project plugin does, is add publishing support to maven repositories.

***build.gradle***
```gradle

publishing { // <1>

    repositories { // <2>
        maven {
            url "http://${nexusHost}/content/repositories/${nexusRepo}-${project.version.toString().endsWith('-SNAPSHOT') ? 'snapshots' : 'releases'}" // <3>
            
            credentials {
                username "${nexusUser}"
                password "${nexusPassword}"
            }
        }
        mavenLocal()
    }
}

<1> Configure the publishing (gradle maven-publish-plugin)
<2> Configure some maven repositories for publishing
<3> Add a nexus repository hosted on myserver, suffixed by {versiontype}, with versiontype being either empty or "-SNAPSHOT" (more on that on the nebula section)
```
### Semantic versioning

Now comes the real question on how to version our project, and how to define its release cycle.
Again nebula comes to the rescue by providing the Netflix biased way via the nebula release plugin, which should be enough for whatever needs you have.

The way it works is really simple, especially compared to the traditional maven way with project versions cluttering in the pom.xml files everywhere, the maven-release-plugin
which has to modify poms, commit them, push the changes.. but when it doesn't work you must rollback.. what would you configure on a CI environment.. You see the point..

Having the version tag in the sources is redundant with the SCM information. The version can be infered from the SCM commit (by traversing the tree).
With this in mind, a release is simply a tag on a specific commit that marks the state of the versioned code at a certain commit (or point in time).

Here's what you need to do in order to use nebula release plugin

***build.gradle***
```gradle

plugins {
    id 'nebula.nebula-release' version '4.0.1' // <1>
}

apply plugin: 'nebula.nebula-release' // <2>

release { // <3>
    // Let the default versioning strategy be the maven style -SNAPSHOT, instead of nebula's devSnapshot
    defaultVersionStrategy = org.ajoberstar.gradle.git.release.opinion.Strategies.SNAPSHOT
}

tasks.release.finalizedBy tasks.publish // <4>

<1> Add the nebula release plugin to the scope of our project
<2> Apply the plugin
<3> Configure the plugin in order to use -SNAPSHOT suffixes for snapshot versions to stick to the maven convention
<4> Automatically publish the artifacts whenever a "release" happpens
```

In nebula's terms, a release can be one of the below

***View nebula release tasks***
```bash
+> ./gradlew tasks
Nebula Release tasks
--------------------
candidate // <1>
devSnapshot
final // <2>
releaseCheck
snapshot // <3>

<1> Releasing a release candidate as a preview version of our project
<2> Releasing a version of our project
<3> "Releasing" a snapshot version of our project
```

snapshot and devSnapshot tasks will not tag the repository. And with the finalize task we have, it will simply publish the latest snapshot to the repository.
candidate and final tasks are a bit different, where they require no uncommited changes, and then tag the current commit with target release version

To bootstrap this whole thing, all you need to do is tag the very first version manually using for example

```git
git tag "v.0.0.1" // tags the current branch tip to have the 0.0.1 version.
```

From here on, whenever you need to bump the version (i.e. release your product), you can use one of the tasks the nebula release plugin provides.

As a concrete example, here we simulate a very basic flow:


***Sample flow: Making some changes***
```bash
+> touch bla
+> git add bla
+> git commit -m "my bla change"
```

Now some user really needs our bla changes, so we will make a release of our project!

***Sample flow: The release candidate***
```bash
+> gradlew candidate
Inferred project: semantic-versioning, version: 0.1.0-rc.1
Unable to determine the host name on this Windows instance
:releaseCheck
:candidate
:prepare
:compileJava
:processResources UP-TO-DATE
:classes
:writeManifestProperties
:jar
:assemble
:compileTestJava UP-TO-DATE
:processTestResources UP-TO-DATE
:testClasses UP-TO-DATE
:test UP-TO-DATE
:check UP-TO-DATE
:build
:release
Tagging repository as v0.1.0-rc.1
Pushing changes in [refs/heads/master, v0.1.0-rc.1] to origin
:generatePomFileForNebulaPublication
:javadoc
:javadocJar
:sourceJar
:publishNebulaPublicationToMavenLocalRepository
:publishNebulaPublicationToMavenRepository
Upload http://myserver/content/repositories/myrepo-releases/com/acme/example/semantic-versioning/0.1.0-rc.1/semantic-versioning-0.1.0-rc.1.jar
...
```

What happened here? Well, simply the following

1. The project is built (compiled, packaged with sources and javadoc etc..)
2. The repository is tagged with v0.1.0-rc.1 and the tag is pushed to the remote git repository
3. The artifacts (with the new RC version) are published to the configured maven repository

Now suppose the candidate is ready to be released, we can do it using

***Sample flow: The official release***
```bash
+> gradlew final
Inferred project: semantic-versioning, version: 0.1.0
Unable to determine the host name on this Windows instance
:releaseCheck
:final
:prepare
:compileJava UP-TO-DATE
:processResources UP-TO-DATE
:classes UP-TO-DATE
:writeManifestProperties
:jar
:assemble
:compileTestJava UP-TO-DATE
:processTestResources UP-TO-DATE
:testClasses UP-TO-DATE
:test UP-TO-DATE
:check UP-TO-DATE
:build
:release
Tagging repository as v0.1.0
Pushing changes in [refs/heads/master, v0.1.0] to origin
:generatePomFileForNebulaPublication
:javadoc
:javadocJar
:sourceJar
:publishNebulaPublicationToMavenLocalRepository
:publishNebulaPublicationToMavenRepository
...
```

Again, the same operations occur, this time tagging the scm repository with v0.1.0 and publishing the artifacts
The next development version will be 0.1.1-SNAPSHOT

***Viewing the next dev version***
```bash
+> gradlew tasks
Inferred project: semantic-versioning, version: 0.1.1-SNAPSHOT
```
You can always manually bump the version, or force it when release using respectively

```bash```
gradlew <snapshot|devSnapshot|candidate|final> -Prelease.scope#<major|minor|patch> // <1>
gradlew <candidate|final> -Prelease.version#1.2.3 // <2>

<1> Choose the release scope of the operation by specifying which part of the version to bump
<2> Force a release version instead of nebula's scheme
```

### The CI part

Here we assume you have a jenkins server running somewhere, with the "new" pipelines scripts.
We will be creating 2 jobs, 1 for building the project, another to handle the release part.

We will create a jenkins pipeline, and the Jenkinsfile describing the build steps is in the repo under ./Jenkinsfile
Here's the content

***Jenkinsfile***
```groovy

node('master') { // <1>
    
   // Mark the code checkout 'stage'....
   stage 'Checkout'

   // Get the code from the Stash repository
   checkout scm // <2>

   // Mark the code build 'stage'....
   stage 'Build'
   
   // Make sure script is runnable
   sh "chmod a+x ./gradlew"
   
   // Run gradle build and deploy // <3>
   wrap([$class: 'ConfigFileBuildWrapper', managedFiles: [[fileId: 'org.jenkinsci.plugins.configfiles.custom.CustomConfig1453803770046', replaceTokens: false, targetLocation: 'gradle.properties', variable: '']]]) {
    sh "./gradlew snapshot generateHtml5 generatePdf --stacktrace"
   }

   publishHTML(target: [allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'build/asciidoc/html5', reportFiles: 'semantic-versioning.html', reportName: 'Documentation']) // <4>

}

<1> Allocate the node master for the build
<2> Checkout the git repo (no repo is specified because it is infered by jenkins when using Jenkinsfile)
<3> Run the snapshot, generateHtml5 and generatePdf tasks, after injecting the gradle.properties from a configured file
<4> Publish the html5 documentation on jenkins
```

## Putting it all together

At this point, we have done the following:

* Created a java project using gradle
* Generated javadoc and a user guide
* Played around with versioning and release cycle
* Integrated with Jenkins pipelines

All those parts are essential (but not sufficient) for any OSS project, of whatever size.
This tutorial focused on the technical aspects, but there are more to it:

* Licensing
* Contributor guide
* ..

You can have more on this on the Netflix nebula page [here](https://nebula-plugins.github.io/).

Thank you for making it this far, and hope this helps you in some way!
