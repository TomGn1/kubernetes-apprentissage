## **Mise en pratique des [notions](1-Les-Bases_Notions.md) sur les bases de Kubernetes**.

---
<a id="index"></a>
**Index** :

I. [**Mise en place d'un cluster k0s**](#i-mise-en-place-dun-cluster-k0s)
1. [Mise en place de l'environnement](#1-mise-en-place-de-lenvironnement)
2. [Installation manuelle d'un cluster k0s](#2-installation-manuelle-dun-cluster-k0s)

II. [**`kubectl` & `curl`**](#ii-kubectl--curl)
1. [La commande `kubectl`](#1-la-commande-kubectl)
2. [`curl`](#2-curl)
3. [Autres commandes utiles](#3-autres-commandes-utiles)

III. [**Lancement des premiers Pods**](#iii-lancement-des-premiers-pods)
1. [`kubectl run`](#1-kubectl-run)
2. [L'option `--dry-run`](#2-loption---dry-run)
3. [Manifests et outputs](#3-manifests-et-outputs)

IV. [**Pods : Multi-conteneurs**](#iv-pods--multi-conteneurs)
1. [Création simple d'un pod multi-conteneurs](#1-création-simple-dun-pod-multi-conteneurs)
2. [Exemple de manifeste multi-conteneurs](#2-exemple-de-manifeste-multi-conteneurs)

V. [**ReplicaSets**](#v-replicasets)
 1. [Création de manifest de ReplicaSet](#1-création-de-manifest-de-replicaset)
 2. [Scaling d'un ReplicaSet](#2-scaling-dun-replicaset)
 3. [Illustration du fonctionnement des labels](#3-illustration-du-fonctionnement-des-labels)
 4. [Exemple d'auto guérison](#4-exemple-dauto-guérison)
 5. [Aperçu des annotations](#5-aperçu-des-annotations)

VI. [**`kubectl` : contextes et optimisations**](#vi-kubectl--contextes-et-optimisations)
1. [Les contextes](#1-les-contextes)
2. [Optimisation de la CLI](#2-optimisation-de-la-cli)

VII. [**Deployments**](#vii-deployments)
1. [Création d'un déploiement](#1-création-dun-déploiement)
2. [`RollingUpdate` et historique](#2-rollingupdate-et-historique)
3. [Fonctionnement interne](#3-fonctionnement-interne)

---
# I. [Mise en place d'un cluster k0s](#index)

- Installation d'une seule node cluster
	- Elle devra assumer la fonction de _worker_.
	- Documentation :
		- [k0s](https://docs.k0sproject.io/stable/install/)
		- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
- [Liens de la vidéo de mise en pratique](https://youtu.be/jRKhHYoFsmU?list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap)
- [Liens du dépôt de la pratique](https://gitlab.com/xavki/kubernetes-tutorials-new-version/-/tree/main/k0s-stack?ref_type=heads)
## 1. Mise en place de l'environnement

Pour la mise en place du cluster **k0s**, je vais utiliser **Vagrant** afin de créer un environnement de travail reproductible et isolé.

>[!NOTE]
>## Qu'est ce que Vagrant ?
>**Vagrant** est un outil d'automatisation qui permet de **créer et de gérer des environnements de machines virtuelles** de manière déclarative via un fichier de configuration appelé **Vagrantfile**. **Vagrant** s’appuie généralement sur un hyperviseur de type 2 (ex. : VirtualBox) pour exécuter une ou plusieurs machines virtuelles.
>## Pourquoi utiliser **Vagrant** ?
> ### Reproductibilité 
> - Un même **Vagrantfile** permet d'obtenir le **même environnement** pour tout le monde.
> ### Déploiement Rapide
> - Création de plusieurs VMs en une seule commande :
>	```bash
>	vagrant up
>	```
>- Démarrage, arrêt et destruction simplifiés :
>	```bash
>	vagrant halt
>	vagrant destroy
>	```
>### Simulation réaliste d'un cluster
>- Possibilité de créer **plusieurs nœuds** (controller / worker)
>- Réseau privés entre VM
>- Simulation proche d'une architecture Kubernetes réelle
>### Provisionning automatisé
>- Installation automatique des dépendances
>- Support des scripts shell, Ansible, etc.
>- Environnement prêt à l'emploi dès le premier démarrage
>### Isolation de l'environnement
>- Pas d'impact sur le système hôte
>- Pas de "pollution" des configurations locales
>- Facile à réinitialiser en cas d'erreur
>### Installation et documentation
>- Installation : https://developer.hashicorp.com/vagrant/install 
>- Documentation : https://developer.hashicorp.com/vagrant/docs

**Vagrant** me permettra donc :
- Créer une ou plusieurs machines virtuelles Linux
- Installer et configurer **k0s** pour former un cluster Kubernetes fonctionnel
- Travailler dans un environnement stable, reproductible et contrôlé

### Exécution de l'environnement :

- Clone ou copie des fichiers du dépôt https://gitlab.com/xavki/kubernetes-tutorials-new-version/-/tree/main/k0s-stack?ref_type=heads
- Ouvrir la console dans ce répertoire.
- Afin de lancer la VM **k0s1**, lancer la commande :
```bash
vagrant up k0s1
```

>[!NOTE]
>Dans un premier temps, je vais lancer l'installation du cluster manuellement en répondant `no` quand il est demandé `Do you want to install the k0s (yes/no) ?`

## 2. Installation manuelle d'un cluster k0s

### Installation du Cluster

- Se connecter à la VM **k0s1** avec la commande :
```bash
vagrant ssh k0s1
```
- Lancer l'installation de **k0s** et lancer le cluster avec les commandes suivantes :
```bash
curl -sSf https://get.k0s.sh | sudo sh
sudo k0s install controller --single
sudo k0s start
```
- Pour vérifier le lancement du cluster, lancer :
```bash
sudo k0s status
sudo k0s kubectl get nodes
```

>[!NOTE]
>La commande `sudo k0s kubectl get nodes` invoque une version embarquée de `kubectl`, automatiquement configurée pour le cluster k0s.

### Installation de `kubectl`

- Une fois le cluster installé et fonctionnel, lancer le téléchargement et l'installation de `kubectl` :
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

- Le fichier `kubeconfig` est le **certificat client** qui permet d'**accéder à l'API Server**, il doit être placé dans un emplacement facilement accessible. Créer ce dossier et ajuster les permissions :
```bash
mkdir /home/vagrant/.k0s
chown vagrant:vagrant -R /home/vagrant/.k0s
```

- Copier le fichier `kubeconfig` dans le nouveau dossier :
```bash
sudo cat /var/lib/k0s/pki/admin.conf > /home/vagrant/.k0s/kubeconfig
```

- Ajuster les permissions sur le fichier :
```bash
chown vagrant:vagrant -R /home/vagrant/.k0s
chmod 600 -R /home/vagrant/.k0s/kubeconfig
```

- Mettre ce fichier dans la variable d'environnement `KUBECONFIG`, cette variable est utilisée, par défaut, par `kubectl` :
```bash
export KUBECONFIG="/home/vagrant/.k0s/kubeconfig"
```

- Pour tester l'installation lancer la commande :
```bash
kubectl get nodes
```

>[!NOTE]
>Pour lancer l'installation automatique du cluster, répondre `yes` quand il est demandé `Do you want to install the k0s (yes/no) ?`

>[!TIP]
>Lors de la création de la VM Debian avec Vagrant et VirtualBox, il peut y avoir un problème de lenteur sur les téléchargements. Ce problème est souvent causé par l'association DNS proxy NAT et `systemd-resolved`. Afin de résoudre ce problème modifier le *vagrantfile* pour désactiver `systemd-resolved` et éditer manuellement le fichier */etc/resolv.conf*.
>
>- Dans le *vagrantfile*, ajouter la configuration DNS :
>```ruby
>  dns = <<-SHELL
>  sudo systemctl disable systemd-resolved || true
>  sudo systemctl stop systemd-resolved || true
>  sudo rm -f /etc/resolv.conf
>  echo "nameserver 1.1.1.1" | sudo tee /etc/resolv.conf
>  echo "nameserver 8.8.8.8" | sudo tee -a /etc/resolv.conf
>  SHELL
>```
>- Puis, ajouter la ligne suivante avant `cfg.vm.provision :shell, :inline => common`:
>```ruby
>cfg.vm.provision :shell, :inline => dns
>```
>

--- 
# II. [`kubectl` & `curl`](#index)

## 1. La commande `kubectl`

```bash
kubectl <verbe> <ressources>
```

- verbes utiles : `get`, `describe`, `apply`, `create`, etc.
- option de verbosité : `-v<level>`
- ressources utiles : `pods`, `nodes`, `deployment`, `replicaset`, etc.
- option de sorties : `-o json`, `-o yaml`,  `-o jsonpath`, `-o wide`, etc.

**Quelques exemples de commandes `kubectl`** :
- Pour afficher les pods :
```bash
kubectl get pods
```

- Pour afficher l'intégralité des pods de tous les namespaces du cluster :
```bash
kubectl get pods --all-namespaces
```

- Pour afficher le pod d'un namespace particulier :
```bash
kubectl get pods -n <nom_du_namespace>
kubectl -n <nom_du_namespace> get pods
```

- Pour avoir une description détaillée (containers, labels, annotation, les états, évènements, etc.) d'un pod en particulier :
```bash
kubectl -n <nom_du_namespace> describe pods <nom_du_pod>
```

- Pour afficher les informations sur les évènements du cluster :
```bash
kubectl get events # Pour le namespace par défaut
kubectl get events -A # Pour afficher les évenements de tous les namespaces
```

- La commande suivante permet d'afficher la sortie d'une commande au format `.yaml`. On retrouve une sortie détaillée `.yaml` du manifeste du pod. Cette commande est utile pour recréer des pods identiques à partir du manifeste d'un pod existant.
```bash
kubectl -n <nom_du_namespace> get pods <nom_du_pod> -o yaml
```

- Pour afficher des informations supplémentaires sur les pods comme les adresses IP, le serveur/node sur lequel il a été alloué, etc.
```bash
kubectl -n <nom_du_namespace> get pods -o wide
```

- Pour afficher des informations supplémentaires sur les nodes.
```bash
kubectl get nodes
kubectl get nodes -o wide
kubectl describe nodes
```

- Pour changer la verbosité des informations affichées, on peut ajouter les arguments `-v<1-9>`. Les informations sont récupérées depuis le fichier *kubeconfig*.
```bash
kubectl describe nodes -v6
kubectl describe pods -v6
```

## 2. `curl`

- Pour utiliser la commande `curl`, il faut un certificat client valide pour accéder à l'APIServer et se connecter au cluster. 
- Dans le fichier *kubeconfig*, on retrouve les certificats encodés en base64 :
```bash
vagrant@k0s1:~$ cat /home/vagrant/.k0s/kubeconfig

apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tL...0tLQo=
    server: https://localhost:6443
  name: local
contexts:
- context:
    cluster: local
    user: user
  name: Default
current-context: Default
kind: Config
users:
- name: user
  user:
    client-certificate-data: LS0tL...tLQo=
    client-key-data: LS0tL...tCg==
```
`certificate-authority-data` est le certificat fourni pour le HTTPS.

`client-certificate-data` et `client-key-data` sont le certificat et sa clé permettant l'authentification auprès du cluster.

- Pour récupérer (dans un fichier .crt ou .key) et désencoder les certificats à l'aide de l'outil `yq` :
```bash
sudo apt install yq
cat ~/.k0s/kubeconfig | yq -r ' .clusters[0].cluster."certificate-authority-data"' | base64 -d > certificate-authority-data.crt
cat /home/vagrant/.k0s/kubeconfig | yq -r ' .users[0].user."client-certificate-data" ' | base64 -d > client-certificate-data.crt
cat /home/vagrant/.k0s/kubeconfig | yq -r ' .users[0].user."client-key-data" ' | base64 -d > client-key-data.key
```

- Une fois les certificats/clés exportées, on peut lancer la commande `curl` en les passants comme arguments :
```bash
curl -iv -L --cert client-certificate-data.crt --key client-key-data.key --cacert certificate-authority-data.crt https://127.0.0.1:6443/api/v1/nodes
```

## 3. Autres commandes utiles

- Pour afficher les informations sur le processus (comme le range d'IP, mode de verbosité, fonctionnement du RBAC, les chemins de certificats clients, etc.)
```bash
ps auxwww | grep kube-apiserver
```
- On peut retrouver les certificats précédemment désencodés dans le répertoire suivant :
```bash
sudo -s
ls /var/lib/k0s/pki
```
- Dans ce même répertoire on peut trouver la configuration du *kubelet* :
```bash
sudo -s
cat /var/lib/k0s/kubelet.conf
```

---

# III. [Lancement des premiers Pods](#index)

Plusieurs méthodes existent pour **créer** un pod :
- Avoir une image et écrire un manifeste
- Utiliser  la commande `kubectl run`
- Utiliser l'option `--dry-run` 
- Utiliser `kubectl explain`
- Utiliser la documentation

>[!CAUTION]
>**Pour rappel** :
> _Kubernetes ne gère pas directement les conteneurs, il orchestre des Pods.  
> Les conteneurs sont créés et exécutés par le container runtime._

## 1. `kubectl run`


- Pour créer un pod avec `kubectl run` on peut lancer la commande suivante :
```bash
kubectl run <pod_name> --image
```

*Exemple* :
```bash
kubectl run nginx --image=nginx
```

>[!IMPORTANT]
>Pour créer un *namespace* avec `kubectl` :
>```bash
>kubectl create ns <nom_du_namespace>
>```
>On peut alors utiliser la commande `kubectl` suivante pour créer un pod dans un *namespace* précis :
>```bash
>kubectl run -n <nom_du_namespace> <pod_name> --image
>```
>*Exemple* :
>```bash
>kubectl run -n demo1 nginx --image=nginx
>```

- Pour obtenir des informations (adresse IP, image, évènements, etc.) sur le pod créé dans le *namespace* :
```bash
kubectl -n demo1 describe pods nginx
```

- Pour détruire un Pod avec la commande `kubectl`, on peut utiliser :
```bash
kubectl delete pods nginx
kubectl -n demo1 delete pods --all # Détruire tous les pods
```

>[!NOTE]
> Par défaut, si plusieurs pods sont supprimés, Kubernetes va attendre que chaque pod soit proprement terminé avant de passer au suivant.

- Il est possible de passer des options lors de la création d'un pod avec `kubectl`, par exemple avec la commande `sleep 10s` :
```bash
kubectl run -n demo1 nginx --image=nginx --command -- sleep 10s
# Ou plus simplement
kubectl run -n demo1 nginx --image=nginx -- sleep 10s
```

>[!NOTE]
>- Dans cet exemple, en passant l'option `sleep 10s` le pod est créé et complété mais entre dans une boucle de plantage.
>- Pour identifier ce problème on peut consulter le manifeste du pod avec la commande :
>```bash
>kubectl -n demo1 get pods nginx -o yaml
>```
>- Dans notre cas, on constate qu'un stratégie de redémarrage automatique est configurée par défaut :
>```yaml
>restartPolicy: Always
>```
>- On peut surcharger (modifier) un élément du manifeste (la `restartPolicy`, dans notre cas) avec la commande suivante :
>```bash
>kubectl run -n demo1 nginx --image=nginx `--overrides='{ "spec": {"restartPolicy": "OnFailure"}}'` --command -- sleep 10s
>```
>- On peut voir que la stratégie a changée dans le manifeste :
>```yaml
>restartPolicy: OnFailure
>```

>[!TIP]
>Créer un pod de debug (avec un conteneur Debian) dans le même *namespace* pour le débug 
>- Création du pod dans le *namespace* `demo1` en activant l'option `-ti` (terminal interactif), sur une image Debian en utilisant `bash`
>```bash
>kubectl -n demo1 run -ti debug --image=debian -- bash
>```
>- Après la création du conteneur, installer des outils pour débugger comme `iputils-ping` ou `curl`
>- Le pod `debug` peut ainsi `ping`, ou `curl` le pod `nginx`.
>- Plus de détails dans la partie [dédiée à ce sujet](6-kubectl_Notions.md#iii-kubectl-exec--kubectl-run)

## 2. L'option `--dry-run`

Cette option permet de **simuler l’exécution d’une commande sans rien créer ni modifier réellement** dans le cluster, elle sert principalement à tester et vérifier avant d’exécuter réellement une action dans Kubernetes.

*Exemple* :
```bash
vagrant@k0s1:~$ kubectl -n demo1 run nginx --image nginx:latest --dry-run=client
pod/nginx created (dry run)
vagrant@k0s1:~$ kubectl -n demo1 get pods
No resources found in demo1 namespace.
```
- `pod/nginx created (dry run)` : la simulation d'un pod est créé
- `No resources found in demo1 namespace.` : le pod n'est pas réellement créé.

La commande `kubectl run ... --dry-run=client -o yaml` génère le manifeste YAML du Pod qui serait créé, sans l’appliquer au cluster :

```bash
vagrant@k0s1:~$ kubectl -n demo1 run nginx --image nginx:latest --dry-run=client -o yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
  namespace: demo1
spec:
  containers:
  - image: nginx:latest
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

On peut exporter puis éditer ce manifeste de pod dans un fichier en lançant la commande suivante :

```bash
kubectl -n demo1 run nginx --image nginx:latest --dry-run=client -o yaml > pod.yml
```

*Fichier `pod.yml` créé sous forme de manifeste* :
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
  namespace: demo1
spec:
  containers:
  - image: nginx:latest
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

## 3. Manifests et outputs

Pour créer un pod depuis un manifeste on peut utiliser deux méthodes:
- Impérative : `kubectl create`
- Déclarative : `kubectl apply`

**Approche impérative** :
```bash
kubectl create -f pod.yml
```
Avec cette approche, "on dit à Kubernetes quoi faire maintenant":
- Kubernetes **crée la ressource une seule fois**
- La commande échoue si la ressource existe déjà
- `kubectl` **n’assure pas le suivi des modifications** du manifeste

**Approche déclarative** :
```bash
kubectl apply -f pod.yml
```
Avec cette approche, on décrit _l’état souhaité_, et Kubernetes s’en charge :
- Kubernetes compare l’état désiré (le manifeste) avec l’état actuel du cluster
- Il **crée la ressource si elle n’existe pas**
- Il **met à jour la ressource si elle existe déjà**
- Les modifications sont **réappliquées de façon idempotente**

### **Détail d’un Pod Kubernetes (vue complète retournée par l’API)**

**Représentation complète d’un objet Pod tel qu’il existe dans le cluster** :
```yaml
apiVersion: v1 # Version de l'API utilisée pour la définition du pod
kind: Pod # Le type de la ressource
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"lb1":"1","run":"nginx"},"name":"nginx","namespace":"demo1"},"spec":{"containers":[{"image":"nginx:latest","name":"nginx","resources":{}}],"dnsPolicy":"ClusterFirst","restartPolicy":"OnFailure"},"status":{}} 
# Rappel de la dernière configuration qui a été ajouté automatiquement par kubectl apply, sous forme d'annotations (un bloc yaml en JSON)
  creationTimestamp: "2026-01-20T13:31:29Z" # La date de création du pod au format ISO 8601
  generation: 1
  labels:
    lb1: "1"
    run: nginx # Le label ajouté au pod
  name: nginx # Le nom du pod, qui doit être unique au sein de son namespace
  namespace: demo1 # Le namespace dans lequel le pod fonctionne
  resourceVersion: "22655" # Identifiant unique de la version du pod
  uid: dddb90f0-d2b4-4023-9801-422b9206d143 # L'ID du pod dans le namespace
spec:
  containers: # Liste des conteneurs qui fonctionnent au sein du pod
  - image: nginx:latest # Le nom de l'image
    imagePullPolicy: Always # Force le container-runtime à télécharger l'image (même si elle est déjà présente localement)
    name: nginx # Le nom de conteneur
    resources: {} # Les requêtes et les limites de ressources définies dans ce conteneur
    terminationMessagePath: /dev/termination-log # Localisation des messages de terminaison du pod
    terminationMessagePolicy: File # La manière dont sera stocker le terminationMessage
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount # Le répertoire dans le conteneur où sont montés les tokens des comptes de service Kubernetes
      name: kube-api-access-x5k5g # Le nom du volume
      readOnly: true # Le montage est en lecture seule 
  dnsPolicy: ClusterFirst # Ici, on utilise la méthode de DNS du Cluster
  enableServiceLinks: true # Permet d'injecter des variables d'environnement pour les services
  nodeName: k0s1 # Le nom de la node où le pod est "scheduled"
  preemptionPolicy: PreemptLowerPriority # Autorise le pod à préempter les pods moins prioritaires
  priority: 0 # Le niveau de priorité de ce pod par rapport à d'autres pods
  restartPolicy: Always # Ici, le conteneur est redémarré automatiquement s’il se termine
  schedulerName: default-scheduler # Le Scheduler utilisé
  securityContext: {} # Permet d'ajouter des éléments de sécurité.
  serviceAccount: default # Le compte de service utilisé
  serviceAccountName: default # Le nom du compte de service utilisé
  terminationGracePeriodSeconds: 30 # Le nombre de secondes attendues par Kubernetes (le kubelet) avant de terminer un pod
  tolerations:
  - effect: NoExecute # Ici, on n'exécute pas de pods
    key: node.kubernetes.io/not-ready # Définit le statut du node. Ici, "not-ready"
    operator: Exists # S'applique à toutes les valeurs de la clé
    tolerationSeconds: 300 # Le nombre de secondes durant lesquels le pod peut rester dans le statut "not-ready"
# Pour résumer : le pod peut rester sur un node "not-ready" pendant 300s avant éviction, valable pour une durée de 300 secondes avant qu'il ne se fasse terminé.
  - effect: NoExecute
    key: node.kubernetes.io/unreachable # Définit le status du node. Cette fois, si le serveur est "unreachable"
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-x5k5g # Le nom du volume mis à disposition pour l'API Kubernetes
    projected:
      defaultMode: 420 # Les permissions en notation octale, ici, équivalente à 0644
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607 # Le temps d'expiration du token en secondes
          path: token # Le chemin, dans ce volume, où le token sera stocké
      - configMap:
          items:
          - key: ca.crt # La clé (ca.crt) de la ConfigMap pour les certificats root du cluster
            path: ca.crt # Le chemin, dans ce volume, où le certificat sera stocké
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status: # Evolution du statut du pod dans le temps
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2026-01-20T13:31:31Z"
    observedGeneration: 1
    status: "True"
    type: PodReadyToStartContainers
  - lastProbeTime: null
    lastTransitionTime: "2026-01-20T13:31:29Z"
    observedGeneration: 1
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2026-01-20T13:31:31Z"
    observedGeneration: 1
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2026-01-20T13:31:31Z"
    observedGeneration: 1
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2026-01-20T13:31:29Z"
    observedGeneration: 1
    status: "True"
    type: PodScheduled
  containerStatuses: # Détails et informations sur les conteneurs
  - containerID: containerd://8f8bf6b77fc0e9e4be69bfc3aba42953cd059a2b07210181476634c7c0cbfb6d
    image: docker.io/library/nginx:latest
    imageID: docker.io/library/nginx@sha256:c881927c4077710ac4b1da63b83aa163937fb47457950c267d92f7e4dedf4aec
    lastState: {}
    name: nginx
    ready: true
    resources: {}
    restartCount: 0
    started: true
    state:
      running: # Statut final du conteneur
        startedAt: "2026-01-20T13:31:30Z"
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-x5k5g
      readOnly: true
      recursiveReadOnly: Disabled
  hostIP: 192.168.144.10 # L'adresse IP de la node ou fonctionne le pod
  hostIPs:
  - ip: 192.168.144.10 # La liste d'adresses IP assignées à la node
  observedGeneration: 1
  phase: Running # L'état actuel du pod
  podIP: 10.244.0.14 # L'adresse IP du pod au sein du cluster Kubernetes
  podIPs: # La liste des adresses IP assignées au pod (ici, une seule)
  - ip: 10.244.0.14
  qosClass: BestEffort # Moyens utilisés par Kubernetes pour assurer la qualité de service. Ajouté automatiquement selon les ressources définies (ou non).
  startTime: "2026-01-20T13:31:29Z" # La date à laquelle le pod a été démarré pour la première fois
```

Un manifeste Kubernetes **décrit uniquement l’état désiré** d’une ressource (`metadata` + `spec`).  
Les champs comme `status`, `uid`, `resourceVersion`, `podIP` ou `nodeName` sont **ajoutés dynamiquement** par Kubernetes et n’apparaissent pas dans un manifeste écrit par l’utilisateur.

---
# IV. [Pods : Multi-conteneurs](#index)

## 1. Création simple d'un pod multi-conteneurs

- Création d'un pod avec un `--dry-run` d'un conteneur _Nginx_, en utilisant sa sortie _yaml_ qui sera redirigée dans un fichier `pod.yml`:
```bash
kubectl run demo1 --image=nginx --dry-run=client -o yaml > pod.yml
```

- Modification du fichier `pod.yml` afin d'y ajouter un autre conteneur `debian` à la suite du premier conteneur `nginx` :
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: demo1
  name: demo1
spec:
  containers:
  - image: nginx
    name: demo1
  - image: debian:latest
    name: demo2
```

- Le pod peut-être lancé avec la commande:
```bash
kubectl apply -f pod.yml
```

>[!NOTE]
>Dans cet exemple, le conteneur `debian` du Pod sera en `CrashLoopBackOff` car l’image Debian ne lance aucun processus de longue durée par défaut. Kubernetes tente donc de redémarrer le conteneur en boucle.
>Pour éviter ce comportement, il est possible d’ajouter une commande dans le manifeste, par exemple un `sleep`.
>```yaml
>- name: debian
>  image: debian:latest
>  command: ["sleep", "infinity"]
>```

## 2. Exemple de manifeste multi-conteneurs

Cet exemple illustre un Pod multi-conteneurs dans lequel :
- un premier conteneur prépare du contenu (téléchargement d’un fichier),
- un second conteneur utilise ce contenu pour le servir via Nginx.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-with-downloader
spec:
  containers:
  - name: downloader
    image: busybox
    command: ['sh', '-c', 'sleep 30s && wget -O /usr/share/nginx/html/index.html https://raw.githubusercontent.com/Semantic-Org/example-github/refs/heads/master/index.html']
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
  volumes:
  - name: shared-data
    emptyDir: {}
```

Les deux conteneurs lancés dans le pod sont les suivants :

**Conteneur `downloader`** :
```yaml
  - name: downloader
    image: busybox
    command: ['sh', '-c', 'sleep 30s && wget -O /usr/share/nginx/html/index.html https://raw.githubusercontent.com/Semantic-Org/example-github/refs/heads/master/index.html && sleep infinity']
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
```
- Ce conteneur a un rôle **ponctuel**
- Il télécharge un fichier `index.html`
- Le fichier est écrit dans un répertoire partagé avec le conteneur Nginx

**Conteneur `nginx`** :
 ```yaml
   - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
 ```
- Ce conteneur expose un serveur HTTP
- Il sert le contenu présent dans `/usr/share/nginx/html`
- Le contenu est fourni par le conteneur `downloader`

**Volumes partagés** :

- Les deux conteneurs montent un volume nommé `shared-data`. Ce volume est monté a partir de son nom `shared-data` dans `/usr/share/nginx/html`. Le volume est **monté uniquement dans les conteneurs qui le déclarent** via `volumeMounts`.
- Dans le manifeste, ce volume est défini au niveau du pod. Le volume est défini dans la section `volumes` du pod, puis monté dans les conteneurs à l’aide de `volumeMounts`.
- Il permet aux conteneurs de partager des fichiers via un même chemin.

Une fois le pod lancé, la page index de `nginx` peut être consultée à l'aide de la commande suivante:
```bash
curl <IP_CONTENEUR>
```

>[!NOTE]
>Dans un manifeste, il est possible de passer des variables d'environnement, par exemple, il pourrait être ajouté la commande suivante :
>```yaml
>command: ['sh', '-c', 'sleep 30s && wget -O /usr/share/nginx/html/index.html https://raw.githubusercontent.com/Semantic-Org/example-github/refs/heads/master/index.html']
>```
>Cette commande affichera une incrémentation de 1 toutes les secondes sur la page d'index de `nginx`.

---
# V. [ReplicaSets](#index)

## 1. Création de manifest de ReplicaSet

- Pour créer simplement le manifest d'un ReplicaSet, il est possible de se baser sur le manifeste d'un déploiement. On peut créer un `--dry-run` d'un déploiement  et le mettre dans un fichier `rs.yml` :

```bash
kubectl create deployment nginx --image nginx --dry-run=client -o yaml > rs.yml
```

- La commande génère un manifeste de **Deployment**, qui servira de base pour créer un ReplicaSet :

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```

- Certaines sections générées automatiquement ne sont pas nécessaires et peuvent être supprimées. Pour créer le manifeste du ReplicaSet, on édite le manifeste de **Deployment** :

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
```

- De la même manière que pour un Pod, on peut créer le ReplicaSet avec la commande suivante :

```bash
kubectl apply -f rs.yml
```

- Les trois Pods décrits dans le manifeste ont été créés : 

```bash
vagrant@k0s1:~$ kubectl get pods
NAME          READY   STATUS    RESTARTS   AGE
nginx-6cgj4   1/1     Running   0          25s
nginx-8z52n   1/1     Running   0          25s
nginx-kw7zc   1/1     Running   0          25s
vagrant@k0s1:~$ kubectl get rs
NAME    DESIRED   CURRENT   READY   AGE
nginx   3         3         3       47s
```

- Pour supprimer un ReplicaSet (et la suppression des Pods qu’il gère) :

```bash
kubectl delete -f rs.yml
```

## 2. Scaling d'un ReplicaSet

- Le scaling d'un ReplicaSet permet d'ajuster dynamiquement le nombre de Pods qu'il gère. Dans l'exemple suivant on va ajuster le nombre de Pods en cours d'exécution de 3 à 10 :

```bash
kubectl scale replicaset nginx --replicas 10
```

```bash
vagrant@k0s1:~$ kubectl get rs
NAME    DESIRED   CURRENT   READY   AGE
nginx   10        10        10      54s
```

>[!NOTE]
>Il est possible de revenir à un nombre inférieur de pods en changeant simplement le nombre de `replicas` de la commande.
>Ces changements sont appliqués immédiatement par le ReplicaSet controller, sans interruption du service.

- Il est aussi possible d'éditer directement le manifeste avec la commande :

```bash
kubectl edit rs nginx
```

## 3. Illustration du fonctionnement des labels

- Pour illustrer le mécanisme de correspondance des labels, on peut créer un Pod simple nommé `demo1` avec le label `app=nginx` :

```bash
kubectl run demo1 --image=nginx --labels app=nginx
```

- On créer un ReplicaSet demandant un nombre de pods (par exemple `5`) qui doivent correspondre au label `app=nginx`.

```bash
vagrant@k0s1:~$ kubectl apply -f rs.yml
replicaset.apps/nginx created
vagrant@k0s1:~$ kubectl get rs
NAME    DESIRED   CURRENT   READY   AGE
nginx   5         5         5       17s
vagrant@k0s1:~$ kubectl get pods --show-labels
NAME          READY   STATUS    RESTARTS   AGE     LABELS
demo1         1/1     Running   0          13m     app=nginx
nginx-br99p   1/1     Running   0          9m45s   app=nginx
nginx-bsz7t   1/1     Running   0          9m45s   app=nginx
nginx-clq99   1/1     Running   0          9m45s   app=nginx
nginx-gvggk   1/1     Running   0          9m45s   app=nginx
```

Après la création du ReplicaSet, on peut observer que **5 Pods sont en cours d’exécution**, comme demandé.  
Le Pod `demo1`, bien qu’il ait été créé indépendamment du ReplicaSet, est **pris en charge** par celui-ci grâce à la correspondance des labels.

Tous les Pods possédant le label `app=nginx` font désormais partie du périmètre de gestion du ReplicaSet.

>[!TIP]
>Pour afficher les Pods qui ont le même label :
> ```bash
> kubectl get pods -l app=nginx
> ```

## 4. Exemple d'auto guérison

- Dans le ReplicaSet suivant, on lui décrit `5` Pods désirés :

```bash
vagrant@k0s1:~$ kubectl get pods
NAME          READY   STATUS    RESTARTS   AGE
nginx-2kkjd   1/1     Running   0          8s
nginx-47stq   1/1     Running   0          8s
nginx-cgh7r   1/1     Running   0          8s
nginx-ss57p   1/1     Running   0          8s
nginx-zr78l   1/1     Running   0          8s
```

- Si l'on supprime des Pods, par exemple de Pod `nginx-2kkjd` :

```bash
kubectl delete pods nginx-2kkjd
```

- Un autre Pod sera automatiquement recréé :

```bash
vagrant@k0s1:~$ kubectl get pods
NAME          READY   STATUS    RESTARTS   AGE
nginx-47stq   1/1     Running   0          3m6s
nginx-6wpxq   1/1     Running   0          5s
nginx-cgh7r   1/1     Running   0          3m6s
nginx-ss57p   1/1     Running   0          3m6s
nginx-zr78l   1/1     Running   0          3m6s
```

- Le ReplicaSet, via son **controller**, s’assure en permanence que le nombre de Pods correspondant à son selector est conforme à l’état désiré décrit dans le manifeste.

## 5. Aperçu des annotations

- Les annotations sont des informations supplémentaires associées aux ressources Kubernetes, utilisées par certains contrôleurs ou composants pour adapter leur comportement.
- Pour ajouter une annotation il suffit d'ajouter le champ `spec.template.metadata.annotations` :

>[!NOTE]
>Dans l'exemple suivant, lors d’une opération de _scale-down_, le ReplicaSet controller peut utiliser l’annotation `controller.kubernetes.io/pod-deletion-cost` pour déterminer **l’ordre de suppression des Pods**.  
>Les Pods avec la valeur la plus faible sont supprimés en priorité.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
      annotations:
        controller.kubernetes.io/pod-deletion-cost: "1"
```

Ou plus simplement avec la commande suivante, qui permet de modifier dynamiquement le comportement de suppression sans redéployer le ReplicaSet :

```bash
kubectl annotate pods <nomDuPod> controller.kubernetes.io/pod-deletion-cost=1
```

---
# VI. [**`kubectl` : contextes et optimisations**](#index)

## 1. Les contextes

- Un **context kubectl** définit l’environnement de travail courant : il associe un cluster, un utilisateur et un namespace.
- Les **contextes** permettent de gérer plusieurs clusters ou environnements (dev, staging, prod) depuis un même poste.
- Les **contextes** sont définis dans le `$KUBECONFIG` (`~/.kube/config`) de la manière suivante :

```yaml
contexts:
- name: dev
  context:
    cluster: dev-cluster
    user: dev-user
    namespace: dev
```

>[!NOTE]
>**Rappel** : 
>Kubectl lit sa configuration depuis un ou plusieurs fichiers kubeconfig, dans lesquels sont définis les clusters, utilisateurs et contexts.

#### Commandes `kubectl` utiles pour la gestion du `kubeconfig`
```bash
# afficher les versions

kubectl version

# lister les configs

kubectl config view

# lister et changer le contexte

kubectl config get-contexts
kubectl config current-context
kubectl config use-context <nomDuContexte>
kubectl config set-context --current --namespace=<nomDuNamespace>

# créer et activer un contexte

kubectl config set-context <nomDuContexte> --namespace <nomDuNamespace> --user <nomUtilisateur> --cluster <nomDuCluster>

# créer un cluster

kubectl config set-cluster <nomDuCluster> --server=https://<api-server>:6443 --certificate-authority=ca.crt

# créer un utilisateur

kubectl config set-credentials <nomUtilisateur> --token=<token>
kubectl config set-credentials <nomUtilisateur> --client-certificate=user.crt --client-key=user.key

# utiliser un contexte

kubectl config use-context <nomDuContexte>

# supprimer un contexte

kubectl config delete-context <nomDuContexte>    
kubectl get namespace # vérifier les namespaces existants
kubectl create namespace <nomDuNamespace> # créer un namespace pour un nouveau contexte

# autre commandes utiles

alias kprod='kubectl config use-context production' 

kubectl get all # ne retourne pas toutes les ressources, uniquement un sous-ensemble
kubectl get pods,replicaset
kubectl ... -w # observe les ressources en temps réel
watch "kubectl ..." # observe les ressources en temps réel

kubectl get componentstatuses # commande dépréciée dans les versions récentes
```

>[!IMPORTANT]
>**Un contexte doit être activé pour être utilisé.**
>
>La création d’un contexte ne modifie pas le contexte courant. Il doit être explicitement activé pour être utilisé par kubectl.

>[!CAUTION]
>Toujours vérifier le contexte courant (`kubectl config current-context`) avant d’exécuter des commandes sur un cluster sensible.
## 2. Optimisation de la CLI

Les commandes `kubectl` pouvant rapidement devenir longues et répétitives, l’optimisation de la CLI est une **bonne pratique** pour gagner en efficacité, limiter les erreurs et améliorer l’expérience utilisateur.

### 2.1 Installation de l'auto-complétions :

```bash
sudo apt-get install bash-completion
echo "source <(kubectl completion bash)" >> ~/.bashrc
source ~/.bashrc
```

### 2.2 Attribution d'alias :

- Pour simplifier les commandes `kubectl` des alias peuvent être configurés :
```bash
alias k='kubectl'
alias kcc='kubectl config current-context'
alias kg='kubectl get'
alias kga='kubectl get all --all-namespaces'
alias kgp='kubectl get pods'
alias kgs='kubectl get services'
alias ksgp='kubectl get pods -n kube-system'
alias kuc='kubectl config use-context'
```

- L’alias `k` permet de raccourcir la commande `kubectl`, mais, par défaut, l’auto-complétion `kubectl` ne s’applique pas aux alias. Pour conserver l’auto-complétion avec cet alias, il est nécessaire de configurer explicitement Bash.
```bash
echo 'complete -F __start_kubectl k' >> ~/.bashrc
```
### 2.3 Installation de `kube-ps1` :

- `kube-ps1` est un outil permettant d'afficher les contextes et namespaces couramment utilisés :
```bash
wget https://github.com/jonmosco/kube-ps1/archive/refs/tags/v0.9.0.tar.gz 
tar xzvf v0.9.0.tar.gz 
sudo cp kube-ps1-0.9.0/kube-ps1.sh /usr/local/bin/
sudo chmod +x /usr/local/bin/kube-ps1.sh
```

- Ajouter au début du fichier `~/.bashrc` :
```bash
source /usr/local/bin/kube-ps1.sh
```

- Puis ajouter `$(kube_ps1)` sur la ligne du prompt :
```
PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\] $(kube_ps1) \$'
```

- Lancer la commande `source ~/.bashrc` pour rafraichir le terminal. De nouveaux éléments informatifs, comme le nom du contexte et le namespace actifs, sont alors affichés :
```bash
vagrant@k0s1:~ (⎈|Default:N/A) $
```

### 2.4 Installation de `kubens` :

- `kubens` est un outil en ligne de commande permettant de **changer rapidement de namespace** dans le contexte `kubectl` courant. Il simplifie la navigation entre namespaces sans avoir à modifier manuellement le `kubeconfig` ou à répéter l’option `--namespace` dans chaque commande.

- Installation de `kubens` :
```bash
sudo curl -sL https://github.com/ahmetb/kubectx/releases/download/v0.9.5/kubens -o /usr/local/bin/kubens && sudo chmod +x /usr/local/bin/kubens
```

- Afficher les namespaces existant :
```bash
vagrant@k0s1:~ (⎈|Default:N/A) $ kubens
default
k0s-autopilot
kube-node-lease
kube-public
kube-system
```

- Changer rapidement de namespace dans le contexte `kubectl` courant :
```bash
vagrant@k0s1:~ (⎈|Default:N/A) $ kubens kube-system
Context "Default" modified.
Active namespace is "kube-system".
```

### 2.5 Installation de `kubectx` :

- `kubectx` facilite la **gestion et le changement de contextes `kubectl`**. Il permet de basculer rapidement entre plusieurs clusters ou environnements, réduisant les risques d’erreurs et améliorant la productivité.

- Installation de `kubectx`:
```bash
sudo curl -sL https://github.com/ahmetb/kubectx/releases/download/v0.9.5/kubectx -o /usr/local/bin/kubectx && sudo chmod +x /usr/local/bin/kubectx
```

- Changer de contexte rapidement avec la commande :
```bash
vagrant@k0s1:~ (⎈|Default:default) $ kubectx Default
Switched to context "Default".
```

>[!NOTE]
>`kubens` et `kubectx` peuvent aussi avoir des alias configurés :
>- `kubens` = `kn`
>- `kubectx` = `kx`

### 2.6 Installation de `k9s` :

- `k9s` est une interface en mode terminal (TUI) pour Kubernetes. Il offre une **visualisation en temps réel** des ressources du cluster et permet d’interagir avec elles de manière plus intuitive qu’avec les commandes `kubectl` classiques.

- Installation de `k9s` :
```bash
sudo -s
wget https://github.com/derailed/k9s/releases/download/v0.32.5/k9s_linux_amd64.deb && apt install ./k9s_linux_amd64.deb && rm k9s_linux_amd64.deb
exit
```

- Lancer `k9s` :
```bash
k9s
```

[**Documentation `k9s`**](https://k9scli.io/)

>[!NOTE]
>`k9s` utilise le contexte `kubectl` courant et respecte le namespace actif.

---
# VII. [**Deployments**](#index)

## 1. Création d'un déploiement

### 1.1 Création :

- Pour créer rapidement un Deployment depuis la CLI :
```bash
kubectl create deployment <nomDuDeploiement> --image=<nomImage> --replicas=<nombreDeReplicas>
```

### 1.2 Consultation et modification du manifeste :

- Le manifeste du Deployment généré peut être consulté et modifié avec :
```bash
kubectl edit deploy nginx
```

- On y retrouve notamment la stratégie de déploiement par défaut :
```yaml
...
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
...
```

>[!NOTE]
>`maxSurge` et `maxUnavailable` peuvent être définis en **valeur absolue** ou en **pourcentage**. Dans ce cas, lors d'une mise à jour, jusqu’à 25 % du nombre total de Pods peuvent être créés en plus pour mettre à jour progressivement les pods. 

### 1.3 Lien entre Deployment, ReplicaSet et Pods :

- Dans la description du pod retourné par la commande `kubectl describe pods <nomDuPodDansLeDeployment>`; on peut observer le ReplicaSet qui le contrôle :

```yaml
Controlled By:  ReplicaSet/nginx-66686b6766
```

>[!NOTE]
>Dans un Deployment le nom des Pods suit le format :
>```bash
><nomDuDeployment>-<idDuReplicaSet>-<idDuPod>
># Exemple :
>nginx-66686b6766-26bc7
>```

- De la même manière, la description d’un ReplicaSet indique le Deployment parent avec la commande `kubectl describe rs nginx-66686b6766`.

```yaml
Controlled By:  Deployment/nginx
```

## 2. `RollingUpdate` et historique

### 2.1 Mise à jour d’un Deployment

- Mettre à jour une image depuis la CLI avec la commande :
```bash
kubectl set image deployment <nomDuDeployment> <nomImage>=<image>
```

>[!NOTE]
>Cette action déclenche automatiquement un **rolling update**.

- On peut voir dans la description du Deployment de nombre de mise à jour. Cette annotation correspond au **numéro de version interne** du Deployment utilisé pour le rollback :
```yaml
Annotations:            deployment.kubernetes.io/revision: 2
```

- Chaque mise à jour incrémente la révision du Deployment :
```bash
vagrant@k0s1:~ (⎈|Default:default) $ kubectl rollout history deployment <nomDuDeployment>
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

- La colonne `CHANGE-CAUSE` peut être assortie d'informations permettant de suivre aisément l'historique des versions. Il suffit d'ajouter ou de modifier le champ suivant dans le manifeste de Deployment :
```bash
nano <manifestDeDeployment>.yml
```

```yaml
metadata:
  labels:
    app: my-app
  name: my-app
  annotations:
    kubernetes.io/change-cause: <contenuInformatif>
```

>[!NOTE]
>**Déprécié** :
>- On peut utiliser la commande pour ajouter une information dans la colonne `CHANGE-CAUSE` :
>```bash
>kubectl rollout history deployment <nomDuDeployment> --record
>```
>```bash
>vagrant@k0s1:~ (⎈|Default:default) $ kubectl rollout history deployment
>deployment.apps/nginx
>REVISION  CHANGE-CAUSE
>1         <none>
>2         <none>
>3         kubectl set image deployments nginx nginx=nginx:1.9.2 --record=true
>```
>- On peut retrouver cette information dans la description du Deployment sous forme d'annotation :
>```yaml
>Annotations:           deployment.kubernetes.io/revision: 3
>                        kubernetes.io/change-cause: kubectl set image deployments nginx nginx=nginx:1.9.2 --record=true
>```

>[!TIP]
>Pour définir le nombre d'éléments affichés avec la commande :
>```bash
>kubectl rollout history deployment <nomDuDeployment>
>```
>On peut décrire le nombre de révisions affichée dans le manifeste de Deployment :
>```yaml
>...
>spec:
>revisionHistoryLimit: 5
>...
>```
>La commande affichera dorénavant 5 révisions en plus de la version courante.

- Pour avoir des informations sur une révision particulière, on peut utiliser la commande :
```bash
kubectl rollout history deployment <nomDuDeployment> --revision=<numeroDeLaRevision>
```

- Pour suivre en temps réel l’état d’un déploiement (_rollout_) et vérifier qu’il s’est terminé avec succès :
```bash
kubectl rollout status deployment <nomDuDeployment>
```

### 2.2 Rollback

Lors d’une mise à jour, Kubernetes **ne supprime pas immédiatement les anciens `ReplicaSets`** associés au `Deployment`.  
Chaque version du Deployment correspond à un `ReplicaSet` distinct.
- Le `ReplicaSet` actif (dernière version) gère les Pods courants
- Les `ReplicaSets` des versions précédentes sont **conservés**, avec un nombre de réplicas généralement à `0`

- Pour revenir à une version antérieure du Deployment :
```bash
kubectl rollout undo deployment <nomDuDeployment> --to-revision=<numeroDeLaRevision>
```

>[!IMPORTANT]
>Un rollback de `Deployment` consiste à **changer quel `ReplicaSet` est actif**, pas à déplacer des Pods entre versions.
>
>Lors d’un rollback vers une version antérieure :
>- Kubernetes **réutilise le `ReplicaSet` correspondant** à la version ciblée
>- Le nombre de réplicas de ce `ReplicaSet` est **augmenté**
>- Les Pods de la version actuelle sont **progressivement supprimés**
>- De nouveaux Pods sont **créés à partir de l’ancien `ReplicaSet`**

### 2.3 Mettre en pause et reprendre un déploiement

- Il est possible de stopper un Deployment en cours d'exécution :
```bash
kubectl rollout pause deployment <nomDuDeployment>
```

- Pour reprendre un Deployment en pause :
```bash
kubectl rollout resume deployment <nomDuDeployment>
```
## 3. Fonctionnement interne

Pour afficher la description détaillée d’un Deployment, on peut utiliser la commande :
```bash
kubectl describe deploy <nomDuDeployment>
```

Dans cette description, on peut observer le champ `ownerReferences`, qui permet d’identifier les relations entre les ressources Kubernetes.

```yaml
...
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: nginx-66686b6766
    uid: 818c0174-df74-4b6e-a060-80d5c3a0bf14
  resourceVersion: "64502"
  uid: d58d2d68-9d6f-488e-8648-c6728b9ad8ed
...
```

Ce champ indique que le **Deployment est propriétaire d’un ReplicaSet**.  
Cela signifie que Kubernetes utilise le Deployment pour créer et gérer automatiquement ce ReplicaSet.

De la même manière, dans la description d’un ReplicaSet, `ownerReferences` indique qu’il est contrôlé par un Deployment.

```yaml
...
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: Deployment
    name: nginx
    uid: b7828cf6-81c0-40fc-9b9a-8df5929abb30
  resourceVersion: "65303"
  uid: 818c0174-df74-4b6e-a060-80d5c3a0bf14
...
```

Cela montre que le **ReplicaSet est rattaché au Deployment**, et qu’il est supprimé automatiquement si le Deployment est supprimé.

---

