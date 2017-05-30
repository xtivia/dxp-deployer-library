### Liferay DXP Deployer Library

This is a simple extraction/adaption of the code from the DXP tools (e.g., blade) developed by Liferay for programmatic control of the deployment of modules (JARs) into an OSGi container using the built-in GoGo shell. This project extracts the key code component associated with the deployment operations and wraps it with an easy-to-use Java API.

Since this library's build results in a single JAR output it can be used for any use case where such programmatic deployment is applicable, but the most notable and perhaps most useful scenario is associated with deployment of JAR files created during Gradle-based builds.

In particular, for any situation where one has developed a custom Gradle build script and needs to manage deployments into the OSGi container from the build script itself this library will be useful. See below for some examples of how to install and use this library in your own Gradle build scripts.

### Installation

To use this library simple download the JAR file from the build/libs directory of this repository (or clone it and build it locally if you prefer). Then include this single JAR in your Java classpath and use the API described below.

### API

The library exposes a single public class, named *Deployer* (in the default package, by design, to make it easier and more concise to use inside tools or build scripts). To create a new instance simply use the constructor to create a new object:

```java
Deployer deployer = new Deployer(11310);
```

Note that the usage of the constructor that accepts a port number for the GoGo shell is optional -- you can use the default constructor if the the default port of 11311 is being used. In all cases the Deployer defaults to using localhost as the target host of the operation.

Once you have constructed a Deployer object as above then simple invoke its *deploy()* method to install a JAR into the OSGi environment. The library will examine the JAR to determine the OSGi bundle ID and then perform a telnet-based operation with GoGo to install the JAR/bundle.

```java
deployer.deploy([path to jar file],true);
```

The *deploy()* method accepts an absolute path to the JAR file to be installed/updated along with a boolean parameter indicated whether or not the bundle should be "started" after installation. A value of *true* is typically supplied here, but for special cases (e.g., OSGi fragments) you may need to instead provide a value of *false*.

There are a couple of things to note about the operation of the *deploy()* method:

1. It handles both installation and update cases inside the same method, i.e. the logic originally written by Liferay that is used here is intelligent enough to determine whether or not the module has been previously installed and will do an OSGi install or update operation as appropriate.
2. The module will be installed/updated "in place", i.e. the OSGi container will refer to it and use it from the path of the supplied JAR location. This means that some thought should be given to the location from which the installation is performed, and should not be in a temporary location that is likely to be deleted.

If for some reason you later determine that you need to uninstall the module/JAR simple invoke the uninstall() method of the library as below:

```java
deployer.uninstall([path to jar file]);
```

And finally a third method is provided in the API for cases where you need to refresh a previously deployed module. This can be the case, for example, when OSGi fragments have been attached to the subject module and wish to execute an OSGi refresh operation so that all classpath dependencies are resolved:

```
deployer.refresh([path to jar file]);
```

### Gradle Usage

This library is especially handy when used in Gradle build scripts, especially as a development aid for rapid installs and updates of the OSGi module under development. It becomes as simple as executing a Gradle task from within your IDE to cause your module to be updated inside the OSGi container.

A simple use case involves the following steps:

1. Configure your Gradle script to use the library. Create a directory named buildLibs at the top level of your Gradle project, drop the dxp-deployer.jar into it, and then add the following code to the top of your build.gradle:

```java
buildscript {
    dependencies {
       classpath files('buildLibs/dxp-deployer.jar')
    }
}
```

2. Add a task to your build file to execute a deployment:

```java
task deploy(dependsOn: 'jar') {
    doLast{
        new Deployer().deploy(jar.archivePath,true)
    }
}
```

Now to deploy your module into the OSGi container, simply run "gradle deploy". This will trigger compilation as necessary, and then will install/update the created JAR into the OSGi container.  Also note this kind of usage of the deployment operation provided by the library can also be useful in "gradle watch" type scenarios where you wish to have the code be automatically re-deployed to the OSGi container after a code change.
