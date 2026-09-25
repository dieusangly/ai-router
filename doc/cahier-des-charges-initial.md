# AI Router — Cahier des charges initial (v0)

**Statut :** v0 (version initiale), à valider avant développement. **Date :** 25 septembre 2026. Le dépôt contient des documents de cadrage, pas encore de service exécutable.

## 1. Problème et objectif

AI Router (routeur de modèles d’intelligence artificielle, IA) expose une API (interface de programmation d’applications) unique. Pour chaque demande, il choisit un modèle autorisé qui atteint le niveau de qualité requis au meilleur coût prévu, contrôle la réponse et garde la preuve de la décision.

Deux objectifs : économies vérifiées (« Auto Savings ») et respect prouvé des règles de données avec possibilité de changer de fournisseur (« Sovereign Evidence »). Le premier cas métier proposé est la LCB-FT (lutte contre le blanchiment de capitaux et le financement du terrorisme) ; son choix comme premier pilote reste à confirmer.

## 2. Parcours attendu pour une demande

1. L’application appelle `/v1/chat/completions` et demande un choix automatique du modèle.
2. Avant tout envoi, le service élimine les routes interdites par les règles du client (région, fournisseur, sensibilité, conservation, budget). Si aucune route n’est permise, il renvoie une erreur explicite.
3. Il choisit un modèle admissible, contrôle la réponse et, si nécessaire, essaie une autre route autorisée.
4. Il transmet la réponse acceptée et consigne le modèle, la règle, le contrôle et le coût pour consultation et export.

## 3. Exigences de la première version

Les codes F1 à F9 identifient les fonctionnalités ; F signifie « fonctionnalité ». Leur numéro ne donne pas un ordre de priorité.

| ID (identifiant) | À construire | Preuve attendue |
| --- | --- | --- |
| F1 | API compatible OpenAI, avec authentification et quota ; réponse vérifiée avant envoi. | Une application obtient une réponse en changeant l’adresse et la clé ; aucun rejet ne lui parvient. |
| F2 | Registre « AI-BOM » (AI Bill of Materials : inventaire des composants d’IA) : version, fournisseur, région, licence, prix et tests. | Toute route utilisable a une fiche datée. |
| F3 | Règles client appliquées avant l’envoi et lors de tout nouvel essai. | Une route interdite est bloquée, sans sortie silencieuse de la région autorisée. |
| F4 | Choix du modèle explicable et reproductible. | Même demande et même configuration : même route et même raison. |
| F5 | Contrôle des réponses, puis nouvel essai autorisé en cas d’échec. | Résultat du contrôle et raison du nouvel essai enregistrés. |
| F6 | Coût comparé à un modèle de référence sur les mêmes demandes. | Économies comptées uniquement sur les réponses acceptées et rapprochées des factures. |
| F7 | Journal de preuves et export par jeu de demandes. | Règle, modèle, contrôle et coût retrouvables, sans texte brut par défaut. |
| F8 | Test de changement vers un fournisseur autorisé. | Rapport de réussite ou d’échec, avec qualité, coût et temps de réponse. |
| F9 | Vue minimale des coûts, contrôles, erreurs et régions. | Résultats consultables et exportables. |

## 4. Données, modèles et prérequis

- Trois jeux de demandes anonymisés ou synthétiques, avec coût de référence et critères de qualité fixés avant les essais ; un jeu LCB-FT si ce pilote est retenu.
- Deux fournisseurs au minimum, dont une alternative autorisée. Vérifier leurs contrats, régions de traitement et règles de conservation avant tout trafic client.
- Qwen, DeepSeek, Mistral, OpenAI et Anthropic sont des candidats à tester, pas des routes validées.
- Technique envisagée : LiteLLM (passerelle à code source ouvert pour plusieurs modèles de langage), Postgres (base de données relationnelle) et stockage objet ; rien n’est encore installé.

## 5. Règles de travail et limites

- AI Router lit temporairement les demandes et réponses. Les journaux ne gardent par défaut que les métadonnées de preuve (règle, modèle, région, contrôle, coût, date), jamais les textes bruts ; protéger les clés et isoler les comptes.
- Dans le parcours vérifié, contrôler la réponse entière avant de l’envoyer. Le streaming (envoi progressif) immédiat ne garantit pas ce contrôle préalable et requiert une décision distincte.
- Tout nouvel essai respecte les mêmes règles. Versionner modèles, règles et tarifs ; ne promettre d’économies qu’après mesure sur des réponses de qualité acceptée.
- Hors périmètre de cette première version : entraînement ou adaptation de modèles, catalogue public, toutes les modalités, hébergement de très grands modèles et optimisation du routeur par apprentissage automatique.

## 6. Critères de réception

1. Trois jeux de demandes passent de bout en bout avec deux fournisseurs ; dix cas de règles bloquent les destinations interdites, y compris lors d’un nouvel essai.
2. Qualité définie avant les essais, sans régression critique. Cible proposée : au moins 70 % de réponses acceptées sans nouvel essai sur les tâches choisies, à confirmer.
3. Pour chaque réponse : modèle et version, région, règle, raison, contrôle et coût exportables ; économies vérifiables face aux factures.
4. Le même jeu est rejoué chez un fournisseur alternatif autorisé ; rapport de réussite ou d’échec avec écarts de qualité, coût et temps de réponse.
5. Une réponse rejetée n’est jamais transmise dans le parcours vérifié ; aucun texte brut dans les journaux de test.

## 7. Estimation initiale de charge

**Hypothèses :** départ sans code ; deux fournisseurs, un pilote, trois jeux de demandes rapidement disponibles, vue minimale et aucun nouveau modèle. Délais contractuels et revue des données exclus.

Un jour-personne (ou jour-homme) représente une journée de travail d’une personne. La charge ci-dessous ne suppose aucun effectif particulier.

| Rôle | Contribution principale | Charge (jours-personnes) |
| --- | --- | --- |
| Responsable produit / métier | Choix du pilote, périmètre, décisions et réception | 3–5 |
| Développement côté serveur et interface | API, fournisseurs, règles, routeur, contrôle, coûts, journal, export et vue minimale | 30–42 |
| Data scientist (spécialiste des données) / évaluation | Jeux de demandes, critères qualité, comparaison des modèles, analyse des échecs | 10–15 |
| DevOps (développement et exploitation) / infrastructure | Environnements, déploiement, secrets et suivi technique | 3–5 |
| Sécurité / DPO (délégué à la protection des données) / juridique | Règles de données, revue des fournisseurs et conditions d’usage | 2–4 |
| QA (assurance qualité) / FinOps (pilotage des coûts) | Tests de bout en bout et rapprochement des coûts | 2–4 |
| Total indicatif | Somme des charges par rôle | 50–75 |

La durée calendaire dépendra de l’effectif et de sa disponibilité ; elle n’est pas estimée ici. Une revue métier approfondie du jeu LCB-FT augmenterait la charge, et les délais fournisseurs allongeraient le calendrier.

## 8. Décisions à prendre avant le premier sprint

1. Premier pilote : économies sur des tâches courantes ou LCB-FT bancaire avec règles de souveraineté ?
2. Quels jeux de demandes, coût de référence et critères de qualité sont validés, et par qui ?
3. Quels fournisseurs et régions sont autorisés après revue des contrats et des données ?
4. Quel identifiant du choix automatique : `auto` ou `sovereign-auto` ?
5. Qui décide pour le produit, la technique, l’évaluation, la sécurité et le budget ?

**Sources de cadrage :** `Kolibree_Launchbook_Auto_Sovereign_Router_v2 (1).docx`, `Kolibree_Sovereign_Router_Launchbook_v1.docx` et `Kolibree_Sovereign_Router_Primer_Slides_v3.pptx`.
