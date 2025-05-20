# mvn-refactoring

## Périmètre de l'étude

Refacto maven = refacto du build.

Pas de refacto du runtime, i.e. pas de migration vers des fonctionnalités de type OSGi. (pas de gestion "runtime lifecycle of the components")

## Exemple de migration...

### Avant

![Sources avant](./docs/mvn-refactoring_1.drawio.png?raw=true)

### Apres

![Sources après](./docs/mvn-refactoring_2.drawio.png?raw=true)

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

```
mvn help:effective-pom
```

### modules maven

[https://www.baeldung.com/maven-multi-module#benefits-of-using-multi-modules](https://www.baeldung.com/maven-multi-module#benefits-of-using-multi-modules)

Mais inconvénient ?. "Maven modules often sit in the same source tree as the main application. Using maven modules outside the source tree actually is more complicated than it`s really worth."
Sont dans le même repo, partagent la même version... On rebuild tout en même temps.

#### modules sans héritage

A noter : on peut avoir des modules, sans relation d'héritage

[https://books.sonatype.com/mvnref-book/reference/pom-relationships-sect-pom-best-practice.html#pom-relationships-sect-multi-vs-inherit](https://books.sonatype.com/mvnref-book/reference/pom-relationships-sect-pom-best-practice.html#pom-relationships-sect-multi-vs-inherit)

[mvnbook-parent](https://github.com/sonatype/maven-guide-en/blob/master/pom.xml)

#### héritage sans modules

Cf (Exemple 6)[https://github.com/avergnaud/mvn-refactoring/tree/main/exemple-6]

#### Exemple

[Exemple 1](https://github.com/avergnaud/mvn-refactoring/tree/main/exemple-1)

[submodule-1/pom.xml](https://github.com/avergnaud/mvn-refactoring/blob/main/exemple-1/projet-chapeau/submodule-1/pom.xml) et [submodule-2/pom.xml](https://github.com/avergnaud/mvn-refactoring/blob/main/exemple-1/projet-chapeau/submodule-2/pom.xml) n'ont PAS comme parent [projet-chapeau/pom.xml](https://github.com/avergnaud/mvn-refactoring/blob/main/exemple-1/projet-chapeau/pom.xml)

### parent pom / héritage

"a benefit of using a parent POM is that we can set up dependency management for all child projects. For example, if you want all your child projects to use the same version of a logging framework, you can lock to that version in the parent POM and the child project will inherit the version in its dependency section"

#### résolution de dépendences transitives + croisées par maven

Maven a sa logique propre pour résoudre un conflit de versions entre deux dépendences.

[https://reflectoring.io/maven-bom/](https://reflectoring.io/maven-bom/)

```
mvn dependency:tree -Dverbose=true
```

- Cas "omitted for duplicate"
- Cas "omitted for conflict with..."

Solutions pour définir/forcer une version :
- définir une dépendence directe
- dans le dependencyManagement ("We should note that defining a dependency in the dependencyManagement section doesn’t add it to the dependency tree of the project, it is used just for lookup reference.")

#### dependencyManagement

- Maven does inherit the <dependencyManagement> section from a parent POM into its children.
- The <dependencyManagement> section is exported from a BOM into its dependent projects.

See (exemple-4/parent/module-a/effective-pom.txt)[https://github.com/avergnaud/mvn-refactoring/blob/main/exemple-4/parent/module-a/effective-pom.txt] and (exemple-5/module-a/effective-pom.txt)[https://github.com/avergnaud/mvn-refactoring/blob/main/exemple-5/module-a/effective-pom.txt]

"Look at how the jar-parent-pom sets up its dependencyManagement section to lock versions of slf4j, Logback, and Logback Contributions. These frameworks have multiple JARs. A child project could depend on any combination of them. By using the dependencyManagement section, we ensure that any direct dependency or transitive dependency on any JAR in the bill of materials will use our specified version. These managed dependencies are also inherited by the war-parant-pom."

#### pluginManagement

- Maven does inherit the <pluginManagement> section from a parent POM into its children.
- The <pluginManagement> section is not exported from a BOM into its dependent projects.

See (exemple-4/parent/module-a/effective-pom.txt)[https://github.com/avergnaud/mvn-refactoring/blob/main/exemple-4/parent/module-a/effective-pom.txt] and (exemple-5/module-a/effective-pom.txt)[https://github.com/avergnaud/mvn-refactoring/blob/main/exemple-5/module-a/effective-pom.txt]

"we can ensure that we always see compiler warnings by configuring the maven-compiler-plugin in the pluginManagement section. This plugin is bound by default to the compile phase of the Maven Lifecycle. By configuring it here we ensure that every child project (jar or war) inherits this configuration."

#### profiles

- By design, Maven does not inherit the <profiles> section from a parent POM into its children. Profiles are resolved early in the model‐building phase and only their effects (activated plugins, dependencies, properties) flow down, not their declarations.
- The <profiles> section is not exported from a BOM into its dependent projects.

See (exemple-4/parent/module-a/effective-pom.txt)[https://github.com/avergnaud/mvn-refactoring/blob/main/exemple-4/parent/module-a/effective-pom.txt] and (exemple-5/module-a/effective-pom.txt)[https://github.com/avergnaud/mvn-refactoring/blob/main/exemple-5/module-a/effective-pom.txt]

"Keep your build‑governance profiles centralized in the parent. Child modules will pick up their effects (e.g. property values, plugin executions) when you activate them with -P, even though help:effective-pom won’t list the <profiles> block itself.

Use help:active-profiles and help:all-profiles to inspect profile activation and availability per module."

### BOM

L'import en mode dependencyManagement importe uniquement la définition/l'exigence de version. Mais la dépendence n'est pas tirée..

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

"We should note that if we use a BOM as a parent for our project, we will no longer be able to declare another parent for our project. This can be a blocking issue if the concerned project is a child module. To bypass this, another way to use the BOM is by dependency." https://reflectoring.io/maven-bom/

#### Exemple

[Exemple 3](https://github.com/avergnaud/mvn-refactoring/tree/main/exemple-3).

[projet-bom/pom.xml](https://github.com/avergnaud/mvn-refactoring/blob/main/exemple-3/projet-bom/pom.xml) n'est PAS un parent de [module-1/pom.xml](https://github.com/avergnaud/mvn-refactoring/blob/main/exemple-3/module-1/pom.xml) ni de [module-2/pom.xml](https://github.com/avergnaud/mvn-refactoring/blob/main/exemple-3/module-2/pom.xml)

Si on veut effectivement tirer la dépendence dans module-1 ou module-2, il faut déclarer la dépendence.

### pipelining

pas une solution à prioriser. Très impactant.

[https://medium.com/eonian-technologies/maven-for-pipelining-part-1-8b850d10a7ee](https://medium.com/eonian-technologies/maven-for-pipelining-part-1-8b850d10a7ee)

## stratégie de test

Comparaison des livrables (buildés), avant/après refacto. Les livrables doivent être identiques aux métadonnées près.

## références

[https://books.sonatype.com/mvnref-book/reference/index.html](https://books.sonatype.com/mvnref-book/reference/index.html)
