
Apprentissage progressif de Kubernetes en 8 blocs. Après chaque bloc ajout des nouvelles compétences acquises au projet DaaS-k8s-v2.
## Kubernetes : Les bases

I. [**Introduction Kubernetes**](1-Les-Bases_Notions.md#i-introduction-kubernetes)
1. [Présentation de Kubernetes](1-Les-Bases_Notions.md#1-présentation-de-kubernetes)
2. [Rappel sur la conteneurisation](1-Les-Bases_Notions.md#2-rappel-sur-la-conteneurisation)
3. [Contexte de Kubernetes](1-Les-Bases_Notions.md#3-contexte-de-kubernetes)
4. [Écosystème et déploiement de Kubernetes](1-Les-Bases_Notions.md#4-écosystème-et-déploiement-de-kubernetes)
5. [Valeur et apports fonctionnels de Kubernetes](1-Les-Bases_Notions.md#5-valeur-et-apports-fonctionnels-de-kubernetes)
6. [Exemples de ce que permet Kubernetes](1-Les-Bases_Notions.md#6-exemples-de-ce-que-permet-kubernetes)

II. [**Architecture Cluster K8s**](1-Les-Bases_Notions.md#ii-architecture-cluster-k8s)
1. [Déclaratif vs Impératif](1-Les-Bases_Notions.md#1-déclaratif-vs-impératif)
2. [Master et Worker](1-Les-Bases_Notions.md#2-master-et-worker)
3. [Introduction au Pod](1-Les-Bases_Notions.md#3-introduction-au-pod)
4. [Fonctionnement d'un cluster Kubernetes](1-Les-Bases_Notions.md#4-fonctionnement-dun-cluster-kubernetes)
5. [Scheduling](1-Les-Bases_Notions.md#5-scheduling)

III. [**L'APIServer**](1-Les-Bases_Notions.md#iii-lapiserver)
1. [Point d'entrée de l'APIServer : les clients](1-Les-Bases_Notions.md#1-point-dentrée-de-lapiserver--les-clients)
2. [Kube-APIServer : Authentification et Autorisation](1-Les-Bases_Notions.md#2-kube-apiserver--authentification-et-autorisation)
3. [KubeConfig](1-Les-Bases_Notions.md#3-kubeconfig)

IV. [**Pods**](1-Les-Bases_Notions.md#iv-pods)
1. [Rappel sur le pod](1-Les-Bases_Notions.md#1-rappel-sur-le-pod)
2. [Exemple de fonctionnement d'un Pod](1-Les-Bases_Notions.md#2-exemple-de-fonctionnement-dun-pod)
3. [Contenu d'un Pod](1-Les-Bases_Notions.md#3-contenu-dun-pod)
4. [Exemple et structure de déploiement](1-Les-Bases_Notions.md#4-exemple-et-structure-de-déploiement)
5. [Multi-conteneurs](1-Les-Bases_Notions.md#5-multi-conteneurs)
6. [Les `status`](1-Les-Bases_Notions.md#6-les-status)

V. [**ReplicaSets**](1-Les-Bases_Notions.md#v-replicasets)
1. [ReplicaSets : présentation](1-Les-Bases_Notions.md#1-replicasets--présentation)
2. [`labels` et `selector`](1-Les-Bases_Notions.md#2-labels-et-selector)

VI. [**Deployments**](1-Les-Bases_Notions.md#vi-deployments)
1. [Deployments : présentation](1-Les-Bases_Notions.md#1-deployments--présentation)
2. [Rollouts strategies](1-Les-Bases_Notions.md#2-rollouts-strategies)

## Kubernetes : Les services

6. **Services - ClusterIP** - Exposition interne cluster

7. **Services - NodePort** - Exposition externe via port node

8. **Services - LoadBalancer** - Intégration cloud providers

9. **Ingress Introduction** - Routage HTTP/HTTPS

10. **Ingress Traefik** (K3s intégré) - Configuration pratique

## Kubernetes : Les volumes

11. **Volumes** - Types volumes (emptyDir, hostPath, etc.)

12. **PersistentVolumes (PV)** - Abstraction stockage

13. **PersistentVolumeClaims (PVC)** - Demande stockage

14. **StorageClasses** - Provisioning dynamique

15. **NFS Volumes** - Montage NFS dans pods

## Kubernetes : Configuration & Secrets

16. **ConfigMaps** - Configuration externalisée

17. **Secrets** - Credentials sensibles

18. **Environment Variables** - Injection config dans pods

19. **Volume ConfigMaps/Secrets** - Montage fichiers config

20. **Best Practices Config** - Patterns recommandés

## Helm :

21. **Introduction Helm** - Pourquoi Helm, concepts charts

22. **Installation Helm** - Setup Helm 3

23. **Déployer Charts** - Helm install, repos publics

24. **Values.yaml** - Customisation charts

25. **Créer Chart Custom** - Structure, templates basiques

## `kubectl` :

26. **kubectl logs** - Consulter logs containers

27. **kubectl describe** - Diagnostiquer problèmes ressources

28. **kubectl exec** - Shell dans pod running

29. **Events K8s** - Événements cluster

30. **Patterns Erreurs Courantes** - ImagePullBackOff, CrashLoopBackOff, etc.

## Kubernetes : Sécurité & accès

31. **RBAC - Rôles & RoleBindings** - Contrôle accès ressources

32. **ServiceAccounts** - Identité pods

33. **Network Policies** - Segmentation réseau pods

34. **Security Context** - runAsUser, capabilities, etc.

35. **Pod Security Standards** - Restricted, Baseline, Privileged

## Kubernetes : Ressources & workloads

36. **Namespaces** - Isolation logique ressources

37. **Resource Quotas** - Limites ressources namespace

38. **LimitRanges** - Limites par pod/container

39. **DaemonSets** - 1 pod par node (monitoring, logs)

40. **StatefulSets** - Apps stateful (bases de données)