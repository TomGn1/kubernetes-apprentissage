
Apprentissage progressif de Kubernetes en 8 blocs. Après chaque bloc ajout des nouvelles compétences acquises au projet DaaS-k8s-v2.
## Bloc 1 :

1. **Introduction Kubernetes** - Pourquoi K8s existe, problèmes résolus

2. **Architecture Cluster K8s** - Master/Worker nodes, API server, etcd, scheduler

3. **Pods** - Unité de base, containers vs pods, lifecycle

4. **ReplicaSets** - Gestion replicas, self-healing

5. **Deployments** - Déploiements déclaratifs, rollouts, rollbacks

## Bloc 2 :

6. **Services - ClusterIP** - Exposition interne cluster

7. **Services - NodePort** - Exposition externe via port node

8. **Services - LoadBalancer** - Intégration cloud providers

9. **Ingress Introduction** - Routage HTTP/HTTPS

10. **Ingress Traefik** (K3s intégré) - Configuration pratique

## Bloc 3 :

11. **Volumes** - Types volumes (emptyDir, hostPath, etc.)

12. **PersistentVolumes (PV)** - Abstraction stockage

13. **PersistentVolumeClaims (PVC)** - Demande stockage

14. **StorageClasses** - Provisioning dynamique

15. **NFS Volumes** - Montage NFS dans pods

## Bloc 4 :

16. **ConfigMaps** - Configuration externalisée

17. **Secrets** - Credentials sensibles

18. **Environment Variables** - Injection config dans pods

19. **Volume ConfigMaps/Secrets** - Montage fichiers config

20. **Best Practices Config** - Patterns recommandés

## Bloc 5 :

21. **Introduction Helm** - Pourquoi Helm, concepts charts

22. **Installation Helm** - Setup Helm 3

23. **Déployer Charts** - Helm install, repos publics

24. **Values.yaml** - Customisation charts

25. **Créer Chart Custom** - Structure, templates basiques

## Bloc 6 :

26. **kubectl logs** - Consulter logs containers

27. **kubectl describe** - Diagnostiquer problèmes ressources

28. **kubectl exec** - Shell dans pod running

29. **Events K8s** - Événements cluster

30. **Patterns Erreurs Courantes** - ImagePullBackOff, CrashLoopBackOff, etc.

## Bloc 7 :

31. **RBAC - Rôles & RoleBindings** - Contrôle accès ressources

32. **ServiceAccounts** - Identité pods

33. **Network Policies** - Segmentation réseau pods

34. **Security Context** - runAsUser, capabilities, etc.

35. **Pod Security Standards** - Restricted, Baseline, Privileged

## Bloc 8 :

36. **Namespaces** - Isolation logique ressources

37. **Resource Quotas** - Limites ressources namespace

38. **LimitRanges** - Limites par pod/container

39. **DaemonSets** - 1 pod par node (monitoring, logs)

40. **StatefulSets** - Apps stateful (bases de données)