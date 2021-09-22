# SoftiCAR Gradle Plugins

The _SoftiCAR Gradle Plugins_ repository is a collection of Gradle plugins used by many SoftiCAR Java projects. It comprises the following plugins:

* SoftiCAR Code Validation Plugin
* SoftiCAR Dependency Validation Publish Plugin
* SoftiCAR Ivy Publish Plugin
* SoftiCAR Legacy Plugin
* SoftiCAR Selenium Grid Plugin
* SoftiCAR Test Logger Plugin

## Individual Plugins

For Gradle multi-projects, a given plugin is either applied to the root project or to the subprojects individually.

### SoftiCAR Code Validation Plugin

The _SoftiCAR Code Validation Plugin_ enables a project to execute self-contained code validation logic, i.e. code validation logic that is implemented in the project itself or in one of its dependencies.

#### Usage

For example, in the _build.gradle_ of the root project write this:
```gradle
plugins {
	// the code validation plugin requires the java library plugin
	id 'com.softicar.gradle.java.library'
	// we usually don't want to apply the plugin to the root project (apply false)
	id 'com.softicar.gradle.code.validation' apply false
}
subprojects {
	apply plugin: 'com.softicar.gradle.java.library'
	apply plugin: 'com.softicar.gradle.code.validation'
	softicarCodeValidationSettings {
		validationEntryPointClass = "com.example.Validator"
		arguments = ["--some-parameter", "12345"]
	}
	check.dependsOn softicarCodeValidation
}
```

### SoftiCAR Dependency Validation Plugin

The _SoftiCAR Dependency Validation Plugin_ is useful to validate manual dependency conflict resolutions, e.g. by failing the build in case of accidential downgrades of dependencies. See the Javadoc of the plugin class for more details.

#### Usage

For example, in the _build.gradle_ of the root project write this:
```gradle
plugins {
	// we usually don't want to apply the plugin to the root project (apply false)
	id 'com.softicar.gradle.dependency.validation' apply false
}
subprojects{
	configurations.all {
		resolutionStrategy {
			failOnVersionConflict()
			force ...
		}
	}
	apply plugin: 'com.softicar.gradle.dependency.validation'
	check.dependsOn softicarDependencyValidation
}
```

### SoftiCAR Ivy Publish Plugin

The _SoftiCAR Ivy Publish Plugin_ provides artifact publication support for Ivy repositories.

* Specifically, it configures an Ivy repository for upload/publishing.
* It adds the tasks _sourcesJar_ and _javadocJar_ producing the respective artifacts.
* And it adds the sources and Javadoc artifacts to the Ivy publication.

#### Usage

For example, in the _build.gradle_ of the root project write this:
```gradle
plugins {
	// the Ivy publish plugin requires the java library plugin
	id 'com.softicar.gradle.java.library'
	// we usually don't want to apply the Ivy publish plugin to the root project (apply false)
	id 'com.softicar.gradle.ivy.publish' apply false
	// apply the release plug-in if you want automatically publish when releasing
	id 'com.softicar.gradle.release'
}
subprojects {
	apply plugin: 'com.softicar.gradle.java.library'
	apply plugin: 'com.softicar.gradle.ivy.publish'

	softicarIvyPublishSettings {
		ivyUploadUrl = 'sftp://some-host:22/some/ivy/repository'
		ivyUploadUsername = 'some-user'
		ivyUploadPassword = 'some-password'
	}

	// to automatically publish when releasing, add the following
	rootProject.afterReleaseBuild.dependsOn publish
}
```

### SoftiCAR Java Library Plugin

The _SoftiCAR Java Library Plugin_ applies the Gradle Java Library plug-in and applies some tweaks:
* It configures the manifest of the generated jar-File.
* It silences some Javadoc compiler diagnostics.

#### Usage

For example, in the _build.gradle_ of the root project write this:
```gradle
plugins {
	id 'com.softicar.gradle.java.library'
}
subprojects {
	apply plugin: 'com.softicar.gradle.java.library'
}
```

### SoftiCAR Release Plugin

The _SoftiCAR Release Plugin_ provides support for creating release tags and commits for a repository.

#### Usage

For example, in the _build.gradle_ of the root project write this:
```gradle
plugins {
	id 'com.softicar.gradle.release'
}
subprojects {
	// to automatically publish the artifacts of all subprojects, you can do this
	rootProject.afterReleaseBuild.dependsOn publish
}
```

To create a release, execute this:
```
./gradlew release
```

### SoftiCAR Selenium Grid Plugin

The _SoftiCAR Selenium Grid Plugin_ is useful for projects executing unit tests based on the [Selenium](https://www.selenium.dev/) framework. See the Javadoc of the plugin class for more details.

#### Usage

To use this plugin, the Gradle daemon must be disabled, e.g. in _gradle.properties_:
```
org.gradle.daemon=false
```

This plugin is applied to the root project directly, e.g. in _build.gradle_:
```gradle
plugins {
	id 'com.softicar.gradle.selenium.grid'
}
```

### SoftiCAR Test Logger Plugin

The _SoftiCAR Test Logger Plugin_ can be used to debug problems (e.g. concerning performance or determinism) during test execution.

#### Usage

For example, in the _build.gradle_ of the root project write this:
```gradle
plugins {
	// the Java plug-in provides the `test` task that the plugin binds to
	id 'com.softicar.gradle.java.library'
	// we usually don't want to apply the plugin to the root project (apply false)
	id 'com.softicar.gradle.test.logger' apply false
}
subprojects {
	apply plugin: 'com.softicar.gradle.java.library'
	apply plugin: 'com.softicar.gradle.test.logger'
}
```

And then execute _Gradle_ with the following parameter:
```
./gradlew -Pcom.softicar.test.logger.enabled=true check
```
