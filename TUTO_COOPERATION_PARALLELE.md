# Coopérer en parallèle — groupes thématiques, interlocuteurs non-Claude

> Écrit le 29/08/2026. Destiné aux agents **et** aux collègues humains qui
> rejoignent. Chaque affirmation porte sa preuve ou est marquée non vérifiée.

## 1. Le diagnostic, mesuré

**22 boîtes C2C existent. Toutes sont individuelles. Aucun groupe thématique.**

Conséquence directe : un sujet qui concerne cinq participants se traite en cinq
messages séparés, sans fil commun. Chacun répond à l'expéditeur, personne ne
voit les réponses des autres, et l'expéditeur devient le seul point où
l'information se rassemble. **C'est un goulot, et c'est ce qui rend la
coopération sérielle.**

Deuxième mesure, du même jour : deux messages de coordination se sont
**croisés** — le « ne construis pas » partant à 12:24, la demande à 12:26.
Personne n'a ignoré personne. **Le canal est plus lent que le travail qu'il
coordonne.** Toute conception qui suppose un aller-retour avant d'agir échouera.

Ces deux constats commandent tout ce qui suit.

---

## 2. Le principe : éventail, pas chaîne

| Sériel — ce qu'on fait aujourd'hui | Parallèle — ce qu'on vise |
|---|---|
| j'écris à A, j'attends, puis à B | j'écris **une fois** au groupe |
| chaque réponse me revient isolée | les réponses partagent un `correlation_id` |
| je suis le seul à voir l'ensemble | chacun voit le fil entier |
| durée = somme des attentes | durée = **la plus longue seule** |

Le mécanisme existe déjà et n'est pas exploité : **`correlation_id` est le fil**.
Le composeur multi-destinataires le pose correctement — N messages, un seul
`correlation_id`, des `message_id` distincts. Il ne manque que la **boîte
partagée** où le fil devient visible.

---

## 3. Créer un groupe thématique C2C

Un groupe est une boîte que **plusieurs identités lisent**.

```bash
mkdir -p c2c-os/03_handoffs/mailboxes/groupe-<theme>/inbox
```

Conventions, pour que ça reste lisible à vingt participants :

| Élément | Règle |
|---|---|
| Nom | `groupe-<theme>` — jamais un nom de personne |
| `recipient_conversation_id` | `groupe-<theme>` |
| `correlation_id` | **le même pour tout le fil**, réponses comprises |
| Réponse | déposée dans la boîte **du groupe**, pas de l'expéditeur |

**Le point qui fait la différence : on répond au groupe.** Répondre à
l'expéditeur reconstitue le goulot qu'on vient de supprimer.

Groupes utiles au vu des chantiers en cours : un groupe par candidature en
cours (`groupe-<nom-de-la-candidature>`), `groupe-infrastructure`,
`groupe-candidatures`, `groupe-accueil-collegues`.

### Le piège qui tue un groupe en silence

**Quatre allowlists, pas une.** `C2C_ALLOWED_SENDERS` ne couvre que le wake
dispatcher ; il y a aussi `AUTOPILOT_ALLOWED_SENDERS`, `allowed_senders` en
JSON dans `worker.py`, et `mcp_c2c_allowed_sender_ids` dans la passerelle.

Un participant absent d'**une seule** voit ses messages mourir en rejet
silencieux, dans une base qu'il ne consultera jamais. **Vérifier les quatre à
l'inscription** — c'est la panne qui a coûté l'incident LEAD2.

---

## 4. Interlocuteurs non-Claude

### ChatGPT

**Capacité vérifiée, longtemps crue absente.** Le contrôle du navigateur
(`desktop-control` : `type`, `click_element`, `find_element`, `scrape`) permet
d'écrire dans une conversation ChatGPT et d'en lire la réponse.

Le protocole tient en trois points :

1. **Coller le message C2C complet** dans le fil ChatGPT — en-tête compris.
   L'en-tête n'est pas décoratif : il porte le `correlation_id` qui rattache la
   réponse au fil.
2. **Demander explicitement la réponse au même format**, sinon elle revient en
   prose libre et le fil se casse.
3. **Récupérer par `scrape`** et déposer dans la boîte du groupe, en signant
   `sender_conversation_id: chatgpt-<sujet>`.

Limite honnête : **c'est du pilotage d'interface, pas une API.** Une refonte de
la page ChatGPT casse le mécanisme sans avertissement. À traiter comme un pont,
pas comme un canal fiable.

### Collègues humains

**Un collègue ne rédigera jamais un en-tête de quatorze champs à la main.**
C'est écrit dans la politique de communication depuis sa rédaction, et c'est
resté vrai.

Le chemin : le **composeur multi-destinataires** produit le message, un point
d'entrée sur le serveur de production l'écrit dans la boîte. **Correction du 03/09/2026** :
contrairement à ce que cette section affirmait depuis le 22/08, le point
d'entrée **existe et tourne** — `aaaa-c2c-entrypoint.service`, actif,
`/opt/<deploiement>/c2c-entrypoint/depose.py`, écoute sur `127.0.0.1:8787`.
Vérifié par `systemctl` et `ss -tlnp`, pas supposé. Ce qui reste vrai :
**zéro appel dans les journaux sur 30 jours** — jamais éprouvé en conditions
réelles — et il n'est exposé sur aucune route Caddy trouvée, donc
inaccessible à un collègue externe aujourd'hui malgré son existence. Motif
identique à "construit, jamais branché", version "construit, jamais essayé
ni exposé."

Tant qu'il n'est pas éprouvé et exposé : le collègue écrit en clair, un agent
met en forme et dépose — ou utilise, pour un collègue avec accès repo,
`services/c2c_gateway/colleague_entry_point.py` (script CLI, prépare le
message et le registre, n'exécute jamais lui-même le commit/push). Lent,
mais honnête — mieux vaut un relais assumé qu'un canal qui a l'air ouvert
et rejette en silence.

---

## 5. Telegram thématique

Telegram sert à ce que le C2C fait mal : **l'alerte et la présence**. Le C2C
est asynchrone et versionné ; Telegram est immédiat et éphémère. Ne pas les
faire se concurrencer.

| Canal | Sert à | Ne sert pas à |
|---|---|---|
| `#<candidature>` (un canal par candidature en cours) | alerter d'une échéance, signaler un blocage | porter le contenu — il vit dans le C2C |
| `#infrastructure` | panne, bascule, redémarrage | décider |
| `#accueil` | premiers pas d'un arrivant | l'archive |

**Règle non négociable** : `MCP_TELEGRAM_WRITE_ENABLED` reste à `false`. Un
message Telegram sortant demande une validation humaine explicite. Un canal
d'alerte qui peut écrire tout seul devient un canal de dégâts.

---

## 5bis. Inventaire complet des canaux -- etat au 02/09/2026, pour deployer demain

Demande de John (02/09 soir) : recenser tous les canaux de connexion avec
les collegues (humains et agents), Telegram en priorite, pour deployer
demain. Inventaire reel, pas suppose -- chaque ligne dit ce qui est
construit, teste, et branche, ou pas.

| # | Canal | Construit | Teste aujourd'hui | Branche en prod | Ce qui manque pour deployer |
|---|---|---|---|---|---|
| 1 | **Telegram thematique** (`#<candidature>`, `#infrastructure`, `#accueil`) | Oui | Non aujourd'hui | Lecture oui, ecriture non | `MCP_TELEGRAM_WRITE_ENABLED=false` par design -- **decision explicite de John requise** pour l'activer, pas une simple bascule technique |
| 2 | **C2C git-mailbox** (boites individuelles) | Oui | Oui, en continu toute la journee | Oui | Rien -- canal de reference, le plus mature |
| 3 | **C2C groupes thematiques** (6 boites + kimi-k3) | Oui, depuis ce matin | Oui, `groupe-candidatures` utilise en reel aujourd'hui | Oui, veilleur actif (16/16, 7 groupes) | Rien pour le mecanisme -- reste a informer les colleges humains de leur existence |
| 4 | **Contact direct LEAD02 <-> allo-desktop** (C2C, ecriture bornee) | Oui, aujourd'hui meme | Oui, canari + rapport + onboarding, bidirectionnel prouve | Oui | Rien -- le plus recent et deja complet |
| 5 | **`ListAgents`/`SendMessage` officiel Claude Code** (voir §7bis) | Livre par Anthropic, pas par nous | Teste (liste vide, aucune session paire ouverte au meme instant) | Non -- jamais utilise entre Allo et moi en conditions reelles | Faire le test reel : deux sessions ouvertes simultanement |
| 6 | **`session-bridge`** (plugin tiers, meme machine) | Oui | Oui, bug reveil-sur-le-plus-ancien trouve et corrige (31/08) | Oui, mais concurrence desormais l'outil officiel (5) pour le meme usage | Decider lequel des deux (5) ou (6) devient le defaut meme-machine |
| 7 | **Pont ChatGPT** (pilotage navigateur, `desktop-control`) | Oui, protocole a 3 points | Non aujourd'hui | Partiellement -- utilise pour LEAD02 avant que son ecriture C2C directe existe | Fragile par construction (pilotage d'interface, pas API) -- a garder en secours, pas en primaire maintenant que (4) existe |
| 8 | **Relais collegue humain** (composeur multi-destinataires + point d'entree sur le serveur de production) | Composeur oui, point d'entree serveur **non code** | Non | Non -- maillon manquant identifie depuis le 22/08 | Coder le point d'entree sur le serveur de production -- le vrai blocant pour que de vrais collegues (pas des agents) rejoignent |
| 9 | **Google Drive (partage de documents)** | Oui, teste le 22/08 | Non aujourd'hui | Oui pour le partage de fichiers, pas pour la messagerie structuree | Verdict deja rendu le 22/08 : trop lent pour un usage courant de messagerie -- reste bon pour du partage de documents ponctuel seulement |

**Priorite de demain, dans l'ordre** : (1) Telegram -- decision de John sur
l'activation en ecriture, sinon rester en lecture/alerte comme aujourd'hui ;
(8) le point d'entree sur le serveur de production -- c'est le seul canal qui bloque reellement
l'arrivee de vrais collegues humains, tous les autres canaux servent des
agents ou des ponts temporaires ; (5) tester `ListAgents`/`SendMessage` en
conditions reelles avant de decider s'il remplace (6).

**Architecture Telegram precisee par John (01/09 17h33-17h38)** :
`Claude -> subagent (agent de recherche + LLM gratuit) -> Telegram (PC ou Android)`. Le
subagent de recherche sert d'intermediaire, pas Claude directement -- coherent
avec `MCP_TELEGRAM_WRITE_ENABLED=false` (Claude ne parle jamais a Telegram
en ecriture sans passer par ce sas). A verifier demain : ce subagent
existe deja (ses outils MCP sont dans `services/`), reste a confirmer qu'il est
bien celui qui porte le pont vers Telegram plutot qu'un nouveau composant.

## 6. Ce qu'un participant fait avant de commencer

```bash
bash ops/deja-fait.sh <mots-cles>     # ca existe deja ?
bash ops/bail.sh voir                  # quelqu'un est dessus ?
bash ops/bail.sh prendre <perimetre> <minutes> "<raison>"
git add c2c-os/02_operational_registers/baux/ && git commit && git push
```

Trois commandes. Le bail est une **déclaration**, pas une demande : on le dépose
et on avance sans attendre de réponse — puisque l'attente est précisément ce qui
ne marche pas.

Et avant de conclure qu'une capacité manque : lire
`CARTE_DES_CAPACITES.md`. Trois fois le 29/08, un agent a conclu à une absence
alors que la capacité existait.

---

## 7. Ne pas s'attendre — la règle qui fait gagner le plus de temps

*Ajouté le 31/08/2026, après vingt minutes perdues entre deux sessions.*

Deux sessions se sont immobilisées mutuellement. L'une s'était arrêtée de
commiter pour laisser l'autre fusionner — geste juste. L'autre a fusionné,
poussé, vérifié… et n'a rien dit pendant vingt minutes. La première attendait un
message pour un fait **visible dans `origin/main` en deux secondes**.

Personne n'a commis d'erreur. C'est le protocole qui était mauvais.

> **On n'attend pas quelqu'un. On attend une condition qu'on peut vérifier
> soi-même.**

La latence d'un message est celle du **destinataire**, pas de l'émetteur : une
boîte se relève à la cadence de celui qui la relève. Un état partagé, lui, se
vérifie à la demande. Coordonner par message ce qui est observable additionne la
latence de l'un à l'incertitude de l'autre.

### Les trois obligations

**1. Tout message « je m'arrête » porte sa condition de reprise.** Jamais
« préviens-moi ». Toujours : *« je m'arrête jusqu'à ce que `origin/main`
contienne `<chemin>` — vérifiable par `git ls-tree -r --name-only origin/main` —
ou au plus tard dans 15 minutes. »* Sans condition écrite, celui qui attend ne
sait pas quoi regarder et se rabat sur sa boîte.

**2. Celui qui attend interroge la condition, pas sa boîte.** La boîte sert aux
décisions et aux informations non déductibles de l'état partagé. Pas à la
synchronisation.

**3. Toute attente porte une échéance.** Un verrou sans TTL ne se libère jamais.
À l'échéance, on reprend **quand même** et on le dit. Mieux vaut un conflit qu'on
résout qu'un blocage que personne ne voit.

### Pourquoi ça compte plus que les vingt minutes

**L'attente est invisible.** Rien ne signale qu'une session est bloquée : elle
ressemble exactement à une session occupée. C'est le même symptôme qu'un service
`active` qui ne produit rien — et le même remède : chercher la **sortie réelle**,
jamais l'état déclaré.

### Deux pièges d'outillage rencontrés le même jour

- **Un `git push` qui réussit peut atterrir sur la mauvaise branche**, sans la
  moindre erreur. Vérifiez avec `git ls-tree -r --name-only origin/main`, pas
  avec le code de sortie du push.
- **Sous Git Bash, un chemin après `origin/main:` peut être réécrit** par la
  conversion MSYS (`origin/main:.claude/x` devient `origin\main;.claude\x`). Le
  contrôle rend alors « absent » sur des fichiers présents. `git ls-tree` n'y est
  pas sujet. **Si vous recevez cette alerte, testez-la sur vos propres commandes
  avant de conclure** — un format non guillemeté n'est pas forcément affecté.

Procédure complète : `ops/bascules/08_NE_PAS_S_ATTENDRE.md`.

---

## 7bis. Un canal officiel Anthropic decouvert le 02/09 -- a distinguer de C2C

Recherche demandee par John (side project W17, catalogue des techniques de
connexion). Trouvaille reelle, pas deja documentee ailleurs dans ce depot :
Claude Code a une **messagerie inter-sessions officielle**, deux outils
(`ListAgents`, `SendMessage`) livres par Anthropic depuis la version 2.1.224
(07/08/2026, Windows natif depuis la 2.1.234). Un Claude peut lister et
contacter par nom une autre session Claude Code -- meme machine, autre
machine du meme compte (via Remote Control), ou session cloud.

**Ce que ca vaut par rapport a C2C git-mailbox et a `session-bridge` :**

| | C2C git-mailbox | `session-bridge` (plugin tiers) | `ListAgents`/`SendMessage` (officiel) |
|---|---|---|---|
| Portee | Inter-organisations, sans limite de distance | Meme machine seulement | Meme machine, ou autre machine du meme compte via Remote Control |
| Ou transite le contenu | Notre propre depot GitHub | Fichiers locaux, jamais externe | **Serveurs Anthropic des que ce n'est pas la meme machine** |
| Durabilite/audit | Total (git, append-only) | Ephemere | Aucune -- texte volatil, jamais l'historique ni les fichiers |
| Contrainte trouvee | Aucune connue | Reveil sur le plus ancien message (corrige, voir §7) | **Une session Windows native et une session WSL2 sur le meme PC ne se voient pas** (repertoires et types de socket differents) |

**Consequence pratique pour la souverainete des donnees (principe `CLAUDE.md`
"Sovereignty by Design")** : `SendMessage` est le bon reflexe pour parler a
une AUTRE session Claude Code sur CE PC, la ou `session-bridge` sert
aujourd'hui -- gain : pas de plugin tiers a maintenir. Mais pour tout ce qui
traverse une frontiere de machine (PC <-> serveur de production <-> serveur secondaire), **C2C reste le
canal a utiliser**, parce que la messagerie officielle passerait par les
serveurs Anthropic pour ce cas, ce que C2C n'a jamais fait.

**Non teste a ce jour** : `ListAgents` execute le 02/09 depuis cette session
a renvoye "No reachable agents" -- aucune autre session avec un inbox ouvert
au meme instant, pas une preuve que le mecanisme ne marche pas. A eprouver
reellement avec Allo (deux sessions ouvertes en meme temps) avant de
recommander quoi que ce soit de plus.

Source : [documentation officielle Claude Code](https://code.claude.com/docs/en/cross-session-messaging).

---

## 8. Ce qui reste à prouver

Traiter ces lignes comme des hypothèses, pas comme des acquis :

- ~~Aucun groupe thématique n'a encore été créé ni testé.~~ **Corrigé le
  02/09/2026 par Allo, avec preuve** : six boîtes existent et tournent
  (`groupe-accueil-collegues`, `groupe-architecture`, `groupe-candidatures`,
  `groupe-disponibilite`, `groupe-infrastructure`, et le groupe de la
  candidature en cours),
  plus `kimi-k3` — sept groupes surveillés par le veilleur depuis 11h25 ce
  jour-là (`vérifiées 16/16`, `errors: []`). Avant ce matin-là, les boîtes
  existaient sans lecteur : un message y écrit ne réveillait personne — le
  motif « construit, jamais branché » une fois de plus. Section 3
  ci-dessus décrit donc un mécanisme **construit et opérationnel**, pas une
  proposition en attente.
- **Le pont ChatGPT n'a pas été éprouvé de bout en bout** — écrire, lire,
  déposer, obtenir une réponse au format.
- ~~Le point d'entrée sur le serveur de production n'existe pas.~~ **Corrigé le 03/09** : il existe
  et tourne (`aaaa-c2c-entrypoint.service`), mais n'a jamais été appelé en
  30 jours et n'est exposé sur aucune route externe — voir section 4.
- **Aucun collègue humain n'a encore parcouru ce parcours en entier.**
- **`ListAgents`/`SendMessage` (§7bis) n'a jamais été essayé en conditions
  réelles entre deux sessions ouvertes en même temps** — la découverte est
  documentaire (lecture de la doc officielle), pas éprouvée.

Un tutoriel qui affirmerait le contraire serait exactement le genre de document
qui fait perdre une journée à celui qui le suit.
