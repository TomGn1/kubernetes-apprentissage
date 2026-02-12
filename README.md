# Kubernetes - Apprentissage

Mes notes d'apprentissage Kubernetes, synchronisées automatiquement depuis mon vault Obsidian privé.


>[!NOTE]
>## CI/CD Automatisé
>
>Ce repository est mis à jour automatiquement via GitHub Actions :
>
>- **Source** : Vault Obsidian privé (notes personnelles)
>- **Déclencheur** : Push sur le dossier `Kubernetes/` du repo privé
>- **Action** : Synchronisation automatique vers ce repo public
>- **Workflow** : `.github/workflows/sync-k8s.yml`

# Contenu

## 1. Kubernetes : Les bases

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

## 2. Kubernetes : Les services

I. [**Services - Présentation**](2-Les-Services_Notions.md#i-services--présentation)

II. [**Services - ClusterIP**](2-Les-Services_Notions.md#ii-services--clusterip) - Exposition interne cluster

III. [**Services - NodePort**](2-Les-Services_Notions.md#iii-services--nodeport) - Exposition externe via port node

IV. [**Services - LoadBalancer**](2-Les-Services_Notions.md#ii-services--loadbalancer) - Intégration cloud providers

V. [**Ingress Introduction**](2-Les-Services_Notions.md#v-ingress-introduction) - Routage HTTP/HTTPS

VI. [**Ingress Traefik**](2-Les-Services_Notions.md#vi-ingress-traefik) (K3s intégré) - Configuration pratique

## 3. Kubernetes : Les volumes

I. [**Volumes**](3-Les-Volumes_Notions.md#i-volumes) - Types volumes (emptyDir, hostPath, etc.)

II. [**PersistentVolumes (PV)**](3-Les-Volumes_Notions.md#ii-persistentvolumes-pv) - Abstraction stockage

III. [**PersistentVolumeClaims (PVC)**](3-Les-Volumes_Notions.md#iii-persistentvolumesclaims-pvc) - Demande stockage

IV. [**StorageClasses**](3-Les-Volumes_Notions.md#iv-storageclasses) - Provisioning dynamique

V. [**NFS Volumes**](3-Les-Volumes_Notions.md#v-nfs-volumes) - Montage NFS dans pods

## 4. Kubernetes : Configuration & Secrets

I. [**ConfigMaps**](4-Configuration-&-Secrets_Notions.md#i-configmaps) - Configuration externalisée

II. [**Secrets**](4-Configuration-&-Secrets_Notions.md#ii-secrets) - Credentials sensibles

III. [**Environment Variables**](4-Configuration-&-Secrets_Notions.md#iii-environment-variables) - Injection config dans pods

IV. [**Volume ConfigMaps/Secrets**](4-Configuration-&-Secrets_Notions.md#iv-configmapssecrets) - Montage fichiers config

V. [**Best Practices Config**](4-Configuration-&-Secrets_Notions.md#v-best-practice-config) - Patterns recommandés

## 5. Helm

I. [**Introduction Helm**](5-Helm_Notions.md#i-introduction-helm) - Pourquoi Helm, concepts charts

II. [**Installation Helm**](5-Helm_Notions.md#ii-installation-helm) - Setup Helm 3

III. [**Déployer des Charts**](5-Helm_Notions.md#iii-déployer-des-charts) - Helm install, repos publics

IV. [**Values.yaml**](5-Helm_Notions.md#iv-valuesyaml) - Customisation charts

V. [**Créer des Charts Custom**](5-Helm_Notions.md#v-créer-des-charts-custom) - Structure, templates basiques

## 6. `kubectl`

I. [**`kubectl logs`**](6-kubectl_Notions.md#i-kubectl-logs) - Isolation logique ressources

II. [**`kubectl describe`**](6-kubectl_Notions.md#ii-kubectl-describe) - Diagnostiquer problèmes ressources

III. [**`kubectl exec` & `kubectl run`**](6-kubectl_Notions.md#iii-kubectl-exec--kubectl-run) - Shell dans pod running

IV. [**Events K8s**](6-kubectl_Notions.md#iv-events-k8s) - Événements cluster

V. [**Patterns d'Erreurs Courantes**](6-kubectl_Notions.md#v-patterns-derreurs-courantes) - ImagePullBackOff, CrashLoopBackOff, etc.

## 7. Kubernetes : Sécurité & accès

I. [**RBAC - Rôles & RoleBindings**](7-Sécurité-&-Accès_Notions.md#i-rbac--rôles--rolebindings) - Contrôle accès ressources

II. [**ServiceAccounts**](7-Sécurité-&-Accès_Notions.md#ii-serviceaccounts) - Identité pods

III. [**Network Policies**](7-Sécurité-&-Accès_Notions.md#iii-network-policies) - Segmentation réseau pods

IV. [**Security Context**](7-Sécurité-&-Accès_Notions.md#iv-security-context) - runAsUser, capabilities, etc.

V. [**Pod Security Standards**](7-Sécurité-&-Accès_Notions.md#v-pod-security-standards) - Restricted, Baseline, Privileged

## 8. Kubernetes : Ressources & workloads

I. [**Namespaces**](8-Ressources-&-Workloads_Notions.md#i-namespaces) - Isolation logique ressources

II. [**Resource Quotas**](8-Ressources-&-Workloads_Notions.md#ii-resource-quotas) - Limites ressources namespace

III. [**LimitRanges**](8-Ressources-&-Workloads_Notions.md#iii-limitranges) - Limites par pod/container

IV. [**DaemonSets**](8-Ressources-&-Workloads_Notions.md#iv-daemonsets) - 1 pod par node (monitoring, logs)

V. [**StatefulSets**](8-Ressources-&-Workloads_Notions.md#v-statefulsets) - Apps stateful (bases de données)