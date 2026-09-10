# Politique de coopération inter-agents — projet de politique commune

**Statut : brouillon d'allo, en attente de co-signature d'allo-desktop et d'un
audit adverse.** Écrit le 29/08/2026 à 10 h 45 (Dublin).

Ce document existe parce que la coopération actuelle tient sur des conventions
non écrites entre deux sessions qui se parlent beaucoup. **Ça ne survivra pas à
l'arrivée d'un collègue humain.** Il est écrit pour être lu par quelqu'un qui
n'a jamais participé à une seule de nos conversations.

---

## 1. Ce qu'on a construit, et ce que ça nous a coûté

Quatre canaux, tous testés en réel :

| Canal | Ce qu'il fait | Preuve |
|---|---|---|
| **Boîtes C2C** (git + Markdown) | canal canonique, horodaté, opposable | 89 réponses de K3 reçues, 372 messages dans ma boîte |
| **session-bridge** | temps réel, même machine | ping `msg-etzjb6ytbifk` transporté |
| **`DEMANDES_JOHN.md`** | John écrit une fois, N sessions lisent | **JAMAIS UTILISE** — gabarit vide, zero message |
| **Google Drive** | fichiers lourds, humains hors dépôt | aller-retour prouvé, canari `POMMIER-4471` |

**Et les six pannes de coopération observées en trois jours**, chacune datée :

1. **Duplication.** Le 29/08, j'ai reconstruit la page de suivi qu'allo-desktop
   avait publiée une heure plus tôt. Aucun mécanisme n'empêchait ça.
2. **Message non lu quatre heures.** Le 28/08, son alerte de 10 h 38 signalant un
   service mort-lettre est restée non lue jusqu'à 14 h.
3. **Mauvais canal lu.** J'ai conclu que K3 était mort en lisant son `outbox`,
   vide par conception ; ses 89 réponses arrivaient sous `reply_MSG-BRIDGE-*`
   dans ma propre boîte.
4. **Propriété supposée, pas vérifiée.** J'ai déclaré le projet W3 mien en
   raisonnant depuis une règle de périmètre ; `git log` dit qu'il est le sien.
5. **Horodatages inventés.** Six de mes messages portaient des heures
   arbitraires, jusqu'à 1 h 25 dans le futur — ce qui casse l'ordre de lecture
   pour tous les agents.
6. **Statut faux.** Des tâches marquées `completed` sans qu'aucune commande ne
   puisse le prouver.

**Ces six pannes n'ont qu'une cause commune : rien n'oblige à vérifier avant
d'affirmer, ni à annoncer avant d'agir.**

---

## 2. Ce qui existe ailleurs — et la trouvaille qui compte

Recherche du 29/08. Sources en fin de document.

### `mcp_agent_mail` — nous avons réinventé la même chose, en moins complet

[`Dicklesworthstone/mcp_agent_mail`](https://github.com/dicklesworthstone/mcp_agent_mail)
est une couche de coordination asynchrone pour agents de code : **identités,
boîtes de réception, fils cherchables, et baux consultatifs sur les fichiers**,
au-dessus de FastMCP + Git + SQLite. Une version Rust existe avec 34 outils,
archive adossée à Git, index SQLite, verrous consultatifs et console TUI.

Son stockage écrit des **artefacts Markdown/JSON canoniques dans un dépôt Git
par projet** — exactement notre architecture, arrivée indépendamment.

**Les deux pièces qui nous manquent :**

- **Le bail consultatif sur un fichier** (*advisory lease*) : un agent annonce
  qu'il travaille sur une zone, avec une **durée de validité**. C'est
  précisément ce qui aurait empêché la duplication du 29/08. Le TTL est
  essentiel : un agent qui plante ne doit pas bloquer les autres pour toujours.
- **L'index de recherche** (SQLite + FTS5) : ma boîte est à **372 fichiers sur
  les 1 000** où l'API GitHub tronque en silence. Un index résout la recherche
  *et* l'échéance.

### Ce qui est devenu natif, et qu'il faut tester avant de construire

- **Agent Teams** (Claude Opus 4.6, février 2026) : une session cheffe engendre
  des coéquipiers, chacun avec son contexte et ses outils ; ils communiquent par
  **boîte aux lettres** et se coordonnent par une **liste de tâches partagée**.
  C'est notre modèle, en natif.
- **Channels** : couche pub/sub intégrée à Claude Code, temps réel **sans
  interrogation périodique ni bricolage d'état partagé**.

### A2A — le standard du dialogue entre agents

[Agent2Agent](https://en.wikipedia.org/wiki/Agent2Agent), v1.0 en avril 2026,
plus de 150 organisations. Là où **MCP relie un agent à ses outils**, **A2A relie
un agent à un autre agent**. Nous faisons du A2A à la main depuis des mois sans
le nommer. Adopter son vocabulaire (fiche d'agent, cycle de vie d'une tâche)
coûte peu et rend le système lisible par un tiers.

### Isolation : les worktrees Git

Pratique établie pour faire travailler plusieurs agents sans conflit : chacun
dans son *worktree*, l'isolation au niveau du système de fichiers, la
coordination au niveau de la **liste de tâches partagée**. Nous en avons déjà —
`.claude/worktrees/allo-desktop-work` — mais sans la liste partagée qui va avec.

---

## 3. Les six règles proposées

Chacune répond à une panne datée de la section 1. Aucune n'est théorique.

### R1 — Annoncer avant de faire, pas après
Avant de commencer un travail visible par les autres — un document, une page,
un service —, **déposer une intention** dans `DEMANDES_JOHN.md` ou dans la boîte
de l'autre. Une ligne suffit : *« je prends X, jusqu'à environ telle heure »*.

*Répare la panne 1.* Une annonce coûte trente secondes ; la duplication du 29/08
a coûté deux heures à deux sessions.

### R2 — Le bail a une durée

> **Complément du 29/08/2026 — inventorier avant de conclure à une absence, et
> ne pas dupliquer sans justification.**
>
> Trois conclusions d'absence fausses le même jour, chez deux agents : le bail
> `VALIDATED` depuis six semaines qu'on s'apprêtait à reconstruire, le trailer
> `Claude-Session` présent dans chaque commit alors qu'on déclarait ne pas
> savoir qui avait fait quoi, et le canal Fable5 — disponible via le modèle
> `fable` — qu'on croyait réservé à un déclenchement humain. Dans les trois cas
> la capacité existait ; **personne ne l'avait inventoriée**.
>
> D'où `c2c-os/00_manifest/CARTE_DES_CAPACITES.md` : ce que chaque agent peut
> réellement faire, chaque ligne portant la commande qui l'établit. Une capacité
> citée sans commande de preuve y est classée « déclaré, non vérifié » et **ne
> se cite pas comme acquise** dans un plan.
>
> **Avant de construire quoi que ce soit :**
>
> ```
> bash ops/deja-fait.sh <mots-cles>    # ca existe deja ?
> bash ops/bail.sh voir                 # quelqu'un est dessus ?
> bash ops/bail.sh prendre <perimetre> <minutes> "<raison>"
> ```
>
> **Pas de redondance, sauf dans trois cas — et ces trois-là sont voulus :**
>
> 1. **Vérification adversariale.** Deux agents vérifient le même fait
>    séparément. Le 29/08, allo et allo-desktop ont lu les mêmes trailers sans
>    se concerter et ont convergé : c'est la convergence de deux mesures
>    indépendantes qui a rendu la conclusion solide, pas l'accord entre elles.
> 2. **Séparation conception/critique.** WRITER et CONTRADICTEUR traitent le
>    même texte, exprès.
> 3. **Sauvegarde.** Un dispositif de secours non redondant n'est pas un
>    dispositif de secours.
>
> Hors de ces trois cas, la seconde construction est du travail perdu — et pire,
> elle **diverge** : deux artefacts qui se ressemblent finissent par se
> contredire, et plus personne ne sait lequel fait foi. C'est ce qui est arrivé
> aux cartographies rédigées à la main, qu'aucune des trois redécouvertes du
> 29/08 n'a réussi à empêcher.

> **Amendement du 29/08/2026 — R2 seule ne peut pas fonctionner, et ce n'est
> pas une question de discipline.**
>
> Ce jour-là, deux sessions ont construit la même page en parallèle. Leurs
> messages de coordination se sont **croisés** : le « ne construis pas » part à
> **12:24**, la demande de coordination à **12:26**. Ni l'une ni l'autre n'a
> ignoré le message de l'autre — il n'était pas encore arrivé.
>
> **Le canal est plus lent que le travail qu'il coordonne.** Écrire un message,
> le commiter, le pousser, attendre qu'il soit lu : plusieurs minutes.
> Construire une page : dix. R2 exige un aller-retour là où il n'y a le temps
> que d'un aller simple. Elle a été enfreinte **trois fois le 29/08** — deux par
> allo, une par allo-desktop. Trois infractions le même jour ne sont pas trois
> défauts d'attention : c'est une règle qui demande l'impossible.
>
> **Ce qui remplace l'annonce : le bail.** On ne demande pas la permission, on
> **déclare** qu'on prend un périmètre, et on avance sans attendre de réponse.
> Celui qui arrive second voit la déclaration et s'arrête.
>
> ```
> bash ops/bail.sh voir                                    # avant de commencer
> bash ops/bail.sh prendre page:carte-services 90 "raison"  # refuse si deja pris
> git add c2c-os/02_operational_registers/baux/ && git commit -m 'bail: ...' && git push
> bash ops/bail.sh rendre page:carte-services               # des que c'est fini
> ```
>
> `prendre` **sort en code 1 et refuse** si le périmètre est déjà tenu — c'est
> une barrière, pas un rappel. Testé sur le scénario réel du 29/08 : la seconde
> demande est rejetée avec le nom du détenteur et sa raison.
>
> **Deux choix de conception qui comptent.** Le bail porte sur le **périmètre**
> et jamais sur la session : l'issue `anthropics/claude-code#76727` montre qu'un
> verrou indexé sur la session est contourné par les worktrees, et allo-desktop
> travaille précisément dans un worktree. Et **un fichier par bail**, jamais un
> registre unique : deux sessions qui déclarent en même temps écrivent deux
> fichiers distincts, donc aucun conflit de fusion — le même patron qui fait que
> les boîtes C2C ne conflictent jamais.
>
> **Ce que le bail ne fait pas.** Il ne supprime pas la course : deux sessions
> qui tirent le dépôt à la même seconde voient toutes deux le périmètre libre.
> Il la réduit d'un aller-retour à un aller simple. Le dire vaut mieux que
> promettre une garantie qu'il n'offre pas.
>
> La politique complète existe depuis le **12/07/2026**
> (`RESOURCE_SCOPED_LEASE_AND_STALE_RECOVERY_POLICY_V1.0.md`, statut
> `VALIDATED OPERATING DECISION`) et **n'avait jamais été branchée**. La règle
> qui aurait évité la duplication du 29/08 dormait validée depuis six semaines.
> `ops/bail.sh` est ce branchement, pas une règle nouvelle.
>
> **Pour les collègues, c'est pire sans bail que pour nous** : ils ne lisent pas
> leur boîte tous les quarts d'heure, donc l'écart entre l'annonce et l'action
> se compte en heures, pas en minutes.
Une intention **expire**. Sans nouvelle de son auteur au-delà du délai annoncé,
n'importe qui peut reprendre le travail. Un agent qui plante ne bloque personne.

*Emprunté au bail consultatif de `mcp_agent_mail`.*

### R3 — Un REQUEST non lu est quelqu'un de bloqué
Lire sa boîte est **le premier point** de chaque tour de travail, avant tout
travail personnel. Répondre, ou écrire pourquoi on ne répond pas. **Jamais le
silence.**

*Répare la panne 2.*

### R4 — Nommer le canal, jamais l'agent
On n'écrit pas « K3 ne répond pas ». On écrit « rien dans `kimi-k3/outbox` », et
on nomme **les canaux non testés**. Une absence ne se déclare jamais depuis un
seul instrument, un seul répertoire, une seule machine.

*Répare les pannes 3 et 4. Détail dans `ops/bascules/07_VERIFIER_AVANT_DECLARER.md`.*

### R5 — Toute heure est lue, jamais estimée
`date -u '+%Y-%m-%dT%H:%M:%SZ'` pour le champ `timestamp_utc` du protocole ;
heure de Dublin pour ce qu'un humain lit. **Ne jamais forcer `TZ=Europe/Dublin`** :
la base de fuseaux est vide sur le PC et renvoie de l'UTC déguisé, y compris en
plein juillet. Sans `TZ`, le système donne juste.

*Répare la panne 5.*

### R6 — « C'est fait » exige une commande de preuve
Toute tâche déclarée finie porte, dans sa description, **la commande exacte qui
l'établit et sa sortie attendue** — sur la sortie réelle, pas sur l'état
déclaré. Si on ne peut pas écrire cette commande, la tâche n'est pas finie :
elle est mal définie.

*Répare la panne 6.*

---

## 4. La répartition des périmètres

Convenue le 28/08, tenue depuis, à réaffirmer par écrit :

| Session | Périmètre | Ne touche pas |
|---|---|---|
| **allo** (`conv-claude-architecture-helper-pc-001`) | SSH vers les deux serveurs, identifiants, chaînes de production, déploiements | poste de travail, tâches Windows, `reminders.json` |
| **allo-desktop** (`conv-allo-desktop-01`) | interface graphique, fichiers locaux, `reminders.json`, tâches planifiées Windows | SSH, identifiants, production |

**En cas de chevauchement** : celui dont c'est le périmètre décide, l'autre
propose. En cas de doute, on demande — c'est moins cher qu'une duplication.

---

## 5. Accueillir un collègue humain

La même séquence que pour un agent, plus trois choses :

1. **Une identité** (`conv-<nom>-<projet>-001`) et une boîte.
2. **L'ajouter à `C2C_ALLOWED_SENDERS`** — sinon ses messages sont reçus et
   **silencieusement ignorés**.
3. **Écrire son périmètre** : ce à quoi il a droit, et surtout ce qu'il ne peut
   pas faire. Par défaut le plus restreint qui permette de travailler.
4. **Tester l'aller-retour** par une question concrète. Tant que ce test n'a pas
   eu lieu, l'arrivant n'est pas connecté.
5. **Un tutoriel dans sa langue**, et un premier échange où il obtient un
   résultat utile. Un collègue qui n'obtient rien de son premier contact ne
   revient pas.

**Ce qui reste non testé, et qu'il ne faut pas affirmer** : qu'un fichier Drive
partagé apparaisse dans le connecteur de la session Claude d'un collègue, sur
**son** compte. Le partage sortant est prouvé ; la réception ne l'est pas.

---

## 6. Ce que je recommande, et ce que je ne recommande pas

**Adopter maintenant** — R1 à R6. Coût nul, chacune répare une panne observée.

**Tester avant de construire** — Agent Teams et Channels. Si le natif fait ce
qu'on bricole, continuer à bricoler serait absurde. **Ce test n'a pas été fait :
je ne recommande pas encore, je recommande de mesurer.**

**Emprunter, pas installer** — de `mcp_agent_mail`, prendre les deux idées (bail
avec TTL, index de recherche) plutôt que le logiciel. Installer un serveur MCP
de plus coûte des jetons à chaque message, et nos boîtes fonctionnent.

**Ne pas faire** — remplacer les boîtes C2C. Elles sont versionnées, opposables,
et survivent à une session éteinte. Aucune des solutions trouvées ne fait mieux
sur ces trois points.

---

## 7. Ce que cette politique ne dit pas encore

- **Agent Teams et Channels ne sont pas testés.** Tout ce qui les concerne est
  une lecture de documentation, pas une mesure.
- **L'accueil d'un collègue humain n'a jamais servi en entier.** Il sera corrigé
  au premier usage réel.
- **Aucun mécanisme n'applique R1 à R6.** Ce sont des règles, pas des garde-fous.
  Une règle qu'on peut ignorer sans conséquence finit ignorée — c'est la leçon
  des dix outils « construits jamais branchés » de cette semaine.

Ces réserves sont là exprès. Un document qui affirme tout avec la même assurance
ne permet pas de savoir sur quoi s'appuyer.

---

## Sources

- [`Dicklesworthstone/mcp_agent_mail`](https://github.com/dicklesworthstone/mcp_agent_mail) — boîtes, identités, baux consultatifs, Git + SQLite
- [Version Rust, 34 outils](https://github.com/Dicklesworthstone/mcp_agent_mail_rust)
- [Agent2Agent (A2A)](https://en.wikipedia.org/wiki/Agent2Agent) — v1.0 avril 2026
- [MCP vs A2A, guide 2026](https://dev.to/pockit_tools/mcp-vs-a2a-the-complete-guide-to-ai-agent-protocols-in-2026-30li)
- [Agent Teams, Claude Opus 4.6](https://www.mindstudio.ai/blog/what-is-claude-code-agent-teams/)
- [Worktrees Git pour agents parallèles](https://www.augmentcode.com/guides/git-worktrees-parallel-ai-agent-execution)
- [Verrouillage au niveau fichier pour bases partagées](https://dev.to/authora/stop-your-ai-coding-agents-from-fighting-file-level-locking-for-shared-codebases-3pej)
- [Système multi-agents, retour d'expérience Anthropic](https://www.anthropic.com/engineering/multi-agent-research-system)

---

## Automations à installer chez un nouvel arrivant

> Ajouté le 29/08/2026. Le critère qui organise cette section : **une
> discipline mécanique tourne sans qu'on y pense ; une discipline manuelle sera
> oubliée.** Ce n'est pas un jugement moral — c'est le constat de la journée,
> où R2 a été enfreinte trois fois par deux agents qui la connaissaient.

### Ce qui est MÉCANIQUE — s'installe et protège sans effort

Ces quatre-là refusent. Un arrivant n'a rien à retenir : il se heurte à la
barrière et comprend.

| Automation | Ce qu'elle refuse | Installée par |
|---|---|---|
| `ops/c2c-validate-messages.py` | un message sans en-tête, ou daté du futur | hook de push |
| `ops/bail.sh prendre` | un périmètre déjà tenu — **code 1** | rien à installer, c'est une commande |
| `ops/generer-existant.py --check` | un `EXISTANT.md` périmé | hook de push |
| sauvegarde + **drill de restauration** | rien, mais **prouve** que la sauvegarde se restaure | timer systemd |

Le drill mérite d'être souligné : une sauvegarde jamais restaurée est une
hypothèse. Le drill du 29/08 a passé à 05:15:50 après un échec la veille — sans
lui, on aurait cru la chaîne saine.

### Ce qui reste MANUEL — donc fragile, et il faut le dire

| Discipline | Pourquoi elle n'est pas mécanisée | Coût observé |
|---|---|---|
| Lire sa boîte au début de chaque tour | rien ne le contraint | un `REQUEST` resté 33 min non lu le 29/08 |
| Prendre le bail **avant** de construire | rien ne le demande avant d'écrire un fichier | deux pages construites en double le 29/08 |
| Répondre **au groupe** et non à l'expéditeur | convention, pas mécanisme | reconstitue le goulot qu'on supprime |
| R6 — « c'est fait » exige une commande de preuve | **déclaratif, rien ne le refuse** | des tâches closes sans preuve |
| Scan anti-secret avant commit | fait à la main | un jeton en clair entré dans l'index le 29/08 |

**Chaque ligne de ce tableau est une dette.** Une discipline manuelle sans date
de mécanisation finira par être enfreinte — la seule question est quand.

### Le kit d'un collègue, dans l'ordre

1. **Une identité** `conv-<nom>-<projet>-001` et une boîte.
2. **Les deux allowlists**, qui ne servent pas à la même chose :
   `C2C_ALLOWED_SENDERS` gouverne le **réveil**, `allowed_senders` du worker
   gouverne l'**exécution**. Être dans la première et pas dans la seconde est
   le réglage **sain** par défaut : son message réveille, il ne déclenche rien.
3. **Les trois commandes**, et rien d'autre à mémoriser :
   ```
   bash ops/deja-fait.sh <mots>          # ca existe deja ?
   bash ops/bail.sh voir                  # quelqu un est dessus ?
   bash ops/bail.sh prendre <perimetre> <minutes> "<raison>"
   ```
4. **Un aller-retour réel** avant de le déclarer connecté. Un canal installé
   n'est pas un canal ouvert — le pont a vécu un mois sans qu'aucune session ne
   s'y enregistre.
5. **Les règles non négociables** : aucun secret sur aucun canal ; aucune
   action irréversible sans validation humaine ; ne jamais valider soi-même un
   `human_validation_required` ; les dossiers d'évaluation européens ne sortent
   jamais vers une API externe.

### Ce qu'on ne demande PAS à un collègue humain

**Produire le format à la main.** Nom de fichier horodaté, en-tête de quatorze
champs, ligne de registre : personne ne l'écrira correctement, et le premier
essai raté décourage durablement.

Le chemin est le point d'entrée `POST /c2c/depose` sur le serveur de production, en service depuis
le 29/08 : il pose l'horodatage, dérive l'identifiant, construit le chemin,
valide, commite et pousse. Le collègue écrit en clair.

**Tant qu'une interface ne l'appelle pas**, un agent fait le relais à la main —
lent, mais assumé. Mieux vaut un relais déclaré qu'un canal qui a l'air ouvert
et rejette en silence.

---

## R7 — Attendre une condition, pas un message

> Règle impérative ajoutée le 31/08/2026 après un incident réel des deux côtés
> le même jour : chacune a attendu un message de l'autre pendant environ vingt
> minutes pour un fait qui était observable directement dans `origin/main`
> depuis la seconde où il est devenu vrai.

**Un état partagé (un fichier présent, une fusion terminée, un commit poussé)
se vérifie à la demande — il ne se relève pas à la cadence de celui qui
l'annonce.** Attendre un message pour un fait que `git fetch` révèle en deux
secondes ajoute la latence de la boîte aux lettres à un délai qui n'existe
pas.

**La règle** : avant de rester en attente d'une confirmation d'un pair sur
quelque chose de vérifiable par soi-même (un sha, un fichier, une ligne de
`git log`), vérifier la condition directement, à intervalle court, plutôt que
d'attendre l'accusé de réception. Le message reste utile — il porte
l'explication, la mesure, ce qui n'est pas dans le diff — mais **le
déblocage lui-même ne doit jamais dépendre de sa réception.**

**Ce qui compte comme observable, et ce qui ne l'en est pas** : un état sur
`origin/main` (fichier, commit, sha) est observable par les deux parties sans
message. Une décision humaine, un jugement, ou tout ce qui vit uniquement
dans la tête ou la session de l'autre ne l'est pas — R7 ne remplace pas la
communication, elle en retire seulement ce qu'on n'avait pas besoin de
demander.

**L'obligation symétrique, côté de celui qui s'arrête** (ajout d'Allo,
co-signé) : R7 met toute la charge sur celui qui attend, ce qui est juste
mais incomplet — si celui qui s'arrête ne nomme pas la condition de reprise,
celui qui attend ne sait pas quoi sonder et se rabat sur sa boîte, exactement
ce qui s'est produit le 31/08. **Tout message « je m'arrête » porte sa
condition de reprise et son échéance**, jamais un simple « préviens-moi » :

> *« Je m'arrête jusqu'à ce que `origin/main` contienne le commit X, ou 15
> minutes maximum. »*

**Ce qu'on fait à l'échéance** : une attente sans échéance est un verrou qui
ne se libère jamais. À l'échéance fixée, on reprend quand même et on le dit
— un conflit se résout après coup, un blocage que personne ne voit ne se
résout pas. `ops/attendre-condition.sh` porte cette mécanique
(`--timeout`) ; la règle dit ce qu'il faut en faire une fois le délai
écoulé.

*Répare la panne du 31/08 : deux sessions bloquées l'une sur l'autre par
un message, alors que la condition attendue était visible depuis vingt
minutes.*

**Outillage** : `ops/attendre-condition.sh` pour sonder un état git (voir
ci-dessus) ; le plugin `session-bridge` pour un signal direct entre deux
sessions sur la même machine (le pont porte le signal, la boîte C2C reste
canonique et porte le contenu — jamais l'inverse).

**Piège trouvé en testant le pont le jour même** : un écouteur bloquant se
réveille sur le PLUS ANCIEN message non traité de sa boîte, pas sur le plus
récent. Les deux sessions ont chacune manqué des signaux frais en croyant
mesurer une interruption en direct, parce qu'un arriéré de vieux messages
(notifications de fin de session, pings) se réveillait en premier. **Règle
de manipulation** : vider la boîte du pont des messages déjà traités avant
d'armer un nouvel écouteur — sinon l'instrument confond un état ancien avec
un événement frais, exactement le motif que R7 existe pour combattre,
retourné contre l'outil censé l'appliquer.

---

## R8 — Une conversation sans travail le DÉCLARE et en reçoit

> Règle impérative ajoutée le 29/08/2026 sur demande de John.

**Une conversation qui n'a plus rien à faire ne s'arrête pas en silence.** Elle
dépose un message `STATUS` dans `groupe-disponibilite` disant qu'elle est
libre, ce qu'elle sait faire, et pour combien de temps.

```
POST /c2c/depose
{"expediteur": "<mon-identite>",
 "destinataires": ["groupe-disponibilite"],
 "type": "STATUS",
 "sujet": "Disponible -- <ce que je peux prendre>",
 "corps": "Perimetre, competences, duree estimee de disponibilite."}
```

### Pourquoi c'est une règle et pas un conseil

Le 29/08, **deux sessions ont passé une partie de l'après-midi en « attente
active »** — l'une attendant un audit, l'autre attendant une réponse — pendant
que la liste des tâches comptait plusieurs chantiers non assignés. Personne
n'était bloqué : chacune ignorait simplement que l'autre était libre.

**Une capacité inoccupée qui ne se déclare pas est indistinguable d'une capacité
absente.** C'est le même défaut que les capacités non inventoriées, appliqué au
temps plutôt qu'aux outils.

### Ce que doit faire celui qui lit `groupe-disponibilite`

Assigner, ou dire pourquoi il n'assigne pas. **Un « je suis disponible » sans
réponse est un `REQUEST` non lu** — il tombe sous R1.

Sources de travail, dans l'ordre :

1. les tâches `pending` qui ne dépendent que de nous ;
2. les disciplines encore **déclaratives** de la carte des capacités — chacune
   est une dette datée ;
3. les vérifications adversariales du travail d'un autre : c'est la seule
   redondance qui rapporte plus qu'elle ne coûte.

### La limite, écrite pour ne pas la découvrir plus tard

Cette règle hérite du défaut du canal : **rien ne réveille `groupe-disponibilite`
non plus** (tâche `ALLO#210`). Tant que le réveil des groupes n'est pas branché, une
déclaration de disponibilité sera vue au prochain cycle de chacun — quelques
minutes pour les sessions actives, une nuit pour LEAD.

C'est mieux que le silence, et ce n'est pas encore de l'affectation temps réel.
Le dire évite de croire la règle plus forte qu'elle n'est.

---

## R9 — Un numéro nu n'est jamais un identifiant partagé

**Née d'un incident daté : le 10/09/2026, `#249` a désigné trois choses
différentes en une seule journée.**

LEAD3 m'a demandé l'état des fils « #249, #250 et #252 ». J'ai cherché les
issues GitHub d'`aaaa-os` : elles s'arrêtent à **#74**. J'ai cherché dans
`MESSAGE_REGISTRY.md` : aucune entrée. Puis, quelques heures plus tard, ma
propre liste de tâches a créé un **#249** — sans aucun rapport avec le sien.

### Ce que la mesure a montré

```
issues GitHub AAAA-Coalition/aaaa-os  : 16 au total, la plus haute est #74
#249 / #250 / #252                    : absentes
liste de tâches de LEAD3              : espace de nommage propre à sa session
liste de tâches d'ALLO                : espace de nommage DIFFÉRENT, arrivé à #249 le même jour
```

**Et le piège est pire qu'un 404** : sous #70, ces numéros **résolvent vers une
autre issue existante**. Une vérification rapide renvoie donc un résultat
**plausible et faux** — le pire cas possible, parce qu'il ne déclenche aucune
alerte.

### La règle

**Entre deux sessions, un numéro nu ne référence rien.** Chaque session possède
son propre espace de nommage de tâches, indépendant des autres et des issues
GitHub. Trois espaces se ressemblent et ne communiquent pas :

| Espace | Forme | Portée |
|---|---|---|
| Issues GitHub | `#67` | le dépôt, partagé |
| Liste de tâches d'une session | `#249` | **cette session seule** |
| Notation interne | `T-249` | ce que la convention locale en dit |

**Ce qui est exigé :**

1. **Référencer par le `message_id` canonique** dès qu'on écrit à quelqu'un
   d'autre. C'est la seule clé stable du système, et elle ne collisionne pas.
2. **Si un numéro est vraiment nécessaire, le qualifier** : `LEAD3#249`,
   `ALLO#249`, `gh:aaaa-os#67`. Sans préfixe, il est illisible.
3. **Ne jamais « vérifier » un numéro sur un dépôt sans contrôler la borne
   haute.** Si le numéro dépasse la plus haute issue existante, ce n'est pas
   une issue — quelle que soit la réponse de l'API.

### Pourquoi c'est une règle et pas un conseil

Parce que le coût est asymétrique. Un numéro ambigu ne provoque pas d'erreur :
il provoque une **recherche qui aboutit ailleurs**, et une réponse confiante
sur le mauvais objet. C'est la même famille que R5 (toute heure est lue, jamais
estimée) et R6 (« c'est fait » exige une commande de preuve) : **l'instrument
répond sans erreur, et sa réponse est fausse.**

Le jour où cette règle est née, le même défaut s'est produit trois fois en
douze heures — sur une heure écrite en UTC au lieu de Dublin, sur un chiffre
budgétaire comparé au mauvais périmètre, et sur ces numéros. Trois fois, la
cause était la même : **une valeur sans son espace de référence.**
