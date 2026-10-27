# Pépite — Plan de projet V0 → V2 et parcours Claude Code

27 octobre 2026 · Ramzi

Version vivante (avec les deux schémas) : https://claude.ai/artifact/B8bnW9WJAuMwotU2gxNaBp

## Résumé et principes

Objectif n°1 : savoir travailler avec Claude Code (CLI, app desktop, agents, skills, SpecKit, hooks) avant que le projet pro ne l'impose. Objectif n°2 : un outil qui transforme un lien d'annonce en verdict argumenté et tient le journal de la recherche. Le prompt maître V3 sert de spécification fonctionnelle : on le découpe, on ne le jette pas.

Principes non négociables :

- Zéro coût au-delà de l'abonnement Max : l'IA de l'outil est Claude Code lui-même (`claude -p`), jamais une clé API, pas d'Agent SDK, pas de serveur payé.
- Tout tourne sur le PC perso ; la machine de mission est hors jeu.
- Un concept Claude Code par session : la V1 est un terrain d'apprentissage, pas une course.
- Le code fait le déterministe (parsing, score, verdict par règles, journal) ; Claude fait le jugement (incohérences, appréciation, questions, message).
- Chaque fait porte un statut de preuve (vérifié / annoncé / déclaré / déduit / inconnu) et « Je ne sais pas » est une valeur légitime : l'outil ne comble jamais un trou.
- L'IA peut rétrograder un verdict avec justification, jamais le promouvoir : traduction en code de « ne valorise jamais artificiellement un bien ».
- V0 tout de suite, V1 pour apprendre, V2 pour le « waouh » ; les spikes V2 démarrent pendant la V1, dans des worktrees jetables.

## Ce que devient le prompt maître

Le prompt maître V3 est déjà une spec : l'essentiel de son contenu est du domaine et des règles métier, pas des instructions pour un LLM, et c'est exactement ce qu'on va transformer en code. Ses points forts sont à garder tels quels : un langage ubiquitaire déjà fixé (PÉPITE → ZAP, défaut fixe, deal breaker, micro-secteur, statut de preuve, seuil de contact), deux modes d'analyse qui sont deux use cases distincts, la règle « n'invente jamais » comme contrainte de conception, et une section 19 (journal) qui décrit déjà le contexte Suivi.

| Section du prompt | Devient | Version |
| --- | --- | --- |
| 1. Priorités, ton, format (« Compréhension : », corrections) | Skill `analyse-rapide` : ton et format seulement | V1 |
| 2A. Analyse rapide | Use case `AnalyserAnnonceRapide` | V1 |
| 2B + 5. Analyse approfondie, QPV, micro-secteur | Use case assisté + connecteurs open data ; manuel dans le projet Claude.ai en attendant | V2 |
| 3, 6, 7, 8. Projet de vie, critères maison / appartement, stationnement | `ProfilRecherche` (config locale) + règles de score et deal breakers | V1 |
| 4. Territoires | Référentiel `Secteur` : table éditable commune / arrondissement / quartier → priorité ; géocodage en V2 | V1, V2 |
| 9. Budget | Value object `Budget` : bandes 310-330 / 335 / 350 k€, plafond privé jamais exporté | V1 |
| 10, 11. Premier contact et style des messages | Use case `RédigerPremierContact` : seuil 380 k€ en code, modèles en templates, 1-2 questions choisies par l'IA | V1 |
| 12. Négociation | Reste conversationnel (chat), hors outil | — |
| 13, 14, 15. Défauts fixes, technique, juridique | Deal breakers codés + checklist de preuves avant visite / offre / compromis | V1 |
| 16. Prix de marché (DVF) | Connecteur DVF | V2 |
| 17, 18. Verdicts et structure de restitution | `Verdict` + règles d'attribution ; rendu markdown (V1) puis carte Telegram (V2) | V1, V2 |
| 19. Journal | Bounded context Suivi, en CQRS | V1 |
| 20. Sources de vérification | Adapters d'enrichissement (Géorisques, SIG Ville, DVF, transports) | V2 |
| 21. Format d'envoi | CLI `pepite analyse <url>` puis partage Telegram | V1, V2 |
| 22. Arbitrages confirmés | Tests de non-régression sur la config | V1 |

Changements à faire avant d'écrire une ligne de code :

1. Séparer trois natures de contenu : règles pour le LLM (ton, format) → skill ; connaissance métier (critères, seuils, secteurs, verdicts) → domaine et config ; processus (modes d'analyse, journal, contact) → use cases.
2. Sortir les données personnelles du dépôt : numéro de téléphone, nom, bandes de budget, plafond → `profil.local.yaml` ignoré par git, avec un `profil.example.yaml` committé.
3. Ajouter un score numérique : le prompt ne produit qu'un verdict, l'IA le rend différemment d'un jour à l'autre. La grille (section Architecture) calcule le score, les règles proposent le verdict, l'IA ne peut que le rétrograder.
4. Assumer que l'analyse approfondie reste semi-automatique : documents, visite et notaire ne s'automatisent pas. La V2 automatise la collecte (risques, QPV, trajets, DVF), Claude synthétise.
5. Transformer les secteurs en données : une table éditable, pas trois paragraphes de prompt.
6. Garder le prompt maître intact pour la V0, avec une ligne en plus : « lien SeLoger → récupère la page ; lien Leboncoin → ouvre-le dans mon Chrome, déplie la description et les critères, lis le DPE et le GES ».

## Les trois versions

La V0 enlève la douleur du copier-coller dès ce soir, la V1 sert à apprendre Claude Code sur un périmètre volontairement réduit, la V2 est l'appli « waouh » qui se construit par flux parallèles une fois la V1 solide.

| | V0 — maintenant | V1 — apprendre | V2 — le waouh |
| --- | --- | --- | --- |
| Capture | Lien SeLoger collé ; Leboncoin ouvert dans ton Chrome | `pepite analyse <url>` | Partager depuis l'app vers Telegram |
| Analyse | Prompt maître dans le projet Claude.ai | Analyse rapide automatisée | Rapide automatique + approfondie assistée |
| Suivi | Éparpillé dans les chats | Journal local, classement en CLI | États, relances, baisses de prix, dashboard |
| Tourne sur | Claude.ai et ton Chrome | Ton PC, ligne de commande | Ton PC, service Spring Boot + Angular |
| Coût | 0 | 0 | 0 |
| Durée | 10 minutes de réglage | 6 sessions, 3 à 4 semaines | Flux continu |

**V0 — zéro code.** Projet Claude.ai + prompt maître + la ligne d'instruction Chrome / SeLoger. Depuis le téléphone : coller le lien dans le chat du projet ; Leboncoin passe tant que le PC tourne avec Chrome et l'extension, sinon coller les liens dans la journée et traiter le soir en lot.

**V1 — périmètre exact.** Dans le périmètre :

- CLI `pepite` : `analyse <url>`, `message <id>`, `classement`, `statut <id> <état>`, `rapport`.
- Sources : SeLoger (HTTP + Jsoup, fixtures HTML), Leboncoin (Playwright, profil Chrome persistant).
- Analyse rapide : parsing avec provenance → score par grille → jugement IA (`claude -p` + skill) → verdict par règles.
- Premier message selon le seuil de 380 k€, à valider par toi avant envoi.
- Journal local (H2 en fichier), classement, rapport markdown.
- Tests : fixtures par source, ArchUnit, evals du skill sur 10 à 15 annonces de ton historique.

Hors périmètre V1, même si c'est tentant : analyse approfondie automatisée, Telegram, dashboard, enrichissement open data, notifications, observabilité.

**V2 — le scénario cible.** Tu partages une annonce depuis l'app Leboncoin vers Telegram. Une minute plus tard, le bot répond : score, verdict, trois points clés, incohérences détectées, inconnues, et quatre boutons : Message, Approfondir, Visite, Zap. Le dashboard classe tout, la carte montre les secteurs et les QPV, les baisses de prix et les relances arrivent seules.

## Architecture cible

Monolithe modulaire Java 21 / Spring Boot 3 en hexagonal, cinq bounded contexts, CQRS léger en V1, événements et outbox en V2 : le même vocabulaire que ton projet pro, pour que tout ce que tu apprends ici se transfère directement.

Pourquoi Java plutôt que TypeScript : le transfert vers le projet pro est l'objectif n°1, et tout l'outillage Claude Code que tu vas construire (CLAUDE.md, hooks Maven / Spotless / ArchUnit, sous-agents Java) se réutilise tel quel. Playwright, Jsoup et Jackson couvrent le scraping sans effort. TypeScript serait plus léger mais t'apprendrait moins pour le pro.

| Contexte | Responsabilité | Objets clés |
| --- | --- | --- |
| Annonces | Capturer et normaliser une annonce, chaque fait avec sa provenance | `Annonce`, `Fait<T>` + `StatutDePreuve`, port `SourceAnnonce` |
| Référentiel | Ce que tu cherches, en données éditables | `ProfilRecherche`, `Budget`, `Secteur` + `NiveauPriorité`, `GrilleDeScore` |
| Analyse | Évaluer une annonce contre le profil | `Évaluation`, `Score`, `DéfautFixe`, `Verdict`, `JugementIA` |
| Suivi | Journal et cycle de vie d'un bien | `DossierBien` (machine à états), `ÉvénementSuivi`, `Relance` |
| Contact | Messages au vendeur | `PremierContact`, `SeuilDeContact` (380 k€), modèles agence / particulier |

> Schéma « architecture hexagonale · 5 contextes, 3 entrées, 5 sorties » : dans le doc Claude (lien en tête). En résumé : à gauche ce qui appelle le domaine (CLI en V1, bot Telegram et web Angular en V2), au centre application + domaine, à droite ce que le domaine appelle (SeLoger HTTP/Jsoup, Leboncoin Playwright, Claude CLI, base H2 puis PostgreSQL, open data et Telegram en V2). Remplacer le HTTP de SeLoger par Playwright ne touche qu'une boîte de droite ; brancher Telegram n'ajoute qu'une boîte de gauche.

Modules Maven : `pepite-domain` (aucune dépendance Spring), `pepite-application` (use cases, ports, commandes et queries), `pepite-adapters` (un module par adapter : `source-seloger`, `source-leboncoin`, `ia-claude`, `persistence`, `report-markdown` ; en V2 `telegram`, `enrichment-*`, `web-api`), `pepite-cli` (Spring Boot + picocli). En V2 s'ajoutent `pepite-runtime` (le service qui tourne sur ton PC) et `pepite-web` (Angular).

CQRS, progressivement : en V1, commandes (`AnalyserAnnonce`, `ChangerEtat`, `EnregistrerMessageEnvoyé`) et queries (`Classement`, `Comparaison`, `ARelancer`) sont des handlers séparés, le read model `Classement` est projeté dans la même transaction. En V2, les commandes émettent des événements de domaine (`AnnonceAnalysée`, `VerdictAttribué`, `PrixBaissé`, `MessageEnvoyé`) publiés par outbox ; les projections du dashboard et les notifications Telegram les consomment. L'historique d'un bien (prix, messages, visites) est naturellement un flux d'événements : c'est ton cas d'école pour l'event sourcing si tu veux aller jusque-là.

Score hybride, le code décide, l'IA juge. Grille de départ sur 100, à calibrer sur ton historique :

| Critère | Source du fait | Poids |
| --- | --- | --- |
| Secteur (forte priorité / prudent / pénalisé) | Référentiel | 20 |
| Prix vs bandes (≤ 330 / ≤ 335 / ≤ 350 / > 350 k€) | Annonce + budget | 20 |
| Configuration (3 ch. + bureau / 2 ch. + bureau / 2 ch. sans bureau) | Annonce + IA | 15 |
| Stationnement (prouvé / annoncé / aucun) | Annonce + statut de preuve | 15 |
| Extérieur utile et intime | Annonce + IA | 10 |
| DPE (A-D / E-G) | Annonce | 10 |
| Charges et copropriété | Annonce | 5 |
| Travaux structurels, nuisances connues | IA, enrichissement en V2 | 5 |

Règles de verdict : un deal breaker confirmé donne ZAP quel que soit le score ; au-dessus de 75 avec peu d'inconnues, PÉPITE ; au-dessus de 75 avec des inconnues déterminantes (adresse, statut du parking), PÉPITE POTENTIELLE ; 55 à 75, À CONTACTER ; 40 à 55, BACKUP ; en dessous, ZAP. L'IA reçoit le verdict proposé et peut le rétrograder d'un cran avec une raison citée, jamais le monter.

Contrat JSON du jugement IA (ce que `claude -p` doit renvoyer, validé par schéma) :

```json
{
  "annonce_id": "lbc:3294104425",
  "faits_extraits": [{"champ": "surface_habitable", "valeur": "100 m²", "statut": "annoncé"}],
  "incoherences": ["80 m² en tête d'annonce, 100 m² dans les informations clés"],
  "defauts_fixes": [{"code": "STATIONNEMENT_NON_PRIVATIF", "statut": "non_verifie"}],
  "bureau_separe": "plausible | non | inconnu",
  "inconnues_determinantes": ["adresse exacte", "statut juridique du parking"],
  "questions_prioritaires": ["Le stationnement est-il un lot privatif ?"],
  "appreciation": "deux phrases, sans vocabulaire commercial",
  "retrogradation": {"appliquer": false, "raison": null}
}
```

Règles ArchUnit, vérifiées à chaque `mvn test` et par un hook Stop : le domaine ne dépend d'aucun framework ; l'application ne connaît que le domaine et ses ports ; seuls les adapters importent Jsoup, Playwright, Spring Data ou Jackson ; un use case par classe, suffixé `UseCase` ; aucune classe du domaine ne lit `profil.local.yaml`.

Stack : Java 21, Spring Boot 3, Maven multi-modules, picocli, Jsoup, Jackson, Playwright Java, H2 en fichier (V1) puis PostgreSQL en Docker (V2), JUnit 5 + AssertJ + ArchUnit, Spotless, GitHub Actions.

## V1 — plan de sessions

Sept sessions de 1 h 30 à 2 h, chacune livre une brique produit et enseigne une brique Claude Code ; une session se termine par une PR fusionnée et trois lignes dans `docs/apprentissage.md` (appris, surpris, à refaire). Les liens pointent vers la [documentation Claude Code](https://code.claude.com/docs/en/overview.md), consultée le 27 octobre 2026.

1. **Session 0 — Installation et premier contact (1 h, avant le démarrage pro).**
    - Produit : dépôt privé `pepite` sur GitHub, README avec les principes.
    - Claude Code : installer la [CLI](https://code.claude.com/docs/en/setup.md), l'[app desktop](https://code.claude.com/docs/en/desktop-quickstart.md) et le [plugin JetBrains](https://code.claude.com/docs/en/jetbrains.md) ; `claude` dans le dépôt ; `/init` ; les trois modes de permission (Plan, Accept edits, Auto) ; `@fichier` ; `/compact`, `/clear`, `/memory`, `/doctor` ; demander à Claude Code d'expliquer Claude Code ([quickstart](https://code.claude.com/docs/en/quickstart.md), [comment ça marche](https://code.claude.com/docs/en/how-claude-code-works.md)).
    - À savoir faire : démarrer, reprendre et nommer une session ; lire un diff avant d'accepter ; interrompre ; revenir en arrière (checkpoints, git).
2. **Session 1 — SpecKit, de la spec aux tâches.**
    - Produit : `constitution.md` (zéro coût, hexagonal, tests par adapter, pas de donnée perso committée), spec de la feature « analyse rapide », plan, tâches.
    - Claude Code : installer [SpecKit](https://github.com/github/spec-kit) (`specify init`), enchaîner constitution → specify → clarify → plan → tasks ; relire et corriger la spec à la main ; comprendre ce que chaque commande génère.
    - À savoir faire : écrire une constitution courte ; faire clarifier plutôt que laisser deviner.
3. **Session 2 — Domaine, CLAUDE.md, premiers harnais.**
    - Produit : modules Maven, domaine (`Annonce`, `Fait`, `Verdict`, `Secteur`, `Budget`), règles ArchUnit, tests unitaires.
    - Claude Code : [CLAUDE.md](https://code.claude.com/docs/en/memory.md) court + `.claude/rules/` par chemin ; [hooks](https://code.claude.com/docs/en/hooks-guide.md) : `PostToolUse` qui lance Spotless, `PreToolUse` qui bloque `rm -rf`, `git push --force` et toute édition de `profil.local.yaml`, `Stop` qui lance `mvn -q test` et renvoie Claude au travail si c'est rouge ; permissions dans `settings.json` ([bonnes pratiques](https://code.claude.com/docs/en/best-practices.md)).
    - À savoir faire : voir Claude se faire rattraper par un hook ; distinguer une règle déterministe (hook) d'une instruction (CLAUDE.md).
4. **Session 3 — Adapters sources, sous-agents, worktrees.**
    - Produit : `SeLogerSource` (Jsoup, fixtures HTML sauvegardées), `LeboncoinSource` (Playwright, profil Chrome persistant, lecture du JSON embarqué dans la page).
    - Claude Code : [sous-agents](https://code.claude.com/docs/en/sub-agents.md) `reviewer` (lecture seule, vérifie les frontières et la couverture) et `qa` ; deux [worktrees](https://code.claude.com/docs/en/worktrees.md), une session par adapter ; [revue locale](https://code.claude.com/docs/en/code-review-local.md) avant PR.
    - À savoir faire : déléguer une tâche cadrée, lire le rapport d'un sous-agent, fusionner deux branches parallèles.
5. **Session 4 — L'analyseur IA : skill, headless, evals.**
    - Produit : skill `analyse-rapide` (ton + jugement, contrat JSON), adapter `ClaudeCliAnalyseur` qui appelle `claude -p --output-format json`, grille de score, règles de verdict ; evals sur 10 à 15 annonces de ton historique avec leur verdict attendu.
    - Claude Code : écrire un [skill](https://code.claude.com/docs/en/skills.md) ; le [mode headless](https://code.claude.com/docs/en/headless.md) (`-p`, `--json-schema`, `--allowedTools`, permissions sans prompt) ; evals avec skill-creator ; token longue durée si un script doit s'authentifier ([authentification](https://code.claude.com/docs/en/authentication.md)).
    - À savoir faire : faire passer un prompt de « mère » à « skill versionné et testé ».
6. **Session 5 — Journal, CQRS, classement, PR flow.**
    - Produit : persistance H2, commandes et queries séparées, read model `Classement`, rapport markdown, commandes CLI `classement`, `statut`, `rapport`.
    - Claude Code : une [agent team](https://code.claude.com/docs/en/agent-teams.md) ou trois sous-agents (dev, reviewer, qa) sur une seule feature ; PR avec `gh` ; CI GitHub Actions (tests + ArchUnit, sans Claude dedans pour rester à zéro coût) ; [`/goal`](https://code.claude.com/docs/en/goal.md) jusqu'au vert.
    - À savoir faire : orchestrer sans micro-manager ; relire une PR produite par des agents comme celle d'un collègue.
7. **Session 6 — Premier message, packaging, rétro.**
    - Produit : `RédigerPremierContact` (seuil 380 k€, modèles agence / particulier), jar exécutable, README d'usage.
    - Claude Code : [tâches planifiées](https://code.claude.com/docs/en/scheduled-tasks.md) (`/loop`) ou tâche desktop « analyse l'inbox » ; export de session ; mise à jour de CLAUDE.md avec les leçons.
    - À savoir faire : dire ce que tu referais autrement ; décider ce qui passe en V2.

Règle de fin de session, sans exception : hooks verts, PR fusionnée, `CLAUDE.md` à jour, aucune donnée personnelle dans le dépôt.

## V2 — le waouh

La V2 tient en une phrase : partager une annonce depuis le téléphone et recevoir, une minute plus tard, un verdict argumenté avec les actions à un clic, pendant que le dashboard et le journal se tiennent seuls à jour. Six flux indépendants, chacun une feature SpecKit dans son worktree, chacun branché sur les ports de la V1.

> Schéma « pipeline V2 · 10 étapes » : dans le doc Claude. Partager (app Leboncoin ou SeLoger) → Bot Telegram (reçoit le lien, file d'attente) → Récupérer (Playwright, HTTP, profil Chrome) → Parser (faits typés avec provenance) → Dédoublonner (Leboncoin et SeLoger) → Enrichir (QPV, risques, trajets, écoles) → Scorer (grille, deal breakers) → Juger (IA : claude -p, skill, contrat JSON) → Verdict (règles, journal, événements) → Notifier (carte Telegram + dashboard). Les étapes Récupérer, Parser, Scorer et Verdict existent déjà en V1 derrière la CLI ; la V2 ajoute les deux bouts (Telegram) et les deux étapes du milieu (Dédoublonner, Enrichir).

| Flux | Contenu | Démarre après |
| --- | --- | --- |
| A — Inbox et temps réel | Bot Telegram (long polling dans `pepite-runtime`), file d'attente, pipeline complet, carte de réponse avec boutons Message / Approfondir / Visite / Zap, commandes `/classement`, `/relances` | Session 5 |
| B — Enrichissement open data | [API Adresse](https://adresse.data.gouv.fr/api-doc/adresse) (géocodage), [Géorisques](https://www.georisques.gouv.fr/doc-api) (risques à l'adresse), périmètres QPV 2024 de SIG Ville en GeoJSON (distance et statut), OSRM ou OpenRouteService (temps vers Aix, La Duranne, Venelles à 8 h), Overpass / OpenStreetMap (commerces, pharmacie, écoles, arrêts à pied), DVF (ventes actées dans la rue), base DPE ADEME (contrôle du DPE annoncé) | Session 3 |
| C — Suivi avancé | Machine à états (repérée → analysée → contactée → visite → offre → compromis / écartée), relances automatiques (pas de réponse sous 5 jours), re-lecture quotidienne des annonces suivies (baisse de prix, retrait), détection des doublons Leboncoin / SeLoger, checklist de visite générée depuis l'analyse, compte rendu post-visite dicté dans Telegram | Session 5 |
| D — Dashboard Angular | Classement par score, carte Leaflet / OpenStreetMap avec secteurs et QPV, comparaison de 2 ou 3 biens, matrice des preuves (vérifié / annoncé / inconnu), funnel repérée → contactée → visitée → offre | Session 5 |
| E — Observabilité (optionnel, pour la pratique) | Actuator + Micrometer → Prometheus → Grafana (annonces par jour, répartition des verdicts, latence de `claude -p`, €/m² par secteur) ; OpenTelemetry → Jaeger, une trace par annonce à travers le pipeline ; tout en docker-compose | Flux A |
| F — Qualité continue | Evals du skill rejouées à chaque changement, fixtures par source, contrat JSON testé, revue de PR par agent | Session 4 |

Tout reste gratuit : Telegram Bot API, API Adresse, Géorisques, SIG Ville, OpenStreetMap et DVF sont des services publics ou ouverts ; OSRM public et OpenRouteService ont un quota gratuit largement suffisant pour une vingtaine d'annonces par jour.

Spikes à lancer pendant la V1, un soir chacun, dans un worktree `spike/*`, conclusion dans `docs/spikes/` :

- [ ] Telegram hello-world : recevoir un lien, répondre « reçu ».
- [ ] Playwright + profil Chrome sur une fiche Leboncoin : extraire le JSON embarqué plutôt que le HTML.
- [ ] API Adresse + Géorisques + QPV sur une adresse connue : les trois appels et leur format.
- [ ] OSRM : temps Marseille → La Duranne à 8 h, comparer avec ton trajet réel.
- [ ] `claude -p --json-schema` appelé depuis Java (ProcessBuilder) : temps de réponse et stabilité du JSON.

Hors scope, et pourquoi : Keycloak (un seul utilisateur, en local), Elasticsearch (quelques centaines d'annonces, PostgreSQL suffit ; exercice possible en V3), messagerie temps réel (Telegram est la messagerie), serveur distant et API payantes (zéro coût), analyse des photos par IA (plus tard, si l'envie reste).

## Organisation du travail

Un dépôt, une branche par feature, une PR relue par un agent puis par toi, et une session de travail qui se termine toujours par quelque chose de fusionné.

- Dépôt privé GitHub `pepite` ; `main` protégée même en solo ; une branche par feature SpecKit (`001-analyse-rapide`, `002-sources`, …) ; squash merge.
- Commits courts et conventionnels (`feat(analyse): grille de score`), sans trailer d'attribution.
- Session type, 1 h 30 à 2 h : objectif écrit en une ligne → Plan mode → implémentation → hooks verts → PR → trois lignes dans `docs/apprentissage.md`.
- Définition de « fini » : tests et ArchUnit verts, PR fusionnée, `CLAUDE.md` à jour, aucune donnée personnelle committée.
- Données : `profil.local.yaml` et la base locale hors git ; sauvegarde hebdomadaire chiffrée (les adresses et les échanges avec les vendeurs sont dedans).
- Parallélisme : V1 sur `main` ; spikes V2 dans des worktrees `spike/*`, jetables, jamais fusionnés tels quels.
- Respect des sites : navigateur piloté par toi, volume humain (20 à 30 fiches par jour au plus), aucun crawl nocturne. Les CGU Leboncoin interdisent l'extraction automatisée et son `robots.txt` ferme les fiches aux robots : le curseur est à toi, et il va dans la constitution.

## Kit de survie Claude Code pour le projet pro

Dans deux jours tu rejoins une équipe qui a déjà ses habitudes Claude Code : l'objectif de la première semaine est de ne rien casser, de lire beaucoup et de copier ce qui marche chez tes collègues.

- [ ] Avant : installer la CLI, l'app desktop et le plugin JetBrains ; `claude doctor` ; une session 0 sur `pepite` pour avoir déjà vu une permission, un diff et un `/compact`.
- [ ] Jour 1 : lire `CLAUDE.md`, `.claude/` (skills, agents, hooks, settings) et `specs/` du dépôt ; demander à Claude Code « explique-moi comment ce dépôt est configuré pour toi ».
- [ ] Chaque tâche démarre en Plan mode ; le plan est validé avant la première édition.
- [ ] Une tâche = une session ; `/compact` quand le contexte grimpe, `/clear` entre deux sujets, `/context` pour voir ce qui est chargé.
- [ ] Lire chaque diff ; aucun `git push` sans relecture ; pas de mode Auto la première semaine.
- [ ] Commits sous l'identité pro du dépôt (`git config user.email` vérifié), messages dans le style du repo, pas de trailer d'attribution (`includeCoAuthoredBy` à `false` dans les settings si l'équipe fait ainsi).
- [ ] `@chemin/fichier` pour cibler ; « lis d'abord X, puis propose » plutôt que « fais ».
- [ ] Sous-agents : `Explore` pour comprendre le code, un `reviewer` avant chaque PR ; pas cinq agents le premier jour.
- [ ] SpecKit : suivre le flux de l'équipe (constitution → specify → clarify → plan → tasks → implement) ; une spec se relit comme une PR.
- [ ] Quand ça part en vrille : Esc, revenir au dernier checkpoint ou à git, reformuler plus petit.
- [ ] Observer comment tes collègues formulent leurs prompts et quels skills ils déclenchent : c'est le meilleur cours disponible.

## Décisions prises (27 octobre 2026)

- Stack : Java 21 / Spring Boot 3.
- Nom du projet : Pépite.
- Inbox V2 : Telegram (le partage existe sur iOS comme sur Android).
- Le PC perso reste allumé en journée ; la V2 tourne dessus.
- Session 0 avant le démarrage pro.
- Jeu d'evals : extraire 10 à 15 analyses de référence avec leur verdict depuis le projet Claude.ai « achat résidence principale ».

Sources : [documentation Claude Code](https://code.claude.com/docs/en/overview.md) (setup, memory, hooks, sub-agents, agent teams, skills, headless, worktrees, desktop, JetBrains ; consultée le 27 octobre 2026), [SpecKit](https://github.com/github/spec-kit), [API Adresse](https://adresse.data.gouv.fr/api-doc/adresse), [Géorisques](https://www.georisques.gouv.fr/doc-api), le prompt maître V3 du 13 octobre 2026.
