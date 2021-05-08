---
layout: post
title:  "Automatisation de processus dans une architecture microservices"
date:   2021-05-09 13:33:29 +0200
categories: blog
---

# Introduction

Un processus (ou workflow) est une série des taches à traiter afin d'obtenir un resultat. On peut trouver des processus un peu partout, dans la vie courante, au sein des entreprises, etc....

Dans une application monolithe les processus sont melangés dans le code de l'application, la logique est hardcodé. Dans une architecture microservice, pour effectuer un processus (par acquerir un nouveau client), on a besoin d'appeler plusieurs microservices. 

Il y a plusieurs façons :
- event driven architecture, chaque composant reçoit et envoie un evenement pour commencer une tache et quand il l'a terminé. Ici on applique le pattern de choregraphie.
- ou appliquer le pattern d'orchestration, et dans ce cas on a besoin d'utiliser un orchestrateur, un workflow engine ou conductor.

Plusieurs outils sont disponibles en version opensource :

- https://www.activiti.org/ : developpé en Java, dans ce tuto nous allons utilisé la version 7 qui est cloud native. Activiti utilise la notation BPMN pour gérer le processus.
- https://netflix.github.io/conductor/ : developpé en Java par Netflix, aussi en architecture Cloud native, utilise JSON comme fichier de configuration de processus.

Certaines personnes amalgament ces deux outils aux anciennes plateformes SOA/ESB qui sont monolithes, centralisées, très complexes et souvent verrouillés par les vendeurs.

Activiti, Conductor sont open source, cloud native, microservices et très proche des developpeurs.

Caracteristiques d'un workflow engine :
- persistance : tracer l'etat de chaque tache, de chaque instance des processus
- scheduling  : 
- versioning  : pouvoir mettre à jour le processus meme s'il y a des longs processus en cours d'execution. 
- visibilité : visualisé les process
- audit data :
- tooling : graphical modeling, technical operations, business monitoring

Pour notre tutoriel, on va realiser le systeme qui permettra de traiter le processus suivant :

![Workflow engine process sample]({{site.baseurl}}/assets/images/workflow-engine-process-sample.png)

Il s'agit d'un processus très simplifié d'un operateur télécom : quand workflow engine reçoit l'ordre, il executé la premiere tache, qui est une tache service (icone engrenage) pour appeler un API, cette tache vérifie l'identité du client en appelant une plateforme KYC par example. Si la vérifie echoue l'ordre est rejeté et le processus se termine, et c'est validé, workflowengine continue la tache suivante "vérifier la notation de crédit du client", ensuite une tache manuelle qui sera fait par un employé pour approuver l'ordre, et ainsi de suite jusqu'à la terminaison de l'ordre.


//TODO