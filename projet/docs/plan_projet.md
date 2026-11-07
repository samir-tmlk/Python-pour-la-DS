# 80 km/h, retour aux 90 km/h et radars : quel effet sur la mortalité routière ?

**Projet Python pour la data science — ENSAE, 2e année (2026-2027)**
Cours de Lino Galiana — [pythonds.linogaliana.fr](https://pythonds.linogaliana.fr/content/annexes/evaluation.html)

> Document de cadrage du projet : problématique, données, méthode, organisation du dépôt, répartition du travail, calendrier et préparation de la soutenance. À lire par les trois membres du groupe avant de commencer, et à faire valider par le chargé de TD.

---

## Sommaire

1. [Résumé du projet](#1-résumé-du-projet)
2. [Contexte](#2-contexte)
3. [Problématique et hypothèses](#3-problématique-et-hypothèses)
4. [Les données](#4-les-données)
5. [Nettoyage et harmonisation](#5-nettoyage-et-harmonisation)
6. [Construction des indicateurs](#6-construction-des-indicateurs)
7. [Analyse descriptive](#7-analyse-descriptive)
8. [Démarche scientifique](#8-démarche-scientifique)
9. [Reproductibilité](#9-reproductibilité)
10. [Organisation du code et du dépôt](#10-organisation-du-code-et-du-dépôt)
11. [Répartition du travail et usage de Git](#11-répartition-du-travail-et-usage-de-git)
12. [Calendrier](#12-calendrier)
13. [Soutenance](#13-soutenance)
14. [Risques et plans de secours](#14-risques-et-plans-de-secours)
15. [Checklist au regard du barème](#15-checklist-au-regard-du-barème)
16. [Références](#16-références)

---

## 1. Résumé du projet

Le 1er juillet 2018, la vitesse maximale autorisée est passée de 90 à 80 km/h sur les routes bidirectionnelles sans séparateur central, hors agglomération. Depuis la loi d'orientation des mobilités (LOM) de décembre 2019, les départements peuvent revenir aux 90 km/h sur tout ou partie de leur réseau. Environ la moitié d'entre eux l'ont fait, à des dates et avec une ampleur très différentes.

Cette situation crée une **expérience naturelle** : à partir de 2020, des départements comparables vivent sous deux régimes de vitesse différents. Le projet exploite cette variation pour mesurer, de façon descriptive mais rigoureuse, l'effet de la vitesse autorisée sur la mortalité routière.

Le projet croise trois familles de données :

- les **accidents corporels** de 2010 à 2024 (fichiers BAAC de l'ONISR, open data) ;
- une **table construite par nos soins** des départements revenus aux 90 km/h, avec dates et ampleur du relèvement (collecte manuelle et documentée à partir de sources publiques) ;
- des **données de contexte** : radars fixes, population, contours géographiques, trafic.

**Livrable :** un dépôt GitHub public contenant un notebook `rapport.ipynb` qui tourne de bout en bout sans erreur, des modules Python dans `src/`, et un `README.md` expliquant comment reproduire les résultats.

---

## 2. Contexte

### 2.1 Chronologie

| Date | Événement |
|---|---|
| Janvier 2018 | Annonce de la mesure par le gouvernement (Comité interministériel de la sécurité routière) |
| 1er juillet 2018 | Passage à 80 km/h sur le réseau bidirectionnel sans séparateur central, hors agglomération |
| 26 décembre 2019 | Promulgation de la LOM : les départements peuvent relever la vitesse à 90 km/h sur leurs routes, après étude d'accidentalité et avis de la commission départementale de sécurité routière |
| 2020 | Premiers relèvements (Corrèze et Cantal au 1er février 2020, puis d'autres départements au fil de l'année) — et confinements liés au Covid |
| 2020-2025 | Relèvements progressifs, partiels ou totaux, dans environ 40 à 50 départements selon les décomptes |

### 2.2 Pourquoi ce sujet est intéressant

- **Une question de politique publique réelle et débattue.** La mesure a été très contestée et le retour aux 90 km/h est une décision politique locale.
- **Une variation exploitable.** Le relèvement n'a pas eu lieu partout en même temps ni de la même façon, ce qui permet de comparer des groupes.
- **Un résultat de référence pour se situer.** L'ONISR a publié une évaluation du relèvement sur 2019-2022. Nous pourrons comparer nos résultats aux siens et expliquer les éventuels écarts.
- **Un vrai travail de données.** Les bases BAAC changent de format en 2019, et il n'existe aucun fichier officiel recensant proprement les relèvements : il faut le construire.

### 2.3 Mécanisme attendu

La littérature en sécurité routière (notamment le « modèle de puissance » de Nilsson, repris par l'ONISR et le Cerema) établit que le nombre de tués varie bien plus que proportionnellement avec la vitesse moyenne pratiquée. Une baisse de quelques km/h de la vitesse moyenne peut donc produire une baisse sensible de la mortalité, surtout pour les chocs frontaux et les sorties de route, qui sont les accidents typiques du réseau secondaire. C'est ce mécanisme que nous cherchons à retrouver dans les données.

> ⚠️ Nous ne disposons pas des vitesses réellement pratiquées. Nous mesurons l'effet d'un **changement de règle**, pas d'un changement de vitesse observé. C'est une limite à mentionner dans le rapport.

---

## 3. Problématique et hypothèses

### 3.1 Question centrale

> **La baisse de la vitesse autorisée à 80 km/h a-t-elle réduit la mortalité routière, et le retour aux 90 km/h l'a-t-il fait remonter ?**

### 3.2 Sous-questions

**Q1 — L'effet de 2018 (contexte).**
La mortalité a-t-elle baissé plus vite après juillet 2018 sur le réseau concerné (routes bidirectionnelles hors agglomération) que sur le réseau non concerné (autoroutes, routes à chaussées séparées, routes en agglomération) ?

**Q2 — L'effet du retour aux 90 km/h (cœur du projet).**
À partir de 2020, la mortalité sur le réseau secondaire a-t-elle évolué différemment dans les départements relevés à 90 km/h et dans ceux restés à 80 km/h ? L'effet est-il plus marqué là où le relèvement est total plutôt que partiel ?

**Q3 — Les radars (optionnel, si le temps le permet).**
Le nombre d'accidents diminue-t-il à proximité d'un radar fixe après sa mise en service ?

### 3.3 Hypothèses à tester

| Hypothèse | Ce qu'on devrait observer si elle est vraie |
|---|---|
| H1 : le 80 km/h a réduit la mortalité | Rupture à la baisse en 2018-2019 sur le réseau concerné, absente ou plus faible sur le réseau non concerné |
| H2 : le retour aux 90 km/h a augmenté la mortalité | Après relèvement, évolution moins favorable dans les départements relevés que dans les départements restés à 80 |
| H3 : l'effet dépend de l'intensité | Écart plus grand pour les départements relevés en totalité que pour ceux relevés partiellement |
| H4 : l'effet passe par certains types d'accidents | Variation concentrée sur les chocs frontaux et les sorties de route, plus faible sur les autres collisions |

Le rapport doit conclure sur chaque hypothèse, y compris quand les données ne permettent pas de trancher.

---

