<a id="index"></a>
**Index** :

I. [**`kubectl logs`**](#i-kubectl-logs) - Consulter logs containers

II. [**`kubectl describe`**](#ii-kubectl-describe) - Diagnostiquer problèmes ressources

III. [**`kubectl exec` & `kubectl run`**](#iii-kubectl-exec--kubectl-run) - Shell dans pod running

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

## 2. `kubectl attach`

`kubectl attach` ne lance **aucun nouveau processus**.

Cette commande se contente de se connecter aux flux d'entrée/sortie (`stdin`, `stdout`, `stderr`) du processus principal déjà en cours dans le conteneur.

On agit donc comme un terminal branché directement sur l'application existante :
- ce que l'on tape → va dans le `stdin` du processus
- ce que le programme affiche → revient via `stdout` / `stderr`

On interagit donc avec le processus tel qu'il tourne déjà, sans en créer un nouveau.

>[!IMPORTANT]
>On en se place jamais _dans_ un conteneur.
>Kubernetes ne fait que créer ou connecter un processus partageant les namespaces du conteneur.

## 3. `kubectl run`

`kubectl run` permet de **crée un pod**.

Dans un contexte de résolution de problèmes, il peut être utiliser pour générer des pods l

---
<a id="iv-events-k8s"></a>
# IV. [**Events K8s**](#index)



---
<a id="v-patterns-derreurs-courantes"></a>
# V. [**Patterns d'Erreurs Courantes**](#index)

