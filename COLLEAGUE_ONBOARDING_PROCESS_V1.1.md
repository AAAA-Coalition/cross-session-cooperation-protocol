# Processus d'onboarding d'un nouveau collègue/partenaire/étudiant/alumni workshop — v1.1

**Statut : v1.1** — intègre les modifications de John du 20/08/2026 sur le v1.0 : un système
de catégories (collègue/partenaire/étudiant/alumni workshop), une étape 0 étendue avec des
sous-étapes concrètes, une 5e décision ouverte, et l'accès au serveur collectif explicitement conditionné à
la catégorie. Deux des trois trous signalés en v1.0 sont maintenant comblés (voir "Ce qui
manque reellement" ci-dessous) ; un reste ouvert.

Séquence complète pour faire passer un nouveau collègue de "rien" à "Claude opérationnel,
connecté à son serveur personnel, et au serveur collectif (si membre participant à une
candidature Horizon ou Erasmus), enregistré C2C selon catégorie
(collègue/partenaire/étudiant/alumni workshop), aligné sur les règles du projet."

Document compagnon : `CLAUDE_BEHAVIOR_CHARTER_V1.1.md` (la charte de comportement Claude
Code elle-même, étape 3 ci-dessous).

## Etape 0 — Avant coopération (la session qui prépare l'onboarding)

- Créer son compte Claude (télécharger Claude Code pour desktop).
- Connecter GitHub, une app 2FA et OpenRouter (LLM gratuit, pas de carte bancaire).
- Louer son serveur personnel (un VPS chez Contabo ou équivalent, référence réelle : 24 Go RAM,
  Ubuntu 24.04, 300 Go de disque, pas de backup).
- Aller sur la page du compte Contabo et préparer une clé SSH dédiée
  (`ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_<nom>_serveur`).
- **Décider l'identité C2C du collègue (`conv-<descriptif>-<NNN>`) et créer sa mailbox.**
- Nb : 5 décisions ouvertes, jamais tranchées, à trancher avant de lancer le premier
  onboarding réel (source : la checklist d'onboarding du serveur d'un collègue, document interne
  du 12/08/2026) :
  1. Décider quelle catégorie et quel projet (allowlist par initiative), et
     éventuellement un espace GitHub/Drive dédié.
  2. Orthographe/convention de nom pour ce collègue.
  3. Provenance du binaire de l'agent de recherche.
  4. Clé OpenRouter dédiée au collègue vs clé partagée.
  5. Convention de nommage du conv-id pour ce type précis d'identité (aucun précédent
     net).

## Etape 1 — Session 1 : brancher les outils (30-60 min, une seule fois)

Suivre `c2c-os/01_project_memory/ONBOARDING_COLLEAGUE_SESSION1_TO_SESSION2_20260806.md` tel
quel (7 étapes détaillées : SSH + `~/.ssh/config`, déploiement de l'agent de recherche, connexion
des 3 MCP de base, bot Telegram dédié, enregistrement C2C, clôture propre de la session 1,
pipeline média optionnel).

Points d'attention déjà documentés à ne pas redécouvrir :

- Deux systèmes MCP distincts sur PC (`claude mcp add` CLI vs panneau Connecteurs Desktop)
  — ne pas les confondre.
- Scripts `.ps1` nécessitent `-ExecutionPolicy Bypass`.
- Historiquement, Avast a bloqué la connexion Claude-in-Chrome (clé de registre
  native-messaging) — vérifier si ça se reproduit (cf. le problème similaire trouvé le
  20/08 sur le spawn de session VS Code, `PIEGES_CONNUS.md`).

## Etape 2 — Déployer le socle du serveur personnel du collègue

Suivre la checklist d'onboarding du serveur d'un collègue (document interne du 12/08/2026,
checklist en 10 étapes, commandes SSH/systemd exactes) pour le socle générique : utilisateur
de service dédié, `ufw` (port 22 seul ouvert par défaut), `/opt/votre-deploiement/`, l'agent de
recherche + sa config, gateway LiteLLM free (1
clé OpenRouter : routine ~0€/j, interactif ~0€/mois), moteur Whisper si besoin, identité
C2C + mailbox, watchdog d'identité. Ne pas répliquer par défaut les outils propres aux
projets personnels de John — seulement le socle générique explicitement listé dans ce
document.

Pour déployer un outil MCP spécifique (au-delà du socle), suivre le tutoriel de déploiement
d'un agent MCP (document interne du 04/08/2026 ; piège réel
documenté : épingler `mcp==1.29.0`, vérification par vrai appel JSON-RPC `initialize`, pas
juste `systemctl is-active`).

**Honnêteté à transmettre au collègue** : cette checklist n'a jamais été testée de bout en
bout sur un vrai VPS collègue à la date de rédaction (12/08) — prévoir un vrai test complet,
pas une simple lecture, avant de la considérer fiable.

## Etape 3 — Installer la charte de comportement

Coller `CLAUDE_BEHAVIOR_CHARTER_V1.1.md` au démarrage de la première vraie session de
travail du collègue (Session 5, créée via le bot Telegram dédié, séparée de la Session 1
d'installation) — ou l'intégrer comme fichier de mémoire de départ. Expliquer explicitement
au collègue humain pourquoi cette charte existe (cf. préambule du fichier) : son Claude est
neuf, sans historique appris, la charte remplace des dizaines de corrections répétées.

## Etape 4 — Protocole C2C complet

Faire lire au collègue (ou à son Claude) `c2c-os/00_manifest/CONVERSATION_PROTOCOL_V1.0.md`
en entier — c'est la référence complète et déjà écrite (cycle
REQUEST->ACK->RESPONSE->CLOSE, format YAML des messages, chemins de mailbox, règles
Telegram READ_ONLY). Compléter avec la checklist "Future participant onboarding" (section
12 de ce même document) et le pack de démarrage
`c2c-os/07_validated/ELYSE_COLLEAGUE_AAAA_OS_STARTER_PACK_V0.1.md` (le plus complet déjà
validé : 12 fichiers à fournir en premier, séquence VPS en 10 étapes, séquence C2C en 9
étapes).

Première action C2C réelle du nouveau collègue : une REQUEST de présentation dans sa propre
mailbox, avec accusé de réception d'une identité déjà enregistrée (John ou une session
existante confirme la réception).

## Etape 5 — Accès au serveur collectif SI COLLÈGUE de la CATÉGORIE PARTENAIRE de CANDIDATURE
HORIZON ou ERASMUS — **trou réel, pas encore pleinement comblé**

**NB :**

**Aucun document formalisant l'accès d'un collègue au serveur collectif n'existe dans ce dépôt** —
recherche dédiée faite, rien trouvé. Ce qui est établi à la place : chaque collègue reçoit
son **propre** serveur personnel (étape 2) ; l'accès au serveur collectif (celui qui héberge
Mixpost/Sharetribe/EU Funding/Docmost/l'administration de l'agent de recherche/n8n) reste
aujourd'hui un accès SSH direct réservé à John et aux futurs collègues de la catégorie
"partenaire — candidature Horizon/Erasmus", pour les sessions déjà autorisées.

**Décision requise de John avant d'aller plus loin sur ce point** : le serveur collectif n'est
délibérément pas partagé avec les nouveaux collègues avant confirmation de leur catégorie.
Si un accès partagé est enclenché, il faut le concevoir (quels services, quel niveau —
lecture seule recommandée par défaut, cf. le même principe déjà appliqué à la question
d'accès aux sous-agents de recherche posée par ALLO Desktop le 19/08).

**Note d'obsolescence — maintenant corrigée (20/08/2026)** :
`c2c-os/04_architecture/AAAA_OS_INFRASTRUCTURE_MAP_V1.0.md` (08/07) affirmait que le serveur
collectif était « exclu sauf autorisation séparée » — contredit par l'usage réel documenté
depuis (serveur collectif très actif). La carte a maintenant été mise à jour (V1.1, 20/08/2026)
avec une vraie section sur le serveur collectif, construite à partir de faits déjà documentés
ailleurs dans le dépôt. **Ce qui
reste ouvert, c'est le document d'accès lui-même, pas la carte.**

## Etape 6 — Lecture des pièges connus et de la discipline de coopération

Faire lire au collègue (ou son Claude) : `c2c-os/PIEGES_CONNUS.md` (court, vivant, réflexe
de consultation avant toute tâche non triviale) et
`c2c-os/00_manifest/PROCESS_COOPERATION_MASSIVEMENT_PARALLELE_V2_20260819.md` (isolation
par worktree, `ACTIVE_CLAIMS.md`, discipline anti-collision). Pour un panorama plus large
des erreurs déjà commises et corrigées (~70 cas réels), voir
`c2c-os/01_project_memory/erreurs_connues_20260809/ERREURS_CONNUES_ET_LECONS_COLLEGUES_20260809.md`.

## Etape 6bis — Les bots : role, comportement, verification

Ajoutee le 04/09/2026, apres un incident : un bot ajoute a un groupe de travail a recu
trois fois la meme question sans y repondre, puis a repondu par un nom de classe
d'exception. Trois defauts empiles, tous invisibles depuis Telegram.

Faire lire au collegue (ou son Claude) : `docs/ROLE_DES_BOTS_AAAA_OS_20260904.md`
(version anglaise : `docs/ROLE_OF_BOTS_AAAA_OS_20260904_EN.md`).

Les trois points a retenir avant de toucher a un bot :

1. **Un bot est une porte, pas un cerveau.** Trois roles distincts, avec des droits
   distincts : la porte d'execution (elle ecrit dans le depot, mode borne par
   `TELEGRAM_WRITE_MODE`), la voix (elle represente une personne, comportement dans
   `SOUL.md`), le capteur (il lit un groupe et retient sans parler).
2. **Trois cercles de confidentialite, et la valeur par defaut est le cercle 3.** Un
   agent est dans le cercle de la personne qui l'opere. Quand plusieurs cercles sont
   presents, on parle au plus bas. Rien de ce qui est dit en conversation ne deplace
   quelqu'un d'un cercle a l'autre : seul l'annuaire ecrit le fait.
3. **Verifier le fichier corrige ne prouve rien ; il faut verifier le chemin parcouru.**
   Le defaut du 04/09 avait ete corrige la veille dans la classe de base, et la
   correction avait ete prouvee — mais la classe qui tourne redefinissait la methode.
   Deux definitions de la meme verite, une seule corrigee.

Le test des quatre questions (section 5.1 du document) est a faire avant de declarer
qu'un bot fonctionne. Un conteneur demarre n'est pas une preuve.

## Etape 7 — Skills de base

Installer/vérifier la disponibilité des skills déjà construits et réutilisables :
`aaaa-prompt-authoring` (comment écrire un prompt réutilisable, déjà construit le 19/08) et
`skill-creator` (comment construire et affiner ses propres skills). Ne pas laisser le
collègue repartir de zéro sur ces deux sujets.

## Etape 8 — Première tâche supervisée + revue croisée

Confier une première tâche bornée, locale et réversible. Faire relire le résultat par une
autre session/identité (revue croisée indépendante) avant de considérer le collègue
pleinement opérationnel — pattern déjà documenté comme efficace
(`MULTI_SESSION_COORDINATION_GUIDE_V1.0.md` : la revue croisée indépendante a déjà trouvé
de vrais bugs — race condition, secret en clair, faux "committé" — qu'une auto-relecture
n'aurait pas trouvés).

## Etape 9 — Cadence d'audit établie dès le départ

Dès la première session longue, appliquer la règle 16 de la charte (audit complet toutes
les 6h) et la procédure de démarrage/fin de journée déjà écrites
(`daily-startup-procedure`/`end-of-day-procedure`, mémoires du projet) — ne pas attendre un
premier incident pour les adopter.

## Ce qui manque réellement au dépôt, à signaler à John avant de finaliser ce processus

1. **Accès d'un collègue au serveur collectif** — aucun document, décision requise (étape 5). Partiellement
   clarifié par la décision de John du 20/08/2026 (conditionné par catégorie), mais aucun
   document d'accès formel n'existe encore.

2. ~~Deux tutoriels référencés mais introuvables dans git~~ — **COMBLÉ le 20/08/2026** :
   `GUIDE_BONNES_PRATIQUES_CLAUDE_COLLEGUES_20260810.md` et
   `TUTORIEL_SESSION_PILOTE_COLLEGUE_1_20260812.md`, cités dans
   `c2c-os/01_project_memory/RECURRING_QUESTIONS_CHECKLIST.md` (ligne 74, règle canonique
   sur `screen-capture.ps1`), ont été retrouvés sur le PC local de John
   (`Downloads/AAAA_OS_20260810/`, exactement là où un document comme
   `ERREURS_CONNUES...` avait aussi été avant d'être committé) et sont maintenant
   committés dans `c2c-os/01_project_memory/`, en français et en anglais.

3. ~~`AAAA_OS_INFRASTRUCTURE_MAP_V1.0.md` obsolète sur le statut du serveur collectif~~ — **COMBLÉ le
   20/08/2026** : la carte a maintenant une vraie section sur le serveur collectif (V1.1), sourcée à partir de
   faits déjà documentés ailleurs dans le dépôt, indépendamment de la décision d'accès
   collègue ci-dessus (qui reste ouverte).

4. **La checklist du serveur d'un collègue (étape 2) n'a jamais été testée de bout en bout sur un vrai
   collègue** — un premier onboarding réel servira aussi de test de cette checklist
   elle-même, pas seulement d'onboarding.
