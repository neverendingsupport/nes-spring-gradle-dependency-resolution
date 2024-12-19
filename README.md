# NES for Spring Gradle Dependency Resolution Troubleshooting
  
Consider this application's following dependency graph prior to using NES versions of Spring. Note in this case the `spring-security-oauth2` transitive dependencies.

```shell
# Others omitted for brevity
+--- org.springframework.security.oauth:spring-security-oauth2:2.5.2.RELEASE
|    +--- org.springframework:spring-beans:4.3.30.RELEASE -> 5.3.39 (*)
|    +--- org.springframework:spring-core:4.3.30.RELEASE -> 5.3.39 (*)
|    +--- org.springframework:spring-context:4.3.30.RELEASE -> 5.3.39 (*)
|    +--- org.springframework:spring-webmvc:4.3.30.RELEASE -> 5.3.39 (*)
|    +--- org.springframework.security:spring-security-core:4.2.20.RELEASE -> 5.8.16 (*)
|    +--- org.springframework.security:spring-security-config:4.2.20.RELEASE -> 5.8.16 (*)
|    +--- org.springframework.security:spring-security-web:4.2.20.RELEASE -> 5.8.16 (*)
|    \--- commons-codec:commons-codec:1.14 -> 1.15
# ...
```

After changing versions to use NES versions, there may be transitive dependencies still pointing to the `org.springframework` group ID. In this case, this will result in multiple copies of the dependency on the classpath.
For example, the following `build.gradle` file has been updated to use NES versions of the following projects:
* Spring Boot
* Spring Framework
* Spring Security
* Spring Security OAuth (legacy project)

```groovy
plugins {
  id "java"
  id "com.herodevs.nes.springframework.boot" version "2.7.18-spring-boot-2.7.20"
  id "io.spring.dependency-management" version "1.0.15.RELEASE"
}

group = "com.herodevs.demo"
version = "0.0.1-SNAPSHOT"

java {
  toolchain {
    languageVersion = JavaLanguageVersion.of(17)
  }
}

ext["spring-framework.version"] = "5.3.39-spring-framework-5.3.44"
ext["spring-security.version"] = "5.8.16-spring-security-5.8.17"

dependencies {
  implementation("com.herodevs.nes.springframework.boot:spring-boot-starter-security")
  implementation("com.herodevs.nes.springframework.boot:spring-boot-starter-web")
  implementation("com.herodevs.nes.springframework.security.oauth:spring-security-oauth2:2.5.2-spring-security-oauth-2.5.3")
  testImplementation("com.herodevs.nes.springframework.boot:spring-boot-starter-test")
  testImplementation("com.herodevs.nes.springframework.security:spring-security-test")
  testRuntimeOnly("org.junit.platform:junit-platform-launcher")
}
```

The resulting dependency graph would look like the following:

```shell
+--- com.herodevs.nes.springframework.boot:spring-boot-starter-security -> 2.7.18-spring-boot-2.7.20
|    +--- com.herodevs.nes.springframework.boot:spring-boot-starter:2.7.18-spring-boot-2.7.20
|    |    +--- com.herodevs.nes.springframework.boot:spring-boot:2.7.18-spring-boot-2.7.20
|    |    |    +--- com.herodevs.nes.springframework:spring-core:5.3.39-spring-framework-5.3.40 -> 5.3.39-spring-framework-5.3.44
|    |    |    |    \--- com.herodevs.nes.springframework:spring-jcl:5.3.39-spring-framework-5.3.44
|    |    |    \--- com.herodevs.nes.springframework:spring-context:5.3.39-spring-framework-5.3.40 -> 5.3.39-spring-framework-5.3.44
|    |    |         +--- com.herodevs.nes.springframework:spring-aop:5.3.39-spring-framework-5.3.44
|    |    |         |    +--- com.herodevs.nes.springframework:spring-beans:5.3.39-spring-framework-5.3.44
|    |    |         |    |    \--- com.herodevs.nes.springframework:spring-core:5.3.39-spring-framework-5.3.44 (*)
|    |    |         |    \--- com.herodevs.nes.springframework:spring-core:5.3.39-spring-framework-5.3.44 (*)
|    |    |         +--- com.herodevs.nes.springframework:spring-beans:5.3.39-spring-framework-5.3.44 (*)
|    |    |         +--- com.herodevs.nes.springframework:spring-core:5.3.39-spring-framework-5.3.44 (*)
|    |    |         \--- com.herodevs.nes.springframework:spring-expression:5.3.39-spring-framework-5.3.44
|    |    |              \--- com.herodevs.nes.springframework:spring-core:5.3.39-spring-framework-5.3.44 (*)
# Others omitted for brevity
|    |    +--- com.herodevs.nes.springframework:spring-core:5.3.39-spring-framework-5.3.40 -> 5.3.39-spring-framework-5.3.44 (*)
|    +--- com.herodevs.nes.springframework:spring-aop:5.3.39-spring-framework-5.3.40 -> 5.3.39-spring-framework-5.3.44 (*)
|    +--- com.herodevs.nes.springframework.security:spring-security-config:5.7.12-spring-security-5.7.13 -> 5.8.16-spring-security-5.8.17
|    |    +--- com.herodevs.nes.springframework.security:spring-security-core:5.8.16-spring-security-5.8.17
|    |    |    +--- com.herodevs.nes.springframework.security:spring-security-crypto:5.8.16-spring-security-5.8.17
# ...
\--- com.herodevs.nes.springframework.security.oauth:spring-security-oauth2:2.5.2-spring-security-oauth-2.5.3
     +--- org.springframework:spring-beans:4.3.30.RELEASE -> 5.3.39 (*)
     +--- org.springframework:spring-core:4.3.30.RELEASE -> 5.3.39 (*)
     +--- org.springframework:spring-context:4.3.30.RELEASE -> 5.3.39 (*)
     +--- org.springframework:spring-webmvc:4.3.30.RELEASE
     |    +--- org.springframework:spring-aop:4.3.30.RELEASE -> 5.3.39 (*)
     |    +--- org.springframework:spring-beans:4.3.30.RELEASE -> 5.3.39 (*)
     |    +--- org.springframework:spring-context:4.3.30.RELEASE -> 5.3.39 (*)
     |    +--- org.springframework:spring-core:4.3.30.RELEASE -> 5.3.39 (*)
     |    +--- org.springframework:spring-expression:4.3.30.RELEASE -> 5.3.39 (*)
     |    \--- org.springframework:spring-web:4.3.30.RELEASE -> 5.3.39 (*)
     +--- org.springframework.security:spring-security-core:4.2.20.RELEASE
     |    +--- aopalliance:aopalliance:1.0
     |    +--- org.springframework:spring-aop:4.3.30.RELEASE -> 5.3.39 (*)
     |    +--- org.springframework:spring-beans:4.3.30.RELEASE -> 5.3.39 (*)
     |    +--- org.springframework:spring-context:4.3.30.RELEASE -> 5.3.39 (*)
     |    +--- org.springframework:spring-core:4.3.30.RELEASE -> 5.3.39 (*)
     |    \--- org.springframework:spring-expression:4.3.30.RELEASE -> 5.3.39 (*)
```
In this case, we have two copies of modules from both the `org.springframework` and `com.herodevs.nes.springframework`. This is also the same for `org.springframework.security` and `com.herodevs.nes.springframework.security`.
To ensure a successful application build that does not have multiple copies of these Spring JARs, the following Gradle features can be utilized. 

## Option 1: Gradle Exclusions

Use Gradle exclusion rules: e.g. `exclude(group: "org.springframework")`.

Example

```groovy
ext["spring-framework.version"] = "5.3.39-spring-framework-5.3.44"
ext["spring-security.version"] = "5.8.16-spring-security-5.8.17"

configurations {
  configureEach {
    exclude(group: "org.springframework")
    exclude(group: "org.springframework.security")
  }
}

dependencies {
  implementation("com.herodevs.nes.springframework.boot:spring-boot-starter-security")
  implementation("com.herodevs.nes.springframework.boot:spring-boot-starter-web")
  implementation("com.herodevs.nes.springframework.security.oauth:spring-security-oauth2:2.5.2-spring-security-oauth-2.5.3")
  testImplementation("com.herodevs.nes.springframework.boot:spring-boot-starter-test")
  testImplementation("com.herodevs.nes.springframework.security:spring-security-test")
  testRuntimeOnly("org.junit.platform:junit-platform-launcher")
}
```
Note that any original transitive dependency from that the application relies on transitively will have to be explicitly added.

## Option 2 Gradle Substitution rules

```groovy
ext["spring-framework.version"] = "5.3.39-spring-framework-5.3.44"
ext["spring-security.version"] = "5.8.16-spring-security-5.8.17"


dependencies {
  implementation("com.herodevs.nes.springframework.boot:spring-boot-starter-security")
  implementation("com.herodevs.nes.springframework.boot:spring-boot-starter-web")
  implementation("com.herodevs.nes.springframework.security.oauth:spring-security-oauth2:2.5.2-spring-security-oauth-2.5.3")
  testImplementation("com.herodevs.nes.springframework.boot:spring-boot-starter-test")
  testImplementation("com.herodevs.nes.springframework.security:spring-security-test")
  testRuntimeOnly("org.junit.platform:junit-platform-launcher")
}
def springFrameworkModules = [
    "spring-aop",
    "spring-aspects",
    "spring-beans",
    "spring-core",
    "spring-test",
    "spring-web"
    //  others from Spring Framework
]

configurations.configureEach {
  resolutionStrategy.dependencySubstitution {
    springFrameworkModules.each { artifact ->
      substitute(module("org.springframework:${artifact}"))
          .using(module("com.herodevs.nes.springframework:${artifact}:${project.ext["spring-framework.version"]}"))
          .because("Replaced with HeroDevs NES")
    }
  }
}

```

This demo also includes a [patch Gradle file](./gradle/nes-dependency-resolution.gradle) using dependency substitution rules for Spring Framework and Spring Security that can be applied to the entire project. 

```groovy
apply from: "gradle/nes-dependency-resolution.gradle"
```

## Testing 

Use Dependency Insight to ensure the exclusion or substitution rules are configured properly.

```shell
./gradlew -q dependencyInsight \
  --configuration runtimeClasspath \
  --dependency org.springframework:spring-core
./gradlew -q dependencyInsight \
  --configuration runtimeClasspath \
  --dependency org.springframework:spring-core

com.herodevs.nes.springframework:spring-core:5.3.39-spring-framework-5.3.44 (selected by rule)
  Variant runtimeElements:
    | Attribute Name                      | Provided     | Requested    |
    |-------------------------------------|--------------|--------------|
    | org.gradle.status                   | release      |              |
    | org.jetbrains.kotlin.localToProject | public       |              |
    | org.jetbrains.kotlin.platform.type  | jvm          |              |
    | org.gradle.category                 | library      | library      |
    | org.gradle.dependency.bundling      | external     | external     |
    | org.gradle.jvm.version              | 8            | 17           |
    | org.gradle.libraryelements          | jar          | jar          |
    | org.gradle.usage                    | java-runtime | java-runtime |
    | org.gradle.jvm.environment          |              | standard-jvm |

org.springframework:spring-core:4.3.30.RELEASE -> com.herodevs.nes.springframework:spring-core:5.3.39-spring-framework-5.3.44
\--- com.herodevs.nes.springframework.security.oauth:spring-security-oauth2:2.5.2-spring-security-oauth-2.5.3
     \--- runtimeClasspath

org.springframework:spring-core:5.3.39 -> com.herodevs.nes.springframework:spring-core:5.3.39-spring-framework-5.3.44
+--- com.herodevs.nes.springframework.security:spring-security-config:5.8.16-spring-security-5.8.17
|    +--- com.herodevs.nes.springframework.boot:spring-boot-starter-security:2.7.18-spring-boot-2.7.20 (requested com.herodevs.nes.springframework.security:spring-security-config:5.7.12-spring-security-5.7.13)
|    |    \--- runtimeClasspath (requested com.herodevs.nes.springframework.boot:spring-boot-starter-security)
|    \--- com.herodevs.nes.springframework.security.oauth:spring-security-oauth2:2.5.2-spring-security-oauth-2.5.3 (requested org.springframework.security:spring-security-config:4.2.20.RELEASE)
|         \--- runtimeClasspath
+--- com.herodevs.nes.springframework.security:spring-security-core:5.8.16-spring-security-5.8.17
|    +--- com.herodevs.nes.springframework.security.oauth:spring-security-oauth2:2.5.2-spring-security-oauth-2.5.3 (requested org.springframework.security:spring-security-core:4.2.20.RELEASE) (*)
|    +--- com.herodevs.nes.springframework.security:spring-security-config:5.8.16-spring-security-5.8.17 (*)
|    \--- com.herodevs.nes.springframework.security:spring-security-web:5.8.16-spring-security-5.8.17
|         +--- com.herodevs.nes.springframework.boot:spring-boot-starter-security:2.7.18-spring-boot-2.7.20 (requested com.herodevs.nes.springframework.security:spring-security-web:5.7.12-spring-security-5.7.13) (*)
|         \--- com.herodevs.nes.springframework.security.oauth:spring-security-oauth2:2.5.2-spring-security-oauth-2.5.3 (requested org.springframework.security:spring-security-web:4.2.20.RELEASE) (*)
\--- com.herodevs.nes.springframework.security:spring-security-web:5.8.16-spring-security-5.8.17 (*)

(*) - Indicates repeated occurrences of a transitive dependency subtree. Gradle expands transitive dependency subtrees only once per project; repeat occurrences only display the root of the subtree, followed by this annotation.

A web-based, searchable dependency report is available by adding the --scan option.
```
