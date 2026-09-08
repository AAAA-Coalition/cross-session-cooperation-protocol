---
title: "Piloter sa session Claude en C2C — guide pratique pour un collègue"
date: 2026-09-05
redige_le: 2026-09-04
statut: "Préparé pour la session de transfert de connaissances du 05/09/2026"
---

# Piloter sa session Claude en C2C — 5 septembre 2026

> **À lire d'abord** : `docs/TUTORIEL_COLLEGUES_20260905.md` (version anglaise
> `docs/TUTORIEL_COLLEGUES_20260905_EN.md`) — c'est le tutoriel d'accueil, il
> présente les quatre canaux, ce qu'est un bot ici, et les pièges datés. Le
> présent document en est la suite pratique : vous avez ouvert votre propre
> session Claude Code, et vous devez maintenant coopérer avec les autres.
> Pour voir les règles en action, les sept incidents du 04/09 sont racontés
> dans `knowledge/MODULE_TUTORIEL_COMPORTEMENT_BOTS_20260904.md`.

Rien ici n'est inventé : chaque commande, chaque chemin, chaque fait daté a
été vérifié dans le dépôt le 04/09/2026. Quand quelque chose n'a pas pu être
vérifié, c'est écrit « à confirmer ».

## 1. Ce qu'est une « conversation », et pourquoi elle a un identifiant

Dans ce projet, une « conversation » n'est pas un fil de discussion : c'est
une identité de travail. Votre session Claude Code, celle de votre collègue,
le bot Telegram, l'agent qui tourne sur un serveur — chacun est une
conversation, et chacun porte un identifiant de la forme
`conv-<descriptif>-<NNN>` (par exemple `conv-allo-desktop-01` ou
`conv-claude-architecture-helper-pc-001`). Cet identifiant est ce qui permet
de vous adresser un message, de savoir qui a écrit quoi, et de retrouver dans
six mois qui a pris quelle décision. Il est porté par le champ
`sender_conversation_id` dans chaque message — jamais par l'identité git, qui
est partagée entre plusieurs sessions et ne prouve donc rien.

Toutes les identités vivent dans un registre unique :
`c2c-os/02_operational_registers/conversation_registry.yaml`. C'est là que la
vôtre est déclarée, avec son statut, le chemin de sa boîte aux lettres et ses
contraintes. Un nouveau venu n'invente pas son identifiant : il lui est
attribué à l'étape 0 du processus d'accueil
(`c2c-os/00_manifest/COLLEAGUE_ONBOARDING_PROCESS_V1.1.md`), qui décide
l'identité, crée la boîte, et l'enregistre. Si vous ne connaissez pas le
vôtre, cherchez votre nom dans le registre ; s'il n'y est pas, demandez —
n'en fabriquez pas un.

Pourquoi cette rigidité ? Parce que le système est fermé par construction, et
qu'il échoue en silence quand on triche. Un expéditeur qui n'est pas dans la
liste des expéditeurs autorisés voit ses messages reçus et silencieusement
ignorés — c'est arrivé, et ça a coûté des jours (le tutoriel d'accueil le
raconte). Et le 03/09/2026, une collègue s'est vu refuser l'envoi vers les
boîtes de groupe avec « Recipient conversation ID is not registered » : les
répertoires existaient depuis le 29/08, mais personne ne les avait déclarés
dans le registre — l'incident est documenté en commentaire dans le registre
lui-même. Réutiliser l'identifiant d'un autre est pire encore : vous signez
alors vos actes du nom de quelqu'un d'autre, dans un système dont toute la
valeur est la traçabilité. Un identifiant inventé ne reçoit rien ; un
identifiant usurpé fait accuser un innocent. Ni l'un ni l'autre, jamais.

## 2. Envoyer un message à quelqu'un d'autre

Votre boîte et celles des autres vivent dans
`c2c-os/03_handoffs/mailboxes/`, un dossier par identité, chacun avec
`inbox/`, `outbox/`, `archive/` et un `CAPABILITIES.yaml`. Un message est un
fichier Markdown déposé dans l'`inbox/` du destinataire, avec un en-tête YAML
(`message_id`, `timestamp_utc`, `sender_conversation_id`,
`recipient_conversation_id`, `message_type`, `in_reply_to`,
`human_validation_required`, entre autres).

On ne fabrique pas ce fichier à la main. On utilise l'outil, dont voici la
ligne d'usage exacte, tirée de son en-tête :

```
ops/deposer-message-c2c.sh <destinataire> <type> <slug> <sujet> <corps.md> [id1,id2,...]
```

Le destinataire est un identifiant de conversation ou une boîte de groupe
(`groupe-architecture`, `groupe-candidatures`, `conv-xxx`...). Le slug est un
identifiant court en MAJUSCULES-TIRETS qui entre dans le nom du fichier. Le
corps est un fichier Markdown sans frontmatter — le script l'ajoute. Et
surtout : le script lit votre identité dans la variable d'environnement
`MOI`, qui vaut par défaut `conv-claude-architecture-helper-pc-001` — si ce
n'est pas vous, exportez la vôtre avant d'envoyer, sinon vous signez du nom
d'un autre.

Pourquoi un script plutôt que d'écrire le fichier soi-même ? Son en-tête le
dit : le 03/09/2026, sur quatorze messages émis dans la journée, treize
portaient une heure tapée à la main, fausse de +1 h 48 à +5 h 29, parce
qu'on incrémentait depuis une base déjà fausse. Le quatorzième, le seul
juste, était le seul qu'un outil avait horodaté. Ce script est cet outil : il
lit l'heure, il ne la tape pas, et il vérifie après coup que l'horodatage
écrit colle à la réalité — sinon il échoue bruyamment.

Le script accepte cinq types de message. `REQUEST` demande quelque chose à
quelqu'un : c'est le type qui ouvre un fil et attend une réponse. `ACK`
confirme qu'on a reçu et lu une demande, avant même d'y avoir travaillé — il
évite à l'expéditeur de se demander si son message est tombé dans le vide.
`RESPONSE` apporte la réponse de fond à une demande. `STATUS` dit où on en
est sans que personne n'ait rien demandé : on annonce ce qu'on entreprend, on
signale un résultat qui contredit ce qu'un autre croit vrai. Et
`DECISION_PROPOSAL` soumet une décision à un humain — car aucun agent ne
peut convertir sa propre recommandation en décision humaine, c'est écrit dans
le protocole (`c2c-os/00_manifest/CONVERSATION_PROTOCOL_V1.1.md`, qui décrit
la séquence complète `REQUEST → ACK → RESPONSE → CLOSE` et les types
optionnels). Le déroulé normal d'un fil est donc : une REQUEST, un ACK
rapide, une RESPONSE quand le travail est fait.

Le sixième argument, optionnel, porte les identifiants des messages auxquels
vous répondez : il remplit le champ `in_reply_to`. Ne le négligez pas. Un fil
non rattaché se perd : la relance automatique qui surveille les demandes
ouvertes ne voit pas qu'on leur a répondu, et elle continue de sonner. C'est
arrivé le 04/09/2026 : une réponse a été émise avec un identifiant tapé de
mémoire — `MSG-...-ACCESLECTUREGROUPE` au lieu de
`MSG-...-ACCESLECTUREMESSAGES`, celui que portait réellement la demande. La
réponse existait, elle ne fermait rien, et la relance a continué de sonner,
avec raison. Depuis, le script refuse tout identifiant de `in_reply_to` qu'il
ne retrouve pas dans votre propre boîte : lisez le champ `message_id` dans le
fichier reçu, ne le retapez pas de mémoire. C'est le même défaut que les
treize horodatages, sur un autre champ.

## 3. Savoir qu'on a du courrier — et le lire en premier

Rien ne vous préviendra sur votre écran : le courrier arrive dans votre
`inbox/` par le dépôt git, et c'est à vous d'aller voir. La procédure
canonique est dans `ops/bascules/05_EN_ATTENTE.md`, point 1, et elle tient en
deux commandes : lister sa propre boîte par date, et balayer toutes les
boîtes pour voir ce qui a bougé dans les dernières vingt-quatre heures.
Adaptez le chemin de la première à votre propre identifiant :

```
ls -t c2c-os/03_handoffs/mailboxes/<votre-identifiant>/inbox | head -8
for d in c2c-os/03_handoffs/mailboxes/*/inbox; do
  n=$(find "$d" -newermt "24 hours ago" -type f 2>/dev/null | wc -l)
  [ "$n" -gt 0 ] && echo "$n depuis 24h -> $d"
done
```

(Sur le serveur de production, un service surveille les boîtes — le script
`c2c-mailbox-watcher.py` et son unité systemd existent dans le dépôt ; son état de marche actuel est à
confirmer. Sur votre poste, rien ne le fait à votre place.)

La règle qui accompagne ces commandes est la première règle de coopération du
projet : ses messages se lisent en premier, avant tout travail personnel. Un
message non lu, c'est quelqu'un de bloqué. Le fait qui a gravé cette règle
date du 28/08/2026 : une alerte envoyée à 10h38 — un service en mort-lettre
depuis quatre heures — est restée non lue jusqu'à 14h, pendant que son
destinataire s'acharnait ailleurs sur un jeton. L'alerte avait trouvé la
panne qu'il avait causée et qu'il ne cherchait pas au bon endroit. Quatre
heures perdues, pour un fichier qui attendait dans sa boîte. Et quand vous
avez lu : répondre, ou écrire pourquoi vous ne répondez pas — jamais le
silence.

## 4. Choisir son canal

Quatre canaux coexistent, et ils ne se concurrencent pas : chacun échoue là
où l'autre excelle. Le guide complet est
`c2c-os/00_manifest/GUIDE_CANAUX_COMMUNICATION.md` ; voici la règle de choix
en un tableau.

| Canal | Quoi | Quand | Quand PAS | Où ça se lit |
|---|---|---|---|---|
| Message C2C direct | un fichier dans l'`inbox/` d'une identité précise | demande ou réponse adressée, qui doit laisser une trace et peut attendre des minutes ou des heures | besoin d'une réponse dans la minute | la boîte du destinataire, indexée par `c2c-os/02_operational_registers/MESSAGE_REGISTRY.md` |
| Boîte de groupe thématique | même mécanique, vers `groupe-architecture`, `groupe-candidatures`, etc. | un sujet qui concerne un périmètre de travail, pas une personne | un message qui engage ou bloque une personne précise — elle peut ne jamais se sentir visée | la boîte de groupe, lue par plusieurs identités |
| Groupe Telegram | discussion en clair, humains et bots mêlés | sujet collectif, question à la cantonade, réponse du bot devant témoins | tout ce qui est confidentiel, tout ce qui doit se retrouver plus tard | dans Telegram, et nulle part ailleurs |
| `switch.md` (racine du dépôt) | l'état laissé à qui prend la suite | fin de session au milieu d'un travail : fait, pas fait, à ne pas refaire | une question qui attend une réponse — personne n'est chargé de le lire | au début d'une prise de poste |

Dans le doute, la boîte C2C directe est le choix qui ne fait jamais de mal :
elle laisse une trace et finit toujours par être lue. À noter aussi : un
message C2C est du texte seul — pour transmettre un fichier, on le commite
dans le dépôt et le message donne son chemin ; et l'écriture vers les boîtes
de groupe par l'outil MCP borné était encore marquée non prouvée dans le
registre au 03/09 (`ACTIVE_GROUP_MAILBOX_WRITE_UNVERIFIED`) — par le script
de dépôt, elle fonctionne.

## 5. Les règles de coopération qui ont un coût mesuré

Quatre règles gouvernent le travail à plusieurs. Aucune n'est une opinion :
chacune a une date et une facture.

**Annoncer avant de faire.** Le 28/08/2026, deux sessions ont construit deux
systèmes de rythme journalier en parallèle, chacune ignorant que l'autre
faisait la même chose ; le doublon n'a été découvert qu'après coup. Depuis,
on envoie un `STATUS` court annonçant ce qu'on entreprend — avant de
commencer, pas après avoir fini. Trente secondes d'annonce contre une journée
de travail dupliqué.

**Transmettre une erreur brute, jamais reformulée.** Le 03/09/2026, une
erreur reformulée par celui qui la rapportait a masqué une panne : le texte
réécrit ressemblait à l'erreur précédente, et tout le monde a cru au même
problème alors qu'une deuxième barrière, distincte, venait d'apparaître.
C'est le collage de l'erreur exacte, mot pour mot, qui l'a révélée. Votre
résumé contient votre hypothèse — et si votre hypothèse était bonne, vous
n'auriez pas d'erreur.

**Ne jamais taper une valeur qu'une machine sait donner.** Les treize
horodatages faux du 03/09 (section 2) en sont la démonstration : une date,
une heure, un compte, un identifiant de message se lisent — dans la sortie
d'une commande, dans le champ du fichier — et ne se retapent pas, ne
s'incrémentent pas depuis le tour précédent. Le seul horodatage juste de
cette journée-là était celui qu'un outil avait produit.

**Une recherche qui ne trouve rien ne prouve rien, tant qu'on n'a pas vérifié
son périmètre.** Le 04/09/2026, une fiche de référence a affirmé qu'aucun
compte de réseau social n'existait pour la Coalition. Faux : ils étaient
recensés depuis le 8 août dans
`c2c-os/01_project_memory/REGISTRY_EXTERNAL_ACCOUNTS_APPS.md` — la recherche
ne couvrait simplement pas ce dossier (c'est le quatorzième défaut de l'audit
`docs/AUDIT_BOT_C2C_20260904.md`). Avant d'écrire « ça n'existe pas », dites
où vous avez regardé, et où vous n'avez pas regardé.

## 6. Ce qu'une session ne fait jamais sans un humain

La règle est dans `CLAUDE.md` à la racine du dépôt, et elle n'admet pas
d'exception : une session n'envoie pas de message à un partenaire, ne publie
rien, ne supprime rien, ne modifie pas un `.env` ni aucun identifiant ou
secret, ne redémarre pas un service de production, ne soumet pas de
candidature et ne s'engage au nom de personne — sans validation humaine
explicite. Le pied de page que le script de dépôt ajoute à chaque message le
rappelle : les actions sensibles exigent une validation humaine séparée, et
l'expéditeur ne peut pas marquer lui-même sa recommandation comme une
décision humaine.

Quand votre travail bute sur une de ces portes, vous ne la franchissez pas et
vous ne restez pas non plus muet : vous écrivez la demande d'autorisation là
où elle sera lue. Dans `switch.md`, l'usage établi est de poser le plan sous
la marque `<ATTEND CONFIRMATION HUMAINE>` — le fichier en contient des
exemples réels : le plan est écrit, prêt à exécuter, et rien ne part tant que
John n'a pas dit oui. Pour une demande adressée à John parmi d'autres, le
fichier `c2c-os/00_manifest/DEMANDES_JOHN.md` existe aussi ; et une
`DECISION_PROPOSAL` en C2C fait la même chose sous forme de message. Dans
tous les cas, le principe est identique : la proposition est écrite, datée,
signée de votre identifiant, et l'action attend.

## 7. Les dix premières minutes d'un nouveau venu

Lisez d'abord `docs/TUTORIEL_COLLEGUES_20260905.md` — les canaux, les bots,
les pièges. Puis le présent document. Puis
`c2c-os/00_manifest/REFERENCE_ORGANISATIONS.md`, la fiche des organisations
et des sigles, et `c2c-os/PIEGES_CONNUS.md`, court et vivant. Ensuite,
ouvrez `c2c-os/02_operational_registers/conversation_registry.yaml` et
trouvez votre propre identifiant : s'il n'y est pas, arrêtez-vous là et
demandez votre enregistrement — rien de ce qui suit ne marchera. Jetez enfin
un œil à `switch.md` pour savoir où en est le travail en cours.

Puis vérifiez que votre canal marche vraiment, en envoyant un premier
message. Le précédent existe : le tout premier message de chaque boîte est
souvent un `STATUS` que l'identité s'adresse à elle-même à l'enregistrement.
Faites de même — écrivez deux lignes de présentation dans un fichier, puis :

```
export REPO=/chemin/vers/votre/clone             # <-- indispensable
export MOI=<votre-identifiant>
ops/deposer-message-c2c.sh <votre-identifiant> STATUS PREMIER-ESSAI "Premier essai de canal" /chemin/vers/corps.md
```

**`REPO` n'est pas optionnel.** Sa valeur par défaut désigne la machine de
John ; sans cet export, le script cherche votre boîte dans un dépôt qui
n'existe pas chez vous. Il refuse désormais de continuer et vous le dit —
mais si vous lisez une version antérieure de ce tutoriel ailleurs, sachez que
l'erreur ressemblait à un problème d'identifiant alors qu'elle n'en était pas
un.

**Et le dépôt du message ne l'envoie pas.** Le script écrit un fichier sur
votre disque, rien de plus : la boîte aux lettres canonique est GitHub. Tant
que vous n'avez pas publié, le destinataire ne voit rien, et les agents qui
lisent le dépôt non plus — leur chargeur de contexte lit GitHub, pas le
disque. Donc, juste après :

```
git add c2c-os/
git commit -m "C2C: premier essai de canal"
git pull --rebase origin main
git push origin main
```

C'est le `push` qui envoie. Le script vous le rappelle à la fin de chaque
dépôt, précisément parce qu'un message resté sur le disque de son auteur
avec un message de succès est la panne la plus coûteuse de ce protocole.

Vous saurez que ça a marché parce que le script est bâti pour échouer
bruyamment, jamais en silence. S'il réussit, il affiche le chemin du fichier
déposé et l'écart entre l'horodatage déclaré et l'heure réelle d'écriture —
allez lire le fichier dans votre `inbox/`, c'est la preuve. S'il échoue, il
vous le dit : mauvais nombre d'arguments (la ligne d'usage s'affiche), corps
introuvable, « boite inexistante » si le destinataire n'a pas de boîte —
c'est le signe d'un identifiant mal orthographié ou non enregistré —,
« IDENTIFIANT INTROUVABLE » si un `in_reply_to` ne correspond à aucun message
de votre boîte, et « TEMOIN EN ECHEC » si l'horodatage écrit s'écarte de plus
de deux minutes du réel. Aucune sortie du tout, ou pas de fichier dans la
boîte à l'arrivée : c'est un échec aussi, même sans message — allez voir le
code de retour. Quand votre premier `STATUS` est dans votre boîte, horodaté
juste, votre canal existe : le prochain message peut partir vers quelqu'un
d'autre, et il commencera probablement par annoncer ce que vous entreprenez.

## Ce que ce document ne couvre pas

Il ne couvre pas l'installation de votre environnement — compte, VPS, clés,
enregistrement de votre identité : c'est le processus d'accueil,
`c2c-os/00_manifest/COLLEAGUE_ONBOARDING_PROCESS_V1.1.md`. Il ne couvre pas
le fonctionnement interne des bots Telegram ni leur vérification — c'est
`docs/ROLE_DES_BOTS_AAAA_OS_20260904.md` et le module d'incidents cité en
tête. Il ne couvre pas le pont de session temps réel entre deux instances sur
la même machine, ni le transport de fichiers par Google Drive et rclone — les
deux sont dans `c2c-os/00_manifest/GUIDE_CANAUX_COMMUNICATION.md`. Il ne
couvre pas les protocoles avancés — successions d'identités, baux,
checkpoints entre sessions sœurs — qui vivent dans
`c2c-os/00_manifest/CONVERSATION_PROTOCOL_V1.1.md`. Et il ne couvre pas le
second bot posé sur le serveur secondaire le 04/09 au soir, dont l'état est dans
`ops/JOURNAL_POSE_JUMEAU_20260904.md`. Enfin, il a été écrit le 04/09/2026 :
comme tout document d'ici, c'est une photographie de ce jour-là, pas un
présent perpétuel — en cas de doute, la source mesurée prime sur le résumé.
