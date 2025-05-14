# mvn-refactoring
étude

[https://books.sonatype.com/mvnref-book/reference/index.html](https://books.sonatype.com/mvnref-book/reference/index.html)

## périmètre

Refacto maven = refacto du build.

Pas de refacto du runtime, i.e. pas de migration vers des fonctionnalités de type OSGi. (pas de gestion "runtime lifecycle of the components")

## principes

### separate versioning of common concerns

"By using a parent POM as a separate Maven project we greatly decrease the complexity of our child projects and provide separate versioning of common concerns"

[https://medium.com/eonian-technologies/maven-for-pipelining-part-1-8b850d10a7ee](https://medium.com/eonian-technologies/maven-for-pipelining-part-1-8b850d10a7ee)

### build+version decoupling

? pouvoir builder chaque jar séparément, puis le war/ear

### composition over inheritance

Contrainte : un pom ne peut avoir qu'un seul parent (et grand-parent...).

Découpler la problématique des dépendences ?

## moyens

### modules maven

[https://www.baeldung.com/maven-multi-module#benefits-of-using-multi-modules](https://www.baeldung.com/maven-multi-module#benefits-of-using-multi-modules)

Mais inconvénient ?. "Maven modules often sit in the same source tree as the main application. Using maven modules outside the source tree actually is more complicated than it`s really worth."
Sont dans le même repo, partagent la même version... On rebuild tout en même temps.

### parent pom

"a benefit of using a parent POM is that we can set up dependency management for all child projects. For example, if you want all your child projects to use the same version of a logging framework, you can lock to that version in the parent POM and the child project will inherit the version in its dependency section"

#### dependencyManagement

"Look at how the jar-parent-pom sets up its dependencyManagement section to lock versions of slf4j, Logback, and Logback Contributions. These frameworks have multiple JARs. A child project could depend on any combination of them. By using the dependencyManagement section, we ensure that any direct dependency or transitive dependency on any JAR in the bill of materials will use our specified version. These managed dependencies are also inherited by the war-parant-pom."

#### pluginManagement

"we can ensure that we always see compiler warnings by configuring the maven-compiler-plugin in the pluginManagement section. This plugin is bound by default to the compile phase of the Maven Lifecycle. By configuring it here we ensure that every child project (jar or war) inherits this configuration."

#### profiles

"Our parent POMs also group properties, pluginManagement, and plugins, into profiles. Many of these profiles are activated by default and allow us to add some really useful functionality to child projects without them even knowing"

#### résolution de dépendences transitives + croisées par maven

[https://reflectoring.io/maven-bom/](https://reflectoring.io/maven-bom/)

### BOM

Composition over inheritance !?

[https://reflectoring.io/maven-bom/](https://reflectoring.io/maven-bom/)

"This file can be used in our projects in two different ways:
- as a parent POM, or
- as a dependency."

--> Composition over inheritance

"Maven will behave exactly like the example with the parent BOM file in terms of dependency resolution. The only thing that differs is how the BOM file is imported.

The import scope set in the dependency section indicates that this dependency should be replaced with all effective dependencies declared in its POM. In other words, the list of dependencies of our BOM file will take the place of the BOM import in the POM file."

```
<dependencyManagement>
  <dependencies>
    ...
    <dependency>
      <groupId>org.jboss.bom</groupId>
      <artifactId>eap6-supported-artifacts</artifactId>
      <version>6.4.0.GA</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
    ...
  </dependencies>
</dependencyManagement>
```

### pipelining

pas une solution à prioriser. Très impactant.

[https://medium.com/eonian-technologies/maven-for-pipelining-part-1-8b850d10a7ee](https://medium.com/eonian-technologies/maven-for-pipelining-part-1-8b850d10a7ee)

## stratégie de test

Comparaison des livrables (buildés), avant/après refacto. Les livrables doivent être identiques aux métadonnées près.

