<a id="index"></a>
**Index** :

I. [**`kubectl logs`**](#i-kubectl-logs) - Consulter logs containers

II. [**`kubectl describe`**](#ii-kubectl-describe) - Diagnostiquer problèmes ressources

III. [**`kubectl exec` & `kubectl run`**](#iii-kubectl-exec--kubectl-run)
1. [`kubectl exec`](1-kubectl-exec)
2. [`kubectl attach`](2-kubectl-attach)
3. [`kubectl run`](3-kubectl-run)

IV. [**Events K8s**](#iv-events-k8s) - Événements cluster

V. [**Patterns d'Erreurs Courantes**](#v-patterns-derreurs-courantes) - ImagePullBackOff, CrashLoopBackOff, etc.

---
<a id="i-kubectl-logs"></a>
# I. [**`kubectl logs`**](#index)



---
<a id="ii-kubectl-describe"></a>
# II. [**`kubectl describe`**](#index)



---
<a id="iii-kubectl-exec"></a>
# III. [**`kubectl exec` & `kubectl run`**](#index)

## 1. `kubectl exec`

`kubectl exec` permet de lancer **un nouveau processus** dans le conteneur ciblé d'un pod.

Ce processus n'est pas exécuté _dans_ le conteneur au sens physique (on ne “rentre” pas dans le pod) mais Kubernetes le crée **dans les mêmes namespaces** que le processus principal du conteneur (PID, réseau, filesystem, etc.).

Ainsi, le processus voit exactement le même environnement que l'application du conteneur. On se place donc _à côté_ du processus principal, avec le même point de vue sur le système.

>[!NOTE]
>La commande `kubectl exec -it ... sh` peut donner l'impression d'entrer dans le conteneur, en réalité on partage simplement son contexte d’exécution.

Les exemples suivants illustrent différents usages courants de `kubectl exec`
### 1.1 Exemples : inspection 

- Création d'un déploiement `nginx` et récupération du nom du Pod :
```bash
kubectl create deploy nginx --image nginx
kubectl get pods
```

_Résultat_ :
```bash
vagrant@k0s1:~ (⎈|Default:default) $ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-66686b6766-tcks6   1/1     Running   0          41s
```

- On peut lancer une commande avec `kubectl exec` en la lançant après un `--`, par exemple pour afficher le contenu d'un fichier, on peut lancer `cat /etc/*elea*` :

```bash
kubectl exec nginx-66686b6766-tcks6 -- cat /etc/*elea*
```

_Résultat_ :
```bash
vagrant@k0s1:~ (⎈|Default:default) $ kubectl exec nginx-66686b6766-tcks6 -- cat /etc/*elea*
PRETTY_NAME="Debian GNU/Linux 13 (trixie)"
NAME="Debian GNU/Linux"
VERSION_ID="13"
VERSION="13 (trixie)"
VERSION_CODENAME=trixie
DEBIAN_VERSION_FULL=13.3
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
cat: /etc/cloud-release: No such file or directory
command terminated with exit code 1
```

- On peut utiliser `env` pour afficher les variables d'environnement d'un Pod donné :
```bash
kubectl exec nginx-66686b6766-tcks6 -- env
```

>[!TIP]
>Exemple d'inspection automatisée sur plusieurs pods :
>```bash
>kubectl get pods -o name | xargs -I {} kubectl exec {} -- cat /etc/hosts | grep nginx
>```
>Pour chaque pod du cluster, exécute `cat /etc/hosts`, puis affiche uniquement les lignes contenant nginx.
### 1.2 Exemple : debug

Pour exécuter une commande complexe dans le conteneur :
```bash
kubectl exec nginx-66686b6766-tcks6 -- /bin/bash -c "apt update && apt install -y iputils-ping && ping google.fr"
```
> Dans cette exemple, le Shell Bash est lancé dans le conteneur, et exécute la commande `apt update && apt install -y iputils-ping && ping google.fr`.
### 1.3 Exemple : interaction

La commande suivante permet d'ouvrir le terminal interactif du Pod :
```bash
kubectl exec nginx-66686b6766-tcks6 -ti -- bash
```

>[!NOTE]
>Dans le cas d’un pod contenant plusieurs conteneurs, il faut préciser le conteneur avec lequel on veut interagir :
>```bash
>kubectl exec nginx-66686b6766-tcks6 -ti -c <nomDuConteneur> -- bash
>```

## 2. `kubectl attach`

`kubectl attach` ne lance **aucun nouveau processus**.

Cette commande se contente de se connecter aux flux d'entrée/sortie (`stdin`, `stdout`, `stderr`) du processus principal déjà en cours dans le conteneur.

On agit donc comme un terminal branché directement sur l'application existante :
- ce que l'on tape → va dans le `stdin` du processus
- ce que le programme affiche → revient via `stdout` / `stderr`

On interagit donc avec le processus tel qu'il tourne déjà, sans en créer un nouveau.

>[!IMPORTANT]
>On ne se place jamais _dans_ un conteneur.
>Kubernetes ne fait que créer ou connecter un processus partageant les namespaces du conteneur.

## 3. `kubectl run`

`kubectl run` permet de **créer rapidement un pod** à partir d’une image conteneur.

Cette commande est souvent utilisée dans un contexte de test ou de dépannage pour lancer un pod temporaire dans un namespace Kubernetes donné. On peut ainsi disposer rapidement d'un environnement minimal pour vérifier la connectivité réseau, tester une image ou exécuter des commandes de diagnostic.

Par défaut, `kubectl run` crée un pod simple, sans contrôleur (Deployment, ReplicaSet, etc.), ce qui le rend adapté aux usages ponctuels plutôt qu'au déploiement d'applications en production.

Les exemples suivants illustrent différents usages courants de `kubectl run`
### 3.1 Exemple : inspection

Pour lancer un Pod temporaire dans le même namespace Kubernetes afin d'effectuer des tests :
```bash
kubectl run test --image alpine --restart=Never --rm -ti -- echo 1
```
Cette commande permet :
- Générer un Pod dans le namespace courant.
- `--restart=Never` crée un Pod simple sans contrôleur ni redémarrage automatique (évite le redémarrage automatique en cas d'échec).
- `--rm` détruit le Pod à la fin de la commande.
- `-ti` active le mode interactif, principalement utile pour ouvrir un shell dans le Pod.
- `-- echo 1` la commande qui sera lancée dans le Pod.

_Résultat_ :
```bash
vagrant@k0s1:~ (⎈|Default:default) $ kubectl run test --image alpine --restart=Never --rm -ti -- echo 1
1
pod "test" deleted from default namespace
vagrant@k0s1:~ (⎈|Default:default) $ kubectl get pods
No resources found in default namespace.
```

### 3.2 Exemple : debug

Pour lancer une commande `curl`, on peut utiliser directement une image dédiée a cet effet :
```bash
kubectl run test --image curlimages/curl --restart=Never --rm -ti -- curl google.fr
```

_Résultat_ :
```bash
vagrant@k0s1:~ (⎈|Default:default) $ kubectl run test --image curlimages/curl --restart=Never --rm -ti -- curl google.fr
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.fr/">here</A>.
</BODY></HTML>
pod "test" deleted from default namespace
vagrant@k0s1:~ (⎈|Default:default) $ kubectl get pods
No resources found in default namespace.
```

### 3.3 Exemple : interaction

- Lancer un Pod de test et accéder à son terminal :
```bash
kubectl run test --image debian --rm -ti -- bash
```

- On peut lancer des commandes directement dans le terminal du Pod :
```bash
apt update && apt install curl -y && curl google.fr
```

- Il est aussi possible de le faire avec la commande, sans entrer dans le terminal du Pod :
```bash
kubectl run test --image debian --rm -ti -- /bin/bash -c "apt update && apt install curl -y && curl google.fr"
```

>[!TIP]
>- **Analogie** : Ces Pods temporaires agissent comme une boîte à outils jetable dans le cluster.
>- Le pod agit comme un environnement Linux éphémère exécuté directement dans le cluster.


---
<a id="iv-events-k8s"></a>
# IV. [**Events K8s**](#index)



---
<a id="v-patterns-derreurs-courantes"></a>
# V. [**Patterns d'Erreurs Courantes**](#index)

