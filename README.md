# Usage
1. Add `apply from: "https://raw.githubusercontent.com/eygraber/bintray-gradle-scripts/master/bintray.gradle"` to the bottom of the `build.gradle` of each module you want to deploy to bintray
2. Either in your root project's buildscript depedencies, or in all of your deployable modules' buildscript dependencies, add:
```
classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.7.3'
classpath 'com.github.dcendents:android-maven-gradle-plugin:1.5'
classpath 'org.jfrog.buildinfo:build-info-extractor-gradle:4.4.12'
```

See [example](#example)

# Uploading artifacts
Assuming we want to upload artifacts from a module called library

## Snapshots
`./gradlew library:artifactoryPublish` 

## Releases
`./gradlew library:bintrayUpload`

## Convenience

Two tasks are added; `deployToBintraySnapshot` and `deployToBintray`. Running these are the equivalent of:

```
// deployToBintraySnapshot
./gradlew library:clean library:build library:artifactoryPublish

// deployToBintray
./gradlew library:clean library:build library:bintrayUpload
```

The tasks are added to the project (will which run the task in each module that defines it), and each module (so you can choose which module to run it on if you don't want it to run on all of them (e.g. `./gradle module:deployToBintray`).

# Properties
The following properties are required to be set for the module that is being deployed

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
* USE_DOKKA - whether the javadoc tasks will use dokka (false by default)

The following properties are only necessary if the bintray repo and package have not been set up yet
* LICENSE - will be empty by default
* LICENSE_URL - will be empty by default
* VCS_URL - a URL pointing to version control (usually just the github root of the project), will be empty by default

## Setting Properties

There are 3 ways you can set these properties (listed in the order they are processed):

1. Through command line properties (`./gradlew deployToBintray -PBINTRAY_USER=<user>`
2. In a `bintray_local_properties.gradle` file at the root of your module
3. In a `bintray_properties.gradle` file at the root of your module
4. In a `bintray_local_properties.gradle` file at the root of your project
5. In a `bintray_properties.gradle` file at the root of your project

**Note:** `bintray_local_properties.gradle` should be added to your .gitignore

This allows for maximum configurability. Let's say you have a repo that has many forks. You want to keep the main project's properties checked in to `bintray_properties.gradle`, but want to make it easy for forks to deploy their own artifacts without having to change that file.

This can be achieved by the forks by using command line properties or `bintray_properties_local.gradle`, because they are processed before `bintray_properties.gradle`.

Because of this, the suggested way of definig properties in `bintray_properties.gradle` and `bintray_properties_local.gradle` is:

`if(!project.hasProperty('<property>')) project.ext.<property> = <value>`

This avoids overwriting properties that have already been set using a method that was processed before the current one.

# Example

Let's use [recycler-view-utils](https://github.com/eygraber/recycler-view-utils) as an example.

* It adds the buildscript dependencies in the project's [`build.gradle`](https://github.com/eygraber/recycler-view-utils/blob/master/build.gradle#L7-L9)
* It adds `bintray_local_properties.gradle` to the [`.gitignore`](https://github.com/eygraber/recycler-view-utils/blob/master/.gitignore#L19)
* It adds a [`bintray_properties.gradle`](https://github.com/eygraber/recycler-view-utils/blob/master/bintray_properties.gradle) to the root of the project containing properties that are common to the project
* It applies this script at the bottom of the adapters [`build.gradle`](https://github.com/eygraber/recycler-view-utils/blob/master/adapters/build.gradle#L38) and click [`build.gradle`](https://github.com/eygraber/recycler-view-utils/blob/master/click/build.gradle#L38)
* It adds a `bintray_properties.gradle` to [adapters](https://github.com/eygraber/recycler-view-utils/blob/master/adapters/bintray_properties.gradle) and [click](https://github.com/eygraber/recycler-view-utils/blob/master/click/bintray_properties.gradle) containing properties that are specific to those modules

You may have noticed that `BINTRAY_KEY` is not set in any of the `bintray_properties.gradle` files. That's because we don't want to add our Bintray API key to a public repo. Since command line properties and `bintray_local_properties.gradle` are not checked into source control, we can put it there.

Now what if someone forks `recycler-view-utils` and wants to use a different versioning system for the click module? They can change the value in `click/bintray_properties.gradle`, but then they will have to resolve merge conflicts every time the version changes upstream, and if they want to merge their changes upstream they'll have to maintain the versioning in a separate branch that gets rebased onto feature branches. Sounds like a mess.

What they could do is create `click/bintray_local_properties.gradle` and add:

`if(!project.hasProperty('ARTIFACT_VERSION')) project.ext.ARTIFACT_VERSION = '0.custom.version`

Because `click/bintray_local_properties.gradle` is processed before `click/bintray_properties.gradle` it will use `0.custom.version` as the `ARTIFACT_VERSION`.
