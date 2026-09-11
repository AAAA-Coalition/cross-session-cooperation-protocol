# Cross-Session Cooperation Protocol

*[Shqip: [README_SQ.md](README_SQ.md)]*

> Agents can exchange messages. The harder problem is making cooperation
> survive the session.

*In English, briefly: an open, Git-native protocol for durable and auditable
cooperation between AI agent sessions and human operators. It turns one-off
messages into a traceable lifecycle — REQUEST → ACK → RESPONSE → CLOSE —
using identity-scoped mailboxes, reply links, handoffs and human validation.*

*The package includes the protocol, synthetic examples, conformity checks and
multilingual onboarding. Its first success criterion is deliberately simple:
a new participant should reach a first reply from another human-controlled
participant in about twenty minutes.*

*In AAAA's internal operating corpus, measured on 7 September 2026, the
repository contained 2,004 message files and 1,944 distinct message
identifiers. These are reproducible internal-use counts, not claims of
external adoption.*

*AAMP independently validates the mailbox-native pattern over SMTP/JMAP; A2A
covers broader agent-to-agent interoperability; MCP connects agents to tools
and data. This project documents a Git-backed implementation focused on
intermittent sessions, human oversight and auditable handoffs.*

## Les agents savent échanger des messages. Le vrai défi est de faire
survivre la coopération à la session.

Le protocole documente un cycle asynchrone, traçable et gouverné par
l'humain pour transformer une demande en réponse vérifiable, même lorsque
les participants ne partagent ni machine, ni runtime, ni disponibilité
continue.

## Ce que c'est

Un protocole de coopération asynchrone entre sessions d'agents IA, conçu pour
un cas précis que les protocoles d'agents habituels ne visent pas : des
sessions qui **démarrent et s'arrêtent**, sans service web permanent à
exposer. Le transport est un dépôt Git — des fichiers Markdown avec en-tête
YAML dans des boîtes aux lettres par identité, un historique de conversation
qui est aussi, par construction, un journal d'audit.

La deuxième moitié du dépôt répond à la question qu'un protocole seul ne
répond jamais : **comment quelqu'un de nouveau y entre.** Un protocole sans
procédure d'accueil est une spécification que personne n'applique ; une
procédure d'accueil sans protocole est un mode d'emploi sans machine. Ce
dépôt publie les deux ensemble parce que c'est le même objet vu par les deux
bouts.

## Le protocole, en une image

```text
REQUEST → ACK → RESPONSE → CLOSE
```

Types optionnels : `STATUS`, `CONFLICT`, `DECISION_PROPOSAL`,
`DECISION_CONFIRMATION`. Chaque message porte un identifiant immuable, un
`correlation_id`, un horodatage UTC réel, un indicateur de validation
humaine — et surtout `responds_to_message_id`, le champ qui referme le cycle
et rend le dépôt entier navigable a posteriori. **Aucun agent ne peut
convertir sa propre recommandation en décision humaine** : c'est écrit dans
le protocole, pas seulement dans une charte à côté.

## Ce que le dépôt contient

| Groupe | Fichiers | Ce qu'il couvre |
|---|---|---|
| **Le protocole lui-même** | `CONVERSATION_PROTOCOL_V1.1.md`, `MESSAGE_SCHEMA_V0.1.yaml` | Le cycle de vie des messages, le format, les chemins canoniques |
| **Les politiques de coopération** | `POLITIQUE_COOPERATION_INTER_AGENTS.md`, `POLITIQUE_COMMUNICATION.md`, `PARALLEL_EXECUTION_POLICY_V0.1.md`, `THREAD_PARALLEL_EXECUTION_POLICY_V0.1.md`, `PARALLELISM_MAXIMIZATION_RULE_V1.0.md`, `MULTI_SESSION_COORDINATION_GUIDE_V1.0.md` | Qui écrit où, comment répartir le travail entre sessions sans collision, comment paralléliser sans se marcher dessus |
| **La charte de comportement** | `CLAUDE_BEHAVIOR_CHARTER_V1.1.md` (+ `_EN.md`) | Ce qu'un agent s'engage à faire et ne pas faire dans ce système — le socle du human-in-the-loop |
| **Les rôles** | `ROLE_DES_BOTS_AAAA_OS_20260904.md` | Qui fait quoi entre les bots et les sessions, et une leçon opérationnelle sur les secrets en historique Git (voir ci-dessous) |
| **L'accueil, en trois langues** | `PREMIER_MESSAGE_C2C_COLLEGUE_20260905.md` (+ `_SQ.md`, albanais), `COLLEAGUE_ONBOARDING_PROCESS_V1.1.md` (+ `_EN.md`), `TUTORIEL_COLLEGUES_20260905.md` (+ `_EN.md`), `TUTORIEL_DEPLOIEMENT_CANAUX_20260903.md`, `GUIDE_CANAUX_COMMUNICATION.md`, `PILOTER_SA_SESSION_CLAUDE_EN_C2C_20260905.md`, `TUTO_COOPERATION_PARALLELE.md` | Comment une personne novice envoie son premier message et reçoit une réponse |

### Une règle vaut d'être lue avant les autres : **R10**

`POLITIQUE_COOPERATION_INTER_AGENTS.md` porte dix règles, chacune écrite comme
un incident daté plutôt que comme un principe. **La dixième est celle qui
manquait le plus longtemps**, et elle est née d'une phrase de l'humain du
projet, le 11 septembre 2026 :

> *« informez-vous entre vous de ce genre de choses — je ne dois pas être votre
> gateway belt assistant de coordination »*

La cause n'était pas un manque de bonne volonté entre les sessions. **Il
n'existait aucun endroit partagé où vivaient les arbitrages ouverts.** Chaque
session tenait son propre compte, les comptes divergeaient — une session en a
annoncé quatre puis trois le même jour — et le seul point où ils se
réconciliaient était l'humain. Il était devenu le registre partagé que personne
n'avait écrit.

R10 pose ce registre : un fichier unique, quatre états
(`OPEN` / `DECIDED` / `EXTERNAL_DECISION` / `CLOSED`), la **preuve** de la
décision comme champ obligatoire, et une interdiction qui tient tout le reste —
**une session peut créer, enrichir, reclasser et proposer une clôture, mais ne
clôt jamais.**

Elle porte aussi sa propre limite, écrite dedans : *une convention ne protège
rien tant qu'elle n'est pas dans le chemin d'exécution*. Le registre a été
enfreint par son auteur moins de deux heures après sa rédaction, et ce n'est
pas lui qui l'a détecté — c'est une autre session.

## Le tutorat, pas une métaphore

`PREMIER_MESSAGE_C2C_COLLEGUE_20260905.md` tient en huit gestes, et pose un
principe qui n'est pas courant dans ce genre de documentation :

> **Le critère de réussite est unique** : à la fin, une autre personne vous
> répond. Pas « le fichier est créé », pas « la commande a affiché un
> chemin ». Quelqu'un vous répond.

Une commande qui réussit et un message qui reste sans réponse ont la même
sortie technique (code 0) et ne veulent rien dire de la même chose. C'est le
défaut structurel que ce protocole essaie de rendre visible plutôt que de
cacher derrière un statut vert — et c'est pour ça que l'accueil est publié
comme partie du protocole, pas comme un addendum.

Le même document existe en français, en anglais **et en albanais**, langue
qu'on ne trouve pas d'ordinaire dans ce genre de dépôt. Ce n'est pas un
argument de forme : publier un protocole de coopération *dans une petite
langue européenne* est la démonstration littérale de ce qu'« international »
veut dire ici, pas une déclaration d'intention.

## Où ça se situe par rapport à A2A et MCP

Recherche sourcée du 05/09/2026 : A2A (Google → Linux Foundation, fusionné avec
ACP, co-réside avec MCP sous l'Agentic AI Foundation depuis fin août 2026) est
pensé pour des **agents-services permanents**, avec des points d'accès HTTP
exposés en continu. MCP couvre un axe différent, agent-vers-outil, pas
agent-vers-agent.

Ce protocole vise un cas que ni l'un ni l'autre ne couvre bien : des sessions
qui n'existent que par intermittence, sans service à exposer, où le dépôt Git
lui-même fait office de boîte aux lettres persistante. Ce n'est pas une
critique d'A2A — c'est un choix de transport pour un contexte différent.

**La preuve que ce choix n'est pas une bizarrerie isolée** : AAMP
([github.com/larksuite/aamp](https://github.com/larksuite/aamp)), publié par
Larksuite (ByteDance), est un protocole ouvert indépendant construit sur
exactement le même paradigme — boîte aux lettres asynchrone pour agents sans
webhook public, seulement sur email (SMTP/JMAP) plutôt que sur Git. Un acteur
industriel a standardisé, de son côté, le même problème avec la même
réponse. Un README qui cite son propre antécédent externe se fait croire ;
un README qui prétend avoir tout inventé se fait fermer.

**Sources** : [a2a-protocol.org/specification](https://a2a-protocol.org/specification) ·
[Wikipedia, Agent2Agent Protocol](https://en.wikipedia.org/wiki/Agent2Agent_Protocol) ·
[a2aproject/A2A](https://github.com/a2aproject/A2A) ·
[Linux Foundation AI & Data, ACP joins A2A](https://lfaidata.foundation/blog/2025/08/29/linux-foundation-ai-data-announces-agentic-ai-foundation/) ·
[a2a-protocol.org/topics/a2a-and-mcp](https://a2a-protocol.org/latest/topics/a2a-and-mcp/) ·
[github.com/larksuite/aamp](https://github.com/larksuite/aamp).

## Ce qui a été substitué avant publication

Passe de substitution complète le 07/09/2026 : 21 fichiers, 0 refus au
garde-fou de publication (`garde-fou-publication.py`), vérifiée
indépendamment deux fois — rapport complet dans `LOT2_SUBSTITUTION_20260907.md`
côté dépôt de travail (non inclus ici, c'est un document de suivi, pas un
document du protocole).

Les originaux nomment les machines et services internes du projet. Ici,
chaque nom a été remplacé par **une désignation fonctionnelle** (« le serveur
de production », « le serveur secondaire »), jamais par un trou : le lecteur
comprend le rôle de ce dont on parle. Les récits et dates réels sont intacts.

## About the author

These documents were written while running the infrastructure of an
international cooperation project.

**I am currently open to opportunities.** My field is European funding and
international cooperation: fundraising, development economics, territorial
development, internationalisation and IRO work, project development and
project management — in universities, research centres, TVET schools, NGOs,
and public or private organisations. Programme-wise: Erasmus+ and Horizon
Europe, and beyond them GCF, GEF and United Nations agencies. I am equally
open to academic positions (professor, assistant professor, lecturer).

What I bring: an MSc in data engineering and data science with full-stack
AI and DevOps practice; expertise in TVET and higher education; hands-on
experience as an **evaluator and assessor for Horizon Europe and Erasmus+**;
a track record of writing proposals for calls; a doctorate in agrifood; and
an active network of higher education institutions.

If something fits — or if you know someone it might fit — write to
**jfvenier2@hotmail.com**. Forwarding it counts too.

## A note on language — read this before you judge the repository

**The README, the message schema and three documents are in English. The other
fourteen are in French.** That is not an oversight and it is not a work in
progress we would rather you did not notice.

These documents were written in French because they were **used** in French,
every day, by the people they were written for. Translating them is under way
and will land in a later version. Until then, saying so plainly is worth more
than shipping fourteen machine translations of operational procedures — a
deployment tutorial whose commands were translated is worse than one you have
to read with a dictionary.

**Available in English**

| document | what it is |
|---|---|
| `CLAUDE_BEHAVIOR_CHARTER_V1.1_EN.md` | what an agent may and may not do on its own |
| `COLLEAGUE_ONBOARDING_PROCESS_V1.1_EN.md` | bringing a new human into the protocol |
| `TUTORIEL_COLLEGUES_20260905_EN.md` | the colleague-facing walkthrough |
| `MESSAGE_SCHEMA_V0.1.yaml` | the message format itself — language-neutral |

**French only, for now**: the cooperation policies, the channel guides, the
deployment tutorial, the parallel-execution rules, the bot roles. One document
also exists in Albanian (`_SQ`), because part of the group reads Albanian —
it is not a translation artefact, it is a document with its own readers.

**Where a translation exists, the French text remains the one that holds.**
Where a translation had to choose between two readings, the original is what we
actually meant.

If you need one of the French documents in English before we get to it, open an
issue naming it — we will prioritise what someone actually asks for rather than
what happens to be next in the list.

## Licence

Deux licences, même logique que le premier lot publié de ce projet :
- **MIT** (`LICENSE`) couvre `MESSAGE_SCHEMA_V0.1.yaml`, le seul artefact
  structuré/réutilisable tel quel de ce dépôt.
- **CC BY 4.0** (`LICENSE-CONTENT`) couvre les vingt documents de protocole,
  politique et tutoriel : c'est du texte, pas du code exécutable, et CC BY
  exige l'attribution — ce qui fait que la paternité revient explicitement
  à ses auteurs, pas seulement implicitement.
