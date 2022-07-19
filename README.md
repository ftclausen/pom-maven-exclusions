# Gradle replication project for adding to maven pom

## Background

We are migrating from the deprecated Gradle "maven" plugin to the "maven-publish" plugin. The main impediment being we
used to use `conf2ScopeMappings` to add dependencies and exclusions to the POM. The two issues are 1) the exclusions
seem no longer available and 2) we can't add to the `test` Maven scope. I should also add that these dependencies and
exclusions are being added to a custom dependency configuration.

## POM example

Location: `build/publications/derfPub/pom-default.xml`

This project replicates an issue I am having to add arbitrary dependencies to the generated Maven POM and also include
transitive exclusions. For example, I want to generate this kind of POM:

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <groupId>example</groupId>
  <artifactId>maven-pom-exclusions</artifactId>
  <version>1.0-SNAPSHOT</version>
  <dependencies>
    <dependency>
      <groupId>com.google.guava</groupId>
      <artifactId>guava</artifactId>
      <version>28.0-jre</version>
      <scope>runtime</scope>
      <exclusions>
        <exclusion>
          <artifactId>module1</artifactId>
          <groupId>com.example</groupId>
        </exclusion>
      </exclusions>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.13.3</version>
      <scope>test</scope>
      <exclusions>
        <exclusion>
          <artifactId>module1</artifactId>
          <groupId>com.example</groupId>
        </exclusion>
      </exclusions>
    </dependency>
  </dependencies>
</project>
```

But, instead, the `jackson.core` one (just being used as an example) is being generated without the `exclusions`
section when using "Next attempt" method described below:

```
<!-- rest of POM omitted - same as above -->
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.13.3</version>
      <scope>test</scope>
      <!-- No exclusions -->
    </dependency>
  </dependencies>
</project>
```

# First attempt - using AdhocComponentWithVariants

Replicate with:

```
./gradlew generatePomFileForDerfPubPublication -P useMapToMavenScope
```

My initial attempt uses the AdhocComponentWithVariants but that gives the below error

```
* Where:
Build file '/Users/fclausen/work/freddiegit/maven-pom-exclusions/build.gradle' line: 108

* What went wrong:
Execution failed for task ':generatePomFileForDerfPubPublication'.
> Invalid Maven scope 'test'. You must choose between 'compile' and 'runtime'
```

See build.gradle for how it is done. It appears that maven-publishing plugin does not let you use the `test` Maven scope by design whereas the old
"maven" plugin allowed you to do this via conf2ScopeMappings.

# Next attempt - roll my own via pom.withXml

Replicate with:

```
./gradlew generatePomFileForDerfPubPublication
```

This almost works except the `ModuleDependency#getExcludeRules` returns nothing for the exclusions. So no exclusions are
added to the POM. See build.gradle for full details.

