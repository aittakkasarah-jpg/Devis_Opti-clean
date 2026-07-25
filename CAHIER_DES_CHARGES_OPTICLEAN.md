# Cahier des charges — Devis & Contrat interactifs Opti'Clean

**Objet du document** : spécifications fonctionnelles et techniques complètes de l'outil, suffisantes pour recréer le projet à l'identique ou en assurer la passation à un développeur professionnel.

**Fichier livré** : `devis_interactif_opticlean_28.html` — fichier HTML unique et autonome (~400 Ko), incluant CSS et JavaScript intégrés. Aucune dépendance externe hormis une police Google Fonts (Inter) chargée par `@import`.

**Version documentée** : Juillet 2026.

---

## 1. Contexte métier

Opti'Clean est une société de nettoyage professionnel (SARL au capital de 1 000 €, Blagnac, 31700, SIRET 106 475 999 00012). L'outil sert à générer, sans serveur ni base de données, un **devis interactif** puis un **contrat juridique complet**, à partir des mêmes informations saisies une seule fois.

### Flux utilisateur
```
Ouverture du fichier HTML dans le navigateur (double-clic)
   → Saisie des informations client, site, prestations, tarification
   → Calculs automatiques (grille tarifaire, TVA, remise, total TTC)
   → Bouton "Imprimer / Exporter PDF" → impression navigateur → PDF du devis
   → Bouton "Exporter PDF Contrat" → modal lieu/date → génération → impression → PDF du contrat
```

### Pourquoi un fichier unique (décision de conception)
Pas de serveur à maintenir, pas de compte à créer, pas de dépendance réseau après le premier chargement. Le fichier se partage par email ou clé USB et fonctionne hors ligne. Compensation : pas de base de données centralisée — la persistance repose sur le `localStorage` du navigateur (limite : les données ne sont disponibles que sur le poste et le navigateur utilisés pour la saisie).

---

## 2. Architecture technique

### 2.1 Structure du fichier

```
devis_interactif_opticlean.html
├── <head>
│   └── <style>            (~36 000 caractères)
│       ├── Variables CSS (:root) : couleurs (dark #1F3B44, accent #8FD3C7, bg #F2EFEB)
│       ├── CSS de base (mise en page, composants, typographie "Inter")
│       ├── @media screen and (max-width: 700px)   → responsive mobile
│       └── @media print                            → mise en page du PDF du devis
│
├── <body>
│   ├── .toolbar          (Réinitialiser / Exporter PDF Devis / Exporter PDF Contrat)
│   ├── .validity-banner
│   ├── .page
│   │   ├── .header        (logo SVG, badge DEVIS, référence, dates)
│   │   ├── .accent-bar
│   │   ├── .parties       (Prestataire fixe + Client éditable, grille 2 colonnes)
│   │   └── .section × N   (description du site, fréquence, prestations, plateforme,
│   │                        tarification, engagement/facturation, conditions, signature)
│   ├── #modal-contrat     (fenêtre modale : lieu + date de signature du contrat)
│
└── <script>              (~47 000 caractères, fonctions globales détaillées en section 7)
```

### 2.2 Technologies

- HTML5 / CSS3 (Flexbox, Grid, sélecteurs `:has()`, `:placeholder-shown`, `text-transform`)
- JavaScript ES5/ES6 vanilla — **aucun framework, aucune librairie externe**
- Police web : Inter (Google Fonts, chargée via `@import` dans le `<style>`)
- Génération PDF : fonction native `window.print()` du navigateur (aucune librairie PDF)
- Persistance : `localStorage` du navigateur (clé `opticlean_devis_v1`)

### 2.3 Compatibilité

Développé et testé sur **Google Chrome** (Windows). Utilise des fonctionnalités CSS modernes (`:has()`, `:placeholder-shown`, `@page`, `print-color-adjust: exact`) qui nécessitent un navigateur récent. Safari et Firefox ont un support partiel de `:has()` — non testé.

---

## 3. Spécifications fonctionnelles — DEVIS

### 3.1 En-tête et informations générales
| Élément | Comportement |
|---|---|
| Logo + badge "DEVIS" | Fixe |
| N° de référence | Saisie manuelle libre (ex : OC-2026-001) |
| Date d'émission | Auto-remplie à la date du jour, modifiable |
| Date de validité | Calculée automatiquement = émission + 30 jours |
| Bandeau de validité | Texte fixe rappelant les 30 jours |

### 3.2 Identité des parties

**Prestataire (fixe, non modifiable dans l'interface)**
- OPTI'CLEAN — SARL au capital de 1 000 €
- 19 rue Isaac Newton, 31700 Blagnac
- Représentants : Hugo FOURNET et Sarah RIGAL
- Téléphones : 06 10 18 09 53 / 06 68 23 41 11
- Email : contact@opticlean.tech
- SIRET : 106 475 999 00012 — TVA : FR25106475999

**Client (saisie libre)** : société, adresse, téléphone, email, contact référent, RC/IF.

### 3.3 Description du site
- Type de site (menu déroulant : Bureau, Coworking, Cabinet médical/juridique, Copropriété, Commerce/Boutique, Entrepôt/Logistique, Autre)
- **Superficie totale** : soit saisie manuelle, soit **calculée automatiquement** à partir du tableau des zones (somme des zones cochées)
- Nombre de niveaux du bâtiment (1 à 5+)
- **Tableau des zones** : Open space/Bureaux, Sanitaires, Cuisine/Coin café, Salles de réunion, Parties communes/Couloirs, + 1 ligne libre — chacune avec case "incluse", superficie estimée, commentaire

### 3.4 Fréquence et planning d'intervention
- Fréquence : 1 à 7 fois/semaine, 2 fois/mois, 1 fois/mois *(NB : "1 fois/mois" n'est pas couvert par la grille tarifaire automatique — champ de compatibilité conservé mais sans calcul associé)*
- Jours d'intervention et horaires (texte libre)
- **Durée par passage** : calculée automatiquement selon la superficie — voir formule section 5.2
- **Passages/mois (estimé)** : calculé automatiquement selon la fréquence
- Date de démarrage souhaitée

### 3.5 Prestations

**Nettoyage courant — inclus à chaque passage** (9 prestations fixes, cochées par défaut, non facturées séparément) :
Aspiration/lavage des sols, vidage sacs poubelles, dépoussiérage, sanitaires, cuisine/coin café, tables/bureaux, plinthes/angles, points de contact, espaces communs.

**Entretien spécifique — sur devis** (3 prestations optionnelles) :
| Prestation | Champs |
|---|---|
| Détartrage sanitaires | case à cocher, nb de passages/mois, P.U. HT, commentaire |
| Nettoyage vitres accessibles | idem |
| Nettoyage moquette | idem |

**Consommables — sur devis** (4 consommables optionnels) :
| Consommable | Champs |
|---|---|
| Savon | case à cocher, quantité/mois, P.U. HT, commentaire |
| Papier toilette | idem |
| Essuie-tout | idem |
| Sacs poubelles | idem |

**Règle métier commune (entretien ET consommables)** : une ligne cochée mais dont la quantité **ou** le prix unitaire est vide est considérée incomplète. Elle est alors exclue :
- du PDF du devis (section Prestations *et* tableau de Tarification),
- du calcul du sous-total,
- mais reste visible à l'écran (mode édition) pour permettre la complétion.

### 3.6 Plateforme de suivi (incluse, informative)
Suivi en temps réel, historique complet, remontée d'informations, rapports mensuels automatiques.

### 3.7 Tarification — tableau récapitulatif

| Colonne | Contenu |
|---|---|
| Désignation | Nom de la prestation |
| Fréquence | Ex : "2x / sem.", "—" (entretien), "Mensuel" (consommables) |
| Qté / mois | Nombre de passages ou quantité |
| P.U. HT | Prix unitaire |
| Montant HT | Qté × P.U. |

**Lignes générées :**
1. **Nettoyage courant** — calculé automatiquement via la grille tarifaire (section 5) selon superficie + fréquence + engagement, ou modifiable manuellement.
2. **Une ligne par prestation d'entretien spécifique cochée et complète.**
3. **Une ligne par consommable coché et complet.**
4. Accès plateforme de suivi (Offert), Fourniture produits et matériel (Inclus) — lignes fixes, sans montant.
5. Sous-total HT / mois.
6. Remise commerciale (case à cocher optionnelle + pourcentage libre, appliquée sur le sous-total).
7. TVA (20 %) — affichée à 2 décimales.
8. **TOTAL TTC / MOIS** — affiché à 2 décimales.
9. Note automatique : prix moyen par passage.

*Note technique : les lignes 2 et 3 sont générées dynamiquement en JavaScript (une `<tr>` par prestation active), et non plus agrégées dans un seul champ — ce choix garantit une présentation identique à l'Annexe 1 du contrat.*

### 3.8 Engagement, facturation, conditions
- **Engagement** : 12 mois / 6 mois / Sans engagement — détermine à la fois le tarif (section 5) et le texte juridique du contrat (Article 7).
- Facturation : mensuelle / trimestrielle / annuelle.
- Délai de paiement : 10 jours / 15 jours / 30 jours fin de mois / à réception / 60 jours.
- Mode de paiement : virement / chèque / espèces.
- Conditions générales résumées à l'écran : accès au site, responsabilité (RC Pro), prestations hors devis, révision tarifaire (fixe, 1×/an), durée & renouvellement (texte auto-généré selon l'engagement), retard de paiement.
- Préavis de résiliation (1 mois, LRAR) : mention fixe, non modifiable.

### 3.9 Signature
Bloc "Bon pour accord" reprenant nom du client, engagement et date de démarrage. Emplacement de signature client (sans mention de cachet).

### 3.10 Fonctions techniques
- **Sauvegarde automatique** : tous les champs sont enregistrés dans `localStorage` 3 secondes après toute saisie ou changement.
- **Restauration** : à l'ouverture, une bannière propose de restaurer les données sauvegardées si elles existent (disparaît après 10 s si ignorée).
- **Réinitialisation** : bouton dédié, avec confirmation, efface tous les champs et le `localStorage`.
- **Export PDF** : utilise `window.print()` avec une feuille de style `@media print` dédiée : masque la barre d'outils, les champs vides, les prestations non cochées/incomplètes ; transforme les champs adresse en texte multi-lignes.
- **Responsive mobile** : mise en page adaptée sous 700px de large.

---

## 4. Spécifications fonctionnelles — CONTRAT

### 4.1 Génération
- Déclenchée par le bouton "Exporter PDF Contrat", disponible en permanence.
- **Validation préalable** : si nom client, date de démarrage, montant HT ou engagement sont manquants, une alerte les liste (la génération reste possible ; les champs manquants apparaissent en évidence — texte rouge encadré `[ Nom du champ ]` — dans le document final).
- Une fenêtre modale demande le **lieu** et la **date de signature**.
- Le contrat est construit dans une **iframe totalement isolée** du devis (voir décision technique 6.3), puis l'impression du navigateur se lance automatiquement après un court délai (800 ms).
- Le numéro de contrat reprend le numéro de devis d'origine.

### 4.2 Structure du document
En-tête (logo, badge CONTRAT, référence, lieu/date de signature) → Parties (Prestataire fixe / Client repris du devis) → 10 articles → Annexe 1 → Conditions générales → Bon pour accord → Signatures → Pied de page.

### 4.3 Les 10 articles (texte juridique complet intégré au code)

| N° | Titre | Contenu clé |
|---|---|---|
| 1 | Objet du contrat et effet | Objet du contrat ; date de prise d'effet **dynamique** (reprise du devis) |
| 2 | Description des prestations de nettoyage | Renvoi à l'Annexe 1 ; horaires décalés possibles ; obligation de moyens ; non-invocabilité de la subjectivité du client |
| 3 | Logistique et accès aux locaux | Fourniture des accès par le Client ; facturation intégrale si accès impossible du fait du Client |
| 4 | Matériel et produits | Produits éco-labellisés fournis par le Prestataire ; matériel lourd à charge du Client sauf accord écrit ; consommables sur devis complémentaire |
| 5 | Volet technologique et accès à l'application | Propriété intellectuelle du Prestataire ; gestion des accès (1 compte/entreprise) ; module d'évaluations ; continuité du service physique en cas de panne applicative |
| 6 | Conditions financières, facturation et révision | Forfait mensuel **dynamique** (montant HT/TTC repris du devis) ; révision annuelle indexée (SMIC, convention collective) ; délai de paiement **dynamique** (repris du devis) ; pénalités (taux BCE + 10 pts + 40 € forfaitaire, art. L.441-10 C. com.) |
| 7 | Durée, engagement et résiliation | **3 versions dynamiques** selon l'engagement choisi (voir 4.4) |
| 8 | Clause de non-sollicitation du personnel | Interdiction d'embauche pendant le contrat + 12 mois après ; indemnité de 6 mois de salaire brut de l'agent concerné |
| 9 | Responsabilités et assurances | RC Pro obligatoire ; plafond de responsabilité = sommes versées sur les 6 derniers mois précédant le sinistre |
| 10 | Litiges et loi applicable | Droit français ; résolution amiable puis Tribunal de Commerce de Toulouse |

### 4.4 Article 7 — les 3 variantes (clé métier)

Le texte de l'Article 7 change automatiquement selon la valeur du champ Engagement du devis :

- **"Sans engagement"** : résiliable à tout moment par l'une ou l'autre partie, sans motif, sous réserve d'un préavis de 1 mois par LRAR. Prestations dues pendant le préavis.
- **"6 mois"** : durée ferme de 6 mois, renouvelable par tacite reconduction (périodes d'1 an), résiliation ≥1 mois avant échéance par LRAR.
- **"12 mois"** : identique, durée ferme de 12 mois.

*(Titre de l'article volontairement neutre — "Durée, engagement et résiliation" — pour rester cohérent quel que soit le cas choisi.)*

### 4.5 Annexe 1 — Proposition commerciale acceptée
Reprend fidèlement le tableau de tarification du devis (section 3.7) : informations clés (client, démarrage, superficie, fréquence, passages/mois, engagement), liste des prestations de nettoyage courant incluses, puis le tableau détaillé ligne par ligne (nettoyage courant + une ligne par entretien spécifique actif + une ligne par consommable actif), sous-total, TVA, total TTC/mois.

### 4.6 Conditions générales (contrat)
Résumé en 6 points : accès au site, responsabilité, prestations hors contrat, révision tarifaire, durée & renouvellement (texte adapté à l'engagement), retard de paiement.

### 4.7 Bon pour accord et signatures
Texte d'engagement (nom client, durée, date de démarrage), lieu et date de signature, deux blocs de signature (Prestataire / Client — sans mention "manuscrite").

---

## 5. Grille tarifaire officielle (2026)

### 5.1 Principe général
La tarification du **nettoyage courant** dépend de trois facteurs croisés : **superficie**, **fréquence de passage**, **engagement**. Les montants sont **lus directement** dans une grille figée (voir décision technique 6.2), jamais recalculés à la volée, pour garantir une correspondance exacte avec les tarifs officiels communiqués par la direction.

### 5.2 Durée estimée par passage
Formule : `durée (h) = ceil(superficie / 75) × 0,5 + 0,5`

| Superficie | Durée |
|---|---|
| 0 à 75 m² | 1,0 h |
| 76 à 150 m² | 1,5 h |
| ... (paliers de 75 m², +0,5h) | ... |
| 901 à 975 m² | 7,0 h |

Au-delà de 975 m², la grille automatique ne s'applique plus (saisie manuelle).

### 5.3 Trois taux horaires selon l'engagement

| Engagement | Taux horaire HT |
|---|---|
| 12 mois | 36 € HT / h |
| 6 mois | 40 € HT / h |
| Sans engagement | 43 € HT / h |

### 5.4 Remises de récurrence (identiques pour les 3 engagements)

| Fréquence | Remise |
|---|---|
| 2 passages/mois, 1 passage/semaine | Aucune |
| 2 passages/semaine | -5 % |
| 3 passages/semaine | -9 % |
| 4 passages/semaine | -11 % |
| 5 passages/semaine | -14 % |
| 6 passages/semaine | -17 % |
| 7 passages/semaine | -20 % |

### 5.5 Grille complète — Engagement 12 mois (36 € HT/h)

| Tranche | 2x/mois | 1x/sem | 2x/sem (-5%) | 3x/sem (-9%) | 4x/sem (-11%) | 5x/sem (-14%) | 6x/sem (-17%) | 7x/sem (-20%) |
|---|---|---|---|---|---|---|---|---|
| 0-75 m² | 72,00 | 156,00 | 296,40 | 425,88 | 555,36 | 670,80 | 776,88 | 873,60 |
| 76-150 m² | 108,00 | 234,00 | 444,60 | 638,82 | 833,04 | 1006,20 | 1165,32 | 1310,40 |
| 151-225 m² | 144,00 | 312,00 | 592,80 | 851,76 | 1110,72 | 1341,60 | 1553,76 | 1747,20 |
| 226-300 m² | 180,00 | 390,00 | 741,00 | 1064,70 | 1388,40 | 1677,00 | 1942,20 | 2184,00 |
| 301-375 m² | 216,00 | 468,00 | 889,20 | 1277,64 | 1666,08 | 2012,40 | 2330,64 | 2620,80 |
| 376-450 m² | 252,00 | 546,00 | 1037,40 | 1490,58 | 1943,76 | 2347,80 | 2719,08 | 3057,60 |
| 451-525 m² | 288,00 | 624,00 | 1185,60 | 1703,52 | 2221,44 | 2683,20 | 3107,52 | 3494,40 |
| 526-600 m² | 324,00 | 702,00 | 1333,80 | 1916,46 | 2499,12 | 3018,60 | 3495,96 | 3931,20 |
| 601-675 m² | 360,00 | 780,00 | 1482,00 | 2129,40 | 2776,80 | 3354,00 | 3884,40 | 4368,00 |
| 676-750 m² | 396,00 | 858,00 | 1630,20 | 2342,34 | 3054,48 | 3689,40 | 4272,84 | 4804,80 |
| 751-825 m² | 432,00 | 936,00 | 1778,40 | 2555,28 | 3332,16 | 4024,80 | 4661,28 | 5241,60 |
| 826-900 m² | 468,00 | 1014,00 | 1926,60 | 2768,22 | 3609,84 | 4360,20 | 5049,72 | 5678,40 |
| 901-975 m² | 504,00 | 1092,00 | 2074,80 | 2981,16 | 3887,52 | 4695,60 | 5438,16 | 6115,20 |

### 5.6 Grille complète — Engagement 6 mois (40 € HT/h)

| Tranche | 2x/mois | 1x/sem | 2x/sem (-5%) | 3x/sem (-9%) | 4x/sem (-11%) | 5x/sem (-14%) | 6x/sem (-17%) | 7x/sem (-20%) |
|---|---|---|---|---|---|---|---|---|
| 0-75 m² | 80,00 | 173,33 | 329,33 | 473,20 | 617,07 | 745,33 | 863,20 | 970,67 |
| 76-150 m² | 120,00 | 260,00 | 494,00 | 709,80 | 925,60 | 1118,00 | 1294,80 | 1456,00 |
| 151-225 m² | 160,00 | 346,67 | 658,67 | 946,40 | 1234,13 | 1490,67 | 1726,40 | 1941,33 |
| 226-300 m² | 200,00 | 433,33 | 823,33 | 1183,00 | 1542,67 | 1863,33 | 2158,00 | 2426,67 |
| 301-375 m² | 240,00 | 520,00 | 988,00 | 1419,60 | 1851,20 | 2236,00 | 2589,60 | 2912,00 |
| 376-450 m² | 280,00 | 606,67 | 1152,67 | 1656,20 | 2159,73 | 2608,67 | 3021,20 | 3397,33 |
| 451-525 m² | 320,00 | 693,33 | 1317,33 | 1892,80 | 2468,27 | 2981,33 | 3452,80 | 3882,67 |
| 526-600 m² | 360,00 | 780,00 | 1482,00 | 2129,40 | 2776,80 | 3354,00 | 3884,40 | 4368,00 |
| 601-675 m² | 400,00 | 866,67 | 1646,67 | 2366,00 | 3085,33 | 3726,67 | 4316,00 | 4853,33 |
| 676-750 m² | 440,00 | 953,33 | 1811,33 | 2602,60 | 3393,87 | 4099,33 | 4747,60 | 5338,67 |
| 751-825 m² | 480,00 | 1040,00 | 1976,00 | 2839,20 | 3702,40 | 4472,00 | 5179,20 | 5824,00 |
| 826-900 m² | 520,00 | 1126,67 | 2140,67 | 3075,80 | 4010,93 | 4844,67 | 5610,80 | 6309,33 |
| 901-975 m² | 560,00 | 1213,33 | 2305,33 | 3312,40 | 4319,47 | 5217,33 | 6042,40 | 6794,67 |

### 5.7 Grille complète — Sans engagement (43 € HT/h)

| Tranche | 2x/mois | 1x/sem | 2x/sem (-5%) | 3x/sem (-9%) | 4x/sem (-11%) | 5x/sem (-14%) | 6x/sem (-17%) | 7x/sem (-20%) |
|---|---|---|---|---|---|---|---|---|
| 0-75 m² | 86,00 | 186,33 | 354,03 | 508,69 | 663,35 | 801,23 | 927,94 | 1043,47 |
| 76-150 m² | 129,00 | 279,50 | 531,05 | 763,04 | 995,02 | 1201,85 | 1391,91 | 1565,20 |
| 151-225 m² | 172,00 | 372,67 | 708,07 | 1017,38 | 1326,69 | 1602,47 | 1855,88 | 2086,93 |
| 226-300 m² | 215,00 | 465,83 | 885,08 | 1271,73 | 1658,37 | 2003,08 | 2319,85 | 2608,67 |
| 301-375 m² | 258,00 | 559,00 | 1062,10 | 1526,07 | 1990,04 | 2403,70 | 2783,82 | 3130,40 |
| 376-450 m² | 301,00 | 652,17 | 1239,12 | 1780,41 | 2321,71 | 2804,32 | 3247,79 | 3652,13 |
| 451-525 m² | 344,00 | 745,33 | 1416,13 | 2034,76 | 2653,39 | 3204,93 | 3711,76 | 4173,87 |
| 526-600 m² | 387,00 | 838,50 | 1593,15 | **2289,10** | 2985,06 | 3605,55 | 4175,73 | 4695,60 |
| 601-675 m² | 430,00 | 931,67 | 1770,17 | 2543,45 | 3316,73 | 4006,17 | 4639,70 | 5217,33 |
| 676-750 m² | 473,00 | 1024,83 | 1947,18 | 2797,80 | 3648,41 | 4406,78 | 5103,67 | 5739,07 |
| 751-825 m² | 516,00 | 1118,00 | 2124,20 | 3052,14 | 3980,08 | 4807,40 | 5567,64 | 6260,80 |
| 826-900 m² | 559,00 | 1211,17 | 2301,22 | 3306,49 | 4311,75 | 5208,02 | 6031,61 | 6782,53 |
| 901-975 m² | 602,00 | 1304,33 | 2478,23 | 3560,83 | 4643,43 | 5608,63 | 6495,58 | 7304,27 |

> ⚠️ La valeur en gras (526-600 m², Sans engagement, 3x/sem = **2 289,10 €**) est un cas limite d'arrondi : la formule pure (temps × taux × passages × remise) donne 2 289,105 €, qui arrondirait normalement à 2 289,11 €. La valeur officielle du barème (2 289,10 €) doit être utilisée telle quelle — ne jamais recalculer cette grille par formule, toujours la relire depuis ce tableau.

### 5.8 Prestations incluses / exclues (rappel officiel du barème)
- **Incluses** : aspiration, lavage des sols, dépoussiérage mobilier, désinfection points de contact/sanitaires, forfait de mise en place.
- **Exclues** (facturées sur devis complémentaire) : vitrerie de grande hauteur, fourniture de consommables sanitaires.
- Base de calcul : régularité de 4,33 semaines/mois (52 semaines / 12 mois).

---

## 6. Règles de cohérence devis ↔ contrat

Le système repose sur un principe simple : **devis et contrat partagent exactement les mêmes champs sources**. Aucune étape de synchronisation manuelle n'est nécessaire.

| Donnée | Champ source (devis) | Répercussion contrat |
|---|---|---|
| Identité client | `client_nom`, `client_adresse`, `client_rc`, `client_contact` | Bloc "Le Client", Annexe 1 |
| Montant HT/TTC | `sous_total_ht`, `total_ttc` | Article 6, Annexe 1 |
| Délai de paiement | `select_delai_paiement` | Article 6 |
| Engagement | `select-engagement` | Tarif (grille), Article 7 (texte), conditions devis |
| Date de démarrage | `date_demarrage` | Article 1 (prise d'effet), Bon pour accord |
| Référence | `ref_num` | Numéro de contrat identique |
| Détail prestations | items entretien/consommables cochés+complets | Annexe 1 (mêmes lignes) |

---

## 7. Référence technique — Fonctions JavaScript

| Fonction | Rôle |
|---|---|
| `parseMoneyToFloat(v)` | Parse un montant formaté ("1 234,56 €") en nombre flottant. Utilisée partout où un champ de tarification doit être relu. |
| `readMontant(id)` | Lit et parse un champ via son id, `0` si absent. |
| `calcTotal()` | Source de vérité unique du sous-total, TVA, TTC. Lit `montant_ht_ligne` + entretien + consommables + remise. |
| `applyGrilleTarifaire()` | Sélectionne le tableau tarifaire correspondant à l'engagement, trouve la tranche de superficie, lit le montant, met à jour tarif/durée/tooltip. |
| `toggleRemise()` | Affiche/masque la ligne de remise commerciale. |
| `resetForm()` | Réinitialise tous les champs (avec confirmation) et efface le `localStorage`. |
| `syncFrequence()` | Synchronise le select fréquence vers les champs tarif liés. |
| `syncEngagement()` | Met à jour le texte des conditions (Durée & Renouvellement, Bon pour accord, note TTC) selon l'engagement. |
| `syncEntretien()` | Calcule chaque prestation d'entretien cochée+complète, génère les lignes dynamiques du tableau de tarification, met à jour le total. |
| `syncConsommables()` | Idem pour les consommables. |
| `calcSuperficieAuto()` | Somme les zones cochées → `superficie_totale` → déclenche `applyGrilleTarifaire()`. |
| `installAddressProxies()` / `syncAddressProxies()` / `removeAddressProxies()` | Gestion de l'affichage multi-ligne des adresses à l'impression. |
| `saveToLocalStorage()` / `restoreFromLocalStorage()` | Persistance locale (debounce 3 s). |
| `openContratModal()` | Valide les champs requis, ouvre la modale lieu/date. |
| `printContrat()` | Construit le HTML complet du contrat (10 articles + Annexe 1 + conditions + signatures), l'injecte dans une iframe isolée, lance l'impression. |

---

## 8. IDs HTML critiques (référence développeur)

**Dates/référence** : `ref_num`, `date_emission`, `date_validite`, `date_demarrage`
**Fréquence/planning** : `freq_select`, `nb_passages`, `superficie_totale`, `duree_passage`
**Tarification (nettoyage courant)** : `tarif_freq`, `qte_passages`, `pu_ht`, `montant_ht_ligne`, `sous_total_ht`, `montant_tva`, `total_ttc`, `prix_passage`, `tarif-tooltip`
**Remise** : `checkbox-remise`, `pct-remise`, `remise-row`, `remise-check-row`, `montant_remise`
**Engagement/conditions** : `select-engagement`, `select_delai_paiement`, `duree-condition`, `duree-bpa`, `engagement-note`
**Entretien spécifique** : `chk-detartrage`/`chk-vitres`/`chk-moquette`, `nb-*`, `pu-*`, `mt-*`, ancre `entretien-anchor-row`, lignes générées classe `.entretien-dynrow`, total `montant-entretien-tarif`
**Consommables** : `chk-savon`/`chk-papier`/`chk-essuie`/`chk-sacs`, `qty-*`, `pu-*`, `mt-*`, ancre `conso-anchor-row`, lignes générées classe `.conso-dynrow`, total `montant-conso-tarif`
**Contrat** : `modal-contrat`, `modal-alert`, `contrat-lieu`, `contrat-date`

---

## 9. Décisions techniques importantes (à respecter en cas de refonte)

1. **Grille tarifaire en lecture directe, jamais recalculée** — les remises étant déjà intégrées aux montants officiels, un recalcul par formule introduit des écarts d'arrondi (jusqu'à plusieurs euros sur les grandes surfaces). Toujours copier les montants du barème officiel tels quels.
2. **Contrat généré dans une iframe isolée** — deux approches alternatives (overlay CSS, manipulation directe du DOM du devis) ont été testées et abandonnées : timing d'impression imprévisible, classes CSS en conflit, perte des event listeners. L'iframe isolée est la seule méthode fiable à 100 %.
3. **Article 7 en 3 versions dynamiques** — le texte juridique change fondamentalement selon l'engagement (durée ferme vs résiliable à tout moment) ; ne jamais fusionner en un texte unique.
4. **Apostrophes typographiques obligatoires dans le JS du contrat** — le contrat est construit par concaténation de chaînes JS délimitées par des guillemets simples (`'`). Toute apostrophe française doit être le caractère typographique `’` (U+2019) et non le guillemet simple ASCII `'`, sous peine de casser la chaîne JavaScript et donc tout le fichier.
5. **Items "incomplets" exclus systématiquement** — une prestation cochée sans quantité/prix est traitée comme absente sur tout le document (devis, tarification, contrat), pas seulement masquée visuellement.
6. **Pas de dépendance à un serveur ou une base de données** — choix assumé pour la simplicité d'usage, au prix de l'absence de synchronisation multi-poste.

---

## 10. Limites connues

- Mono-poste : `localStorage` non partagé entre navigateurs/ordinateurs.
- Numéro de devis saisi manuellement (pas d'auto-incrémentation).
- Fréquence "1 fois / mois" présente dans le menu mais non couverte par la grille tarifaire automatique.
- Au-delà de 975 m², tarification manuelle requise.
- Testé uniquement sur Google Chrome.

---

## 11. Pistes d'évolution possibles

- Auto-incrémentation du numéro de devis (format `OC-[ANNÉE]-[NNN]`, via `localStorage`, remise à 001 chaque janvier).
- Sauvegarde manuelle d'un devis figé (export d'un fichier HTML avec valeurs pré-remplies), pour réouverture ultérieure.
- Extension de la grille tarifaire pour couvrir la fréquence "1 fois / mois" et les superficies > 975 m².
- Intégration d'un service de signature électronique (actuellement gérée hors outil).

---

*Document rédigé à partir de l'état du fichier `devis_interactif_opticlean_28.html` en juillet 2026. Toute évolution du code doit être répercutée dans ce document pour en conserver la valeur de référence.*
