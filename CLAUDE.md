# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Le projet

Pépite transforme un lien d'annonce immobilière (SeLoger, Leboncoin) en verdict argumenté et tient le journal de la recherche. C'est aussi, et d'abord, un terrain d'apprentissage de Claude Code : un concept par session, la V1 n'est pas une course. Tout est en français (docs, commits, code métier).

**État du dépôt** : pas encore de code. Il contient `docs/plan.md` (la référence : versions V0 → V2, architecture cible, plan des sessions, décisions) et `docs/apprentissage.md` (journal, trois lignes par session). Lire `docs/plan.md` avant toute tâche qui touche à l'architecture ou au périmètre.

## Règles non négociables

- **Zéro coût** : l'IA de l'outil est Claude Code lui-même (`claude -p`), jamais une clé API, pas d'Agent SDK, pas de serveur payé.
- **Aucune donnée personnelle committée** : `docs/private/`, `profil.local.yaml`, `*.local.yaml`, `data/` et les bases locales sont hors git (voir `.gitignore`). Le prompt maître V3 est dans `docs/private/prompt-maitre.md` ; on peut le lire sur demande, jamais en copier le contenu (nom, téléphone, budget) dans un fichier suivi. Les bandes de budget et le plafond vont dans `profil.local.yaml`, avec un `profil.example.yaml` committé.
- **Le code fait le déterministe, l'IA juge** : parsing, score, verdict par règles et journal sont du code testable ; Claude produit les incohérences, l'appréciation, les questions, le message. L'IA peut rétrograder un verdict d'un cran avec une raison citée, jamais le promouvoir.
- **Statut de preuve sur chaque fait** (vérifié / annoncé / déclaré / déduit / inconnu). « Inconnu » est une valeur légitime : l'outil ne comble jamais un trou.
- **Hors périmètre V1** même si c'est tentant : analyse approfondie automatisée, Telegram, dashboard, enrichissement open data, notifications, observabilité. Les spikes V2 vivent dans des worktrees `spike/*`, jamais fusionnés tels quels.

## Architecture cible (V1)

Monolithe modulaire Java 21 / Spring Boot 3, hexagonal, CQRS léger. Modules Maven prévus :

- `pepite-domain` : aucune dépendance Spring ni framework. Cinq bounded contexts : Annonces (`Annonce`, `Fait<T>` + `StatutDePreuve`), Référentiel (`ProfilRecherche`, `Budget`, `Secteur`, `GrilleDeScore`), Analyse (`Évaluation`, `Score`, `DéfautFixe`, `Verdict`, `JugementIA`), Suivi (`DossierBien` machine à états, `ÉvénementSuivi`), Contact (`PremierContact`, `SeuilDeContact` à 380 k€).
- `pepite-application` : use cases (une classe par use case, suffixe `UseCase`), ports, commandes et queries séparées.
- `pepite-adapters/*` : `source-seloger` (Jsoup + fixtures HTML), `source-leboncoin` (Playwright, profil Chrome persistant), `ia-claude` (`claude -p --output-format json`, contrat JSON validé par schéma), `persistence` (H2 fichier), `report-markdown`.
- `pepite-cli` : Spring Boot + picocli, commandes `analyse <url>`, `message <id>`, `classement`, `statut <id> <état>`, `rapport`.

Règles ArchUnit à faire respecter dès que le code existe : seuls les adapters importent Jsoup, Playwright, Spring Data ou Jackson ; l'application ne connaît que le domaine et ses ports ; aucune classe du domaine ne lit `profil.local.yaml`.

Score sur 100 calculé par la grille (secteur 20, prix 20, configuration 15, stationnement 15, extérieur 10, DPE 10, charges 5, travaux 5) ; verdicts PÉPITE / PÉPITE POTENTIELLE / À CONTACTER / BACKUP / ZAP attribués par règles, deal breaker confirmé = ZAP quel que soit le score. Les seuils et la grille sont dans `docs/plan.md`, section Architecture cible.

## Commandes

Aucune pour l'instant. Une fois le squelette Maven créé (session 2) : `mvn -q test` lance tests unitaires et ArchUnit ; les hooks prévus lancent Spotless après édition et `mvn -q test` au Stop.

## Façon de travailler

- Une branche par feature SpecKit (`001-analyse-rapide`, `002-sources`, …), PR, squash merge sur `main`.
- Commits courts et conventionnels (`feat(analyse): grille de score`, `docs: …`), **sans trailer d'attribution**.
- Démarrer chaque tâche en Plan mode ; le plan est validé avant la première édition.
- Fin de session : tests verts, PR fusionnée, ce fichier à jour, trois lignes (appris / surpris / à refaire) ajoutées dans `docs/apprentissage.md`.
- Sites sources : volume humain (20 à 30 fiches par jour au plus), navigateur piloté par l'utilisateur, aucun crawl nocturne.
