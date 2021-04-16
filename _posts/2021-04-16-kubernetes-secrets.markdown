---
layout: post
title:  "Kubernetes: gestion de secrets"
date:   2021-03-16 13:33:29 +0200
categories: blog
---

# La gestion des secrets dans Kubernetes à l'époque de GitOps

## Introduction
Ces derniers temps, on attend de plus en plus parler de GitOps avec Kubernetes. Grosso modo, GitOps, c'est tout gérer via Git, 
les fichiers de configuration, les fichiers de description de déploiement, et donc par conséquent les identifiants et mot de passe que les applications ont besoin pour fonctionner, 
par exemple l'identifiant et mot de passe pour la base de données.

Mettre ces informations directement sur Git n'est pas vraiment une bonne idée, de point de vue sécurité, et beaucoup d'entreprise continue à gérer les secrets manuellement via une commande kubectl et stocker les identifiants et mot de passe dans un outil comme keepass.
Et ceci déroge au concept de GitOps.

Pour répondre à ce problématique que la fondation CNCF étudie les outils suivants afin de les inclure dans l'éco-système cloud-native :

- [Hashicorp Vault](https://www.vaultproject.io/), [AWS Secrets Manager](https://aws.amazon.com/fr/secrets-manager/), Certificate Manager, AWS KMS
- Bitnami Sealed Secret, Encrypted repositories
- [Google Secrets Management](https://cloud.google.com/solutions/secrets-management), [Sops](https://github.com/mozilla/sops)

Plus d'info :
https://www.cncf.io/announcements/2021/02/23/cncf-provides-insights-into-secrets-management-tools-with-latest-end-user-technology-radar/


