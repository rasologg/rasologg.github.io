---
layout: post
title:  "Java : developper avec Maven"
date:   2021-04-16 13:33:29 +0200
categories: blog
---

# Introduction

Maven (plus precisement Apache Maven) est un outil de gestion de dépendances en Java. Mais il permet aussi d'automatiser differentes étapes utiles pendant un developpement logiciel : tests et qualité, compilation, packaging, archivage et documentation.

De manière generale, une application écrite en Java, par exemple un microservice, a besoin d'utiliser une base de données, de lancer un serveur web HTTP ou d'effectuer des calculs, etc....

Pas besoin de réinventer la roue, quelqu'un a déjà fait le travail à notre place et a publié ses projets sur Internet, c'est la magie de l'Open Source.

Pour réutiliser ces composants Open Source dans notre projet, on a besoin d'un outil comme Maven. Il nous aide à résoudre beaucoup de problématiques comme la compatibilité des versions, l'inter-dependance entre composants, etc...

Dans le monde Java, il y a trois principaux outils pour gérer les dependances : 

- [Apache Maven](https://maven.apache.org)
- [Gradle](https://gradle.org/)
- [Apache Ant](https://ant.apache.org/)

Apache Maven reste le plus populaire et le plus utilisé même si Gradle monte en puissance.

Apache Maven a commencé en 2002, très stable, robuste et supporté par la majorité des IDEs (Intellij, Eclipse, ...), il est aussi utilisé par quasiment tous les projets Open Source.

# Installation

En tant que Développeur, on utilise très souvent plusieurs outils et en plusieurs versions. Pour faciliter l'installation et la gestion de ces outils, on peut utiliser [SDK Man](https://sdkman.io/).

La procedure à suivre pour installer SDKMan est disponible à l'adresse suivante https://sdkman.io/install.

Une fois SDKMan est installé, lancé la commande suivante pour installer Java JDK (s'il n'est pas déjà installé sur votre ordinateur) :

```
sdk install java 11.0.10.hs-adpt
```

La 11.0.10.hs-adpt est la derniere version de OpenJDK11 disponible sur la plateforme sdkman au moment où j'écris cet article mais vous pouvez afficher la dernière version via la commande :

```
sdk ls java
```

Après avoir installé Java, voici la commande pour installer Maven :

```
sdk install maven
```

A noter que Maven est integré par défaut dans l'IDE comme Intellij, Eclipse. On peut l'utiliser directement mais personnelement je prefere utiliser la ligne de commande et avoir une installation à l'exterieur de l'IDE.


Pour vérifier que l'installation a bien fonctionné :

```
mvn --version
```

Le résultat devrait afficher la version de Maven et son emplacement.


# Créer un projet Java via Maven

On peut créer un projet Java/Maven via un IDE type Intellij ou Eclipse, mais pour comprendre le fonctionnement, on va d'abord créer un projet Maven via la ligne commande :

```
mvn archetype:generate -DgroupId=io.rasologg.tutoriel -DartifactId=java-maven-project -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

Si tout va bien, des fichiers seront téléchargés et Maven va créer un le projet contenant structuré comme ci-dessous :

```
java-maven-project
├── pom.xml
└── src
    ├── main
    │   └── java
    │       └── io
    │           └── rasologg
    │               └── tutoriel
    │                   └── App.java
    └── test
        └── java
            └── io
                └── rasologg
                    └── tutoriel
                        └── AppTest.java
```
## Archetype

Simplement un template de projet, ici on utilise "maven-archetype-quickstart" est genere un projet Java très basique à partir du template. On peut trouver une liste des archetypes inclus par défaut dans Maven https://maven.apache.org/archetypes/ . On peut écrire son propre archetype.

## SuperPOM

Pour bien utiliser Maven, il faut respecter quelques règles comme l'utilisation d'un fichier XML nommé pom.xml à la racine du projet, la structure du projet, etc...

Ces règles sont definies dans le SuperPOM, il se trouve dans un fichier jar (MAVEN_HOME/lib/maven-model-builder-x.y.z/org/apache/maven/model/pom-4.0.0.xml)


## POM (Projet Object Model)
Le pom.xml est le fichier qui décrit ce que Maven doit faire.

Voici le contenu créé par la ligne de commande ci-dessus :

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>io.rasologg.tutoriel</groupId>
  <artifactId>java-maven-project</artifactId>
  <packaging>jar</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>java-maven-project</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
    </dependency>
  </dependencies>
</project>
```

Le fichier XML commence par le tag "project" et doit respecter le schema XSD indiqué.

### modelVersion

modelVersion est obligatoire et contient toujours "4.0.0" qui correspond à la version du model qu'on a vu dans le SuperPOM. Cette valeur existe depuis la version 2.x de Maven.

Avec Maven 1.x, le modelVersion est "3.0.0".

La "5.0.0" est actuellement en status "brouillon" (Draft) https://cwiki.apache.org/confluence/display/MAVEN/POM+Model+Version+5.0.0


### Maven coordinates

Maven utilise trois tags (groupId, artifactId, version) pour identifier un projet.

Le *groupId* permet de grouper tous les projet par entité, souvent le nom de domaine inversé de la société ou la personne qui a developpé le projet.

Le tag *artifactId* c'est le nom du projet.

le tag *version* comme son nom l'indique, c'est pour preciser la version du projet, generalement on suit le [SemVer](https://semver.org/) x.y.z mais on peut y ajouter un "qualifier" x.y.z-qualifier. Un qualifier specifique utile pendant le developpement est le "SNAPSHOT", le SNAPSHOT permet d'indiquer à Maven que le projet est en cours de developpement.


### Autres tags

Le packaging est le type d'archivage utilisé pour le projet : jar, war, rar.

Les autres tag : *name*, *url* sont optionnels, on peut ajouter aussi le tag *description*

Le tag *dependencies* permet d'ajouter tous les dependences utilisés par le projet, dans notre cas, on a "junit" ajouté par défaut pour les tests unitaires.

Tout au long de l'evolution, on ajoutera autant de *"dependency"* que necessaire, par exemple pour utiliser une base de données.

## Maven repository

Un repository est un serveur distant où la communauté open source partage d'autres projets, et à partir duquel on peut télécharger les projets qu'on a besoin pour construire notre application.

Trois types de repository : local, central et remote.

Local se trouve sur la machine de developpement, par défaut il se trouve dans le repertoire USER_HOME/.m2/

Central est le repository par défaut utilisé par Maven, l'adresse de ce repository se trouve dans le superPOM et qui est https://repo.maven.apache.org/maven2.

Remote autres repository public, accessible via Internet ou privé interne à l'entreprise.

## Comment compiler et tester le projet

Avant de compiler, modifions un peu notre fichier "pom.xml" en y ajoutant quelques "properties"

```xml
...
<url>http://maven.apache.org</url>
<properties>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <maven.compiler.source>11</maven.compiler.source>
  <maven.compiler.target>11</maven.compiler.target>
</properties>
<dependencies>
...
```
Ces proprietes nous permettent de configurer l'encodage en "UTF-8", de specifier la version du compilateur java qu'on souhaite utiliser.
Et maintenant, lançons la commande pour compiler :

```
mvn compile
```

```log
[INFO] Scanning for projects...
[INFO]
[INFO] --------------< io.rasologg.tutoriel:java-maven-project >---------------
[INFO] Building java-maven-project 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ java-maven-project ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /workspace/tuto/java-maven-project/src/main/resources
[INFO]
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ java-maven-project ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to /workspace/tuto/java-maven-project/target/classes
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.380 s
[INFO] Finished at: 2021-04-04T18:17:37+02:00
[INFO] ------------------------------------------------------------------------
```
Un nouveau repertoire "target" vient d'être créé pour contenir les fichiers compilés qui sont le ".class".

Pour lancer les tests unitaires :


```
mvn test
```

La commande "test" lance automatiquement la compilation aussi, c'est le cycle de vie d'une commande dans Maven.

```
[INFO] Scanning for projects...
[INFO]
...

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running io.rasologg.tutoriel.AppTest
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.005 sec

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  0.808 s
[INFO] Finished at: 2021-04-04T18:26:38+02:00
[INFO] ------------------------------------------------------------------------
```

Et maintenant pour "packager" notre application, il suffit tout simplement de lancer la commande:

```
mvn package
```
Un fichier jar est créé dans le repertoire "target".

Pour supprimer le repertoire "target"

```
mvn clean
```

On peut enchainer des commandes

```
mvn clean package
```

Et si on souhaite eviter l'execution des tests (chose qu'il faut absolument eviter mais ça arrive de vouloir packager sans passer par une étape de test unitaire qui peut être très long si le projet est assez compliqué)

```
mvn clean package -DskipTests
```

# Cycle de vie d'un build maven

Maven n'est pas seulement pour gérer les dependances d'un projet mais il gere aussi les differentes étapes de la construction d'une application. Maven gere ces differentes via des plugins et en suivante une cycle de vie bien defini : validate -> compile -> test -> package -> verify -> install -> deploy.

validate : verifier la structure du projet
compile : compiler le code source
test : lancer les tests unitaires
package : créer un package contenant le code source compilé
verify : lancer les tests d'integration
install : copier le package dans le repository local 
deploy : transferer le package vers un repository distant.

# Comment chercher les dependences

https://search.maven.org/

https://mvnrepository.com/


# Dependences transitives

Une dependence peut dependre d'autres projets, Maven recupere automatiques ces autres projets lors de la compilation. 

Par exemplem, si on ajoute la libraire "hibernate-core" dans notre projet, celle-ci emmenera d'autres libraires dans notre projet.

```xml
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
    </dependency>
      <dependency>
      <groupId>org.hibernate</groupId>
      <artifactId>hibernate-core</artifactId>
      <version>6.0.0.Alpha7</version>
    </dependency>
  </dependencies>
```
Ceci peut nous emmener des dependences qu'on ne souhaite pas, qu'on n'a pas besoin dans notre projet, Maven nous propose une solution pour exclure ces dependences.

# Exclure les dependences

Pour exclure les dependences, on peut utiliser le tag exclusions/exclusion:

```xml
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
      <dependency>
      <groupId>org.hibernate</groupId>
      <artifactId>hibernate-core</artifactId>
      <version>6.0.0.Alpha7</version>
      <exclusions>
        <exclusion>
          <groupId>org.jboss.logging</groupId>
          <artifactId>jboss-logging</artifactId>
        </exclusion>
      </exclusions>
    </dependency>
  </dependencies>
```

# Dependency mediation

S'il y a un conflit sur la version d'une dependance, Maven utilise la plus proche du graphe de dependances (nearest definition).

Par exemple, le projet A depend des deux projets B et C la version 2.0, et B depend aussi du projet C mais la version 1.1, Maven choisira la version 2.0 du projet C.

# Dependance cyclique

Un projet A depend du projet B et B depend de A, est une dependance cyclique. Maven ne peut pas gerer ce genre de dependance.

# Scope

Un autre moyen de filtrer les dependences, on peut utiliser le "scope", par exemple des dependences qui ne sont utiles que pour les tests unitaires seulement, on peut ajouter le scope "test".

```xml
<dependency>
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>3.8.1</version>
  <scope>test</scope>
</dependency>
```

La valeur par défaut d'un scope est "compile", d'autres valeurs sont possible : runtime, provided

# Version

La methode la plus simple et la plus sur est de specifier une valeur fixe de la version de dependence, mais Maven nous permet d'ajouter un peu plus de flexibilité
```xml
<version>[4.1,]</version> <!-- la version la plus recente après 4.1-->
<version>[4.1,4.5]</version> <!-- la version la plus recente entre 4.1 et 4.5 -->
<version>[4.1,4.5)</version> <!-- la version entre 4.1 et avant la 4.5, 4.5 exclue -->
<version>[,4.5]</version> <!-- le maximum 4.5 -->
```

# Effective POM

Plus haut on a parlé d'un Super POM, la configuration par défaut de Maven. Ce super POM est ajouté automatiquement par Maven au projet, qui nous donne l'effective POM. 

A noter que le contenu du super pom peut être surcharger par le pom du projet.

On peut regarder l'effective pom via la ligne de commande suivante :

```
mvn help:effective-pom -output=effective-pom.xml
```

# Comment fonctionne Maven (architecture) ?

![Architecture de Maven]({{site.baseurl}}/assets/images/maven-architecture.png)

```
mvn help:effective-settings -Doutput=effective-settings.xml
```

# Plugins
Les plugins apportent beaucoup d'outil utile pour le projet, par exemple pour compiler, pour executer les tests.
Par défaut Maven ajoute beaucoup de plugins venant du superPOM, et certaines configurations ne nous conviennent pas, et pour resoudre ce probleme il suffit de surcharger la configuration.

Par exemple, pour changer la version de Java, on surcharge la configuration du plugin maven-compiler-plugin

```xml
<build>
  <plugins>
    <plugin>
      <artifactId>maven-compiler-plugin</artifactId>
      <configuration>
        <source>11</source>
        <target>11</target>
      </configuration>
    </plugin>
  </plugins>
</build>
```

# Multi-modules
Techniquement on peut avoir un projet Java composé d'un seul module (un seul pom.xml), mais au fur et à mesure de l'evolution du projet, une restructuration est necessaire, par exemple les codes qui gere l'accès aux données dans un repertoire et les codes qui gerent la logique metier dans un autre repertoire.
C'est pour gérer ce genre de problematique que Maven propose le multi-module.

```
projet-parent
├── pom.xml
    ├── module1
         ├── pom.xml
    ├── module2
         ├── pom.xml
```

# Packaging avec Jib

Le packaging est le type d'archivage utilisé pour le projet : jar, war, rar. Avant l'utilisation du Cloud et la conteneurisation, on utilise le war ou la jar pour deployer l'application.

Avec l'architecture Cloud Native, le packaging final est maintenant une image conteneur (image docker). Cette image peut être construite de differente façon, via un fichier Dockerfile, mais dans le cadre de Maven, on s'interesse au plugi [Jib](https://github.com/GoogleContainerTools/jib).

Jib est un plugin Maven developpé par Google qui permet de créer une image conteneur en mode 100% Java. Via un Dockerfile, on est obligé d'installer l'application Docker parce qu'on a besoin du demon docker.




# Astuces

```
mvn help:describe -Ddetail=true -Dplugin=compiler
```
```
mvn help:help -Ddetail=true
```
