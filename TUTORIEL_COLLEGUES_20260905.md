---
title: "Tutoriel pour les collègues — canaux de communication, bots, coopération"
date: 2026-09-05
redige_le: 2026-09-04
statut: "Préparé pour la session de transfert de connaissances du 05/09/2026"
version_anglaise: "docs/TUTORIEL_COLLEGUES_20260905_EN.md"
---

# Tutoriel pour les collègues — 5 septembre 2026

Ce document est écrit pour quelqu'un qui arrive sans aucun contexte. Il ne
remplace pas les documents de référence du projet ; il dit lesquels lire, dans
quel ordre, et il rassemble ce qu'il faut savoir avant de participer : quels
canaux de communication existent et lequel sert à quoi, ce qu'est un bot ici
et ce qu'il n'est pas, comment on coopère sans se marcher dessus, et les
erreurs qui ont déjà coûté cher — avec leur date, parce qu'une leçon sans date
est une opinion.

Tout ce qui suit est tiré de documents existants, mesurés sur les systèmes en
production. Rien n'y est promis qui n'ait été prouvé.

## 1. Les canaux de communication, et lequel sert à quoi

Il existe plusieurs façons de se parler dans ce projet, et le bon choix n'est
pas évident au premier jour. La confusion coûte : une question posée sur le
mauvais canal peut rester sans réponse pendant des jours, ou au contraire être
lue par des gens à qui elle n'était pas destinée. Voici les quatre canaux
principaux, et la règle de choix.

**Le groupe Telegram** est le canal des sujets collectifs. Tout le monde y
voit tout, humains comme bots. On l'utilise quand une information concerne le
groupe entier, quand on veut poser une question à la cantonade, ou quand on
veut que le bot y réponde devant témoins. On ne l'utilise pas pour quoi que ce
soit de confidentiel — un groupe mélange presque toujours des personnes qui
n'ont pas les mêmes droits d'accès, et la règle en vigueur est qu'on y parle
comme au participant le moins autorisé. On ne l'utilise pas non plus pour ce
qui doit laisser une trace durable et retrouvable : Telegram se relit mal, et
le bot lui-même ne voit que les quarante derniers messages. Ça se lit dans
Telegram, et nulle part ailleurs.

**Les boîtes aux lettres C2C**, dans `c2c-os/03_handoffs/mailboxes/`, servent
à envoyer un message d'une conversation à une autre — typiquement d'une
session de travail Claude à une autre, ou d'un collègue à une session. Chaque
identité enregistrée a son dossier, avec une boîte de réception. Le message
est un fichier dans le dépôt git : il est donc versionné, daté, traçable, et
il attend son destinataire même si celui-ci est éteint ou absent pendant des
jours. C'est le canal canonique : quand on veut qu'une demande soit lue,
tracée et opposable, c'est ici. On ne l'utilise pas quand on a besoin d'une
réponse dans la minute — la latence se compte en minutes ou en heures, au
rythme où les sessions relèvent leur boîte. Deux choses à savoir avant le
premier envoi : un message C2C est du texte seul, sans pièce jointe (pour
transmettre un fichier, on le commite dans le dépôt et le message donne son
chemin) ; et l'expéditeur doit être dans la liste des expéditeurs autorisés,
sans quoi ses messages sont reçus et silencieusement ignorés — c'est arrivé,
et ça a coûté des jours. Ça se lit dans le dépôt, et le registre des messages
(`c2c-os/02_operational_registers/MESSAGE_REGISTRY.md`) en tient l'index.

**Le pont de session** relie en temps réel deux instances Claude qui tournent
sur la même machine. C'est le seul canal instantané : on pose une question, la
session d'en face répond dans la seconde, avec tout son contexte. On
l'utilise quand on a besoin d'une réponse maintenant et qu'on sait que l'autre
session est allumée. On ne l'utilise pour rien d'autre, parce qu'il ne laisse
aucune trace et ne survit pas à une session éteinte : une question posée sur
le pont à une session endormie est perdue, purement et simplement. Piège
connu, payé le 28/08 : l'enregistrement sur le pont expire, et un identifiant
annoncé à 14h peut ne plus exister dix minutes plus tard — on revérifie ses
pairs juste avant d'envoyer. Il n'y a pas d'endroit où relire le pont : c'est
le prix de l'instantané.

**Le fichier `switch.md`**, à la racine du dépôt, n'est pas un message adressé
à quelqu'un : c'est ce qu'on laisse à celui qui prend la suite. Quand une
session s'arrête au milieu d'un travail, elle y écrit où elle en est, ce qui
est fait, ce qui ne l'est pas, et ce qu'il ne faut surtout pas refaire. On
l'utilise pour l'état d'un travail en cours, jamais pour une question qui
attend une réponse — personne en particulier n'est chargé de le lire, c'est
celui qui reprend le travail qui le lit. Ça se lit au début d'une prise de
poste, avant de toucher à quoi que ce soit.

La règle de choix tient en quatre lignes : une réponse maintenant et l'autre
session tourne, c'est le pont ; ça doit laisser une trace et être lu même plus
tard, c'est une boîte C2C ; un sujet qui concerne tout le collectif, c'est le
groupe ; ce qu'on laisse en partant, c'est `switch.md`. Ces canaux ne se
concurrencent pas — ils échouent chacun là où l'autre excelle. Le guide
complet, qui couvre aussi le transport de vrais fichiers par Google Drive, est
`c2c-os/00_manifest/GUIDE_CANAUX_COMMUNICATION.md`.

## 2. Ce qu'est un bot ici, et ce qu'il n'est pas

Un bot, dans ce projet, est une porte, pas un cerveau. Il fait passer une
intention humaine depuis un endroit où les gens sont déjà — Telegram,
typiquement — vers l'endroit où le travail se fait vraiment, et il fait
revenir le résultat. Il lit, il rédige, il propose. Il n'exécute rien
d'irréversible : il n'écrit pas dans le dépôt, il n'envoie rien au nom de
personne, il ne publie pas, il ne s'engage pas, il ne crée ni ne gère de
groupe. Toute action visible par un tiers passe par un humain —
concrètement, par John. Cette règle n'est pas une politesse de façade : elle
est écrite dans le comportement du bot et doublée dans le code par un mode
d'écriture borné, dont le changement est une décision humaine datée.

Voici ce que le bot C2C sait faire aujourd'hui. Cette liste vient de l'audit
du 04/09/2026 (`docs/AUDIT_BOT_C2C_20260904.md`), et chaque capacité y a été
exercée en vrai, pas seulement déployée : lire le dépôt canonique —
documents, listing de dossier, recherche plein texte, sur douze préfixes
autorisés ; chercher sur le web, prouvé en retrouvant l'échéance de
TRANSFO-05, le 23/09/2026 à 17h00 heure de Bruxelles ; lire un PDF ou un
document bureautique, interroger la base de connaissances interne et la
recherche d'articles ; relire les quarante derniers messages du groupe,
prouvé, quarante messages restitués ; envoyer un fichier en pièce jointe,
uniquement depuis une liste déclarée, prouvé dans les deux sens — envoi
réussi, et refus effectif hors liste. Rien d'autre. Si quelqu'un vous affirme
que le bot fait autre chose, la réponse juste est de vérifier dans l'audit
avant de le croire.

Deux comportements du bot surprennent les nouveaux venus et ne sont pas des
pannes. D'abord, le bot lit tout le groupe mais ne prend la parole que s'il
est interpellé — une mention de son nom, une réponse à l'un de ses messages —
ou si c'est une personne autorisée qui écrit. Une question posée dans le vide
par quelqu'un hors de la liste restera sans réponse, et c'est voulu. Ensuite,
un bot qui répond n'est pas la preuve que la chaîne derrière lui fonctionne,
et un bot muet n'est pas forcément cassé. Le document qui explique tout cela,
y compris comment vérifier qu'un bot fonctionne vraiment, est
`docs/ROLE_DES_BOTS_AAAA_OS_20260904.md`.

## 3. Coopérer — entre agents, et entre humains

Trois règles gouvernent la coopération ici. Elles ont chacune été payées
avant d'être écrites.

**Annoncer avant de faire.** Le 28/08/2026, deux sessions ont construit deux
systèmes de rythme journalier en parallèle, chacune ignorant que l'autre
faisait la même chose. Le travail en double n'a été découvert qu'après coup.
Depuis, la règle est d'annoncer ce qu'on entreprend — dans le groupe si le
sujet est collectif, dans la boîte C2C du périmètre concerné sinon — avant de
commencer, pas après avoir fini. Trente secondes d'annonce coûtent moins cher
qu'une journée de travail dupliqué.

**Transmettre une erreur brute, jamais reformulée.** Le 03/09/2026, une
erreur reformulée par celui qui la rapportait a masqué une panne : le texte
réécrit ressemblait à l'erreur précédente, et on a cru avoir affaire au même
problème alors qu'une deuxième barrière, distincte, venait d'apparaître.
C'est le collage de l'erreur exacte, mot pour mot, qui l'a révélée. Quand
quelque chose échoue chez vous, copiez le message d'erreur tel quel — le vrai
texte, pas votre résumé. Votre résumé contient votre hypothèse, et si votre
hypothèse était bonne, vous n'auriez pas d'erreur.

**Une règle de coordination doit être exécutable par ceux à qui on
l'adresse.** Une règle énoncée fin août — « un sujet collectif va dans le
groupe » — n'était en pratique exécutable que par les sessions capables
d'écrire dans le canal en question. Pour les autres, c'était un vœu. Avant
d'édicter une règle de coopération, vérifiez que chaque destinataire a
réellement le moyen de la suivre ; sinon vous n'avez pas écrit une règle,
vous avez écrit un reproche d'avance.

## 4. Les pièges qui ont coûté cher, avec leur date

**Une recherche qui ne trouve rien ne prouve rien, tant qu'on n'a pas vérifié
son périmètre.** Le 04/09/2026, une fiche de référence a affirmé qu'aucun
compte de réseau social n'existait pour la Coalition. C'était faux : les
comptes étaient recensés depuis le 8 août dans
`c2c-os/01_project_memory/REGISTRY_EXTERNAL_ACCOUNTS_APPS.md`. La recherche
qui a produit cette phrase ne couvrait simplement pas ce dossier. Le trou
n'était pas dans le dépôt, il était dans l'endroit où l'on avait cherché.
Avant d'écrire « ça n'existe pas », demandez-vous toujours où vous avez
regardé, et où vous n'avez pas regardé. C'est le piège le plus utile à
connaître pour un nouveau, parce que c'est celui qu'on commet dès la première
semaine.

**Vérifier le fichier corrigé ne prouve rien ; il faut vérifier le chemin
parcouru.** Le 03/09/2026, un filtre défectueux a été corrigé dans un
fichier, et la correction a été prouvée : les bonnes valeurs étaient dans le
fichier, le service avait redémarré. Sans effet — le code qui tournait
réellement était une sous-classe qui redéfinissait la méthode corrigée, et
refaisait le même test avec l'ancienne valeur. Deux définitions de la même
vérité, une seule corrigée. Le bot est resté muet jusqu'au lendemain. Quand
vous déployez une correction, la contre-épreuve passe par l'interface réelle
— refaire l'action qui échouait — jamais par la relecture du fichier.

**Un fait manquant se comble par un document lisible, pas par une règle.** Le
04/09/2026, devant dix-huit personnes, le bot a développé le sigle d'une de nos
organisations en un nom d'organisation qui n'a rien à voir. Il n'a pas désobéi :
aucun document de son contexte ne décrivait cette organisation, et le modèle a inventé une expansion
plausible. Interdire d'inventer ne donne pas la bonne réponse ; il a fallu
écrire une fiche de faits (`c2c-os/00_manifest/REFERENCE_ORGANISATIONS.md`)
et purger la mémoire de groupe qui contenait déjà la version fausse — car une
règle ne bat pas un fait faux déjà présent dans le contexte. La leçon vaut
pour tout le monde, humains compris : quand quelqu'un se trompe faute de
savoir, la réponse est de rendre le fait lisible là où il cherchera, pas
d'ajouter une consigne.

Ces trois pièges ont un air de famille : dans les trois cas, quelque chose
avait l'air vrai — une recherche vide, un fichier corrigé, une règle écrite —
et personne n'avait vérifié le chemin entre l'apparence et la réalité. C'est
le réflexe central de ce projet : on vérifie le branchement, pas l'existence.

## 5. Premiers pas concrets

Si vous arrivez aujourd'hui, voici l'ordre de lecture. D'abord ce document.
Puis `docs/ROLE_DES_BOTS_AAAA_OS_20260904.md`, qui explique les bots que vous
allez croiser dans le groupe et comment savoir s'ils fonctionnent. Puis
`c2c-os/00_manifest/REFERENCE_ORGANISATIONS.md`, la fiche des organisations
et des sigles — c'est elle qui vous évitera de dire une fausseté sur l'une de nos organisations ou
de donner un mauvais lien de réseau social. Ensuite `c2c-os/PIEGES_CONNUS.md`,
court et vivant, à consulter avant toute tâche non triviale. Enfin, le jour
où vous recevez votre propre environnement de travail, le processus complet
est dans `c2c-os/00_manifest/COLLEAGUE_ONBOARDING_PROCESS_V1.1.md` — il
existe aussi en anglais.

Pour poser une question : si elle concerne tout le monde, le groupe
Telegram ; si elle s'adresse à une session ou une personne précise et peut
attendre quelques heures, sa boîte C2C ; si elle est urgente et que vous
savez la session allumée sur la même machine, le pont. Dans le doute, la
boîte C2C est le choix qui ne fait jamais de mal : elle laisse une trace et
finit toujours par être lue.

Reste la question la plus importante : comment savoir si ce que vous croyez
vrai l'est encore. La mémoire canonique du projet est le dépôt git, pas les
conversations, pas les bots — un chiffre sorti d'un bot sans source n'est pas
un fait. Chaque document sérieux du dépôt porte une date et souvent une ligne
de statut : lisez-les, car une page est une photographie du jour où elle a
été écrite, pas un présent perpétuel. Ce tutoriel lui-même sera faux un
jour ; il a été écrit le 04/09/2026, et tout ce qu'il affirme sur les
capacités du bot vaut pour cette date. Si l'enjeu le mérite, remontez à la
source mesurée — l'audit, le registre, le fichier de production — plutôt
qu'au document qui la résume. Et si vous vous surprenez à écrire « ça
n'existe pas », relisez le premier piège de la section 4 avant d'appuyer sur
Entrée.
