
# Conteneur (Container)

Encapsule une application sous une forme portable et facile à déployer. Le conteneur contient tous les elements necessaires au fonctionnement de l'application, et il peut fonctionner sur n'importe quel systeme compatible, sur n'importe quel cloud, sans aucun changement.

Pour demarrer un conteneur, un conteneur runtime est necessaire, ce conteneur runtime qui permet à l'application (conteneur) de communiquer à la machine hôte.

Conteneur runtime : docker, lxc, runc, cri-o, rkt, containerd, podman, buildah.

cf. image (draw.io)

Un conteneur est fortement inspiré des fonctionnalités offertes par le systeme Linux.
Au sein du runtime, chaque conteneur est isolé par le concept de linux kernel namespaces. Cette isolation s'applique sur le network, file, users, processes, IPCs.

L'utilisation d'un conteneur est standardisé et suit la specification d'OCI (Open Containers Initiative) https://opencontainers.org. Ce qui permet de deployer et executer n'importe quel conteneur sur n'impore quel runtime.

On peut télécharger un conteneur depuis Internet et le lancer, par exemple https://hub.docker.com/, https://quay.io/, mais on peut aussi le construire.
```
docker run --name mon-serveur-web -d nginx:1.19
Unable to find image 'nginx:1.19' locally
1.19: Pulling from library/nginx
75646c2fb410: Pull complete
6128033c842f: Pull complete
71a81b5270eb: Pull complete
b5fc821c48a1: Pull complete
da3f514a6428: Pull complete
3be359fed358: Pull complete
Digest: sha256:bae781e7f518e0fb02245140c97e6ddc9f5fcf6aecc043dd9d17e33aec81c832
Status: Downloaded newer image for nginx:1.19
99d68c4d2eb755d3c094084d46006c26bf6f32ebf133fe436b9fadc6739b7163
```
```
docker ps
99d68c4d2eb7   nginx:1.19                       "/docker-entrypoint.…"   30 seconds ago   Up 29 seconds   80/tcp    mon-serveur-web
```

Pour construire un conteneur, plus communement appelé image, il faut écrire un fichier ou utiliser un outil. Le plus utilisé est le fichier Dockerfile.

```
FROM centos
MAINTAINER Rasolo <gege.eric@gmail.com>
ADD ...
RUN ...
CMD ...
```

# Kubernetes

Une plateforme pour deployer, gerer un conteneur.

# Kubernetes architecture overwiev

(cf draw.io)

# Operator

un moyen d'etendre les ressources que l'API Serveur de Kubernetes peut gerer. On appelle ceci CRD (Custom Resource Definitions).


# Kubernetes Controller:

```
for {
    desired := getDesiredState()
    current := getCurrentState()
    makeChanges(desired, current)
}
```

# Kubernetes API Server

c'est un RESTful interface qui expose les services necessaire à la gestion d'un clusteur Kubernetes

(cf draw.io)

# Interegir avec Kubernetes
## dashbord
## kubectl
L'outil pour administrer Kubernetes.
https://kubernetes.io/docs/reference/kubectl/cheatsheet/
https://kubectl.docs.kubernetes.io/

kubectl auth can-i get pods

## curl
kubectl proxy --port=8001 &

curl http://localhost:8001

curl http://localhost:8001/api/v1/namespaces/default/pods

# Comment on developpe pour Kubernetes ?

Build l'application -> La conteneuriser -> La deployer et executer sur K8S

# Pod
Est une abstration d'un serveur. Un pod peut contenir plusieurs conteneurs dans un seul namespace et ils sont exposé via une seule adresse IP.

Un pod doit être demarré à partir d'un deployment, sinon il ne sera pas redeployer quand le noeud s'arrete.
```
apiVersion: v1
kind: Pod
metadata:
  name: busiboxPod
  namespace: default
spec:
  containers:
  - name: busybox
    image: busybox
    command:
      - sleep
      - "100"
``` 

kubectl explain pods

Generalement, un pod contient un seul conteneur mais, dans certains cas on a besoin de lancer multiples conteneur dans un pod:

- init container : pour initialiser une configuration par exemple
- sidecar container : produire des fonctionnalités supplementaires au conteneur principal, par exemple gestion des logs, synchronisation, metriques, tracing
- ambassador container
- adapter container

## Comment investiguer un probleme dans un pod ?

kubectl describe pod podname (qui lit le status courant du pod dans la base de données etcd)
kubectl logs (lit le contenu stdout du pod, s'il y a plusieurs conteneurs il faut ajouter l'option -c avec le nom du conteneur)
kubectl exec (executer une commande dans un conteneur, s'il y a plusieurs conteneurs il faut ajouter l'option -c)

## Monitorer un pod
kubectl get pods --all-namespaces -o wide

## securityContext d'un pod
```
apiVersion: v1
kind: Pod
metadata: 
  name: nginxsecure
spec:
  securityContext:
    runAsNonRoot: true
  containers:
  - image: nginx
    name: nginx
```
# Job

# Cronjob

# Namespace
Pour isoler les ressources.

kubectl -n namespace

