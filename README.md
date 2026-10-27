# Pépite

Pépite transforme un lien d'annonce immobilière en verdict argumenté et tient le journal de la recherche. Le projet sert d'abord de terrain d'apprentissage de Claude Code (CLI, app desktop, agents, skills, SpecKit, hooks), avant que le projet pro ne l'impose.

## Principes non négociables

- Zéro coût au-delà de l'abonnement Max : l'IA de l'outil est Claude Code lui-même (`claude -p`), jamais une clé API, pas d'Agent SDK, pas de serveur payé.
- Tout tourne sur le PC perso ; la machine de mission est hors jeu.
- Un concept Claude Code par session : la V1 est un terrain d'apprentissage, pas une course.
- Le code fait le déterministe (parsing, score, verdict par règles, journal) ; Claude fait le jugement (incohérences, appréciation, questions, message).
- Chaque fait porte un statut de preuve (vérifié / annoncé / déclaré / déduit / inconnu) et « Je ne sais pas » est une valeur légitime : l'outil ne comble jamais un trou.
- L'IA peut rétrograder un verdict avec justification, jamais le promouvoir : traduction en code de « ne valorise jamais artificiellement un bien ».
- V0 tout de suite, V1 pour apprendre, V2 pour le « waouh » ; les spikes V2 démarrent pendant la V1, dans des worktrees jetables.

Le plan complet est dans [docs/plan.md](docs/plan.md).
