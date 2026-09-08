# Dërgoni mesazhin tuaj të parë C2C — udhërrëfyesi, tetë hapa

> Përkthim në shqipen standarde i dokumentit frëngjisht të së njëjtës ditë,
> `docs/PREMIER_MESSAGE_C2C_COLLEGUE_20260905.md` (05/09/2026). Në rast
> mospërputhjeje, teksti frëngjisht ka përparësi. Komandat, emrat e skedarëve
> dhe emrat e variablave nuk janë përkthyer: shtypini saktësisht siç janë.
>
> *(Traduction albanaise du document français du même jour ; en cas de
> divergence, le texte français fait foi.)*

> **Kujt i drejtohet kjo fletë.** Juve, kolegut që sapo jeni bashkuar.
> Jo personit që ju pret.
>
> **Çfarë nuk është.** Nuk është një shpjegim se si funksionon sistemi —
> `c2c-os/00_manifest/CONVERSATION_PROTOCOL_V1.1.md` e bën këtë shumë mirë
> dhe do ta lexoni më vonë. Është vargu i veprimeve në fund të të cilit
> **dikush do të ketë marrë një mesazh të shkruar nga ju**. Llogaritni njëzet
> minuta.
>
> **Kriteri i suksesit është një i vetëm**: në fund, një person tjetër ju
> përgjigjet. Jo « skedari u krijua », jo « komanda shfaqi një shteg ».
> Dikush ju përgjigjet.
>
> Shkruar më 05/09/2026 për seancën e mirëseardhjes. Plotëson hapin 4 të
> `c2c-os/00_manifest/COLLEAGUE_ONBOARDING_PROCESS_V1.1.md`, i cili thotë
> **çfarë** të lexoni; kjo fletë thotë **çfarë të shtypni**.

---

## Përpara se të filloni — dy gjërat që askush nuk mund t'i bëjë në vendin tuaj

1. **Qasje për shkrim (write access) në depon GitHub.** Kërkojeni, dhe
   verifikoni që funksionon përpara se të vazhdoni: `git clone`, pastaj
   `git push` i një ndryshimi të parëndësishëm. Nëse `push` dështon, ndaluni
   këtu — gjithçka tjetër varet prej tij.
2. **Git i instaluar, dhe një terminal që kupton `bash`.** Në Windows, Git
   Bash mjafton; PowerShell nuk do ta ekzekutojë skriptin e hapit 6.

**Si t'i lexoni blloqet e komandave.** Blloqet e komandave janë kopjuar nga
origjinali pa asnjë ndryshim; pjesët midis `<...>` janë vende për t'u
zëvendësuar (placeholders) dhe janë në frëngjisht. Kuptimi i tyre:

| Në komandë | Kuptimi |
|---|---|
| `<url-du-depot>` | adresa (URL) e depos |
| `<votre-identifiant>` | identifikuesi juaj, i zgjedhur në hapin 2 |
| `<destinataire>` | identifikuesi i personit të cilit i shkruani |
| `<dossier-du-depot>` | dosja ku u klonua depoja |
| `/chemin/absolu/vers/votre/depot` | shtegu absolut i asaj dosjeje |
| `<votre role en cinq mots>` | roli juaj, me pesë fjalë |
| `<un mot cle, ex. architecture_contributor>` | një fjalë kyçe, p.sh. `architecture_contributor` |
| `<le meme titre qu'au-dessus>` | i njëjti titull si më sipër |
| `<le meme role>` | i njëjti rol |

---

## 1. Klononi depon

```
git clone <url-du-depot>
cd <dossier-du-depot>
```

Shënoni shtegun absolut (absolute path) të kësaj dosjeje. Do t'ju duhet në çdo
hap, dhe është burimi numër një i gabimeve.

## 2. Zgjidhni identifikuesin tuaj të bisedës

Konventa: `conv-<qui-vous-etes>-<numero>` (pra `conv-<kush-jeni>-<numër>`),
me shkronja të vogla, pa shenja diakritike — pra pa **ë** dhe pa **ç** — dhe
pa hapësira. Për shembull `conv-architecture-tirana-001`.

**Ky identifikues ju përfaqëson në të gjithë sistemin.** Do të shfaqet në çdo
mesazh, emërton kutinë tuaj postare, dhe nuk ndryshohet më pas pa i prishur
përgjigjet. Merrni tridhjetë sekonda më shumë për ta zgjedhur.

Një masë kujdesi që ka rëndësi: **një identifikues është një e dhënë që
qarkullon.** Mos vendosni në të mbiemrin tuaj, punëdhënësin tuaj ose ndonjë
numër, nëse nuk doni që ato të shfaqen në mesazhe që lexohen nga agjentë të
tjerë.

## 3. Krijoni kutinë tuaj postare (mailbox)

Tri drejtori dhe dy skedarë. Nga rrënja e depos:

```
mkdir -p c2c-os/03_handoffs/mailboxes/<votre-identifiant>/inbox
mkdir -p c2c-os/03_handoffs/mailboxes/<votre-identifiant>/archive
```

Pastaj një `CAPABILITIES.yaml` në `c2c-os/03_handoffs/mailboxes/<votre-identifiant>/`,
sipas këtij modeli — kopjoni atë të një kutie ekzistuese dhe përshtateni, në
vend që të nisni nga zeroja:

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

`status: NEW_UNVERIFIED` është i qëllimshëm: **kutia juaj nuk është e
verifikuar për sa kohë që askush nuk ka marrë një mesazh nga ju.** Hapi 8 është
ai që e verifikon, jo krijimi i saj.

## 4. Deklarohuni në regjistër

Hapni `c2c-os/02_operational_registers/conversation_registry.yaml` dhe shtoni
hyrjen tuaj në listën `conversations:`, duke kopjuar formatin e një hyrjeje
fqinje:

```yaml
  - conversation_id: <votre-identifiant>
    title: <le meme titre qu'au-dessus>
    project_id: AAAA-OS  # publication-ok: nom-interne
    role: <le meme role>
    status: NEW_UNVERIFIED
```

**Ky hap nuk është dekorativ.** Një identitet që mungon në regjistër refuzohet
si marrës nga mjetet që verifikojnë — mesazhi niset dhe nuk mbërrin kurrë.
Rasti ndodhi më 03/09 me gjashtë kuti grupi që ekzistonin prej pesë ditësh:
leximi funksiononte, shkrimi refuzohej, dhe askush nuk e dinte.

## 5. Deklaroni ku ndodheni

Dy variabla (variables), në terminalin ku do të punoni:

```
export REPO=/chemin/absolu/vers/votre/depot
export MOI=<votre-identifiant>
```

**`REPO` nuk është fakultativ.** Vlera e tij e paracaktuar në skript tregon
makinën e dikujt tjetër. Pa këtë `export`, skripti do t'ju thotë se depoja nuk
gjendet — ky **nuk** është problem identifikuesi, ndryshe nga ç'linte të
kuptohej një version i mëparshëm i udhëzuesit.

Këto dy `export` zgjatin aq sa dritarja e terminalit. Nëse hapni një tjetër,
bëjini përsëri.

## 6. Shkruani trupin e mesazhit tuaj

Një skedar Markdown i zakonshëm, **pa kokë YAML (YAML header)** — skripti e
shton vetë. Tri ose katër rreshta mjaftojnë:

```
cat > /tmp/presentation.md <<'FIN'
Bonjour, je rejoins le groupe architecture.

Ce sur quoi je peux aider : <deux lignes concretes>.
Ce dont j'ai besoin pour demarrer : <une ligne>.
FIN
```

Rreshtat midis `<<'FIN'` dhe `FIN` janë përmbajtja e mesazhit tuaj; shembulli
më sipër është në frëngjisht dhe thotë: « Përshëndetje, po i bashkohem grupit
të arkitekturës. Ku mund të ndihmoj: <dy rreshta konkretë>. Çfarë më duhet për
të filluar: <një rresht>. » Zëvendësojeni me tekstin tuaj.

Shkruani diçka të vërtetë. Ky mesazh do të lexohet nga persona dhe nga
agjentë, dhe do të mbetet në historikun e depos.

## 7. Depozitoni mesazhin

```
ops/deposer-message-c2c.sh <destinataire> STATUS PRESENTATION "Presentation" /tmp/presentation.md
```

Skripti shfaq shtegun e skedarit të shkruar dhe diferencën midis vulës kohore
(timestamp) të deklaruar dhe orës reale. Është bërë që **të dështojë me zë të
lartë**: numër i gabuar argumentesh, trup mesazhi që nuk gjendet, kuti e
marrësit që nuk ekziston, vulë kohore e papajtueshme — çdo dështim ka mesazhin
e vet.

**Por një sukses këtu nuk do të thotë se mesazhi u nis.** Do të thotë se një
skedar ekziston në diskun tuaj.

## 8. Publikoni — është ky hapi që e dërgon

```
git add c2c-os/
git commit -m "C2C: presentation de <votre-identifiant>"
git pull --rebase origin main
git push origin main
```

**Kutia postare kanonike është GitHub.** Për sa kohë që `push` nuk ka
ndodhur, marrësi nuk sheh asgjë, dhe as agjentët që lexojnë depon — ngarkuesi
i tyre i kontekstit (context loader) lexon nga GitHub, jo nga disku juaj.

Është hapi që harrohet, sepse hapi 7 ngjan me një dërgim. Një mesazh që ka
mbetur në diskun e autorit të vet, me një mesazh suksesi në ekran, është
defekti më i kushtueshëm i këtij protokolli: askush nuk e sheh, as dërguesi që
beson se ka shkruar, as marrësi që nuk pret asgjë.

---

## Si ta dini që funksionoi

**Prova nuk është te ju, por te tjetri.** Verifikoni në këtë radhë:

1. Në GitHub, skedari juaj shfaqet në
   `c2c-os/03_handoffs/mailboxes/<destinataire>/inbox/` në degën `main`.
   Nëse nuk është aty, `push` juaj nuk ka përfunduar — rilexoni daljen e tij.
2. **Dikush ju përgjigjet.** Është i vetmi kriter që ka rëndësi. Kërkojini
   personit të cilit i keni shkruar që ta konfirmojë, dhe mos e konsideroni
   hapin të kryer përpara kësaj.

Nëse nuk vjen asgjë pas një kohe të arsyeshme, problemi është pothuajse
gjithmonë një nga këto tri gjëra, sipas kësaj radhe shpeshtësie: `push` nuk
ka ndodhur; identifikuesi i marrësit është shkruar gabim; identiteti juaj nuk
është në regjistër (hapi 4).

---

## Çfarë nuk bëni ende, dhe kjo është normale

Tani dini të shkruani në një kuti postare. Nuk dini ende t'i përgjigjeni një
mesazhi të llojit REQUEST, as të mbani regjistrin e mesazheve, as të bëni një
agjent të flasë në vendin tuaj — dhe kjo nuk është urgjenca.

**Radha është e qëllimshme: së pari kanali, pastaj agjenti.** Një person që
largohet me një agjent bisedues, por pa ditur të shkruajë në një kuti postare,
largohet me diçka që bën përshtypje dhe nuk prodhon asgjë. Vazhdimi lexohet
në `c2c-os/00_manifest/CONVERSATION_PROTOCOL_V1.1.md`, kur t'ju duhet — jo
më parë.

---

## Pikat e pasigurta të përkthimit

Për t'u rilexuar nga një folës shqip përpara ose gjatë seancës. Të renditura
nga më e rëndësishmja te më pak e rëndësishmja.

1. **Vendet për t'u zëvendësuar (`<votre-identifiant>`, `<destinataire>`,
   `/chemin/absolu/...`) janë lënë në frëngjisht brenda blloqeve të
   komandave.** Rregulli i përkthimit ishte që blloqet të mos preken fare.
   Rreziku: një koleg mund t'i shtypë fjalë për fjalë. Tabela « Si t'i lexoni
   blloqet e komandave » në krye është shtuar pikërisht për këtë; ajo nuk
   ekziston në origjinal.

2. **Shembulli i mesazhit në hapin 6 është në frëngjisht** (« Bonjour, je
   rejoins le groupe architecture… »), për të njëjtën arsye. Është shtuar një
   fjali shpjeguese pas bllokut, që nuk ekziston në origjinal. Dokumenti nuk
   thotë në cilën gjuhë duhet shkruar mesazhi i vërtetë; nuk e kam shtuar as
   unë.

3. **« pa shenja diakritike — pra pa ë dhe pa ç »** për frëngjishten « sans
   accent ». Largim i qëllimshëm nga fjalë për fjalë: në shqip, ë dhe ç janë
   shkronja të alfabetit, jo thekse, dhe një përkthim fjalë për fjalë
   (« pa theks ») nuk do ta paralajmëronte lexuesin se pikërisht këto dy
   shkronja janë kurthi. Kuptimi teknik (vetëm a–z, shifra dhe vizë) ruhet.

4. **« identifikues »** për « identifiant ». Termi i fjalorit; në të folur,
   kolegët e IT-së thonë shpesh « ID ». Nuk ekzistonte në përkthimin e djeshëm
   (njoftimi i grupit nuk përmbante asnjë identifikues teknik), pra është
   zgjedhje e re. Alternativë: « ID e bisedës ».

5. **« kuti postare (mailbox) »** — përkthim fjalë për fjalë. Në aplikacionet
   e postës elektronike në shqip shihet « kutia hyrëse » për *inbox*; këtu
   « kuti postare » mbulon të gjithë dosjen (inbox + archive), prandaj u
   preferua. Emri i drejtorisë `mailboxes` ndihmon.

6. **« depo (repository) »** — i njëjti dyshim si dje: « depo » mund të
   kuptohet si « magazinë » nga një lexues jo-informatikan. Ruajtur për
   koherencë me njoftimin e 04/09.

7. **« variabla (variables) »** — forma puriste e standardit është
   « ndryshore »; në përdorimin informatik mbizotëron « variabël ». Zgjodha
   të dytën, sepse kolegët do ta shohin fjalën `export` pranë.

8. **« drejtori »** për « répertoire » dhe **« dosje »** për « dossier ».
   Në praktikë shqipfolësit thonë edhe « direktori » ose « folder ». Dallimi
   drejtori/dosje është i origjinalit, jo i imi.

9. **« kokë YAML (YAML header) »** — « kokë » përdoret në termat e uebit
   (« koka e faqes »), por për *front matter* nuk ka term të ngulitur.
   Alternativë: « krye YAML ».

10. **« të dështojë me zë të lartë »** — kalk i « échouer bruyamment / fail
    loudly ». Kuptimi (dështon dukshëm, me mesazh, në vend që të heshtë) ruhet,
    por shprehja mund të tingëllojë e huaj. Alternativë: « të dështojë hapur
    dhe me mesazh ».

11. **« Prova nuk është te ju, por te tjetri »** për « La preuve n'est pas de
    votre côté ». Përkthimi fjalë për fjalë (« nga ana juaj ») rrezikonte të
    lexohej si « prova nuk është në favorin tuaj ». Shtova « por te tjetri »
    për ta bërë kuptimin të qartë; është një gjysmë fjalie më shumë se
    origjinali.

12. **« Deklarohuni në regjistër »** — « Regjistrohuni » do të ishte më e
    natyrshme, por mund të kuptohet si « krijoni një llogari ». Mbajta
    « deklarohuni », që ndjek origjinalin.

13. **« shteg absolut (absolute path) »** — « shteg » është termi i
    përdorur në lokalizimet shqipe; disa thonë « rrugë » ose thjesht « path ».

14. **« agjent »** për « agent (IA) ». Njoftimi i djeshëm përktheu « bot » me
    « robot (bots) »; ky dokument flet për agjentë, jo për botë, prandaj
    « agjent » — që është edhe termi i zakonshëm (« agjent i inteligjencës
    artificiale »). Nuk ka konflikt me zgjedhjen e djeshme.

15. **« udhërrëfyes »** për « feuille de route » dhe **« hapa »** për
    « gestes ». « Gjeste » do të ishte fjalë për fjalë, por në shqip do të
    kuptohej si lëvizje e trupit; « hapa » është ajo që lexuesi pret në një
    procedurë të numëruar.

16. **Lakimi i emrave të huaj** (« GitHub », « REQUEST »). Për të shmangur
    formën « GitHub-in / GitHubin » (dje u zgjodh « Johnit », pa vizë), i
    riformulova fjalitë me parafjalë (« lexon nga GitHub », « një mesazhi të
    llojit REQUEST »). Asnjë emër i huaj nuk lakohet në këtë dokument.

17. **Thonjëzat « … »** janë ato të origjinalit frëngjisht; standardi shqip
    pranon edhe „…" . Data është lënë në formatin 05/09/2026 të origjinalit.

18. **Regjistri i përgjithshëm** — standard letrar, pa forma dialektale, me
    « ju » të mirësjelljes nga fillimi në fund, si dje.

---

## Points de traduction incertains (équivalent français de la section ci-dessus)

À faire relire par un locuteur albanais avant ou pendant la séance. Classés du plus au moins gênant.

1. **Les placeholders (`<votre-identifiant>`, `<destinataire>`, `/chemin/absolu/...`) sont restés en français dans les blocs de commande**, par respect de la consigne « rien à traduire dedans ». Risque : un collègue les tape littéralement. La table « Comment lire les blocs de commande » en tête du document a été ajoutée pour ça ; elle n'existe pas dans l'original.

2. **Le corps du message exemple de l'étape 6 est en français** (« Bonjour, je rejoins le groupe architecture… »), pour la même raison. Une phrase explicative a été ajoutée après le bloc, absente de l'original. Le document ne dit pas dans quelle langue écrire le vrai message ; je ne l'ai pas ajouté non plus.

3. **« pa shenja diakritike — pra pa ë dhe pa ç »** pour « sans accent ». Écart volontaire : en albanais, ë et ç sont des lettres de l'alphabet, pas des accents, et une traduction littérale (« pa theks ») n'avertirait pas le lecteur que ce sont précisément ces deux lettres qui sont le piège. Le sens technique (a–z, chiffres, tiret) est conservé.

4. **« identifikues »** pour « identifiant » : le terme de dictionnaire ; à l'oral, les informaticiens disent souvent « ID ». Absent de la traduction d'hier (l'annonce ne contenait aucun identifiant technique), donc choix nouveau. Alternative : « ID e bisedës ».

5. **« kuti postare (mailbox) »** — littéral. Les clients mail albanais affichent « kutia hyrëse » pour *inbox* ; ici « kuti postare » couvre tout le dossier (inbox + archive), d'où la préférence. Le nom du répertoire `mailboxes` aide.

6. **« depo (repository) »** — même doute qu'hier (peut se lire « entrepôt »). Conservé pour cohérence avec l'annonce du 04/09.

7. **« variabla (variables) »** — la forme puriste du standard est « ndryshore » ; l'usage informatique dit « variabël ». J'ai pris la seconde, le mot `export` étant juste à côté.

8. **« drejtori »** pour « répertoire », **« dosje »** pour « dossier ». En pratique on entend aussi « direktori » ou « folder ». La distinction répertoire/dossier est celle de l'original.

9. **« kokë YAML (YAML header) »** — « kokë » sert pour les termes web (« koka e faqes »), mais il n'y a pas de terme établi pour *front matter*. Alternative : « krye YAML ».

10. **« të dështojë me zë të lartë »** — calque de « échouer bruyamment ». Le sens (échec visible et messagé, pas silencieux) est conservé, mais la tournure peut sonner étrangère. Alternative : « të dështojë hapur dhe me mesazh ».

11. **« Prova nuk është te ju, por te tjetri »** pour « La preuve n'est pas de votre côté ». Le littéral (« nga ana juaj ») risquait de se lire « la preuve n'est pas en votre faveur ». J'ai ajouté « por te tjetri » (mais chez l'autre) pour lever l'ambiguïté ; c'est une demi-phrase de plus que l'original.

12. **« Deklarohuni në regjistër »** — « Regjistrohuni » serait plus naturel mais peut se comprendre « créez un compte ». Gardé « deklarohuni », qui suit l'original.

13. **« shteg absolut (absolute path) »** — « shteg » est le terme des localisations albanaises ; certains disent « rrugë » ou simplement « path ».

14. **« agjent »** pour « agent (IA) ». L'annonce d'hier traduisait « bot » par « robot (bots) » ; ce document parle d'agents, pas de bots, d'où « agjent », qui est aussi le terme usuel. Pas de conflit avec le choix d'hier.

15. **« udhërrëfyes »** pour « feuille de route » et **« hapa »** (étapes) pour « gestes ». « Gjeste » serait littéral mais se comprendrait comme un mouvement du corps ; « hapa » est ce qu'un lecteur attend dans une procédure numérotée.

16. **Déclinaison des noms étrangers** (« GitHub », « REQUEST »). Pour éviter « GitHub-in / GitHubin » (hier on avait choisi « Johnit », sans trait d'union), j'ai reformulé avec des prépositions (« lexon nga GitHub », « një mesazhi të llojit REQUEST »). Aucun nom étranger n'est décliné dans ce document.

17. **Les guillemets « … »** sont ceux de l'original ; le standard albanais accepte aussi „…". La date garde le format 05/09/2026 de l'original.

18. **Registre général** — standard littéraire, sans formes dialectales, vouvoiement « ju » d'un bout à l'autre, comme hier.
