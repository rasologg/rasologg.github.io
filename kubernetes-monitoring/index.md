# Installer Prometheus Operator pour Kubernetes

Installer l'operateur via la ligne de commande suivante, si vous voulez specifier un autre namespace que le default, vous pouvez ajouter le "-n votre_namespace"

```
kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/master/bundle.yaml
```

ça devrait afficher les messages suivants

```
customresourcedefinition.apiextensions.k8s.io/alertmanagerconfigs.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/alertmanagers.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/podmonitors.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/probes.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/prometheuses.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/prometheusrules.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/servicemonitors.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/thanosrulers.monitoring.coreos.com created
clusterrolebinding.rbac.authorization.k8s.io/prometheus-operator created
clusterrole.rbac.authorization.k8s.io/prometheus-operator created
deployment.apps/prometheus-operator created
serviceaccount/prometheus-operator created
service/prometheus-operator created
```

Pour vérifier que l'operateur est bien installé

```
kubectl get deploy
```

```
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
prometheus-operator   0/1     1            0           10s
```

Maintenant on peut configurer le RBAC, afin d'autoriser Prometheus à acceder à l'API de Kubernetes.
Créer un fichier prometheus_rbac.yaml et ajouter le contenu ci-dessus :

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus
rules:
- apiGroups: [""]
  resources:
  - nodes
  - nodes/metrics
  - services
  - endpoints
  - pods
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources:
  - configmaps
  verbs: ["get"]
- apiGroups:
  - networking.k8s.io
  resources:
  - ingresses
  verbs: ["get", "list", "watch"]
- nonResourceURLs: ["/metrics"]
  verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: prometheus
  namespace: default
```

Ensuite, executer la commande suivante

```
kubectl apply -f prometheus_rbac.yaml
```

On devrait avoir des messages comme ci-dessous

```
serviceaccount/prometheus created
clusterrole.rbac.authorization.k8s.io/prometheus created
clusterrolebinding.rbac.authorization.k8s.io/prometheus created
```

Créer un fichier YAML pour deployer prometheus (par exemple prometheus.yaml) contenant le code ci-dessous

```yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: prometheus
  labels:
    app: prometheus
spec:
  image: quay.io/prometheus/prometheus:v2.22.1
  nodeSelector:
    kubernetes.io/os: linux
  replicas: 2
  resources:
    requests:
      memory: 400Mi
  securityContext:
    fsGroup: 2000
    runAsNonRoot: true
    runAsUser: 1000
  serviceAccountName: prometheus
  version: v2.22.1
  serviceMonitorSelector: {}
```

ensuite, lancer la commande

```
kubectl apply -f prometheus.yaml
```
```
prometheus.monitoring.coreos.com/prometheus created
```

On vérifie

```
kubectl get prometheus
```
On devrait obtenir le resultat suivant

```
NAME         VERSION   REPLICAS   AGE
prometheus   v2.22.1   2          12s
```

Et une autre commande

```
kubectl get pod -l app=prometheus
```
Et un autre resultat :-)

```
NAME                      READY   STATUS    RESTARTS   AGE
prometheus-prometheus-0   2/2     Running   1          69s
prometheus-prometheus-1   2/2     Running   1          69s
```

Et une derniere étape pour exposer le service afin que prometheus soit accessible via Grafana. 
Créer un autre fichier YAML, nommer le comme vous le voulez, par prometheus_service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: prometheus
  labels:
    app: prometheus
spec:
  ports:
  - name: web
    port: 9090
    targetPort: web
  selector:
    app: prometheus
  sessionAffinity: ClientIP
```

Et on applique le fichier

```
kubectl apply -f prometheus_service.yaml
```
qui devrait afficher ce message

```
service/prometheus created
```

Et on vérifie si tout va bien

```
kubectl get service
```

```
NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kubernetes            ClusterIP   10.96.0.1        <none>        443/TCP    72m
prometheus            ClusterIP   10.107.134.128   <none>        9090/TCP   11s
prometheus-operated   ClusterIP   None             <none>        9090/TCP   2m26s
prometheus-operator   ClusterIP   None             <none>        8080/TCP   5m56s
```

Et voilà, on a un prometheus UP&RUNNING ;-) mais sans Grafana le travail n'est pas bien fait.

Pour installer Grafana, on passe par Helm, il faut installer Helm si vous ne l'avez pas encore.

Ajoutons le repo de bitnami

```
helm repo add bitnami https://charts.bitnami.com/bitnami
```
Ensuite, on installe grafana, en mode quickstart ;-)

```
helm install grafana bitnami/grafana
```

ça devrait afficher un message de ce type

```
NAME: grafana
LAST DEPLOYED: Thu Apr  8 17:32:32 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

1. Get the application URL by running these commands:
    echo "Browse to http://127.0.0.1:8080"
    kubectl port-forward svc/grafana 8080:3000 &

2. Get the admin credentials:

    echo "User: admin"
    echo "Password: $(kubectl get secret grafana-admin --namespace default -o jsonpath="{.data.GF_SECURITY_ADMIN_PASSWORD}" | base64 --decode)"
```


