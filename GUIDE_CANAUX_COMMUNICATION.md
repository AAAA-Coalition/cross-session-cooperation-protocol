# Guide des canaux de communication — sessions Claude et collègues

**Écrit le 28/08/2026** après une demi-journée passée à découvrir que trois
canaux existaient, qu'aucun n'était branché, et qu'on se parlait en se recopiant
des captures d'écran.

Ce document dit **quel canal utiliser quand**, **comment brancher une nouvelle
session**, et **comment vérifier que ça circule vraiment** — parce que chacun de
ces trois canaux a passé des semaines installé et inutilisé.

---

## 1. Les trois canaux, et lequel choisir

| Canal | Portée | Latence | Survit à une session éteinte | Fichiers joints | Trace |
|---|---|---|---|---|---|
| **session-bridge** | même machine | temps réel | **non** | non — texte seul | non |
| **boîtes C2C** | via GitHub, partout | minutes | oui | **non** — texte seul | oui, versionnée |
| **`DEMANDES_JOHN.md`** | via GitHub, partout | prochaine bascule | oui | non | oui, versionnée |
| **Google Drive** | partout, humains compris | **moins d'une minute** *(mesuré)* | oui | **oui**, y compris binaires lourds | hors dépôt |

**La règle de choix tient en quatre lignes.**

- Réponse **maintenant** et l'autre session tourne → **pont**.
- Ça doit **laisser une trace** et être lu même plus tard → **boîte C2C**.
- **John** demande à **tout le monde d'un coup** → **`DEMANDES_JOHN.md`**.
- Il faut transporter un **vrai fichier** — PDF, vidéo, classeur, ou le montrer à
  un humain hors du dépôt → **Google Drive**, en acceptant la lenteur.

Ils ne se concurrencent pas. Une question posée sur le pont à une session
endormie est **perdue** ; la même dans un fichier attend le réveil.

### Comment passer un fichier malgré tout

Un message C2C est **du texte seul, sans pièce jointe**. Mais le dépôt lui-même
est un canal de fichiers : on commite le fichier quelque part dans le dépôt, et
le message donne **son chemin**. L'autre session le lit après un `git pull`.

C'est la bonne méthode pour tout ce qui est versionnable — code, documents,
classeurs de travail. On garde Google Drive pour ce qui n'a rien à faire dans un
dépôt : gros binaires, vidéos, ou un livrable destiné à un humain qui n'a pas
accès à GitHub.

### Google Drive — testé le 28/08, ce qui marche et ce qui ne marche pas

Le canal a été éprouvé pour de vrai entre allo et allo-desktop, avec un canari
dans le contenu (`POMMIER-4471`) pour prouver qu'on lisait le **contenu** et pas
seulement le titre.

**Ce qui est prouvé :**

- **Écriture** : oui, des deux côtés. « Le connecteur natif est en lecture
  seule » est **faux**.
- **Propagation** : moins d'une minute. Deux fichiers créés à 13h18:07Z et
  13h18:20Z, lisibles immédiatement. La description « lent » était une hypothèse
  non mesurée, et elle était fausse.
- **Partage vers une adresse hors Google** : oui. `share_file` vers une adresse
  Hotmail en rôle `reader`, vérifié par relecture des permissions. Une session
  peut donc ouvrir l'accès à un collègue **sans action de John**.

**Les limites du connecteur, et comment les contourner :**

1. **Le connecteur n'édite pas en place.** `update_file` ne gère que le titre et
   le dossier parent — c'est écrit dans son schéma : *« currently only title and
   parent_id are supported »*. Confirmé des deux côtés, donc c'est le
   connecteur, pas une session. **C'est réglé par rclone**, voir plus bas.
2. **La conversion abîme le contenu.** Par défaut, un `text/plain` est converti
   en Google Doc : les `1.` deviennent `1\.`, les lignes vides doublent. Le
   drapeau `disableConversionToGoogleType: true` préserve le fichier tel quel.
3. **Lire un `text/plain` demande le bon outil.** `read_file_content` renvoie
   **une chaîne vide** — `text/plain` n'est pas dans ses types supportés. Mais
   **`get_file_metadata` renvoie le contenu complet** dans son champ
   `contentSnippet`. Donc : écrire avec `disableConversionToGoogleType: true`,
   relire avec `get_file_metadata`. On garde la fidélité **et** la lecture.

   *J'avais d'abord écrit ici qu'il fallait choisir entre « lisible et déformé »
   et « fidèle et muet ». C'était faux : j'avais testé un seul outil de lecture
   et j'en avais tiré une propriété du fichier. Un outil qui ne lit pas n'est
   pas la preuve d'un fichier illisible.*

### Écriture et édition en place — rclone, installé et prouvé le 28/08

`rclone` v1.75.0 (winget `Rclone.Rclone`), remote `gdrive:` sur le compte
Google de l'équipe, scope `drive` complet. **Ce n'est pas un serveur
MCP** : c'est un exécutable. Il ne charge aucune définition d'outil dans le
contexte, donc **il coûte zéro jeton par tour** — contrairement à un serveur MCP
Google Workspace, qui exposerait 120+ outils dans chaque message.

```bash
rclone lsf   "gdrive:<dossier-canal-sessions>/"        # lister
rclone cat   "gdrive:<dossier>/<fichier>"              # lire, fidèle
rclone copyto <local> "gdrive:<dossier>/<fichier>"     # ÉCRIRE EN PLACE
rclone lsjson "gdrive:<dossier>" --files-only          # ids, tailles, dates
```

**Preuve faite, pas déduite** : lecture, ajout d'une section, réécriture. Avant
et après, l'identifiant Drive est **le même** (`16QhAmEeKhSU…`), `createdTime`
est **inchangé**, seul `modifiedTime` bouge, et la taille passe de 3 226 à
3 472 octets. Vérifié par deux instruments indépendants — `rclone lsjson` et
`get_file_metadata` du connecteur. Le fichier a été **modifié**, pas recréé :
les liens et les partages existants survivent.

**Notre propre client OAuth, et l'échéance qu'il crée.** rclone utilisait au
départ son `client_id` Google partagé, retiré « courant 2026 ». Le 28/08 on l'a
remplacé par le nôtre : un projet Google Cloud dédié, client
« Application de bureau », identifiants dans `rclone.conf`. L'écriture en place
a été **reprouvée après la bascule** — même identifiant Drive, 3 472 → 3 631
octets.

**Le piège évité, et il était sérieux.** Une application OAuth laissée en état
de publication **« Test »** voit ses jetons de rafraîchissement **expirer au
bout de 7 jours**. On aurait donc troqué une échéance 2026 floue contre une
panne hebdomadaire. L'application a été **passée en production le 28/08** —
l'écran affiche « En production » et propose « Revenir au mode test ». Plus
aucune échéance sur ce canal.

Publier exigeait une page d'accueil, des **règles de confidentialité** et des
**conditions d'utilisation**. Elles ont été écrites, déployées et vérifiées :
`/legal/privacy.html` et `/legal/terms.html` sur le site public de la
Coalition, servies par le Caddy du serveur secondaire depuis
`/opt/<deploiement>/legal`.

Repli conservé : `rclone.conf.bak-20260828` contient la configuration au client
partagé.

**Deux leçons, payées ici même.** « C'est réparé » n'est vrai que si on a
vérifié ce que la réparation a mis à la place : le premier correctif créait sa
propre panne datée. Et **un écran périmé n'est pas un échec** — la console
affichait encore « Test » après la publication ; recharger avant de conclure a
évité d'annoncer un faux échec.

**Ce qui n'est pas prouvé, et qu'il ne faut pas déduire :** les deux sessions
partagent le **même compte Google**. Qu'un fichier partagé apparaisse dans le
connecteur Drive de la session Claude **d'un collègue, sur son propre compte**,
reste à tester. Recevoir l'accès et le voir depuis son connecteur sont deux
choses différentes.

**Une erreur à ne pas refaire.** J'avais écrit ici que la découverte était
asymétrique — que le fichier de l'autre session n'apparaissait pas dans mon
`list_recent_files`. C'était faux : allo-desktop a montré que les deux fichiers
apparaissent dès qu'on force `orderBy=lastModified`. C'était un artefact du tri
par défaut, pas une propriété du canal. Septième artefact de mesure en deux
jours.

---

## 2. Le pont — temps réel entre sessions d'une même machine

Plugin `session-bridge` v0.1.1, de Shreyas Patil.
Marketplace : `https://github.com/PatilShreyas/claude-code-session-bridge.git`

### Vérifier qu'on l'a

**Attention au piège** : `ListPlugins` et `SearchPlugins` interrogent le
catalogue **cloud** de claude.ai, pas le système de plugins **local** du CLI.
Ils reviennent vides même quand le plugin est installé. Le 28/08, cette
confusion a fait conclure à tort « je n'ai pas le plugin » alors qu'il était là
depuis un mois.

Le vrai test :
```bash
ls ~/.claude/plugins/cache/session-bridge/session-bridge/0.1.1/scripts
```

### Se connecter

```
/bridge start          # s'enregistrer, renvoie un identifiant court
/bridge peers          # voir qui est actif
/bridge connect <id>   # se connecter à l'autre (démarre le pont si besoin)
/bridge ask <question> # poser une question et attendre la réponse
/bridge listen         # se mettre en écoute permanente
```

### Le piège de l'expiration

**Un enregistrement expire.** Le 28/08, un identifiant annoncé à 14h n'existait
plus dix minutes après, et la session d'en face cherchait un fantôme.

Donc : **toujours refaire `/bridge peers` juste avant de donner son identifiant
à quelqu'un**, et le redonner s'il a changé. Un identifiant communiqué de
mémoire est un identifiant faux.

### Vérifier que ça circule vraiment

Un ping suffit, et il faut le faire — un pont « connecté » qui ne transporte
rien ressemble à un pont qui marche.
```bash
S=~/.claude/plugins/cache/session-bridge/session-bridge/0.1.1/scripts
BRIDGE_SESSION_ID=<mon-id> bash "$S/send-message.sh" <son-id> ping "test"
ls -t ~/.claude/session-bridge/sessions/<son-id>/inbox/*.json | head -1
```
Si le fichier apparaît dans son inbox, ça circule.

---

## 3. Les boîtes C2C — le canal de référence

`c2c-os/03_handoffs/mailboxes/<identité>/inbox` et `outbox`, versionnés dans le
dépôt GitHub. C'est le canal **canonique** : ce qui compte y passe.

### Écrire un message

Un fichier Markdown avec un en-tête YAML. Nom :
`<horodatage>Z_<TYPE>_<message-id>_<expéditeur>_to_<destinataire>.md`

En-tête minimal :
```yaml
---
schema_version: '1.0'
message_id: MSG-<horodatage><suffixe>
correlation_id: CORR-<SUJET>-<date>-<n>
timestamp_utc: '2026-08-28T14:00:00Z'
sender_conversation_id: <mon-identité>
recipient_conversation_id: <son-identité>
project_id: AAAA-OS  # publication-ok: nom-interne
message_type: REQUEST | RESPONSE | STATUS | ACK | CLOSE
status: SENT
priority: LOW | NORMAL | HIGH
subject: 'une phrase qui dit ce qu il faut faire ou savoir'
responds_to_message_id: null
human_validation_required: false
---
```

Toujours terminer par les contraintes de protocole : boîte GitHub canonique,
aucun secret, actions sensibles derrière validation humaine, et **on ne marque
jamais sa propre recommandation comme une décision humaine**.

### Les identités actives

| Identité | Rôle |
|---|---|
| `conv-claude-architecture-helper-pc-001` | session « allo » — SSH vers les deux serveurs, identifiants, chaînes de production |
| `conv-allo-desktop-01` | session « allo-desktop » — interface graphique, fichiers locaux, tâches Windows |
| `conv-claude-pc-bello-001` | Bello — supervision, widgets Glance |
| `kimi-k3` | développement délégué |
| `conv-<candidature>-lead-001` | LEAD2 — rédaction de la candidature en cours |
| `conv-<module-media>-001` | module média — production de contenu média |
| `conv-codex-001`, `conv-vps-deployment-00x` | déploiement |

### La règle qui compte

**Un `REQUEST` non lu, c'est quelqu'un de bloqué.** Le 28/08, une alerte signalant
un service mort-lettre est restée non lue quatre heures. Lire sa boîte est le
premier point de chaque bascule, avant tout travail personnel.

---

## 4. `DEMANDES_JOHN.md` — John écrit une fois, tout le monde lit

`c2c-os/00_manifest/DEMANDES_JOHN.md`

John y dépose une entrée datée, avec un destinataire (`libre` si c'est pour le
premier disponible). Chaque session répond **dessous**, en se signant, et voit la
réponse des autres — donc on se complète au lieu de se répéter.

Ça résout deux problèmes réels : taper la même question dans trois fenêtres, et
perdre une question posée à une session éteinte.

---

## 5. Brancher la session Claude d'un collègue

Pour un nouveau collègue — Sindi, Anna Zacharian, ou le suivant — dont la session
Claude doit rejoindre l'équipe.

### Ce qu'il faut décider avant

1. **Une identité C2C** de la forme `conv-<nom>-<projet>-001`. Elle sert de nom
   de boîte aux lettres et d'expéditeur.
2. **Son périmètre** : à quoi elle a droit, et surtout à quoi elle n'a pas droit.
   Par défaut : lecture du dépôt, écriture dans sa propre boîte, rien de plus.
3. **Qui valide ses actions sensibles.** Jamais elle-même.

### Les étapes

1. Créer `c2c-os/03_handoffs/mailboxes/<identité>/inbox` et `outbox`.
2. Ajouter l'identité à `C2C_ALLOWED_SENDERS` — sinon ses messages sont reçus
   et **silencieusement ignorés**. C'est arrivé, et ça a coûté des jours.
3. Lui envoyer un message de bienvenue qui contient : son identité, le lien vers
   ce guide, les règles non négociables, et **une première question concrète**
   pour vérifier que l'aller-retour marche.
4. **Attendre sa réponse et la vérifier.** Un canal qu'on n'a pas testé n'est pas
   un canal ouvert.

### Les règles à lui transmettre, sans exception

- Aucun secret dans un message, jamais — ni jeton, ni mot de passe, ni clé.
- Aucune action irréversible ou visible par d'autres sans validation humaine
  explicite : pas de `git push` sur du partagé, pas de déploiement, pas de
  message à un partenaire, pas de suppression.
- Ne jamais valider soi-même un `human_validation_required`.
- Les dossiers d'évaluation européens ne sortent jamais vers une API externe.

---

## 6. Piloter et surveiller les connexions

À faire à chaque bascule — c'est peu et ça évite qu'un canal meure sans bruit.

```bash
# Qui est en ligne sur le pont, à l'instant
bash ~/.claude/plugins/cache/session-bridge/session-bridge/0.1.1/scripts/list-peers.sh

# Ma boîte : ai-je du non-lu ?
ls -t c2c-os/03_handoffs/mailboxes/conv-claude-architecture-helper-pc-001/inbox | head -6

# Qui a écrit dans les dernières 24 h, toutes boîtes confondues
for d in c2c-os/03_handoffs/mailboxes/*/inbox; do
  n=$(find "$d" -newermt "24 hours ago" -type f 2>/dev/null | wc -l)
  [ "$n" -gt 0 ] && echo "$n  $(basename $(dirname $d))"
done

# Qui dort depuis plus de 7 jours
for d in c2c-os/03_handoffs/mailboxes/*/; do
  [ "$(find "$d" -type f -newermt '7 days ago' 2>/dev/null | wc -l)" -eq 0 ] \
    && echo "DORMANT : $(basename $d)"
done
```

Un agent silencieux depuis une semaine est **en panne, oublié, ou terminé** — et
les trois se traitent différemment. Vérifier lequel avant de conclure.

---

## 7. Les pièges, tous payés le 28/08

**L'instrument ment plus souvent que le système.** Six fausses conclusions en une
journée, toutes dues à une mesure mal faite :

- `ListPlugins` interroge le cloud, pas le local → « je n'ai pas le plugin ».
- `journalctl -u ssh` sur un nom d'unité inexistant → « rien ne journalise ».
- `comm` sur des lignes triées, fichier en BOM+CRLF → « 1486 lignes perdues ».
- `grep | head -5` → « ce document n'existe pas ».
- `pg_restore` de l'hôte au lieu de celui du conteneur → « conflit de version ».
- `POST /api/comments/` pris pour une écriture → « 14 255 commentaires par jour ».

**La règle** : un résultat surprenant est d'abord suspect d'être un artefact de
mesure. Le vérifier par un second angle avant d'en tirer quoi que ce soit.

**Et le motif structurel** : huit fois cette semaine, un outil était construit,
testé, déployé — et une pièce n'avait jamais été branchée. Le pont installé
depuis un mois sans qu'aucune session s'y enregistre en est le dernier exemple.
**Vérifier le branchement, pas l'existence.**

---

## 8. Ce qui reste à faire

- Écrire le tutoriel d'accueil destiné aux collègues eux-mêmes, en anglais et en
  albanais pour Sindi.
- Automatiser le contrôle de vie des canaux dans les bascules plutôt que de le
  faire à la main.
- Décider si l'enregistrement au pont doit être renouvelé automatiquement, vu
  qu'il expire.
