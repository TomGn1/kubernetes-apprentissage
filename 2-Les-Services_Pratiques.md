## **Mise en pratique des [notions](2-Les-Services_Notions.md) sur les services de Kubernetes**.

---
<a id="index"></a>
**Index** :

I. [**Création des premiers Services**](#i-création-des-premiers-services)
1. [Déployer une application de test](#1-déployer-une-application-de-test)
2. [Accès direct à un Pod](#2-accès-direct-à-un-pod)
3. [Problème rencontré lors de la montée en charge](#3-problème-rencontré-lors-de-la-montée-en-charge)
4. [Création du Service](#4-création-du-service)
5. [Accès au Service](#5-accès-au-service)
6. [Exécution d'un service en mode déclaratif](#6-exécution-dun-service-en-mode-déclaratif)


---
# I. [Création des premiers Services](#index)

## 1. Déployer une application de test

Pour illustrer le fonctionnement des Services, nous allons déployer une application simple qui retourne le nom du Pod ayant répondu à la requête HTTP. :
```bash
kubectl create deployment hello --image paulbouwer/hello-kubernetes:1.8
```

## 2. Accès direct à un Pod

Chaque Pod reçoit une **adresse IP propre** sur le réseau des Pods. Il est donc possible d’interroger directement un Pod via son IP.

Récupération de l’IP du Pod :
```bash
kubectl get pods -o wide
```

Puis requête directe :
```bash
curl -s 10.244.0.113:8080 | grep hello
```

Exemple de réponse :
```bash
	   <td>hello-78b95d695d-l2bjs</td>
```

## 3. Problème rencontré lors de la montée en charge


Lorsque l’on augmente le nombre de Pods :
```bash
kubectl scale deploy hello --replicas 5
```
Plusieurs Pods sont créés, chacun avec :
- sa propre adresse IP
- une durée de vie potentiellement courte

**Problèmes** :
- il n’existe **aucun point d’accès stable**
- il faut connaître l’IP exacte de chaque Pod
- les Pods peuvent être recréés à tout moment

Un service résout précisément ces problèmes

## 4. Création du Service

On expose le Deployment via un Service de type **ClusterIP** (par défaut) :
```bash
kubectl expose deployment hello --port 8080
```

Lister les services du namespace :
```bash
kubectl get service
```

```bash
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
hello        ClusterIP   10.108.170.97   <none>        8080/TCP   20s
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP    8d
```

On peut voir le service nommé `hello` avec qui possède une **IP stable** (ClusterIP), avec un **port exposé** (:8080).

>[!NOTE]
>La colonne `EXTERNAL-IP` indique l’adresse IP accessible depuis l’extérieur du cluster pour certains types de Services (comme `LoadBalancer`).  
>Pour les Services de type `ClusterIP`, cette valeur est vide, car l’accès est limité au réseau interne du cluster.

## 5. Accès au Service

Maintenant, nous allons interroger directement le **Service**, et non plus un Pod précis :
```bash
curl -s 10.108.170.97:8080 | grep hello
```

Requêtes successives :
```bash
      <td>hello-78b95d695d-zmgc2</td>
      <td>hello-78b95d695d-54rl6</td>
      <td>hello-78b95d695d-gpwds</td>
```

À chaque requête :
- un **Pod différent** peut répondre
- le Service effectue un **load-balancing** vers les Pods disponibles.

>[!IMPORTANT]
>- Seuls les Pods **en état `Running` et `Ready`** peuvent recevoir du trafic via un Service.
>- Les Pods non prêts sont automatiquement exclus du routage.

## 6. Exécution d'un service en mode déclaratif

Afficher le manifeste YAML d’un Service existant :
```bash
kubectl get services hello -o yaml
```

### Génération d’un manifeste de Service `ClusterIP`

Générer un manifeste d'un service de type `ClusterIP` via un `--dry-run` :
```bash
kubectl create service clusterip <nomDuService> --tcp <portDuService>:<portDuPod> --dry-run=client -o yaml > svc.yaml
```

_Par exemple_ :
```bash
kubectl create service clusterip hello --tcp 8081:8080 --dry-run=client -o yaml > svc.yaml
```

Fichier `svc.yaml` généré :
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: hello
  name: hello
spec:
  ports:
  - name: 8081-8080  # Valeur informelle
    port: 8081       # Le port du Service
    protocol: TCP
    targetPort: 8080 # Le port des Pods
  selector:
    app: hello
  type: ClusterIP
# Le champ status est géré par Kubernetes et ne doit pas être défini dans un manifeste appliqué par l’utilisateur.
#status:            
#  loadBalancer: {}
```

>[!CAUTION]
>Les `selectors` du Service doivent correspondre aux labels des Pods ciblés, sans quoi aucun trafic ne sera routé.

### Application du Service

Appliquer le **Service** au déploiement :
```bash
kubectl apply -f svc.yaml
```

Vérification :
```bash
vagrant@k0s1:~ (⎈|Default:default) $ kubectl get services
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
hello        ClusterIP   10.99.69.172   <none>        8081/TCP   14s
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP    8d
```

Test :
```bash
curl -s 10.99.69.172:8081 | grep hello
```

```bash
      <td>hello-78b95d695d-l2bjs</td>
      <td>hello-78b95d695d-z9ctn</td>
      <td>hello-78b95d695d-gpwds</td>
```
Les réponses proviennent de Pods différents, démontrant le load-balancing interne.

### Suppression d'un Service

Supprimer un **Service** :
```bash
kubectl delete service <nomDuService>
```

### Création d'un Service `NodePort`

Créer un service de type `NodePort`:
```bash
kubectl create service nodeport hello --tcp 8081:8080
```

```bash
vagrant@k0s1:~ (⎈|Default:default) $ kubectl get services
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello        NodePort    10.100.154.11   <none>        8081:31568/TCP   15s
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP          8d
```

Le service devient accessible via :  `<nodeIP>:31568`

Avec un Service de type `NodePort`, Kubernetes expose le Service sur un port fixe de chaque nœud du cluster (ici,`:31568`).  
Les Pods associés au Service deviennent accessibles depuis l’extérieur du cluster via l’adresse IP d’un nœud et le port attribué.