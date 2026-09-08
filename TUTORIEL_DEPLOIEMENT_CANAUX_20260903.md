# Déployer les canaux de communication avec un collègue

> **Version du 03/09/2026.** Écrit à la demande de John : *« mets à jour les
> tutoriels déploiement des 7-8-9 canaux de communication avec collègues
> (Telegram en premier) »*.
>
> **Ce tutoriel ne décrit que ce qui est mesuré comme fonctionnel au 02/09 à
> 23h.** Chaque canal porte son état réel et sa commande de vérification. Un
> canal annoncé qui ne marche pas coûte plus cher qu'un canal absent : le
> collègue attend une réponse qui ne viendra jamais.

---

## Les neuf canaux — la liste de John, et leur état réel

Liste énumérée par John sur WhatsApp le 01/09 (17h23 → 17h38). L'état à droite
est mesuré, pas supposé.

| # | Canal | Ce qu'il permet | État au 02/09 |
|---|---|---|---|
| 1 | **MCP click & browse** | Claude agit sur le PC : clic, saisie, lecture d'écran | ✅ actif (`desktop-control`) |
| 2 | **Chrome** | navigation pilotée, extraction de pages | ✅ via le même MCP |
| 3 | **BRIDGE** (session-bridge) | Claude Code ↔ Claude Code sur la même machine | ⚠️ **même compte uniquement** |
| 4 | **Conversation-à-conversation Anthropic** | sous-agents, `SendMessage` | ✅ actif |
| 5 | **C2C — 3 couches** | boîtes aux lettres sur GitHub, 11 identités | ✅ **le canal de référence** |
| 6 | **Google Drive** | partage de documents avec des humains | ✅ connecteur actif |
| 7 | **A2A** | protocole agent-à-agent inter-organisations | ⛔ pas branché ici |
| 8 | **SSH direct entre VPS** | serveur de production ↔ serveur secondaire, tunnel PostgreSQL compris | ✅ actif |
| 9 | **Telegram** | notification vers un humain, PC ou Android | ⚠️ **RÉCEPTION SEULE** |

---

# 1. TELEGRAM — à traiter en premier, et à comprendre avant de déployer

## Ce qu'il faut dire au collègue dès la première phrase

**Telegram fonctionne aujourd'hui dans un seul sens : il reçoit.** Un collègue
peut être notifié ; il ne peut pas répondre et être entendu.

```
TELEGRAM_WRITE_MODE        = READ_ONLY
MCP_TELEGRAM_WRITE_ENABLED = false
```

Ce n'est **pas une panne à réparer**, c'est une décision de sécurité du projet.
Un agent qui écrit sur Telegram peut contacter des humains hors du système ; la
règle de `CLAUDE.md` l'interdit sans validation humaine explicite.

> **Vérifier soi-même, et sur TOUS les conteneurs — pas un seul :**
> ```bash
> ssh <serveur-de-production> "sudo -n bash -c 'for C in \$(docker ps --format \"{{.Names}}\" | grep -i telegram); do
>   echo \"--- \$C\"; docker exec \$C printenv 2>/dev/null |
>   grep -E \"^(MCP_TELEGRAM_WRITE_ENABLED|TELEGRAM_WRITE_MODE)=\"; done'"
> ```
>
> **Trois pièges, tous rencontrés le 02/09 :**
>
> **1. Ces variables ne sont pas dans les unités systemd.** Les y chercher donne
> un faux négatif. Elles vivent dans les surcharges compose du poller
> (`docker-compose.poller.yml`, `.agent.yml`, `.local-llm.yml`, `.c2c-write-v1.yml`).
>
> **2. Lire la config ne suffit pas** : il faut lire l'environnement du
> **processus**. Une config juste jamais appliquée à la pile en marche est
> exactement ce qui a bloqué le canal de LEAD02 pendant des jours.
>
> **3. Et le pire — les deux conteneurs ne disent PAS la même chose.**

### ⚠️ Correction du 03/09 : le contrôle n'est PAS uniforme

J'avais écrit ici que Telegram était en lecture seule, « vérifié par `printenv` ».
**Je n'avais interrogé qu'un conteneur sur deux.** Mesure complète :

| Conteneur (serveur de production) | `MCP_TELEGRAM_WRITE_ENABLED` | `TELEGRAM_WRITE_MODE` |
|---|---|---|
| `...-telegram_mcp-1` | `false` ✅ | `READ_ONLY` ✅ |
| `...-telegram_poller-1` | `false` ✅ | **`C2C_REQUESTS`** ⚠️ |

**Pourquoi** : la surcharge `docker-compose.c2c-write-v1.yml` force `READ_ONLY`,
mais **ne s'applique qu'au conteneur MCP**. Le poller est démarré avec
`--env-file .env.poller`, qui porte `TELEGRAM_WRITE_MODE=C2C_REQUESTS`.

**Ce que ça change, et ce que ça ne change pas — la nuance compte :**

- **L'écriture *vers Telegram* reste fermée.** `MCP_TELEGRAM_WRITE_ENABLED=false`
  sur les deux. Aucun agent ne peut envoyer un message Telegram.
- **Ce qui change** : une commande `/handoff` **tapée dans Telegram par John**
  crée un message C2C avec le statut `SENT` au lieu de `DRAFT`. Ce n'est pas un
  agent autonome — c'est un outil qui obéit à un humain identifié
  (`ALLOWED_TELEGRAM_USER_IDS`, un seul, dans un seul groupe).

**Donc : pas une violation de `CLAUDE.md`, mais un contrôle annoncé comme tenu
qui ne l'est qu'à moitié.** À trancher avec John : est-ce voulu ?

> **La leçon, et elle vaut pour tout ce tutoriel** : vérifier un conteneur et
> conclure sur le service est le même défaut que balayer `.env` et conclure sur
> les secrets. *Le trou est dans l'endroit où l'on regarde.*

## Déploiement de la réception — ce qui marche aujourd'hui

**Étape 1 — le collègue rejoint le canal.** Rien à installer de son côté :
Telegram sur téléphone ou PC suffit. John l'ajoute au groupe concerné.

**Étape 2 — vérifier que le poller est vivant.** Un poller `Up` qui ne reçoit
rien est le cas classique de la panne muette.

```bash
ssh <serveur-secondaire> "sudo -n docker ps --filter name=telegram --format '{{.Names}}  {{.Status}}'"
ssh <serveur-secondaire> "sudo -n docker logs --since 1h c2c-telegram-poller-telegram_mcp-1 2>&1 | tail -20"
```

Des `Recoverable poller error: ReadTimeout` sur `getUpdates` sont **normaux** —
c'est le motif habituel du long-polling, pas une panne.

**Étape 3 — la preuve, et rien d'autre.** Ne conclure « ça marche » qu'après
avoir vu **un message réel arriver**, pas après avoir vu un conteneur vert.

## L'architecture visée pour la bidirectionnalité — la conception de John

John l'a écrite le 01/09 à 17h36 :

```
claude ──── subagent (agent de recherche + LLM gratuit) ──── telegram (PC ou Android)
```

**Ce n'est pas « ouvrir l'écriture Telegram ».** C'est interposer un sous-agent
qui porte la responsabilité de ce qui sort. La question de sécurité change de
nature : elle passe de *« autorise-t-on l'écriture ? »* à *« quel intermédiaire
répond de ce qui est envoyé, et selon quelles bornes ? »*

**Ce que ça suppose, et qui n'existe pas encore** : une liste blanche
d'émetteurs, des types de messages bornés, et l'interdiction pour l'agent
d'émettre lui-même une confirmation de décision humaine — exactement les trois
disciplines qui rendent le canal C2C sûr aujourd'hui.

**Ce n'est pas déployable ce mois-ci.** Le dire franchement au collègue.

---

# 2. C2C — le canal de référence, et le seul vraiment bidirectionnel

C'est celui qui marche le mieux : **40 messages échangés en 12 heures** le
02/09, entre quatre entités différentes.

## Comment ça marche, en une phrase

Une boîte aux lettres est un répertoire dans le dépôt GitHub. Écrire un message
= déposer un fichier Markdown dans l'`inbox` du destinataire. Le lire = lire le
répertoire. **Git est le transport ; il n'y a pas de serveur à maintenir.**

```
c2c-os/03_handoffs/mailboxes/<destinataire>/inbox/<horodatage>_<TYPE>_<id>_<de>_to_<vers>.md
```

## Les 28 boîtes, dont 6 groupes thématiques

```bash
cd /chemin/vers/votre/clone && ls c2c-os/03_handoffs/mailboxes/
```

Groupes : `groupe-architecture`, `groupe-candidatures`, `groupe-infrastructure`,
`groupe-accueil-collegues`, `groupe-disponibilite`, et un groupe dédié à la
candidature en cours (`groupe-<nom-de-la-candidature>`).

> **Écart de conception signalé par John**, et pas encore corrigé :
> *« thematics groups mixing humans and conversations »*. Les six groupes
> actuels ne contiennent **que des agents**. Ils devaient mêler humains et
> conversations. C'est une correction à porter, pas un détail.

## Les deux façons d'écrire, et leur différence réelle

| Voie | Qui l'utilise | Registre |
|---|---|---|
| **outil MCP borné** | LEAD02 et les identités de la liste blanche | ligne écrite **automatiquement** |
| **dépôt git direct** | moi, allo-desktop | ligne à écrire **à la main** |

**Les deux sont légitimes.** Le 02/09 j'ai affirmé que la voie git « ne peut pas
enregistrer » — **c'était faux**, et LEAD02 l'a correctement requalifié :
*le canal peut, c'est ma procédure qui omettait*. allo-desktop, elle, enregistre
les siens depuis toujours.

**L'invariant à tenir, quelle que soit la voie :**

```
1 fichier de boîte  ↔  1 message_id  ↔  1 ligne de registre
```

Le registre vit dans `c2c-os/02_operational_registers/MESSAGE_REGISTRY.md`.

> **Piège de comptage mesuré** : le motif `MSG-[0-9]{20}` **ignore 112 lignes**
> — tous les identifiants comportant des lettres. Compter avec `^| *MSG-`.
> Il subsiste une **collision** connue : `MSG-20260821140000000084` désigne deux
> messages différents (expéditeurs inversés, corrélations différentes). À
> réparer sous écrivain unique.

## La règle de coordination qui manquait

Le 02/09 à 20h55, allo-desktop et moi avons écrit notre bilan du soir dans
`switch.md` **en même temps**. Conflit de rebase sur un fichier de 6 000 lignes.

> **Pour tout fichier partagé à fort trafic** (`switch.md`,
> `MESSAGE_REGISTRY.md`, les cartes) : **on annonce avant d'écrire**, dans
> `groupe-infrastructure`. Trois lignes. C'est moins cher qu'un rebase en
> conflit.

## Quel canal pour quoi

| Nature de l'échange | Canal |
|---|---|
| demande à **une** conversation | sa boîte individuelle |
| sujet concernant **plusieurs** | le **groupe** thématique |
| journal de bord personnel | `switch.md`, section datée et signée, **append seulement** |
| alerte à un humain | Telegram (réception seule aujourd'hui) |
| **un fait établi que d'autres réutiliseront** | **le dépôt, pas un message** |

**La dernière ligne est la plus importante.** Un message se perd ; un fichier se
cite. Plusieurs mesures importantes du 02/09 n'existent que dans des messages
C2C — donc inutilisables par un tiers qui arrive demain.

---

# 3. SESSION-BRIDGE — utile, mais connaître sa limite

Permet à deux sessions Claude Code de la **même machine et du même compte** de
se parler.

```bash
bash "$HOME/.claude/plugins/cache/session-bridge/session-bridge/0.1.1/scripts/list-peers.sh"
```

**La limite qui compte** : il ne peut pas atteindre une entité hébergée
ailleurs. LEAD02 tourne chez OpenAI — le pont ne l'atteindra jamais. Pour un
collègue distant, c'est **C2C ou rien**.

Autre limite mesurée : **l'enregistrement du pont expire**. Un pont qui
« marchait hier » et ne répond plus n'est pas cassé, il est désinscrit :
relancer `register.sh`.

---

# 4. GOOGLE DRIVE — le canal vers les humains non techniques

Le seul qui ne demande **rien** au collègue : il reçoit un lien, il ouvre.

Connecteurs actifs : recherche, lecture, création, partage de fichiers.

**Contrainte du projet, non négociable** : les dossiers d'évaluation UE ne
sortent pas vers des API externes. Le Drive sert aux documents de travail
partagés, pas aux pièces sensibles.

> **Piège daté** : notre application OAuth Google Drive doit rester **publiée**,
> sinon le jeton meurt tous les 7 jours (tâche #208).

---

# 5. SSH ENTRE MACHINES — le canal qui ne se voit pas

Serveur de production ↔ serveur secondaire, avec un tunnel PostgreSQL en 15432.

```bash
ssh <serveur-secondaire> "systemctl is-active aaaa-pg-tunnel.service"
ssh <serveur-secondaire> "ss -ltnp | grep 15432"
```

**Discipline de secret, apprise à ses dépens** : jamais de valeur en ligne de
commande — `sudo` journalise la commande entière. Copie fichier à fichier,
jamais `echo`, jamais de variable en ligne. Pour comparer deux secrets sans les
afficher, comparer leurs empreintes :

```bash
sha256sum < fichier_a | cut -c1-12    # puis comparer les deux empreintes
```

---

# 6. LE MCP DE CONTRÔLE DU PC — le plus puissant, le plus à encadrer

*« ALLOW CLAUDE TO ACT ON YOUR PC »*, selon les mots de John.

Permet clic, saisie, lecture d'arbre d'interface, presse-papier, capture
d'écran. **C'est le canal qui permet de faire à la place de quelqu'un**, donc
celui qui demande le plus de prudence : ce qu'il touche appartient à l'humain.

Ordre de préférence quand plusieurs voies existent — du meilleur au pire :
connecteur applicatif → script (PowerShell) → arbre d'accessibilité
(`find_element`, `click_element`) → **clic aux coordonnées en dernier recours**.

---

# 7. A2A — annoncé, pas branché

Protocole agent-à-agent inter-organisations. **Rien n'est déployé ici.** Le
mentionner à un collègue comme disponible serait faux.

Il est lié à la tâche #100 (*accélérer sur l'accès agent-à-agent face à EUACC*)
et reste au niveau conception.

---

# Ordre de déploiement recommandé pour un nouveau collègue

| Jour | Canal | Pourquoi cet ordre |
|---|---|---|
| **1** | Telegram en réception | zéro installation, valeur immédiate, attentes cadrées dès le départ |
| **1** | Google Drive | partage de documents sans compétence technique |
| **2** | C2C | le vrai canal de travail — demande de comprendre les boîtes et le registre |
| **3** | SSH | seulement si le collègue opère une machine |
| **4** | MCP contrôle PC | seulement sur sa propre machine, et avec son accord explicite |
| — | session-bridge | inutile pour un collègue distant |
| — | A2A | rien à déployer |

---

# La vérification qui vaut pour tous les canaux

Notre échelle interne, et la seule ligne qui compte :

| Niveau | Ce que ça veut dire |
|---|---|
| S0 | écrit |
| S1 | branché |
| S2 | **produit réellement** en local |
| S3 | **surveillé** — une panne se voit |
| **S4** | **reçu et utilisable par le destinataire** |

**Un canal ne s'annonce à un collègue qu'en S4.** Le 02/09, trois composants
étaient en S2 — ils produisaient vraiment — et pourtant rien n'arrivait : les
boîtes de groupe que personne ne lisait, le détecteur de chaînes muettes que
rien n'appelait, le canal de LEAD02 dont la config n'était pas appliquée à la
pile en marche.

> **Dans les trois cas, un rapport « ça produit » aurait été exact et trompeur.**

---

## Journal des versions

| Date | Changement |
|---|---|
| 03/09/2026 | Création. Liste des 9 canaux verbatim de John ; états mesurés ; Telegram traité en premier avec sa contrainte de lecture seule dite franchement ; l'architecture à sous-agent présentée comme le chemin et non l'existant ; les pièges de mesure documentés à côté de chaque commande. |
