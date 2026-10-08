# Projet 2 — Data Warehouse : Consignes de clôture : 17 octobre 2026 à 23h59

## Contexte et objectif

Le Projet 2 commence exactement là où le Projet 1 s'arrête. Tu ne refais pas l'extraction : tu récupères le **CSV nettoyé produit par ton Projet 1**, tu le charges dans **Snowflake**, puis tu construis un véritable **Data Warehouse** avec **dbt**.

L'objectif n'est pas de faire tourner quelques commandes SQL ou un `dbt run`. C'est de savoir répondre à une question plus importante, et de pouvoir le prouver :

> **Comment transformer un fichier de données nettoyées en une plateforme analytique structurée, testée, documentée et reproductible ?**

L'architecture attendue est de type **Medallion** :

```text
CSV nettoyé du Projet 1
          │
          ▼
      SNOWFLAKE
          │
          ▼
      ┌─────────┐
      │ BRONZE  │  Données telles que reçues du Projet 1
      └────┬────┘
           │ dbt
           ▼
      ┌─────────┐
      │ SILVER  │  Données préparées pour l'analyse
      └────┬────┘
           │ dbt
           ▼
      ┌─────────┐
      │  GOLD   │  Modèles analytiques
      └─────────┘
```

Le modèle en étoile (**Star Schema**) est **recommandé mais non obligatoire**. Si tu choisis une autre modélisation, tu dois démontrer qu'elle répond mieux aux besoins analytiques du projet.

- **Dataset** : le même que le Projet 1 — [NYC TLC Trip Record Data — Yellow Taxi (2023)](https://data.cityofnewyork.us/Transportation/2023-Yellow-Taxi-Trip-Data/4b4i-vvec)
- **Dictionnaire des champs** : [TLC Trip Record Data Dictionary (PDF)](https://www.nyc.gov/assets/tlc/downloads/pdf/data_dictionary_trip_records_yellow.pdf)
- **Projet précédent** : [Projet 1 — Pipeline API](https://github.com/ksilogacademy/data-engineer-01-pipeline-api)

---

## 1. Prérequis

Avant de commencer, assure-toi d'avoir en main :

- **Le Projet 1 terminé**, avec son **CSV nettoyé** et son **rapport de qualité**.
- **Un compte Snowflake** fonctionnel, avec les droits pour créer un warehouse, une database, des schemas, un stage et des tables.
- **Un environnement Python fonctionnel**, avec **dbt Core** et l'adaptateur Snowflake (`dbt-snowflake`).
- **Une base correcte en SQL** et des notions sur les bases de données relationnelles.
- **Des bases Git/GitHub** : tu vas versionner ton travail proprement, en excluant tout ce qui ne doit jamais être public.

Tu dois aussi comprendre, avant de commencer ou en cours de projet, les notions suivantes : database, schema, table, stage, warehouse Snowflake, `COPY INTO`, clé primaire, clé étrangère, grain, dimension, fait, agrégation, Data Warehouse, architecture Medallion.

### Sources et documentation

Les liens ci-dessous couvrent tout ce dont tu as besoin pour ce projet. Commence par la documentation officielle plutôt que par des tutoriels trouvés au hasard :

- **Snowflake** — documentation générale : [docs.snowflake.com](https://docs.snowflake.com/)
- **Snowflake** — vue d'ensemble du chargement de données : [Data Loading Overview](https://docs.snowflake.com/en/user-guide/data-load-overview)
- **Snowflake** — création et gestion des stages : [CREATE STAGE / Stage management](https://docs.snowflake.com/en/sql-reference/ddl-stage)
- **Snowflake** — chargement dans une table : [COPY INTO](https://docs.snowflake.com/en/sql-reference/sql/copy-into-table)
- **dbt** — documentation générale : [dbt Developer Hub](https://docs.getdbt.com/)
- **dbt** — modèles : [Models](https://docs.getdbt.com/docs/build/models)
- **dbt** — sources : [Sources](https://docs.getdbt.com/docs/build/sources)
- **dbt** — tests : [Data Tests](https://docs.getdbt.com/docs/build/data-tests)
- **dbt** — Jinja et macros : [Jinja and Macros](https://docs.getdbt.com/docs/build/jinja-macros)
- **dbt** — seeds : [Seeds](https://docs.getdbt.com/docs/build/seeds)
- **dbt** — snapshots : [Snapshots](https://docs.getdbt.com/docs/build/snapshots)
- **dbt** — analyses : [Analyses](https://docs.getdbt.com/docs/build/analyses)
- **dbt** — documentation et lineage : [Documentation](https://docs.getdbt.com/docs/build/documentation)

Si l'un de ces points n'est pas clair pour toi, c'est le moment de combler la lacune — pas après avoir construit la moitié de ton entrepôt.

---

## 2. Consigne

À partir du CSV nettoyé de ton Projet 1, construis un Data Warehouse qui réalise, dans l'ordre, les étapes suivantes :

1. **Ingestion** : charger le CSV dans une table **Bronze** avec les mécanismes natifs de Snowflake (file format, stage, `PUT`, `COPY INTO`), puis prouver que le chargement est complet.
2. **Préparation (Silver)** : transformer la Bronze avec **dbt** en données typées, standardisées et cohérentes.
3. **Modélisation (Gold)** : construire avec **dbt** des modèles analytiques adaptés aux besoins ci-dessous, avec un grain clairement défini.
4. **Qualité et documentation** : tester les propriétés qui rendent l'entrepôt fiable, documenter les modèles et générer le lineage.

L'ensemble doit être reproductible par un tiers à partir de ton README, sans intervention cachée.

### Le besoin analytique

Imagine qu'une équipe d'analystes reçoive ton CSV. Il est exploitable, mais ce n'est pas encore un Data Warehouse. Elle veut pouvoir répondre à des questions comme :

- **Activité** : combien de trajets ? Comment se répartissent-ils selon les heures et les jours ? Quelles périodes concentrent le plus de trajets ?
- **Temps et trajet** : quelle durée et quelle distance moyennes ? Comment la durée évolue-t-elle avec la distance ? Existe-t-il des trajets anormalement longs ou courts ?
- **Revenus** : quel montant total, quel montant moyen par trajet, quel pourboire moyen ? Comment évoluent-ils selon les périodes ?
- **Paiement** : quels modes de paiement, quelle part de chacun, et les montants moyens diffèrent-ils selon le mode ?
- **Localisation** : quels lieux de départ et d'arrivée sont les plus fréquents ? Quels trajets ou zones concentrent le plus d'activité ?
- **Qualité** : les données du Projet 1 sont-elles toujours cohérentes après ingestion ? Quelles anomalies doivent être bloquées, lesquelles peuvent simplement être documentées ?

**Ces questions ne signifient pas que tu dois créer une table par question.** Comprends d'abord le besoin, identifie les données nécessaires, définis leur grain, puis choisis une modélisation cohérente.

---

## 3. Directives

### Les données

Le Projet 2 part du **CSV nettoyé généré par ton Projet 1**, et uniquement de lui. Tu ne dois pas :

- télécharger un autre dataset ;
- utiliser un dataset déjà préparé trouvé sur Internet ;
- remplacer ton CSV par un autre fichier ;
- refaire l'extraction API pour contourner le travail du Projet 1.

Le Projet 2 utilise volontairement **le même mois et le même fichier** que le Projet 1, qui peut atteindre plusieurs centaines de Mo : l'objectif est de se concentrer sur le Data Warehouse et dbt sans consommer inutilement les ressources de ton compte Snowflake. Tu n'as donc pas besoin de charger plusieurs mois.

### Comprendre le dataset avant de construire

Ne commence pas par écrire du SQL. Avant de créer quoi que ce soit, analyse le résultat du Projet 1 et réponds, par écrit, à ces questions :

- Quelles colonnes sont présentes dans le CSV, et quel est le type logique de chacune ?
- Quelle colonne (ou combinaison de colonnes) permet d'identifier un trajet ? Peut-on vraiment la considérer comme un identifiant unique ?
- Quelles colonnes représentent le temps, la localisation, le paiement, les montants, la distance, la durée, le fournisseur ?
- Lesquelles sont des attributs descriptifs, lesquelles servent à calculer des mesures ?
- Lesquelles doivent rester en Bronze même si Gold ne les utilise pas ?
- Quelles anomalies du Projet 1 doivent être contrôlées à nouveau dans Snowflake ?

Cette analyse fait partie de tes livrables.

### Définir le grain

Avant de créer un modèle, définis explicitement le **grain** de chaque table importante :

```text
fact_trips       → une ligne représente un trajet.
dim_date         → une ligne représente une date.
dim_payment_type → une ligne représente un mode de paiement.
```

Ce ne sont que des exemples : tu détermines toi-même le grain adapté à ton modèle. Pour chaque table Gold, tu dois pouvoir dire **ce que représente exactement une ligne** et **pourquoi ce grain permet de répondre aux questions analytiques**. Une table dont le grain n'est pas clairement défini n'est pas considérée comme correctement modélisée.

### Architecture Snowflake et ingestion

Crée l'environnement Snowflake avec des **scripts SQL versionnés** (pas avec dbt). Le nom des objets est libre, mais leur rôle doit être identifiable. Il te faut au minimum : un **warehouse**, une **database**, trois **schemas** (Bronze, Silver, Gold), un **file format CSV**, un **stage** et une **table Bronze**.

Le flux d'ingestion est : CSV local → `PUT` → stage → `COPY INTO` → table Bronze. Tu dois pouvoir vérifier, avec les commandes de contrôle de Snowflake : combien de fichiers et de lignes ont été chargés, si des erreurs sont apparues, si le nombre de lignes correspond au fichier source, et si les types chargés sont ceux attendus.

> Pourquoi séparer les données en plusieurs couches plutôt que de charger directement le CSV dans une table finale ?

### Architecture Medallion

- **Bronze** : les données telles qu'elles arrivent du Projet 1, aussi proches que possible de la source. *Quelles transformations ne doivent surtout pas être effectuées ici ?*
- **Silver** : les données préparées pour l'analyse — types standardisés, colonnes dérivées utiles, normalisation, gestion des valeurs particulières, règles métier, contrôles de cohérence. Réalisée avec **dbt** (couches `staging` et, si besoin, `intermediate`).
- **Gold** : les données destinées à l'analyse, réalisées avec **dbt** (couche `marts`). Le Star Schema est recommandé ; si tu en choisis un autre : *pourquoi est-il plus adapté aux questions analytiques du projet ?*

### dbt : le projet

Crée un véritable projet dbt, pas seulement quelques modèles. Tu dois comprendre le rôle de `dbt_project.yml`, de `profiles.yml`, des dossiers `models/`, `tests/`, `macros/`, `seeds/`, `snapshots/`, `analyses/`, ainsi que de `source()`, `ref()` et de Jinja.

**Sources.** Déclare tes tables Bronze comme **sources dbt** et utilise `source()` dans les modèles qui les consomment. Ajoute des contrôles de fraîcheur ou de qualité sur les sources quand c'est pertinent. *Pourquoi `source()` plutôt que le nom de la table Snowflake écrit en dur ?*

**Staging.** Le staging transforme progressivement la Bronze en données propres : renommage, typage, cast, normalisation, colonnes dérivées, nettoyage léger. *Quelle différence fais-tu entre le nettoyage du Projet 1 et les transformations du staging dbt ?*

**Intermediate.** Cette couche n'est pas obligatoire si elle n'apporte aucune valeur. Si tu en crées une, tu dois expliquer pourquoi certaines étapes méritent leur propre modèle. *Quand une transformation reste-t-elle dans staging, et quand mérite-t-elle un modèle intermédiaire ?*

**Gold.** À partir des besoins identifiés, détermine tes faits, dimensions, mesures, clés, relations et agrégations. Un modèle possible est un `fact_trips` entouré de `dim_date`, `dim_location`, `dim_payment` et `dim_vendor`, mais **ce n'est qu'une piste** : tu dois justifier ton propre modèle.

**Tests.** Les tests ne sont pas optionnels. Réfléchis à `not_null`, `unique`, `accepted_values` et `relationships` lorsqu'ils sont pertinents, puis traduis en tests les propriétés qui doivent absolument être vraies pour que ton entrepôt soit fiable. Ne teste pas une colonne uniquement parce que dbt permet de la tester : **chaque test doit protéger une règle de qualité identifiable**. Distingue aussi une *anomalie à documenter* (avertissement) d'une *erreur bloquante* (échec).

**Jinja et macros.** Utilise Jinja là où il apporte une vraie valeur, et crée **au moins une macro utile**, qui résout un vrai problème de répétition (pas une macro créée pour cocher la case). *Quel problème ton usage de Jinja résout-il par rapport à du SQL écrit en dur ?*

**Seeds.** Utilise un seed si tu identifies une petite table de référence, stable et utile, suffisamment légère pour être versionnée dans Git. Un seed n'est **pas** un moyen de stocker ton dataset NYC Taxi, qui reste chargé dans Snowflake. Si aucune donnée de référence ne s'y prête, justifie-le plutôt que d'en créer une artificiellement.

**Snapshots.** Étudie le fonctionnement des snapshots et détermine si une donnée de ton projet mérite d'être historisée. Si oui, mets en place un snapshot adapté ; si non, documente *pourquoi un snapshot n'est pas pertinent pour ces données*. Ne crée pas un snapshot artificiel.

**Analyses.** Utilise le dossier `analyses/` pour au moins une requête qui répond à une question analytique ou technique sans être matérialisée dans l'entrepôt (par exemple : quelles relations entre distance, durée et montant total ?). Le résultat doit être expliqué.

**Documentation et lineage.** Documente au minimum tes sources, modèles, colonnes importantes, grain des tables principales, clés, mesures et règles métier, puis génère la documentation dbt. *Si un analyste te demande d'où vient une métrique de Gold, comment remontes-tu jusqu'au CSV ?* Tu dois pouvoir montrer le chemin :

```text
CSV → Bronze → source() → staging → intermediate → Gold → métrique
```

### Reproductibilité et secrets

Un autre développeur doit pouvoir, en suivant ton README : configurer Snowflake, placer le CSV, créer le stage, charger les données, configurer dbt, exécuter les modèles, lancer les tests et générer la documentation. Documente les commandes nécessaires.

**Aucun secret ne doit être présent dans Git** : mot de passe Snowflake, token, clé privée, identifiant sensible, fichier `.env`, et donc ton `profiles.yml` s'il contient des identifiants. Utilise des variables d'environnement, et fais en sorte que ton `.gitignore` empêche ces fichiers d'être versionnés. Une fuite dans l'historique Git reste une fuite même si le fichier est supprimé ensuite.

### Maîtrise des coûts Snowflake

Ne laisse pas un warehouse tourner inutilement. Tu dois comprendre ce qu'est un warehouse, quand il consomme des crédits, comment le suspendre et limiter son usage, et pourquoi il ne faut pas multiplier les traitements inutiles. Tu dois pouvoir expliquer les mesures que tu as prises.

### Liberté d'architecture

L'arborescence du dépôt proposée plus bas est un minimum, pas un carcan : elle peut évoluer si tu peux la justifier, à condition de rester **cohérente**, **documentée** dans ton `README.md` et **lisible à l'échelle du projet**.

---

## 4. Livrable

### Ce que tu dois rendre

Un dépôt Git propre contenant au minimum :

```text
.
├── README.md
├── .gitignore
├── dbt_project.yml
├── models/
│   ├── staging/
│   ├── intermediate/
│   └── marts/
├── tests/
├── macros/
├── seeds/
├── snapshots/
├── analyses/
├── sql/
│   └── snowflake/
└── docs/
```

- **Les scripts SQL Snowflake** (dans `sql/snowflake/`) qui créent le warehouse, la database, les schemas, le file format, le stage, la table Bronze et les objets nécessaires au chargement.
- **Le projet dbt complet**, exécutable à partir du dépôt.
- **Un `README.md`** qui explique : le contexte et l'objectif, l'architecture, les prérequis, l'installation, la configuration Snowflake et dbt, le chargement du CSV, l'exécution, les tests, la documentation, les choix de modélisation et autres choix importants (avec leur *pourquoi*), et les limites du projet. Il doit aussi répondre aux questions « à résoudre » posées dans ces consignes.
- **Une documentation d'architecture** (dans `docs/`) : un schéma du flux CSV → stage → Bronze → Silver → Gold, qui montre aussi tes principales tables Gold, et ton analyse du dataset.
- **Des preuves** (dans `docs/`) permettant de vérifier le chargement Snowflake, le nombre de lignes Bronze, les modèles Silver et Gold, les tests dbt, la documentation dbt et le lineage. Vérifie que tes captures ne montrent aucun identifiant de compte sensible.

### Barème de validation — /100 points

Ton projet sera noté selon le barème ci-dessous. Chaque ligne est vérifiée soit automatiquement (script), soit par une lecture humaine ou assistée (pour juger la qualité d'une justification, pas juste sa présence).

**Règle éliminatoire — prioritaire sur tout le reste :** si un mot de passe, un token, une clé privée ou tout autre secret Snowflake apparaît à un seul endroit de l'historique Git (même dans un commit ancien, même supprimé depuis), **le score est automatiquement 0/100**, quel que soit le reste du projet.

**Seuil de réussite : 70/100.** En dessous, le projet est « à revoir » ; à partir de 70, il est « validé ».

#### Bloc 1 — Snowflake et ingestion (15 pts) · *vérification automatique*

| Critère                                                                                                          | Points |
| ---------------------------------------------------------------------------------------------------------------- | ------ |
| Environnement créé par scripts SQL versionnés (warehouse, database, 3 schemas, file format, stage, table Bronze) | 5      |
| Chargement par `PUT` + `COPY INTO`, avec un nombre de lignes Bronze égal à celui du CSV source                   | 6      |
| Warehouse configuré pour maîtriser la consommation (suspension automatique, taille adaptée)                      | 4      |

#### Bloc 2 — dbt et modélisation (25 pts) · *mixte*

| Critère                                                                                                                  | Points | Vérification       |
| ------------------------------------------------------------------------------------------------------------------------ | ------ | ------------------ |
| Projet dbt structuré : sources déclarées, staging, `source()`/`ref()`, et `dbt build` qui s'exécute sans erreur          | 8      | Auto               |
| Modèles Gold avec grain défini, faits/dimensions identifiés, clés documentées, modèle justifié par les besoins analytiques | 10     | Humaine / assistée |
| Jinja, au moins une macro utile, une analyse SQL, et un seed ou un snapshot (ou leur absence justifiée)                  | 7      | Humaine / assistée |

#### Bloc 3 — Qualité et reproductibilité (20 pts) · *vérification automatique*

| Critère                                                                                                      | Points |
| ------------------------------------------------------------------------------------------------------------ | ------ |
| Tests dbt présents, couvrant des règles de qualité identifiables (clés, valeurs acceptées, relations)        | 10     |
| Projet relançable par un tiers à partir du README, sans valeur codée en dur qui devrait être un paramètre    | 6      |
| `.gitignore` correct, aucun fichier de configuration sensible ni donnée volumineuse versionnés               | 4      |

#### Bloc 4 — Documentation et présentation (40 pts) · *mixte*

| Critère                                                                                                                                                                         | Points | Vérification                 |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ | ---------------------------- |
| `README.md` complet et structuré (objectif, architecture, installation, configuration, exécution, tests, limites)                                                               | 10     | Auto (présence des sections) |
| Documentation dbt générée : modèles, colonnes, grain, clés, mesures, règles métier, lineage                                                                                     | 10     | Mixte                        |
| Les choix (Medallion, modélisation, staging vs Projet 1, macro, seed/snapshot, maîtrise des coûts) sont expliqués et argumentés dans le README, pas seulement mentionnés        | 15     | Humaine / assistée           |
| Dépôt propre et preuves présentes (chargement, lignes Bronze, tests, lineage), sans secret ni fichier temporaire                                                                | 5      | Auto                         |

**Exemple concret pour le critère « choix » :** lister n'est pas argumenter.

- ❌ *Listé* : « J'ai ajouté un test `unique` sur `trip_id`. »
- ✅ *Argumenté* : « J'ai ajouté un test `unique` sur la clé de `fact_trips` parce que le grain est un trajet par ligne : un doublon fausserait tous les comptages et les moyennes, et le CSV du Projet 1 ne contient pas d'identifiant natif, donc j'ai construit cette clé moi-même. »

Même chose pour chaque table, transformation, test, macro, seed ou snapshot : le README doit dire *pourquoi* ils existent. Et si tu décides de ne pas utiliser une fonctionnalité, justifie pourquoi elle n'est pas pertinente.

---

## 5. Soumission

**📅 Date limite : samedi 17 octobre 2026, 23h59 (heure de Dakar).**

Une fois ton projet terminé et poussé sur GitHub (dépôt public, sinon je ne peux pas y accéder) :

- **Commente le post LinkedIn du projet** avec le lien direct vers ton dépôt GitHub.
- Vérifie avant de poster que le lien est bien public (teste-le en navigation privée) et qu'il pointe vers la racine du dépôt, pas vers un fichier ou une branche spécifique.
- Un seul commentaire par apprenant suffit ; si tu push des corrections après coup, un commentaire de mise à jour avec « edit » en préfixe évite la confusion sur quelle version évaluer.

---

## Checklist finale avant de considérer le projet terminé

- [ ] Le CSV provient bien du Projet 1, et le nombre de lignes Bronze est égal à celui du fichier source.
- [ ] Les scripts Snowflake créent tout l'environnement (warehouse, database, schemas, file format, stage, table Bronze), et le warehouse ne tourne pas inutilement.
- [ ] Les responsabilités de Bronze, Silver et Gold sont documentées, et le grain de chaque table importante est défini.
- [ ] `dbt build` passe : sources déclarées, `source()` et `ref()` utilisés, au moins une macro utile, une analyse, un seed et un snapshot (ou leur absence justifiée).
- [ ] Chaque test protège une règle de qualité identifiable.
- [ ] La documentation dbt est générée et le lineage vérifié.
- [ ] Aucun secret (mot de passe, token, clé, `.env`, `profiles.yml` avec identifiants) n'est versionné ni présent dans l'historique Git.
- [ ] `README.md` est à jour, permet une prise en main autonome, et argumente les choix au lieu de les lister.