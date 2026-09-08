# Protokolli i Bashkëpunimit Ndërmjet Sesioneve

*[English: [README.md](README.md)]*

> Agjentët mund të shkëmbejnë mesazhe me njëri-tjetrin. Problemi i vështirë
> është ta bësh bashkëpunimin të mbijetojë përtej sesionit.

## Çfarë është ky depo

Një protokoll i hapur për **bashkëpunim mes sesioneve të ndryshme** — mes
agjentëve të inteligjencës artificiale, dhe mes tyre dhe njerëzve që punojnë me
ta.

Nuk është një aplikacion që duhet instaluar. Është një mënyrë e rënë dakord për
t'i shkëmbyer kërkesat, vendimet dhe provat, në mënyrë që **asgjë të mos humbasë
kur një sesion mbyllet dhe një tjetër fillon**.

Gjithçka mbështetet mbi Git: çdo mesazh është një skedar, me dërguesin,
marrësin, përgjigjen dhe mbylljen e vet. Kjo do të thotë se historiku ekziston
me të vërtetë, dhe se askush nuk mund ta ndryshojë atë pa lënë gjurmë.

## Pse ekziston

Sepse e matëm problemin, dhe nuk ishte ai që prisnim.

Mbi **858 çifte kërkesë/përgjigje reale**, koha mesatare e përgjigjes ishte
**53 minuta**. Menduam se faji ishte i kanalit — se duhej një lidhje më e
shpejtë. Por matja tregoi se **transporti përbënte vetëm 0,2 % të vonesës**.

Katër sekonda mbi pesëdhjetë e tre minuta. Kanali nuk ishte kurrë problemi.

Ajo që mungonte ishte **një zile**: një sesion e shihte kutinë e vet vetëm kur
dikush i fliste. Një mesazh që mbërrinte në një sesion të heshtur qëndronte aty
derisa një njeri ta rindizte. Ky protokoll ekziston për ta zgjidhur atë — jo
shpejtësinë, por **mbërritjen, vendimin dhe mbylljen e provuar**.

## Çfarë gjeni brenda

| Dokument | Për çfarë shërben |
|---|---|
| `MESSAGE_SCHEMA_V0.1.yaml` | formati i mesazhit — i pavarur nga gjuha |
| `CLAUDE_BEHAVIOR_CHARTER_V1.1_EN.md` | çfarë lejohet të bëjë një agjent vetëm, dhe çfarë jo |
| `COLLEAGUE_ONBOARDING_PROCESS_V1.1_EN.md` | si futet një person i ri në protokoll |
| `TUTORIEL_COLLEGUES_20260905_EN.md` | udhëzuesi praktik për kolegët |
| `PREMIER_MESSAGE_C2C_COLLEGUE_20260905_SQ.md` | **në shqip** — mesazhi juaj i parë, hap pas hapi |
| `POLITIQUE_COOPERATION_INTER_AGENTS.md` | rregullat e bashkëpunimit (frëngjisht) |
| `TUTORIEL_DEPLOIEMENT_CANAUX_20260903.md` | si vihen në punë kanalet (frëngjisht) |

## Për gjuhën — e themi hapur

**Tre dokumente janë në anglisht, katërmbëdhjetë janë në frëngjisht, dhe një
është në shqip.**

Kjo nuk është një harresë. Këto dokumente u shkruan në frëngjisht sepse u
**përdorën** në frëngjisht, çdo ditë, nga njerëzit për të cilët u shkruan.
Përkthimet janë duke u bërë dhe do të vijnë në një version të ardhshëm.

Deri atëherë preferojmë ta themi sesa të botojmë katërmbëdhjetë përkthime
automatike të procedurave operative. **Një udhëzues vendosjeje ku janë përkthyer
edhe komandat është më i keq se një udhëzues që duhet lexuar me fjalor.**

Dokumenti në shqip nuk është një mbetje përkthimi: ai ekziston sepse një pjesë e
grupit lexon shqip, dhe ka lexuesit e vet.

**Nëse ju duhet një nga dokumentet frëngjisht në shqip ose anglisht përpara se
ne t'ia arrijmë**, hapni një *issue* duke e emërtuar. Do t'i japim përparësi
asaj që dikush kërkon vërtet, jo asaj që vjen radhës në listë.

## Si të filloni

Mos e lexoni të gjithë depon. Filloni me **një mesazh të vetëm**.

1. Hapni `PREMIER_MESSAGE_C2C_COLLEGUE_20260905_SQ.md` — është në shqip dhe ju
   çon hap pas hapi.
2. Shkruani mesazhin tuaj të parë dhe **dërgojeni** me `git push`.
3. Prisni përgjigjen, dhe **mbylleni fillin me shkrim**, qoftë edhe me një fjalë.

Dy gracka që i bien të gjithëve, dhe që po jua themi paraprakisht:

- **Hapi që shkruan nuk është hapi që dërgon.** Derisa `git push` të mos ketë
  ndodhur, askush nuk e sheh mesazhin tuaj. E kemi paguar këtë disa herë.
- **Një refuzim nuk është një gabim.** Do të thotë thjesht se dikush shtyu
  përpara jush. Bëni `git pull --rebase`, pastaj shtyni sërish. **Kurrë
  `--force`.**

## Çfarë nuk e bën ky protokoll

Nuk e ngjesh kontekstin dhe nuk e ruan kujtesën e një agjenti — ajo është një
punë tjetër, e njohur si *cross-session memory*, dhe e bëjnë vegla të tjera.

Ne merremi me gjysmën tjetër: **disa sesione, disa modele, dhe njerëzit e tyre**,
me një gjurmë të verifikueshme se kush kërkoi çfarë, kush vendosi, dhe mbi cilën
provë.

## Licenca

Dy licenca:

- **MIT** (`LICENSE`) për çdo element kodi.
- **CC BY 4.0** (`LICENSE-CONTENT`) për dokumentet — janë tekst, jo kod i
  ekzekutueshëm, dhe CC BY kërkon atribuim.
