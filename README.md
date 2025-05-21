# mvn-refactoring

Ce document est une étude pour les projets de refonte Maven.

Plan :
- [Périmètre de l'étude](https://github.com/avergnaud/mvn-refactoring?tab=readme-ov-file#p%C3%A9rim%C3%A8tre-de-l%C3%A9tude)
- [Exemple de migration](https://github.com/avergnaud/mvn-refactoring?tab=readme-ov-file#exemple-de-migration)
- [Principes](https://github.com/avergnaud/mvn-refactoring?tab=readme-ov-file#principes)
- [Moyens](https://github.com/avergnaud/mvn-refactoring?tab=readme-ov-file#moyens)
- []()
- []()

## Périmètre de l'étude

On envisage uniquement une refonte ("migration", "refacto") de build Maven.

On ne considère pas les refontes du runtime, i.e. pas de refonte vers des fonctionnalités de type OSGi.

Le livrable produit doit être le même avant et après refacto (aux métadonnées de build près).

## Exemple de migration

### Avant

- Tout le code source est dans le même repo git
- Le pom parent déclare des modules
- Le livrable est buildé dans son ensemble
- Les versions des modules et du parent sont identiques
- Le build du parent est lent (on reconstruit tout le livrable à chaque build)

![Sources avant](./docs/mvn-refactoring_1.drawio.png?raw=true)

### Apres

- On utilise l'héritage (pom parent) pour le pluginManagement et les profiles
- On utilise la composition (bom) pour spécifier les versions de dépendences
- Chaque module peut être dans son propre repo git
- Le pom parent ne déclare pas ses modules. Les "modules" sont déclarés en tant que dépendences
- Chaque module peut être rebuildé séparément.
- Les versions des modules et du parent sont a priori différentes
- Le build du parent est rapide (on ne fait que récupérer les modules en dépendences)

![Sources après](./docs/mvn-refactoring_2.drawio.png?raw=true)

## Principes

### Factorisation des exigences transverses

"Separate versioning of common concerns"

En utilisant:
- un pom parent
- ou un BOM

...comme projet Maven distinct, on réduit la complexité des pom enfants et on obtient un versionnement séparé des exigences transverses.

### Découplage des versions et des builds

On veut pouvoir builder chaque jar (module) séparément, puis le livrable complet (war, ear).

### "Composition" over inheritance

Contrainte : un pom ne peut avoir qu'un seul pom parent (et grand-parent...).

On cherche à découpler au maximum chaque module de son parent, en utilisant des dépendences plutôt que l'héritage.

## Moyens

### Modules maven

Les projets Maven "multi-modules" présentent des avantages:

"a benefit of using a parent POM is that we can set up dependency management for all child projects. For example, if you want all your child projects to use the same version of a logging framework, you can lock to that version in the parent POM and the child project will inherit the version in its dependency section"

[https://www.baeldung.com/maven-multi-module#benefits-of-using-multi-modules](https://www.baeldung.com/maven-multi-module#benefits-of-using-multi-modules)

Les projets Maven "multi-modules" présentent aussi des inconvénients : parfois tous les modules se retrouvent dans le même repo git. Ils partagent tous la même version et sont buildés ensemble.

[Exemple 2](https://github.com/avergnaud/mvn-refactoring/tree/main/exemple-2/projet-chapeau)

#### Modules sans héritage

A noter : on peut avoir des modules, sans relation d'héritage

[https://books.sonatype.com/mvnref-book/reference/pom-relationships-sect-pom-best-practice.html#pom-relationships-sect-multi-vs-inherit](https://books.sonatype.com/mvnref-book/reference/pom-relationships-sect-pom-best-practice.html#pom-relationships-sect-multi-vs-inherit)

[mvnbook-parent](https://github.com/sonatype/maven-guide-en/blob/master/pom.xml)

#### Héritage sans modules

Un pom peut hériter d'un parent, sans que le parent le déclare en tant que module.

Cf [Exemple 6](https://github.com/avergnaud/mvn-refactoring/tree/main/exemple-6).

[Exemple 1](https://github.com/avergnaud/mvn-refactoring/tree/main/exemple-1)

[submodule-1/pom.xml](https://github.com/avergnaud/mvn-refactoring/blob/main/exemple-1/projet-chapeau/submodule-1/pom.xml) et [submodule-2/pom.xml](https://github.com/avergnaud/mvn-refactoring/blob/main/exemple-1/projet-chapeau/submodule-2/pom.xml) n'ont PAS comme parent [projet-chapeau/pom.xml](https://github.com/avergnaud/mvn-refactoring/blob/main/exemple-1/projet-chapeau/pom.xml)

### Résolution de dépendences transitives + croisées par maven

Maven a sa logique propre pour résoudre un conflit de versions entre deux dépendences :

[https://reflectoring.io/maven-bom/](https://reflectoring.io/maven-bom/)

```
mvn dependency:tree -Dverbose=true
```
Cf :
- Cas "omitted for duplicate"
- Cas "omitted for conflict with..."

Solutions pour définir/forcer une version :
- définir une dépendence directe
- dans le dependencyManagement ("We should note that defining a dependency in the dependencyManagement section doesn’t add it to the dependency tree of the project, it is used just for lookup reference.")

### dependencyManagement

- Maven fait hériter la section <dependencyManagement> d’un POM parent dans ses POM enfants.
- La section <dependencyManagement> est exportée depuis un BOM vers les POM qui en dépendent.

Cf [exemple-4/parent/module-a/effective-pom.txt](https://github.com/avergnaud/mvn-refactoring/blob/main/exemple-4/parent/module-a/effective-pom.txt) and [exemple-5/module-a/effective-pom.txt](https://github.com/avergnaud/mvn-refactoring/blob/main/exemple-5/module-a/effective-pom.txt)

"By using the dependencyManagement section, we ensure that any direct dependency or transitive dependency on any JAR in the bill of materials will use our specified version. These managed dependencies are also inherited by the war-parant-pom."

### pluginManagement

- Maven fait hériter la section <pluginManagement> d’un POM parent dans ses POM enfants.
- La section <pluginManagement> n'est PAS exportée depuis un BOM vers les POM qui en dépendent.

Cf [exemple-4/parent/module-a/effective-pom.txt](https://github.com/avergnaud/mvn-refactoring/blob/main/exemple-4/parent/module-a/effective-pom.txt) and [exemple-5/module-a/effective-pom.txt](https://github.com/avergnaud/mvn-refactoring/blob/main/exemple-5/module-a/effective-pom.txt)

"we can ensure that we always see compiler warnings by configuring the maven-compiler-plugin in the pluginManagement section. This plugin is bound by default to the compile phase of the Maven Lifecycle. By configuring it here we ensure that every child project (jar or war) inherits this configuration."

### profiles

- Maven ne fait PAS hériter les <profiles> d’un POM parent dans ses POM enfants. Les profiles sont résolus tôt dans la phase de construction du modèle, et seuls leurs effets (plugins activés, dépendances, propriétés) sont transmis, pas leurs déclarations. 
- La section <profiles> n'est PAS exportée depuis un BOM vers les POM qui en dépendent.

Cf [exemple-4/parent/module-a/effective-pom.txt](https://github.com/avergnaud/mvn-refactoring/blob/main/exemple-4/parent/module-a/effective-pom.txt) and [exemple-5/module-a/effective-pom.txt](https://github.com/avergnaud/mvn-refactoring/blob/main/exemple-5/module-a/effective-pom.txt)

"Keep your build‑governance profiles centralized in the parent. Child modules will pick up their effects (e.g. property values, plugin executions) when you activate them with -P, even though help:effective-pom won’t list the <profiles> block itself.

Use help:active-profiles and help:all-profiles to inspect profile activation and availability per module."

### BOM

La technique de BOM maven consiste à importer en tant que dépendence une liste de versions pour des composants définis.

L'import en mode dependencyManagement importe uniquement la définition/l'exigence des versions. Mais la dépendence n'est pas effectivement définie.

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

### Pipelining

Le pipelining n'est pas une solution à prioriser. Très impactant.

[https://medium.com/eonian-technologies/maven-for-pipelining-part-1-8b850d10a7ee](https://medium.com/eonian-technologies/maven-for-pipelining-part-1-8b850d10a7ee)

## Stratégie de test

Les tests sont des TNR. Un TNR consiste à comparer les livrables (buildés), avant et après refacto. Les livrables doivent être identiques aux métadonnées près.

## Référence

[https://books.sonatype.com/mvnref-book/reference/index.html](https://books.sonatype.com/mvnref-book/reference/index.html)
