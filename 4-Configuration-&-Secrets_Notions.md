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
>En Yaml, l'utilisation des `"` ou `'` sont recommandées, elles permettent d'éviter certaines erreurs sur le traitement de valeurs en précisant la valeur exacte.

Les `labels`et `annotations` sont des metadatas qui vont contribuer à plusieurs ressources de Kubernetes (ex. : les pods, les services, les deployments, les ingress, etc.).

Leurs usages sont différents :
- Labels : organiser et manager les objets Kubernetes
- Annotations : intéragir avec des éléments ajoutés à Kubernetes ou externes à Kubernetes
## 1. Les Labels

Exemple d'usage des labels :
- Environnement
- Rôles (frontend, backend)
- Equipes 
- Versions
- Verbosité
- Localisation
- etc.

## 2. Les annotations






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


