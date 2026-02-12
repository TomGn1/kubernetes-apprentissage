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

- `kubectl exec` : Un processus est lancé dans les même namespaces que le processus initial du pod ciblé. Quand on lance ce processus on ne va pas "dans le pod" mais on se place "à côté" de celui-ci, pour avoir le même point de vu que lui.

- `kubectl attach` : Permet de streamer les flux `stdin`, `sdtout` et `stderr`. On se "plug" sur le stdin, pour lancer des commandes sur sur le stdin du processus, le stdout va retourner les éléments des commandes lancées et stderr va afficher les éléments en erreurs.

>[!NOTE]
>On en se place jamais dans un conteneur mais à "côté" de celui-ci, dans son namespaces.
>


---
<a id="iv-events-k8s"></a>
# IV. [**Events K8s**](#index)



---
<a id="v-patterns-derreurs-courantes"></a>
# V. [**Patterns d'Erreurs Courantes**](#index)

