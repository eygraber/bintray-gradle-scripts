# Usage
1. Add `classpath 'org.jfrog.buildinfo:build-info-extractor-gradle:4.4.9'` to your root project's buildscript dependencies
2. Add `apply from: "https://raw.githubusercontent.com/eygraber/bintray-gradle-scripts/master/bintray.gradle"` to the bottom of the `build.gradle` of each module you want to deploy to bintray
3. Either in your root project's buildscript depedencies, or in all of your deployable modules' buildscript dependencies, add:
```
classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.7.3'
classpath 'com.github.dcendents:android-maven-gradle-plugin:1.5'
```

# Uploading artifacts
Assuming we want to upload artifacts from a module called library

## Snapshots
`./gradlew library:clean library:build library:artifactoryPublish` 

## Releases
`./gradlew library:clean library:build library:bintrayUpload`

# Properties
The following properties are required to be set on the project for the module that is being deployed

* BINTRAY_USER - the username of the bintray username
* BINTRAY_KEY - the API key of the bintray account
* BINTRAY_REPO
* BINTRAY_PACKAGE
* ARTIFACT_VERSION
* ARTIFACT_NAME

If `ARTIFACT_NAME` is not set, `BINTRAY_PACKAGE` will be used as the name of the artifact

If you are deploying to a repo that is owned by an organization, `BINTRAY_ORG` must be set to that organization's name

The following properties are optional, and will have the default value noted
* BINTRAY_PUBLISH - whether the artifacts should be automatically published (true by default)
* BINTRAY_OVERRIDE_VERSION - whether the artifacts will override an existing artifact with the same version (false by default)
* BINTRAY_DRY_RUN - whether the upload will actually occur or only run through the steps up until the upload (false by defualt)
* DESCRIPTION - will be empty by default

The following properties are only necessary if the bintray repo and package have not been set up yet
* LICENSE - will be empty by default
* LICENSE_URL - will be empty by default
* VCS_URL - a URL pointing to version control (usually just the github root of the project), will be empty by default

There are 3 ways these properties are set (and in the following order):
1. Through command line properties (`./gradlew <tasks...> -PBINTRAY_USER=<user>`
2. In a `bintray_local_properties.gradle` file at the root of your project and/or module
3. In a `bintray_properties.gradle` file at the root of your project and/or module

To set properties using the file based methods, add the following line per property you wish to set in the file:

`if(!project.hasProperty('<property>')) project.ext.<property> = <value>`

This is because properties should not be overridden.

If you don't want to enter your API key through the command line, I suggest adding `bintray_local_properties.gradle` to your `.gitignore`, and putting BINTRAY_KEY in the project root `bintray_local_properties.gradle`.
