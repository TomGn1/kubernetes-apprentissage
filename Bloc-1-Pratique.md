## **Mise en pratique des premières** [**notions**](Bloc-1-Notions.md#iii-pods)

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

IV. [Pods : Multi-conteneurs](#iv-pods--multi-conteneurs)

---
# I. [Mise en place d'un cluster k0s](#index)

- Installation d'une seule node cluster
	- Elle devra assumé la fonction de worker
	- Documentation :
		- k0s : https://docs.k0sproject.io/stable/install/
		- Kubectl : https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
- Liens de la vidéo de mise en pratique : https://youtu.be/jRKhHYoFsmU?list=PLn6POgpklwWo6wiy2G3SjBubF6zXjksap
- Liens du dépôt de la pratique : https://gitlab.com/xavki/kubernetes-tutorials-new-version/-/tree/main/k0s-stack?ref_type=heads
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
>- Environnement prêt à l'emplois dès le premier démarrage
>### Isolation de l'environnement
>- Pas d'impact sur le système hôte
>- Pas de "pollution" des configurations locales
>- Facile à réinitialiser en cas d'erreur
>### Installation et documentation
>- Installation : https://developer.hashicorp.com/vagrant/install 
>- Documentation : https://developer.hashicorp.com/vagrant/docs

**Vagrant** me permettra donc :
- Créer une ou plusieurs machines virtuelles Linux
- Installé et configurer **k0s** pour former un cluster Kubernetes fonctionnel
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
- Le fichier `kubeconfig` est le **certificat client** qui permet d'**accédé à l'APIServeur**, il doit être placé dans un emplacement facilement accessible. Créer ce dossier et ajuster les permissions :
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
>Lors de l'a création de la VM Debian avec Vagrant et VirtualBox, il peut y avoir un problème de lenteur sur les téléchargements. Ce problème est souvent causé par l'association DNS proxy NAT et `systemd-resolved`. Afin de résoudre ce problème modifier le *vagrantfile* pour désactiver `systemd-resolved` et éditer manuellement le fichier */etc/resolv.conf*.
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
- Ressources utiles : `pods`, `nodes`, `deployment`, `replicaset`, etc.
- option de sorties : `-o json`, `-o yaml`,  `-o jsonpath`, `-o wide`, etc.

**Quelques exemples de commandes `kubectl`** :
- Pour afficher les pods :
```bash
kubectl get pods
```
- Pour afficher l'intégralité des pods de tout les namespaces du cluster :
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
kubectl get events -A # Pour afficher les évenements de tout les namespaces
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
`certificate-authority-data` est le certificats fournit pour le HTTPS.

`client-certificate-data` et `client-key-data` sont le certificat et sa clé permettant l'authentification auprès du cluster.

- Pour récupérer (dans un fichier .crt ou .key) et désencoder les certificats à l'aide de l'outils `yq` :
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

- Pour détruire un pod avec la commande `kubectl`, on peut utiliser :
```bash
kubectl delete pods nginx
kubectl -n demo1 delete pods --all # Détruire tout les pods
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
>- Dans cet exemple, en passant l'option `sleep 10s` le pod est créé et complété mais entre dans un boucle de plantage.
>- Pour identifier ce problème on peut consulter le manifeste du pod avec la commande :
>```bash
>kubectl -n demo1 get pods nginx -o yaml
>```
>- Dans notre cas, on constate qu'un stratégie de redémarrage automatiques est configurée par défaut :
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
# Rappel de la dernière configuration qui a été ajouté automatiquement par kubectl apply, sous forme d'annotations (un bloc yaml en.json)
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
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount # Le répertoire dans le conteneur où sont montésles tokens des comptes de service Kubernetes
      name: kube-api-access-x5k5g # Le nom du volume
      readOnly: true # Le montage est en lecture seule 
  dnsPolicy: ClusterFirst # Ici, on utilise la méthode de DNS du Cluster
  enableServiceLinks: true # Permet d'injecter des variables d'environnement pour les services
  nodeName: k0s1 # Le nom de la node où le pod est "scheduled"
  preemptionPolicy: PreemptLowerPriority # Autorise le pod à préempter les pods moins prioritaires
  priority: 0 # Le niveau de priorité de ce pod par rapport à d'autres pods
  restartPolicy: Always # Ici, le conteneur est redémarré automatiquement s’il se termine
  schedulerName: default-scheduler # Le Scheduler utilisé
  securityContext: {} # Permet d'ajouter des éléments de sécutité.
  serviceAccount: default # Le compte de service utilisé
  serviceAccountName: default # Le nom du compte de service utilisé
  terminationGracePeriodSeconds: 30 # Le nombre de secondes attendues par Kubernetes (le kubelet) avant de terminer un pod
  tolerations:
  - effect: NoExecute # Ici, on n'exécute pas de pods
    key: node.kubernetes.io/not-ready # Définit le statut du node. Ici, "not-ready"
    operator: Exists # S'applique à toutes les valeurs de la clé
    tolerationSeconds: 300 # Le nombre se secondes durant lesquels le pod peut rester dans le statut "not-ready"
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
  startTime: "2026-01-20T13:31:29Z" # La date à laquel le pod a été démarré pour la première fois
```

Un manifeste Kubernetes **décrit uniquement l’état désiré** d’une ressource (`metadata` + `spec`).  
Les champs comme `status`, `uid`, `resourceVersion`, `podIP` ou `nodeName` sont **ajoutés dynamiquement** par Kubernetes et n’apparaissent pas dans un manifeste écrit par l’utilisateur.

---
# IV. [Pods : Multi-conteneurs](#index)

