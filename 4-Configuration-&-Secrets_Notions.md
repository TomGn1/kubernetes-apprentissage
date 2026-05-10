<a id="index"></a>
**Index** :

I. [**Labels et annotations**](#i-labels-et-annotations)

1. [Les Labels](#1-les-labels)
2. [Les annotations](#2-les-annotations)
3. [Différences techniques](#3-différences-techniques)

II. [**ConfigMaps**](#ii-configmaps) - Configuration externalisée

III. [**Secrets**](#iii-secrets) - Credentials sensibles

IV. [**Environment Variables**](#iv-environment-variables) - Injection config dans pods

V. [**Volume ConfigMaps/Secrets**](#v-configmapssecrets) - Montage fichiers config

VI. [**Best Practices Config**](#vi-best-practice-config) - Patterns recommandés

---
<a id="i-labels-et-annotations"></a>
# I. [**Labels et annotations**](#index)

>[!NOTE]
>**Rappel utile pour les labels** :
> En YAML, encadrer les valeurs avec `"` ou `'` est recommandé, notamment pour les labels dont la valeur ressemble à un booléen ou un nombre (`version: "1.0"`, `enabled: "yes"`). Sans guillemets, le parser YAML les interprète comme float ou bool.

Les `labels` et `annotations` sont des **métadonnées** attachées aux objets Kubernetes (Pods, Services, Deployments, Ingress, etc.). Ils partagent la même syntaxe `key: value` mais ont des rôles bien distincts : 
- **Labels** : métadonnées **identifiantes**, utilisées pour organiser, grouper et sélectionner les objets. 
- **Annotations** : métadonnées **non-identifiantes**, utilisées pour attacher des informations contextuelles destinées à des outils, contrôleurs ou opérateurs humains. 

La règle simple : si une donnée doit servir à **cibler** des objets, c'est un label. Sinon, c'est une annotation.
## 1. Les Labels

Les labels permettent de **filtrer et sélectionner** des objets pour appliquer des actions de manière mutualisée. Ils sont indexés par l'API server, ce qui les rend efficaces pour les requêtes. 

**Cas d'usage typiques :** 
- Sélection des Pods par un Service (via le `selector`) 
- Sélection des Pods par un Deployment ou ReplicaSet 
- Filtrage en ligne de commande : `kubectl get pods -l env=prod` 
- Règles de scheduling : `nodeSelector`, `nodeAffinity`, `podAffinity` / `podAntiAffinity` 
- Routage par les NetworkPolicies 
- Regroupement logique pour la supervision et la journalisation 

**Deux types de selectors :** 
- *Equality-based* : `env = prod`, `tier != frontend` 
- *Set-based* : `env in (prod, staging)`, `tier notin (frontend)`

### Exemples :

- Créer un pod avec le label désiré :
```bash
kubectl run <podName> --image <imageName> -l version=<x.x.x>
```

- Pour filtrer un objet à l'aide d'un label :
```bash
kubectl get pod -l version=<x.x.x>
```

- Pour ajouter un label depuis la CLI :
```bash
kubectl label deploy <deploymentName> <labelKey>=<labelValue>
```

- Pour modifier la valeur d'un label existant :
```bash
kubectl label deploy <deploymentName> --overwrite <labelKey>=<labelValue>
```

- Retirer un label d'un objet :
```bash
kubectl label deploy <deploymentName> <labelKey>-
```

> [!WARNING]
> Lors de l'ajout ou modification de labels dans un manifest, il faut faire attention à **où** ils sont placés : les labels ne se propagent pas automatiquement aux objets enfants.
>
> Dans un Deployment, il existe trois emplacements distincts :
>
> ```yaml
> apiVersion: apps/v1
> kind: Deployment
> metadata:
>   labels:        # (1) labels du Deployment lui-même
>     app: nginx
> spec:
>   selector:
>     matchLabels: # (2) labels que le Deployment cherche pour gérer ses Pods
>       app: nginx
>   template:
>     metadata:
>       labels:    # (3) labels appliqués aux Pods créés
>         app: nginx
> ```
>
> - **(1)** sert à retrouver le Deployment lui-même (`kubectl get deploy -l app=nginx`).
> - **(2)** doit correspondre à **(3)**, sinon le Deployment ne reconnaît pas ses propres Pods (et le manifest est rejeté à l'`apply` car ce champ est immuable une fois créé).
> - **(3)** est ce que les Services et autres selectors externes utilisent pour cibler les Pods.
>
> En pratique : un label ajouté en (1) n'apparaîtra **pas** sur les Pods. Pour cibler les Pods avec un Service, le label doit être dans le `template` (3).

### Convention de nommage

Une clé de label est composée de deux parties optionnellement séparées par un `/` : `préfixe/nom: valeur`

- Le **préfixe** est un sous-domaine DNS (ex. `app.kubernetes.io`, `argoproj.io`). Il est optionnel mais **fortement recommandé** dès qu'un label est consommé par un outil tiers, pour éviter les collisions entre projets.
- Le **nom** est obligatoire, 63 caractères max, doit commencer et finir par un caractère alphanumérique, et peut contenir `-`, `_`, `.` au milieu.
- La **valeur** suit les mêmes règles que le nom (63 caractères max, alphanumérique en début/fin), ou peut être vide.

Les préfixes `kubernetes.io/` et `k8s.io/` sont **réservés** aux composants core de Kubernetes.

**Labels natifs à Kubernetes** : 
- Documentation : https://kubernetes.io/docs/reference/labels-annotations-taints/

## 2. Les Annotations

Les annotations sont des **métadonnées non-identifiantes** attachées aux objets Kubernetes. Contrairement aux labels, elles ne sont **jamais utilisées par les selectors** : aucun contrôleur, aucun service, aucune règle de scheduling ne se base sur elles pour cibler des objets.

Leur rôle est de **transporter de l'information contextuelle** : configuration d'un contrôleur tiers, métadonnées de build, documentation, état interne géré par un outil, etc.

> [!NOTE]
> Règle simple : si la donnée sert à **cibler** des objets, c'est un label. Si elle sert à **décrire** ou **configurer**, c'est une annotation.

### Cas d'usage typiques

- **Configuration de contrôleurs** : la plupart des Ingress controllers, cert-manager, ou ArgoCD lisent leur configuration via des annotations.
  
```yaml
  metadata:
    annotations:
      cert-manager.io/cluster-issuer: "letsencrypt-prod"
      traefik.ingress.kubernetes.io/router.entrypoints: "websecure"
      argocd.argoproj.io/sync-wave: "1"
```

- **Métadonnées de build et de release** : commit Git, version, date de build, équipe responsable, lien vers la doc.

```yaml
  metadata:
    annotations:
      git.commit: "a3f2c91"
      build.date: "2026-05-10"
      owner: "team-platform@example.com"
```

- **État interne géré par les outils** : `kubectl apply` stocke par exemple la dernière configuration appliquée dans `kubectl.kubernetes.io/last-applied-configuration` pour gérer les diffs lors des prochains apply.

- **Activation de comportements optionnels** : Prometheus scrape par exemple les pods qui portent les annotations `prometheus.io/scrape: "true"` et `prometheus.io/port: "8080"`.

### Convention de nommage

Comme pour les labels, une annotation peut être préfixée par un domaine DNS pour éviter les collisions entre outils. C'est une **bonne pratique fortement recommandée** dès qu'une annotation est consommée par un outil tiers :

- `cert-manager.io/...` → annotations lues par cert-manager
- `argocd.argoproj.io/...` → annotations lues par ArgoCD
- `nginx.ingress.kubernetes.io/...` → annotations lues par l'Ingress controller NGINX

Les préfixes `kubernetes.io/` et `k8s.io/` sont **réservés** au projet Kubernetes lui-même.

## 3. Différences techniques

| Critère                   | Labels                          | Annotations                       |
|---------------------------|---------------------------------|-----------------------------------|
| Rôle                      | Identifier, sélectionner        | Décrire, configurer               |
| Utilisé par les selectors | Oui                             | Non                               |
| Indexé par l'API server   | Oui                             | Non                               |
| Taille de la valeur       | 63 caractères max               | Pas de limite stricte             |
| Caractères autorisés      | Restreint (alphanumérique + `-`, `_`, `.`) | Libre (URLs, JSON, multilignes…)  |
| Exemple de requête        | `kubectl get pods -l env=prod`  | Pas de requête possible           |

---
<a id="ii-configmaps"></a>
# II. [**ConfigMaps**](#index)



---
<a id="iii-secrets"></a>
# III. [**Secrets**](#index)



---
<a id="iv-environment-variables"></a>
# IV. [**Environment Variables**](#index)



---
<a id="v-configmapssecrets"></a>
# V. [**Volume ConfigMaps/Secrets**](#index)



---
<a id="vi-best-practice-config"></a>
# VI. [**Best Practices Config**](#index)


