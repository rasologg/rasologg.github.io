---
layout: post
title:  "Quarkus: développer un microservice cloud-native"
date:   2021-04-30 13:33:29 +0200
categories: blog
---

# Introduction
Depuis l'avènement de Docker, Kubernetes, Cloud, beaucoup de frameworks ont vu le jour.
Pour rappel, la philosophie de Java est : écrire le code une fois et lancer sur n'importe quel système d'exploitation (le fameux WORA : Write Once Run Anywhere).
Ce choix n'a que des avantages, on constate aussi des problèmes comme le temps de démarrage de l'application, la consommation des resources memoire (RAM).

Ce WORA a du sens à l'époque on deployait l'application sur un système precis Linux, Windows, MacOS, mais maintenant on crée une image et on lance l'application avec une plateforme d'execution d'un conteneur comme Docker par exemple.
En production on orchestre avec Kubernetes ou d'autres plateforme equivalentes.

Alors pourquoi ne pas construire l'application comme un vrai natif "Cloud" ? Pour répondre à cette question que des frameworks comme Quarkus ont vu le jour.

Quarkus, est un framework developpé par RedHat, open source et apporte des avantages comme :

- taille du package réduite
- utilisation de memoire RAM fortement réduite
- temps de démarrage accéléré pour gérer la montée en charge facilement dans un système cloud (Kubernetes) (scale up)

Ceci grace à la compilation AOT (Ahead-Of-Time), en une phrase, compile l'application directement en binaire proche du CPU, contrairement à l'ancien mode JIT (Just-In-Time) qui interprete le bytecode au moment où on appelle.

En tant que développeur, Quarkus apporte aussi des avantages comme :

- l'auto-chargement des modifications, pas besoin d'arrêter et rédémmarrer l'application à chaque fois qu'on modifie le code
- on se focalise à l'écrire des tests fonctionnels

# Créer un projet

Plusieurs façons de créer un projet Quarkus :

- via le portail https://code.quarkus.io/ à la Spring Initializr.
- via votre IDE préféré comme Intellij, ou Microsoft VS Code
- via Maven

Avec Maven :
```
mvn io.quarkus:quarkus-maven-plugin:1.13.3.Final:create -DprojectGroupId=io.rasologg.survey 
-DprojectArtifactId=survey -DprojectVersion=1.0.0-SNAPSHOT 
-DclassName=io.rasologg.survey.web.SurveyResource -Dpath=/surveys
```
Cette commande devrait afficher le message ci-dessous et crée un projet nommé "survey".

```
-----------
...
applying codestarts...
🔠 java
🧰 maven
🗃 quarkus
📜 config-properties
🛠 dockerfiles
🛠 maven-wrapper
🐒 resteasy-example

-----------

[SUCCESS] ✅ quarkus project has been successfully generated in:
--> /Users/g.rasolondraibe/workspace/tuto/survey
-----------
[INFO]
[INFO] ========================================================================================
[INFO] Your new application has been created in /Users/g.rasolondraibe/workspace/tuto/survey
[INFO] Navigate into this directory and launch your application with mvn quarkus:dev
[INFO] Your application will be accessible on http://localhost:8080
[INFO] ========================================================================================
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  6.064 s
[INFO] Finished at: 2021-05-01T10:00:05+02:00
[INFO] ------------------------------------------------------------------------
```
Voici la structure du projet créé par le plugin Maven de quarkus

```
survey
├── README.md
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src
    ├── main
    │   ├── docker
    │   │   ├── Dockerfile.jvm
    │   │   ├── Dockerfile.legacy-jar
    │   │   ├── Dockerfile.native
    │   │   └── Dockerfile.native-distroless
    │   ├── java
    │   │   └── io
    │   │       └── rasologg
    │   │           └── survey
    │   │               └── web
    │   │                   └── SurveyResource.java
    │   └── resources
    │       ├── META-INF
    │       │   └── resources
    │       │       └── index.html
    │       └── application.properties
    └── test
        └── java
            └── io
                └── rasologg
                    └── survey
                        └── web
                            ├── NativeSurveyResourceIT.java
                            └── SurveyResourceTest.java
```

Pour lancer le projet qu'on vient de créer

```
❯ cd survey
❯ ./mvnw quarkus:dev
```

```
[INFO] Scanning for projects...
[INFO]
[INFO] ---------------------< io.rasologg.survey:survey >----------------------
[INFO] Building survey 1.0.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- quarkus-maven-plugin:1.13.3.Final:dev (default-cli) @ survey ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 2 resources
[INFO] Nothing to compile - all classes are up to date
Listening for transport dt_socket at address: 5005
__  ____  __  _____   ___  __ ____  ______
 --/ __ \/ / / / _ | / _ \/ //_/ / / / __/
 -/ /_/ / /_/ / __ |/ , _/ ,< / /_/ /\ \
--\___\_\____/_/ |_/_/|_/_/|_|\____/___/
2021-05-01 10:11:47,612 INFO  [io.quarkus] (Quarkus Main Thread) survey 1.0.0-SNAPSHOT on JVM (powered by Quarkus 1.13.3.Final) started in 1.861s. Listening on: http://localhost:8080
2021-05-01 10:11:47,652 INFO  [io.quarkus] (Quarkus Main Thread) Profile dev activated. Live Coding activated.
2021-05-01 10:11:47,652 INFO  [io.quarkus] (Quarkus Main Thread) Installed features: [cdi, resteasy]
```

![Quarkus page de bienvenu]({{site.baseurl}}/assets/images/quarkus-welcome-page.png)

Ouvrons le projet avec un IDE et analysons son contenu.

Comme vous l'avez remarqué, il n'y a que SurveyResource.java qui a été créé, pas de class main.
Quarkus ne crée pas une class "main" dans le projet, pas comme les autres frameworks comme Spring. 
Le projet utilise la class par défaut de Quarkus : io.quarkus.runner.GeneratedMain.
Mais ceci n'empeche pas de créer une class main personalisée si on le souhaite.

Par exemple :
```java
import io.quarkus.runtime.Quarkus;
import io.quarkus.runtime.annotations.QuarkusMain;

@QuarkusMain
public class SurveyMain {
    public static void main(String[] args) {
        Quarkus.run(args);
    }
}
```

Quarkus crée un repertoire "docker" contenant des Dockerfiles qu'on utilisera pour créer l'image du conteneur. 
Il est aussi possible d'utiliser Jib pour créer l'image du conteneur sans passer par un Dockerfile.

Le fichier application.properties nous permettra d'ajouter les configurations specifique à l'application.

Pour l'instant, notre microservice n'a que ceci comme code
```java
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

@Path("/surveys")
public class SurveyResource {

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public String hello() {
        return "Hello RESTEasy";
    }
}
```
Grosso modo, ce code expose un REST API via la route (@Path) "/surveys", et la méthode HTTP GET (@GET)
nous produit (@Produces) du texte (MediaType.TEXT_PLAIN).

Si on teste rapidement via cURL, on a le resultat ci-dessous. 
Plus tard, on utilisera d'autre outil de test plus complet comme Postman ou SoapUI.

```
❯ curl http://localhost:8080/surveys
Hello RESTEasy
```

@Path : pour designer la route
@Produces et @Consumes : pour designer le format de données produit et consommé, ça peut être du JSON, XML, etc...
@GET : pour designer la méthode HTTP accepté pour la route, comme son nom l'indique ça correspond au HTTP GET.
Et donc, 
- @POST pour POST
- @PUT pour PUT
- @PATCH pour PATCH
- @DELETE pour DELETE
- @HEAD pour HEAD



@QueryParam : des parametres de type clé=valeur, par exemple /surveys?test=toto

@PathParam : des parametres envoyés dans la route, par exemple /surveys/1000/questions

@MatrixParam

@FormParam

@BeanParam

@HeaderParam

//TODO ...