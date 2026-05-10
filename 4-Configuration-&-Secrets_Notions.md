<a id="index"></a>
**Index** :

I. [**Labels et annotations**](#i-labels-et-annotations)

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

## 2. Annotations










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


