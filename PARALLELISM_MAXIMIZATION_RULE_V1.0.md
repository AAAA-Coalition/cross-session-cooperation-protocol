# Règle de maximisation du parallélisme — AAAA OS

Version: 1.0
Status: VALIDATED DECISION (John V, 2026-07-23)

## Principe

Avant d'exécuter une séquence d'opérations, se demander systématiquement :
**lesquelles sont réellement indépendantes et peuvent tourner en même temps ?**
Le matériel (RAM/GPU du PC) n'est quasiment jamais le facteur limitant réel
(confirmé ce soir) -- la vitesse vient de la structure du travail, pas de
la machine.

## Ce qui marche (validé en réel ce soir)

1. **Tâches de fond multiples en parallèle** -- installer Node.js, Codex
   CLI et Claude Code CLI simultanément sur le serveur secondaire plutôt que l'un après
   l'autre ; interroger K3 sur une question pendant qu'un déplacement de
   fichier tourne en fond.
2. **Appels K3 simultanés** -- testé explicitement : 2 appels concurrents
   sur la même clé OpenRouter, aucun conflit, aucune limite de débit
   atteinte à cette échelle.
3. **Délégation parallèle Codex + K3** sur des questions indépendantes
   (ex. avis comparatif sur les alternatives C2C) -- converge plus vite
   qu'en séquentiel, et la convergence elle-même est un signal de qualité.
4. **Flux non-concurrents du schéma de coopération** (code K3, vérification
   Codex, revue 004/005, orchestration) -- tournent en parallèle entre eux
   tant qu'ils ne touchent jamais le même fichier au même moment.

## Ce qui ne marche PAS (testé et abandonné, ne pas réessayer)

- **Multiplexage SSH (ControlMaster) sur ce PC Windows/Git-Bash** : casse
  la connexion (`Connection reset by peer`) au lieu de l'accélérer --
  limitation connue des sockets Unix sous Windows pour ce cas précis.
  Chaque commande SSH garde son coût de poignée de main individuel
  (~1-2s), c'est un coût accepté, pas un problème à retenter avec cette
  méthode sur cette plateforme.
- Processus arrière-plan lancés à la main (`nohup ... & disown`) dans le
  Bash tool : ne survivent pas de façon fiable entre les appels d'outils
  sur ce Windows -- utiliser le mécanisme de tâche de fond natif de
  l'outil à la place (déjà la pratique depuis la découverte de ce soir).

## Règle pratique

Avant de lancer une série de commandes : identifier les opérations sans
dépendance entre elles (aucun fichier partagé, aucun résultat de l'une
nécessaire pour l'autre) et les lancer dans le même tour plutôt qu'en
séquence. Ne pas essayer d'optimiser la couche transport (SSH, réseau) au-delà
de ce qui est déjà validé -- l'effort de parallélisation porte sur la
structure des tâches, pas sur l'infrastructure de connexion.
