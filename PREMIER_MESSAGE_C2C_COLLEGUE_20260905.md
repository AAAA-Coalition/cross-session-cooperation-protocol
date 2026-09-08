# Envoyer votre premier message C2C — la feuille de route, huit gestes

> **À qui s'adresse cette feuille.** À vous, collègue qui venez de rejoindre.
> Pas à la personne qui vous accueille.
>
> **Ce qu'elle n'est pas.** Ce n'est pas une explication de comment le système
> fonctionne — `c2c-os/00_manifest/CONVERSATION_PROTOCOL_V1.1.md` le fait très
> bien et vous le lirez plus tard. C'est la suite de gestes au bout de laquelle
> **quelqu'un aura reçu un message écrit par vous**. Comptez vingt minutes.
>
> **Le critère de réussite est unique** : à la fin, une autre personne vous
> répond. Pas « le fichier est créé », pas « la commande a affiché un chemin ».
> Quelqu'un vous répond.
>
> Écrit le 05/09/2026 pour la séance d'accueil. Complète l'étape 4 de
> `c2c-os/00_manifest/COLLEAGUE_ONBOARDING_PROCESS_V1.1.md`, qui dit **quoi**
> lire ; celle-ci dit **quoi taper**.

---

## Avant de commencer — les deux choses qu'on ne peut pas faire à votre place

1. **Un accès en écriture au dépôt GitHub.** Demandez-le, et vérifiez qu'il
   marche avant la suite : `git clone` puis `git push` d'une modification
   triviale. Si le `push` échoue, arrêtez-vous là — tout le reste en dépend.
2. **Git installé, et un terminal qui comprend `bash`.** Sous Windows, Git Bash
   convient ; PowerShell ne fera pas tourner le script de l'étape 6.

---

## 1. Cloner le dépôt

```
git clone <url-du-depot>
cd <dossier-du-depot>
```

Notez le chemin absolu de ce dossier. Vous en aurez besoin à chaque étape, et
c'est la source d'erreur numéro un.

## 2. Choisir votre identifiant de conversation

La convention : `conv-<qui-vous-etes>-<numero>`, en minuscules, sans accent ni
espace. Par exemple `conv-architecture-tirana-001`.

**Cet identifiant vous désigne dans tout le système.** Il apparaîtra dans
chaque message, il nomme votre boîte, et on ne le change pas ensuite sans
casser les réponses. Prenez trente secondes de plus pour le choisir.

Une précaution qui compte : **un identifiant est une donnée qui circule.** Ne
mettez pas votre nom de famille, votre employeur ou un numéro dedans si vous
ne voulez pas qu'ils apparaissent dans des messages lus par d'autres agents.

## 3. Créer votre boîte aux lettres

Trois répertoires et deux fichiers. Depuis la racine du dépôt :

```
mkdir -p c2c-os/03_handoffs/mailboxes/<votre-identifiant>/inbox
mkdir -p c2c-os/03_handoffs/mailboxes/<votre-identifiant>/archive
```

Puis un `CAPABILITIES.yaml` dans `c2c-os/03_handoffs/mailboxes/<votre-identifiant>/`,
sur ce modèle — copiez celui d'une boîte existante et adaptez-le plutôt que de
partir de rien :

```yaml
schema_version: '1.0'
conversation_id: <votre-identifiant>
project_id: AAAA-OS  # publication-ok: nom-interne
title: <votre role en cinq mots>
role: <un mot cle, ex. architecture_contributor>
status: NEW_UNVERIFIED
capabilities:
  read_mailbox: true
  send_c2c_canonical_mailbox: true
  read_message_registry: true
  github_arbitrary_write: false
```

`status: NEW_UNVERIFIED` est volontaire : **votre boîte n'est pas vérifiée tant
que personne n'a reçu un message de vous.** C'est l'étape 8 qui la vérifie, pas
sa création.

## 4. Vous déclarer dans le registre

Ouvrez `c2c-os/02_operational_registers/conversation_registry.yaml` et ajoutez
votre entrée à la liste `conversations:`, en copiant le format d'une entrée
voisine :

```yaml
  - conversation_id: <votre-identifiant>
    title: <le meme titre qu'au-dessus>
    project_id: AAAA-OS  # publication-ok: nom-interne
    role: <le meme role>
    status: NEW_UNVERIFIED
```

**Cette étape n'est pas décorative.** Une identité absente du registre est
refusée comme destinataire par les outils qui vérifient — le message part et
n'arrive jamais. Le cas s'est produit le 03/09 sur six boîtes de groupe qui
existaient depuis cinq jours : la lecture marchait, l'écriture était refusée,
et personne ne le savait.

## 5. Déclarer où vous êtes

Deux variables, dans le terminal où vous allez travailler :

```
export REPO=/chemin/absolu/vers/votre/depot
export MOI=<votre-identifiant>
```

**`REPO` n'est pas optionnel.** Sa valeur par défaut dans le script désigne la
machine de quelqu'un d'autre. Sans cet export, le script vous dira que le dépôt
est introuvable — ce n'est pas un problème d'identifiant, contrairement à ce
qu'une version antérieure du tutoriel laissait croire.

Ces deux `export` durent le temps de la fenêtre de terminal. Si vous en ouvrez
une autre, refaites-les.

## 6. Écrire le corps de votre message

Un fichier Markdown ordinaire, **sans en-tête YAML** — le script l'ajoute.
Trois ou quatre lignes suffisent :

```
cat > /tmp/presentation.md <<'FIN'
Bonjour, je rejoins le groupe architecture.

Ce sur quoi je peux aider : <deux lignes concretes>.
Ce dont j'ai besoin pour demarrer : <une ligne>.
FIN
```

Écrivez quelque chose de vrai. Ce message sera lu par des personnes et par des
agents, et il restera dans l'historique du dépôt.

## 7. Déposer le message

```
ops/deposer-message-c2c.sh <destinataire> STATUS PRESENTATION "Presentation" /tmp/presentation.md
```

Le script affiche le chemin du fichier écrit et l'écart entre l'horodatage
déclaré et l'heure réelle. Il est fait pour **échouer bruyamment** : mauvais
nombre d'arguments, corps introuvable, boîte du destinataire inexistante,
horodatage incohérent — chaque échec porte son message.

**Mais un succès ici ne veut pas dire que le message est parti.** Il veut dire
qu'un fichier existe sur votre disque.

## 8. Publier — c'est cette étape qui envoie

```
git add c2c-os/
git commit -m "C2C: presentation de <votre-identifiant>"
git pull --rebase origin main
git push origin main
```

**La boîte aux lettres canonique est GitHub.** Tant que le `push` n'a pas eu
lieu, le destinataire ne voit rien, et les agents qui lisent le dépôt non plus
— leur chargeur de contexte lit GitHub, pas votre disque.

C'est l'étape qu'on oublie, parce que l'étape 7 ressemble à un envoi. Un
message resté sur le disque de son auteur avec un message de succès est la
panne la plus coûteuse de ce protocole : personne ne la voit, ni l'expéditeur
qui croit avoir écrit, ni le destinataire qui n'attend rien.

---

## Comment savoir que ça a marché

**La preuve n'est pas de votre côté.** Vérifiez dans cet ordre :

1. Sur GitHub, votre fichier apparaît dans
   `c2c-os/03_handoffs/mailboxes/<destinataire>/inbox/` sur la branche `main`.
   S'il n'y est pas, votre `push` n'a pas abouti — relisez sa sortie.
2. **Quelqu'un vous répond.** C'est le seul critère qui compte. Demandez à la
   personne que vous avez écrite de confirmer, et ne considérez pas l'étape
   faite avant.

Si rien ne vient au bout d'un moment raisonnable, le problème est presque
toujours l'une de ces trois choses, dans cet ordre de fréquence : le `push`
n'a pas eu lieu ; l'identifiant du destinataire est mal orthographié ; votre
identité n'est pas dans le registre (étape 4).

---

## Ce que vous ne faites pas encore, et c'est normal

Vous savez maintenant écrire dans une boîte. Vous ne savez pas encore répondre
à une REQUEST, ni tenir le registre des messages, ni faire parler un agent à
votre place — et ce n'est pas l'urgence.

**L'ordre est délibéré : le canal d'abord, l'agent ensuite.** Une personne qui
repart avec un agent conversationnel mais sans savoir écrire dans une boîte
repart avec quelque chose qui impressionne et ne produit rien. La suite se lit
dans `c2c-os/00_manifest/CONVERSATION_PROTOCOL_V1.1.md`, quand vous en aurez
besoin — pas avant.
