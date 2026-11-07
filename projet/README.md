# 80 km/h, retour aux 90 km/h et radars : quel effet sur la mortalité routière ?

Projet *Python pour la data science*, ENSAE 2e année (2026-2027), cours de Lino Galiana.

> **État : brouillon.** La collecte et l'harmonisation des données sont en place ; l'analyse et la rédaction sont en cours. Le cadrage complet est dans [`docs/plan_projet.md`](docs/plan_projet.md).

## Problématique

Le 1er juillet 2018, la vitesse maximale est passée de 90 à 80 km/h sur les routes bidirectionnelles hors agglomération. Depuis la loi d'orientation des mobilités (décembre 2019), les départements peuvent revenir aux 90 km/h : 50 l'avaient fait fin 2024, à des dates et sur des linéaires très différents. Nous exploitons cette expérience naturelle pour mesurer l'effet de la vitesse autorisée sur la mortalité routière.

## Données

| Source | Accès | Usage |
|---|---|---|
| Accidents corporels BAAC 2010-2024 (ONISR) | API data.gouv.fr | Tués, accidents, réseau, collisions |
| Départements revenus aux 90 km/h | **Table construite** : [`data/departements_90.csv`](data/departements_90.csv), méthode dans [`docs/collecte_departements.md`](docs/collecte_departements.md) | Groupes `G80`, `G90_partiel`, `G90_total` |
| Population au 1er janvier par département (INSEE) | API Melodi | Taux par habitant |
| Ventes de carburants routiers par département (SDES) | API DiDo | Indicateur indirect du trafic (2020) |
| Contours des départements (IGN) | `cartiflette` | Cartes |
| Radars fixes (ministère de l'Intérieur) | API data.gouv.fr | Q3 (optionnel) |

Seule la table des relèvements est versionnée ; tout le reste est téléchargé par le code.

## Reproduire les résultats

```bash
git clone https://github.com/samir-tmlk/Projet-Python-DS.git
cd Projet-Python-DS
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

python -m src.collecte     # télécharge les 60 fichiers BAAC (~350 Mo) dans data/raw/
python -m src.nettoyage    # harmonise et écrit data/processed/{accidents,usagers}.parquet
```

Puis ouvrir `rapport.ipynb` et lancer « Exécuter tout ». Les deux commandes ci-dessus sont facultatives : le notebook reconstruit les données s'il ne les trouve pas.

**Cache S3 (SSP Cloud).** Si les variables `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`, `AWS_S3_ENDPOINT` (fournies par le SSP Cloud) et `S3_DOSSIER_PROJET` (par exemple `mon-bucket/projet-80kmh`) sont définies, le notebook lit d'abord les Parquet sur S3 et les y écrit après reconstruction. Les identifiants vont dans un fichier `.env` non versionné.

## Organisation

```
├── data/departements_90.csv   # table des relèvements, construite et sourcée
├── docs/
│   ├── plan_projet.md          # cadrage du projet
│   └── collecte_departements.md
├── src/
│   ├── collecte.py             # API data.gouv, INSEE, SDES, cartiflette, S3
│   ├── nettoyage.py            # harmonisation BAAC 2010-2024
│   ├── indicateurs.py          # panels, taux, groupes, différence de différences
│   └── graphiques.py
└── rapport.ipynb               # le rapport
```

## Premiers contrôles

- Les tués par an en France métropolitaine recalculés à partir des BAAC harmonisés correspondent aux bilans officiels de l'ONISR (3 992 en 2010, 3 244 en 2019, 2 541 en 2020, 3 193 en 2024).
- La part des accidents sur le réseau concerné par le 80 km/h est stable autour de 2019 (20,1 % en 2018, 19,8 % en 2019) : pas de rupture artificielle liée au changement de format.
- Avant 2015, les coordonnées GPS manquent pour environ deux tiers des accidents, ce qui limite l'analyse des radars aux années récentes.

## Conventions

- Branche par fonctionnalité, fusion dans `main` par pull request relue.
- Sorties du notebook nettoyées avant commit : `nb-clean clean rapport.ipynb`.
- Formatage : `ruff format src/`.
