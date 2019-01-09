[<img src="http://sling.apache.org/res/logos/sling.png"/>](http://sling.apache.org)

 [![Build Status](https://builds.apache.org/buildStatus/icon?job=sling-org-apache-sling-feature-1.8)](https://builds.apache.org/view/S-Z/view/Sling/job/sling-org-apache-sling-feature-1.8) [![Test Status](https://img.shields.io/jenkins/t/https/builds.apache.org/view/S-Z/view/Sling/job/sling-org-apache-sling-feature-1.8.svg)](https://builds.apache.org/view/S-Z/view/Sling/job/sling-org-apache-sling-feature-1.8/test_results_analyzer/) [![Maven Central](https://maven-badges.herokuapp.com/maven-central/org.apache.sling/org.apache.sling.feature/badge.svg)](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22org.apache.sling%22%20a%3A%22org.apache.sling.feature%22) [![JavaDocs](https://www.javadoc.io/badge/org.apache.sling/org.apache.sling.feature.svg)](https://www.javadoc.io/doc/org.apache.sling/org.apache.sling.feature) [![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)

# The Sling OSGi Feature Model

OSGi is a platform capable of running large applications for a variety of purposes, including rich client applications, 
server-side systems and cloud and container based architectures. 

As these applications are generally based on many bundles, describing each bundle individually in the application 
definition becomes unwieldy once the number of bundles reaches a certain level. 

Addinally, OSGi has no mechanism to describe other elements of the application definition, such as configuration or custom artifacts.

The Sling OSGi Feature Model introduces a higher level to describe OSGi applications that encapsulates the details of the various 
components that the application is built up from. It allows the description of an entire OSGi-based application based on reusable 
components and includes everything related to this application, including bundles, configuration, framework properties, capabilities, 
requirements and custom artifacts.

# Features

Features are the central concept of the Feature Model. Features are typically defined in a Feature file which is
a JSON file. An example feature file can be found here: https://github.com/apache/sling-org-apache-sling-feature-io/blob/master/design/feature-model.json
and the JSON Schema for feature files is available from here: https://github.com/apache/sling-org-apache-sling-feature-io/blob/master/src/main/resources/META-INF/feature/Feature-1.0.0.schema.json

All features have a unique identity.
Features can have an optional `title`, `description`, `vendor` name, `license`.

Features typically reference one or more bundles that declare the behaviour provided by the feature. These
bundles may have dependencies on other bundles or capabilities in the feature, e.g. defined by other features or bundles.

Features can define OSGi Configurations that will be provided into the runtime with the feature and features can also declare additional requirements and capabilities over and above the ones coming from the bundles
that are part of the feature.  

Features can be declared _from scratch_ or can use another pre-existing feature as a prototype. In this case
the new feature starts off as a feature identical to its prototype, with some exceptions:

* it does not get the prototype's identity. 
* bundles, configurations and framework properties can be removed from the prototypel, in the `removals` section.
* anything declared in the feature definition of a feature based on a prototype adds or overrides to 
what came from the prototype. 

Features can also be marked as `final` and/or `complete`.
A `final` feature cannot be used as a prototype for another feature. A feature marked as `complete` indicates
that all its dependencies are met by capabilities inside the feature, i.e. it has no external dependencies. 

A Feature Launcher can be used to launch features into a running process with an OSGi Framework. 
The launcher can typically be provided with a number of feature files that should be launched together.
Overrides for variables defined in the feature models can be provided on the launcher commandline. 

Tooling exists to analyze and validate features, and to aggregate and merge multiple features into a single
feature, which can be used to create higher level features from a combination of lower-level ones. Most of 
the tooling is accessible through the slingfeature-maven-plugin: https://github.com/apache/sling-slingfeature-maven-plugin  

The following diagrams show a typical workflow when working with feature files:

<img src="diagrams/Develop.jpg" width="700"/>

Features are authored as JSON Feature Files. 
The slingfeature-maven-plugin provides analyzers and aggregators that check features and can combine them into larger features. The maven plugin can also be used to publish features to a Maven Repository.

<img src="diagrams/RunningSystem.jpg" width="700"/>

To create a running system from a number of feature files, features are selected from a Maven Repository,
they are validated for completeness and optionally additional features are pulled in through the OSGi Resolver 
(not yet implemented). A final system feature has no unresolved dependencies. It is passed to the Feature Launcher
along with optional additional features the provide functionality on top of what is defined in the system feature. 
The Feature Launcher creates a running process containing an OSGi Framework provisioned with the feature's contents.
  
## Feature Identity

A feature has a unique id. Maven coordinates (https://maven.apache.org/pom.html#Maven_Coordinates) provide a well defined and accepted way of uniquely defining such an id. The coordinates include at least a group id, an artifact id, a version and optionally a type and classifier.

While group id, artifact id, version and the optional classifier can be freely choosen for a feature, the type/packaging is defined as `slingosgifeature`.

## Maven Coordinates

Maven coordinates are used to define the feature id and to refer to artifacts contained in the feature, e.g. bundles, content packages or other features. There are two supported ways to write down such a coordinate:

* Using a colon as a separator for the parts: groupId:artifactId[:type[:classifier]]:version as defined in https://maven.apache.org/pom.html#Maven_Coordinates
* Using a mvn URL: `'mvn:' group-id '/' artifact-id [ '/' [version] [ '/' [type] [ '/' classifier ] ] ] ]`

In some cases only the coordinates are specified as a string in one of the above mentioned formats. In other cases, the artifact is described through a JSON object. In that case, the *id* property holds the coordinates in one of the formats.


## Bundles

A feature normally declares a number of bundles that are contained in the feature. The bundles are not stored inside the feature
but referenced via their Maven Coordinates in the `bundles` section of the feature model.

Individual bundles are either referenced as a string value in the `bundles` array in the feature model, or they can be specified 
as objects in the `bundles` array. In that case the `id` for the bundle must be specified. Additional metadata can also be placed 
here. 

Multiple versions of a bundle with the same group ID and artifact ID are allowed. In this case both must be specified in the 
`bundles` section.

## Configuration

OSGi Configuration Admin configuration is specified in the `configurations` section of the feature model.

The configurations are specified following the format defined by the OSGi Configurator 
specification: https://osgi.org/specification/osgi.cmpn/7.0.0/service.configurator.html 
Variables declared in the Feature Model variables section can be used for late binding of variables, 
they can be specified with the Launcher, or the default from the variables section is used.
Factory configurations can be specified using the named factory syntax, which separates
the factory PID and the name with a tilde '~'.


## Requirements and Capabilities

In order to avoid a concept like "Require-Bundle" a feature does not explicitly declare dependencies to other features. These are declared by the required capabilities, either explicit or implicit. The implicit requirements are calculated by inspecting the contained bundles (and potentially other artifacts like content packages).

Features can also explicitly declare additional requirements the have over and above the ones coming from the bundles. This is done
in the `requirements` section of the Feature Model.

Features can also declare additional capabilities that are provided by the feature in addition to the capabilities provided by 
the bundles. For example a number of bundles might together provide an `osgi.implementation` capability, which is not provided
by any of those bundles individually. The Feature can be used to add this capability to the provided set. 

Additional capabilities are specified in the `capabilities` section of the Feature Model. 

## Prototype

A feature can be defined based on a prototype. This means that the feature is effectively a copy of the feature marked as its
prototype. Everything in the prototype is copied to the new feature, except for its `id`. The new feature will get a new ID.
Then the prototype is processed with regard to the defined elements of the feature itself. 

This processing happens as follows:
* Removal instructions for an include are handled first
* A clash of artifacts (such as bundles) between the prototype and the feature is resolved by picking the version defined last, which
is the one defined by the feature, not its prototype. Artifact clashes are detected based on Maven Coordinates, not on the content
of the artifact. So if a prototype defines a bundle with artifact ID `org.sling:somebundle:1.2.0` and the feature itself declares 
`org.sling:somebundle:1.0.1` in its `bundles` section, the bundle with version `1.0.1` is used, i.e. the definition in the feature
overrides the one coming from the prototype.
* Configurations will be merged by default, later ones potentially overriding newer ones:
  * If the same property is declared in more than one feature, the last one wins - in case of an array value, this requires redeclaring all values (if they are meant to be kept)
* Later framework properties overwrite newer ones.
* Capabilities and requirements are appended - this might result in duplicates, but that doesn't really hurt in practice.
* Extensions are handled in an extension specific way, by default the contents are appended. In the case of extensions of type 
`Artifact` these are handled just like bundles. Extension merge plugins can be configured to perform custom merging.

Prototypes can provide an important concept for manipulating existing features. For example to replace a bundle in an existing feature and deliver this as a modified feature.

# Extensions

TBD

# Launching

A launcher for feature models is available in this project: https://github.com/apache/sling-org-apache-sling-feature-launcher

# Tooling

The primary tooling around the feature model is provided through Maven by the Sling Feature Maven Plugin: https://github.com/apache/sling-slingfeature-maven-plugin
See the readme of the plugin for more information.  

# References

The links below provide additional information regarding the Feature Model.

* [Requirements](requirements.md)
* [File format](https://github.com/apache/sling-org-apache-sling-feature-io/blob/master/design/feature-model.json)

* [Prototype](prototype.md)
* [API Controller Proposal](apicontroller.md)
* [Application Configuration Proposal](appconf.md)
