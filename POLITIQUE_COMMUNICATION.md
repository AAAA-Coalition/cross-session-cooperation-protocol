# Politique de communication — sessions, agents, LLM, collègues

**Statut : proposition d'allo, en attente d'accord d'allo-desktop.**
Le descriptif est dans `GUIDE_CANAUX_COMMUNICATION.md` : *ce qui existe*.
Ici : *ce qu'on fait, pourquoi, et qui tranche*.

---

## Le principe

**Le canal se choisit sur la durée de vie de l'information, pas sur le confort
de l'expéditeur.**

Une information qui doit survivre à une session éteinte ne passe jamais par un
canal éphémère, même si c'est le plus rapide sous la main. C'est la règle qui
gouverne tout le reste.

---

## 1. La règle de choix — un seul chemin par situation

| La situation | Le canal | Pourquoi celui-là |
|---|---|---|
| J'ai besoin d'une réponse **pour continuer maintenant**, et l'autre tourne | **pont** | seul canal en temps réel |
| Je transmets un résultat, une alerte, une demande qui **engage** | **boîte C2C** | horodaté, versionné, opposable |
| John s'adresse à **tout le monde** | **`DEMANDES_JOHN.md`** | écrit une fois, lu par tous |
| Je dois transmettre un **fichier versionnable** | **dépôt + chemin dans un message C2C** | le fichier vit dans l'historique |
| Fichier **non versionnable** ou destiné à un humain hors dépôt | **Google Drive** | seul canal qui porte les binaires lourds |

**En cas de doute, la boîte C2C.** C'est le canal par défaut : ce qu'on y met ne
se perd pas, et une trace de trop n'a jamais coûté cher.

---

## 2. Ce qui vaut pour tous les interlocuteurs

La politique ne distingue pas les sessions Claude, les agents LLM et les
humains. Elle distingue **ce que l'interlocuteur peut recevoir**.

| Interlocuteur | Canal principal | Particularité |
|---|---|---|
| Sessions Claude (allo, allo-desktop, bello) | C2C + pont | le pont seulement sur la même machine |
| Agents LLM (K3, agent de recherche, passerelle voix, module média) | C2C uniquement | pas de pont, pas d'interface |
| Orchestrateurs (candidature, deployment, codex) | C2C uniquement | mission écrite, réponse écrite |
| Collègues humains | **C2C s'ils coopèrent sur un livrable**, sinon WhatsApp/courriel/Drive | voir §2 bis |
| John | `DEMANDES_JOHN.md` + conversation directe | il ne lit pas les boîtes C2C |

**Conséquence à ne pas oublier :** un agent LLM ne « voit » pas un message qu'on
lui adresse hors de sa boîte. Écrire dans `switch.md` en espérant qu'il le lise
ne marche pas — c'est le genre d'illusion qui fait qu'on attend une réponse qui
ne viendra jamais.

### 2 bis. Les collègues humains dans le C2C — arbitrage de John, 29/08/2026

Ce document disait « **jamais le C2C** » pour les humains. **C'est corrigé.** Un
collègue qui coopère sur un livrable — une candidature, un budget, une relecture
— doit pouvoir écrire dans le C2C, sinon sa contribution reste hors de la trace
et hors de portée des agents.

La ligne de partage n'est pas humain/agent, c'est **ce sur quoi porte
l'échange** : coopération sur un livrable → identité C2C et boîte ; échange
social ou hors périmètre → WhatsApp, courriel, Drive.

**Deux réserves, écrites pour qu'on ne les oublie pas devant Sindi :**

1. **Quatre allowlists, pas une.** `C2C_ALLOWED_SENDERS` ne couvre que le wake
   dispatcher. Il y a aussi `AUTOPILOT_ALLOWED_SENDERS` (`autopilot.py`),
   `allowed_senders` en JSON (`worker.py`) et `mcp_c2c_allowed_sender_ids`
   (`c2c_gateway/app/config.py`). N'en ouvrir qu'une laisse les messages du
   nouveau venu mourir en rejet, dans une base que lui ne verra jamais.
2. **Aucun humain ne produira le format à la main** : nom de fichier horodaté,
   en-tête de quatorze champs, ligne de registre unique. Tant qu'il n'existe pas
   un moyen de saisie simple, le premier message d'un collègue échouera en
   silence — l'exact contraire du « premier échange utile » promis au §4.

---

## 3. Les cinq règles de conduite

Chacune est née d'un échec réel, daté.

### 3.1 Un `REQUEST` non lu, c'est quelqu'un de bloqué
Lire sa boîte est **le premier point** de chaque tour de travail, avant tout
travail personnel. Le 28/08, une alerte signalant un service mort-lettre est
restée non lue quatre heures pendant que son destinataire s'acharnait ailleurs.

Répondre, ou écrire pourquoi on ne répond pas. **Jamais le silence.**

### 3.2 Signaler avant de dupliquer, pas après
Si un chantier chevauche visiblement un travail en cours ailleurs, envoyer un
message de coordination **avant** d'avancer dessus. Le 28/08, deux rythmes
journaliers ont été construits en parallèle sans que personne ne le sache.

### 3.3 Vérifier que le canal vit, pas qu'il existe
Un canal installé n'est pas un canal ouvert. Le pont était installé depuis un
mois **sans qu'aucune session s'y enregistre**. Une identité C2C absente de
`C2C_ALLOWED_SENDERS` reçoit des messages qui sont **silencieusement ignorés**.

Avant de compter sur un canal : un aller-retour réel, prouvé.

### 3.4 Ne jamais recopier un identifiant de mémoire
L'enregistrement au pont **expire**. Un identifiant annoncé à 14h n'existait plus
dix minutes après, et l'autre session cherchait un fantôme.

Toujours refaire `/bridge peers` juste avant de donner son identifiant.

### 3.5 Aucun secret sur aucun canal
Ni jeton, ni mot de passe, ni clé — dans aucun message, aucun document partagé,
aucune ligne de commande. `sudo` journalise la commande entière ; `ps` expose
les arguments. Un secret se transmet par un fichier en mode 600, ou pas du tout.

---

## 4. Accueillir un nouvel arrivant — sessions, agents, collègues

La même séquence, quel que soit le type d'interlocuteur.

1. **Lui donner une identité** (`conv-<nom>-<projet>-001` pour un agent, un
   canal nommé pour un humain).
2. **Écrire son périmètre** : ce à quoi il a droit et surtout ce qu'il ne peut
   pas faire. Par défaut le plus restreint qui permette de travailler.
3. **Le brancher réellement** : boîte créée **et** identité ajoutée à
   `C2C_ALLOWED_SENDERS`. L'un sans l'autre donne un canal muet.
4. **Tester l'aller-retour** par une première question concrète, et attendre la
   réponse. Tant que ce test n'a pas eu lieu, l'arrivant n'est pas connecté.
5. **Lui transmettre les règles non négociables** : aucun secret, aucune action
   irréversible ou visible par d'autres sans validation humaine, jamais valider
   soi-même un `human_validation_required`, et les dossiers d'évaluation
   européens ne sortent jamais vers une API externe.

Pour un **collègue humain**, ajouter : un tutoriel dans sa langue, et un premier
échange où il obtient un résultat utile. Un collègue qui n'a rien obtenu de son
premier contact ne revient pas.

---

## 5. Qui tranche quoi

- **Périmètres entre sessions** : accord entre les sessions concernées, écrit
  dans ce document. Au 28/08 — allo prend le SSH vers les deux VPS, les
  identifiants et les chaînes de production ; allo-desktop prend l'interface
  graphique, les fichiers locaux, `reminders.json` et les tâches Windows.
- **Ouverture d'un canal vers un nouvel interlocuteur** : décision de John,
  parce que ça engage le projet vis-à-vis d'un tiers.
- **Tout ce qui est irréversible ou visible par d'autres** : John, toujours. Une
  session ne valide jamais sa propre recommandation.

---

## 6. Ce que cette politique ne dit pas encore, et qu'il ne faut pas inventer

**Google Drive n'a pas été testé** comme canal entre agents. Je l'ai décrit comme
« lent » sans l'avoir mesuré, et allo-desktop a refusé de le confirmer faute de
l'avoir éprouvé — elle a eu raison. Tant qu'un aller-retour réel n'a pas eu lieu,
ce canal reste une hypothèse dans ce document, pas un fait.

**L'accueil d'un collègue humain** décrit ci-dessus n'a pas encore servi en
entier. Il sera corrigé au premier usage réel.

Ces deux réserves sont là exprès. Un document qui affirme tout avec la même
assurance ne permet pas de savoir sur quoi s'appuyer.
