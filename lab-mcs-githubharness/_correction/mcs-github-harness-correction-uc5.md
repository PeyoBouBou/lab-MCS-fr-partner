# Correction : cas d’usage nº 5, la Skill « dossier d’escalade »

Document réservé à l’animateur. Il contient la Skill de référence, les réponses aux questions de documentation, les résultats attendus des tests et les éléments de réponse aux questions de compréhension.

> [!IMPORTANT]
> Ne distribuez pas ce fichier avec le lab. Gardez-le hors du répertoire `mcs-github-harness/` remis aux participants, ou partagez-le seulement après la séance.

---

## ✅ Réponses aux questions de documentation

| Question | Réponse attendue |
|----------|------------------|
| Les trois champs de **Create from blank** | **Name**, **Description** et **Instructions** |
| Les contraintes du **Name** | Uniquement des lettres minuscules, des chiffres et des tirets ; le nom ne commence ni ne se termine par un tiret |
| La base de l’activation | L’orchestrateur active une Skill en comparant le message de l’utilisateur à la **description** de la Skill |
| Le contenu recommandé des **Instructions** | Une description claire de la tâche, des étapes pas à pas, les exigences de format de réponse, les cas limites et leur traitement, les références aux outils à utiliser |

Sources : [Vue d’ensemble des Skills](https://learn.microsoft.com/microsoft-copilot-studio/agents-experience/skills-overview) et [Créer une Skill pour un agent](https://learn.microsoft.com/microsoft-copilot-studio/agents-experience/skills-create).

---

## 🧱 La Skill de référence

### Name

```text
dossier-escalade
```

### Description

```text
Prépare le dossier d'escalade standard de Northgate Energy pour une éolienne, avant toute décision d'intervention. À utiliser dès qu'un analyste veut escalader une éolienne, préparer un dossier ou un argumentaire, justifier l'envoi d'une équipe, ou demande s'il faut intervenir, envoyer quelqu'un ou déclencher une maintenance sur une turbine comme WTG-114. Ne pas utiliser pour une simple question sur une limite, un seuil, un délai d'intervention ou la liste des éoliennes en alarme.
```

### Instructions

```text
# Dossier d'escalade Northgate Energy

Cette Skill prépare le dossier qu'un analyste présente avant qu'une décision d'intervention soit prise. Elle collecte les faits, les compare aux limites et à la politique, et s'arrête là : l'analyste décide.

## Procédure

1. Identifiez l'éolienne. Si l'analyste n'en nomme aucune, appelez list_assets, présentez les éoliennes en alarme et demandez laquelle traiter avant de continuer.
2. Relevez l'état actuel avec get_asset_status : code d'alarme, vibration du multiplicateur, température du multiplicateur et puissance produite.
3. Vérifiez la validité de la mesure. Recherchez dans le manuel de maintenance du fabricant le seuil minimal de puissance à partir duquel les mesures sont comparables, et confirmez que l'éolienne le dépasse. Si ce n'est pas le cas, dites-le clairement et indiquez que la mesure ne peut pas fonder une décision.
4. Comparez chaque valeur aux seuils d'avertissement et d'action du manuel. Ne vous fiez pas au libellé du code d'alarme : le contrôleur enregistre le premier seuil franchi et ne recode pas l'alarme lorsque la valeur continue de monter. Indiquez le code qui correspond réellement à la valeur mesurée.
5. Analysez la tendance. Recherchez dans les normes Northgate la fenêtre et le seuil de déclenchement sur tendance, puis récupérez la vibration du multiplicateur sur cette fenêtre avec query_telemetry. Si l'outil renvoie une erreur de période trop longue, découpez la période et réessayez.
6. Vérifiez les travaux ouverts. Appelez list_work_orders pour l'éolienne, ouvrez chaque ordre de travail ouvert avec get_work_order et dites si ses tâches traitent réellement le problème constaté.
7. Déterminez le niveau de gravité selon les normes Northgate de performance et d'intervention. Citez la règle qui a déterminé ce niveau, le délai d'intervention et l'escalade prévus.
8. Rédigez le dossier au format ci-dessous.

## Format du dossier

Utilisez exactement ces rubriques, dans cet ordre :

- **Éolienne** : identifiant et site
- **État actuel** : code d'alarme, vibration, température, puissance
- **Validité de la mesure** : exploitable ou non, et pourquoi
- **Comparaison aux limites** : chaque valeur face à son seuil, et le code d'alarme réellement correspondant
- **Tendance** : évolution sur la fenêtre des normes, comparée au seuil de déclenchement
- **Travaux ouverts** : ordres de travail ouverts et s'ils traitent le problème
- **Niveau de gravité** : niveau, règle appliquée, délai d'intervention, escalade
- **Recommandation** : ce que l'analyste devrait décider

## Règles

- Ne créez, n'ouvrez et ne planifiez aucun ordre de travail. Terminez le dossier en rappelant que la décision appartient à l'analyste.
- Indiquez la source de chaque chiffre cité : nom de l'outil ou du document.
- Si une donnée manque ou si un outil ne peut pas répondre, écrivez "non disponible" et expliquez pourquoi. N'estimez jamais une valeur.
```

### Pourquoi cette Skill fonctionne

| Choix de conception | Raison |
|---------------------|--------|
| La description emploie les mots de l’analyste : *escalader*, *envoyer une équipe*, *intervenir*, *argumentaire* | C’est ce vocabulaire que l’orchestrateur compare au message ; le mot « escalade » seul ne suffirait pas à déclencher sur « Faut-il envoyer une équipe ? » |
| La description dit explicitement quand **ne pas** l’utiliser | Elle écarte les questions de simple consultation, qui relèvent des connaissances |
| Aucun seuil n’est écrit dans les instructions | Les limites et les niveaux de gravité restent dans les documents de connaissances, seule source de vérité. Si le manuel change de révision, la Skill reste juste |
| Chaque étape nomme l’outil à appeler | La documentation recommande de référencer les outils ; cela rend l’exécution plus prévisible |
| L’étape 4 rappelle le piège du code d’alarme | C’est l’erreur d’analyse la plus probable sur WTG-114, identifiée au cas d’usage nº 2 |
| L’étape 5 prévoit l’erreur de période trop longue | La Skill reste robuste même si la fenêtre de tendance dépassait la limite de 90 jours de l’outil |
| La règle sur les ordres de travail est répétée | Elle figure dans les instructions de l’agent et dans la Skill : deux raisons de la respecter, comme au cas d’usage nº 2 |
| Le format de sortie est imposé | Le besoin métier exige « toujours le même dossier, dans le même ordre » |

> [!NOTE]
> Les participants n’ont pas à reproduire ce texte mot pour mot. Une Skill est correcte dès qu’elle se déclenche sur les deux demandes positives, reste à l’écart sur les deux demandes négatives et remplit la grille de contrôle.

---

## 🧪 Résultats attendus des tests

| Prompt | Skill chargée ? | Comportement attendu |
|--------|-----------------|----------------------|
| « Je veux escalader WTG-114, préparez-moi le dossier. » | Oui | `Loading skill: dossier-escalade`, puis télémétrie, connaissances, ordres de travail et dossier complet |
| « Faut-il envoyer une équipe sur WTG-114 ? Montez-moi l’argumentaire. » | Oui | Même déroulé que ci-dessus |
| « Quelle est la limite d’action des vibrations du multiplicateur de l’Aeris 3.2 ? » | Non | Réponse depuis les connaissances : limite d’action de **4,5 mm/s** |
| « Quel est le délai d’intervention prévu pour un niveau S2 ? » | Non | Réponse depuis les normes Northgate : envoi d’une équipe sous **72 heures** |

Éléments attendus dans le dossier sur WTG-114, tous établis dans les cas d’usage précédents :

* **Vibration du multiplicateur de 4,6 mm/s**, au-dessus de la limite d’action de **4,5 mm/s**.
* **Code d’alarme A212** (seuil d’avertissement) alors que la valeur correspond au seuil d’action **A214**.
* Mesure **valide** : l’éolienne produit au-dessus de **30 % de la puissance nominale**.
* Tendance comparée au seuil de **0,75 mm/s sur une fenêtre de 60 jours**.
* **WO-00001** ouvert, dont les trois tâches (érosion des pales, ventilateur du convertisseur, anémomètre) ne concernent pas le multiplicateur.
* Niveau de gravité **S2**, avec envoi d’une équipe sous **72 heures**.
* **Aucun ordre de travail créé.**

> [!WARNING]
> Ce document n’a pas été rejoué intégralement dans Copilot Studio au moment de sa rédaction. Faites une répétition complète avant la séance, en particulier sur deux points : le déclenchement de la Skill sur la question « Faut-il envoyer une équipe ? », et le niveau de gravité que l’agent retient en combinant la mesure ponctuelle et la tendance.

---

## 🛠️ Dépannage pour l’animateur

| Symptôme | Cause probable | Correction |
|----------|----------------|------------|
| La Skill ne se charge sur aucune demande | Description trop abstraite, qui décrit la procédure au lieu des demandes | Réécrire la description avec les verbes de l’analyste |
| La Skill se charge aussi sur les questions de limite | Description trop large, sans exclusion | Ajouter une phrase « Ne pas utiliser pour… » |
| Le nom est refusé | Majuscules, espaces, accents ou tiret en début ou fin de nom | Respecter le format : minuscules, chiffres, tirets |
| Le dossier cite des seuils faux ou périmés | Seuils recopiés en dur dans les instructions | Retirer les valeurs et demander à l’agent de les chercher dans les connaissances |
| Le dossier ne mentionne pas WO-00001 | Étape des ordres de travail absente ou vague | Nommer `list_work_orders` et `get_work_order` dans la procédure |
| La Skill modifiée ne change rien | Test lancé dans la même conversation, ou agent non enregistré | Sélectionner **Save**, puis relancer dans un **New chat** |

---

## 🎓 Éléments de réponse aux questions de compréhension

* **Quels mots ont permis de charger la Skill sur « Faut-il envoyer une équipe ? » ?** Ceux de la description qui décrivent l’intention de l’analyste plutôt que la procédure : *envoyer une équipe*, *intervenir*, *argumentaire*. L’orchestrateur compare le message à la description ; si elle ne parlait que « d’escalade conforme aux normes », cette demande ne la déclencherait pas.
* **Pourquoi ne pas écrire les seuils dans la Skill ?** Parce qu’un seuil est un fait, et les faits relèvent des connaissances. Recopié dans la Skill, il crée une deuxième source de vérité qui divergera à la prochaine révision du manuel. La Skill décrit **comment** vérifier, les documents disent **par rapport à quoi**.
* **Et si le dossier devait être préparé à chaque conversation ?** La Skill ne serait plus le bon composant : une procédure qui s’applique à chaque tour relève des **instructions**. La Skill a de la valeur précisément parce qu’elle reste hors du contexte quand elle n’est pas pertinente. En pratique, un agent dédié à l’escalade, dont c’est l’unique rôle, porterait cette procédure dans ses instructions.
