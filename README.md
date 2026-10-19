# VVB – Mailspring-plugin

Gemensam grund för VVB:s utveckling av ett Mailspring-plugin, med kodkonventioner,
ärendehantering, granskning och ett kontrollerat PR-flöde för människor och botkonton.
Processreglerna nedan gäller alla bidrag, även AI-genererad kod.

## Status och avgränsning

Repositoryt innehåller ännu Mailsprings exempelplugin, inte en färdig VVB-produkt.
Knappen i meddelanderedigeraren visar en dialog och ändrar utkastets ämnesrad.
Sidopanelen visar den fokuserade kontaktens namn eller en platshållare.
Paketnamnet är fortfarande `sample`, versionen `0.1.0`.
VVB:s verksamhetsfunktioner, integrationer och slutliga paketidentitet ska beslutas
i separata ärenden med acceptanskriterier innan de implementeras.

Grunden omfattar:

- Gemensamma kod-, säkerhets- och granskningsregler i denna beskrivning.
- Inställningar för editorer i [`.editorconfig`](.editorconfig).
- Formulär för fel, förbättringar och tekniska uppgifter i
  [`.github/ISSUE_TEMPLATE/`](.github/ISSUE_TEMPLATE/).
- [PR-mall](.github/pull_request_template.txt) och [kodägare](.github/CODEOWNERS).
- [Byggkontroll](.github/workflows/ci.yml) och
  [Dependabot-konfiguration](.github/dependabot.yml).
- En [importerbar grenregelmall](.github/rulesets/vvb-default-branch.json).
  **Filen aktiverar inte något skydd på GitHub på egen hand.**

## Struktur och tekniska ramar

| Sökväg | Ansvar |
| --- | --- |
| `src/main.js` | Pluginets livscykel och registrering av komponenter. |
| `src/*.jsx` | React-komponenter och anslutning till Mailsprings stores. |
| `styles/main.less` | LESS-stilar med Mailsprings UI-variabler och mixins. |
| `lib/` | Genererad ES2017/CommonJS-kod som distribueras med pluginet. |
| `package.json`, `package-lock.json` | Paketmetadata, byggkommando och låsta beroenden. |
| `tsconfig.json` | Befintlig kompilering av JavaScript/JSX med TypeScript. |
| `.github/` | Ärende- och PR-flöde, kodägarskap, CI och regelmall. |

Ändra källkod i `src/`, inte manuellt i `lib/`. Bygg och checka in motsvarande
`lib/`-ändringar i samma PR: Mailspring transpilerar inte pluginet vid installation.
Checka aldrig in `node_modules/`, lokala loggar eller kontodata.
Skapa nya kataloger först när en konkret funktion behöver dem.

### Kompatibilitet och arkitektur

- Behåll ES2017/CommonJS och befintlig JSX-kompilering tills en separat migrering
  har godkänts. CI använder Node.js 22 för byggverktygen; det är inte samma sak som
  Mailsprings inbyggda Electron-/Node-miljö.
- `package.json` anger Mailspring `>=1.6.3`. Dokumentera den faktiskt manuellt
  testade versionen och operativsystemet; metadata är inte ett kompatibilitetstest.
- Använd Mailsprings exporter för React, komponentregister och stores.
  Inför inte en andra React-runtime eller en ny arkitektur utan motivering.
- Registrera i `activate()` och avregistrera i `deactivate()`. Frigör listeners,
  timers och andra resurser när komponenter avmonteras eller pluginet stängs av.
- Hantera tom kontakt, saknat utkast och upprepade monteringar. Undvik onödiga
  omrenderingar när utkast ändras vid varje tangenttryckning.
- Beroenden ska vara motiverade och granskas för licens, säkerhet och kompatibilitet.
  Uppdatera låsfilen med npm, inte för hand. Native-moduler ingår inte i ramen.
- Lägg bara till `windowTypes` för fönster där funktionalitet finns.
  `syncInit: true` kräver ett dokumenterat behov, eftersom det påverkar uppstarten.

## Lokal utveckling och verifiering

Använd Node.js 22 och npm. Från repositoryts rot:

```sh
npm ci --ignore-scripts --no-audit --no-fund
npm run build
git diff -- lib/
```

Installationen använder den befintliga låsfilen och kör inga installationsskript.
Det finns ännu **ingen lint- eller automatiserad testsvit**, bara `npm run build`.
Inför tester när testinfrastruktur eller ny testbar logik tillkommer; påstå inte
att byggkontrollen testar Mailsprings beteende.

Installera projektmappen via Mailsprings **Install a Plugin...**, eller länka den
till Mailsprings `packages`-katalog. På macOS ligger den under
`~/Library/Application Support/Mailspring/packages`. För andra operativsystem,
leta upp datakatalogen via **Developer > Show Mailsync Logs**.
Bygg först och starta sedan om Mailspring eller ladda om huvudfönstret i
utvecklarverktygen för att se ändringar.

### Manuell kontroll före PR

Använd ett testkonto och syntetiska meddelanden:

1. Öppna huvudfönstret och ett separat composer-fönster; kontrollera att
   komponenterna registreras utan fel.
2. Klicka på exempelknappen i ett testutkast: dialogen ska visas och ämnet ändras
   med ` - It Worked!`. Skicka inte ett riktigt meddelande som del av testet.
3. Växla mellan kontakter och ett läge utan kontakt. Kontrollera namn respektive
   platshållare i sidopanelen.
4. Inaktivera och återaktivera pluginet. Kontrollera att komponenter och
   prenumerationer inte dubbleras.
5. Notera Mailspring-version, OS, resultat och eventuella begränsningar i PR:n.

CI kör installation, bygg och kontroll av att `lib/` är uppdaterad, även för
dokumentations-PR:er så att samma obligatoriska statuskontroll alltid kan köras.
Checken heter exakt **`VVB build`**. Den körs för PR:er och kan startas manuellt.
Gröna checks ersätter inte manuell kontroll eller mänsklig granskning.

## Kodkonventioner

- UTF-8, LF, två mellanslag och avslutande radbrytning enligt `.editorconfig`.
- JavaScript/JSX: semikolon, `const` som standard och `let` vid ombindning.
  Använd camelCase för variabler/metoder, PascalCase för komponentklasser och
  kebab-case för komponentfiler.
- Befintlig kod har både enkla och dubbla citattecken. Följ den berörda filens
  stil; blanda inte funktionella ändringar med generell omformatering.
- Behåll befintliga klasskomponenter och namnge `displayName` unikt.
  Definiera relevanta `propTypes` och håll renderingen fri från sidoeffekter.
- Använd plugin-specifika CSS-klasser så att Mailsprings övriga UI inte påverkas.
  Återanvänd UI-variabler i stället för hårdkodade temafärger.
- Namn i kod och tekniska commit-typer är på engelska. Ärenden, PR-beskrivningar
  och VVB-dokumentation får vara på svenska; håll terminologin konsekvent.
- Kommentarer förklarar varför, inte självklar syntax. Dokumentera ändrat
  beteende, kompatibilitet och verifiering tillsammans med ändringen.
- Små, sammanhållna ändringar prioriteras. Nya verktyg, breda refaktoreringar och
  ändringar av beroenden eller pluginets behörigheter kräver uttrycklig motivering.

## Säkerhet och VVB-data

E-post, kontaktuppgifter, bilagor och utkast ska behandlas som skyddsvärda data.
Ingen funktion får lägga till nätverksanrop, telemetri, extern AI-bearbetning eller
beständig lagring av sådana data utan beskrivet ändamål, dataminimering och
uttryckligt godkännande från ansvarig förvaltare.

- Lägg inte lösenord, API-nycklar, tokens, kundmeddelanden eller personuppgifter
  i kod, ärenden, PR:er, loggar, skärmbilder eller testdata.
- Validera externa värden före filåtkomst, Electron-anrop och rendering.
  Använd inte `eval` eller osanitiserad HTML för meddelandeinnehåll.
- Varken CI eller botar ska få tillgång till e-postkonton eller produktionsdata.
  Använd syntetiska exempel och maskera känsliga uppgifter.
- Misstänkta hemligheter ska spärras/roteras omedelbart; att bara radera dem i en
  senare commit räcker inte.

### Rapportera en sårbarhet

Publicera inte exploateringsdetaljer eller känsliga data i ett vanligt ärende.
Använd repositoryts **Security > Advisories > Report a vulnerability** om privat
rapportering är aktiverad. Saknas funktionen, be `@kristianlyckebris` om en privat
kontaktväg utan att dela detaljer offentligt. Administratören ska aktivera privat
sårbarhetsrapportering före extern lansering och ange en förvaltad kontaktväg.
Ingen svarstid eller supportorganisation utlovas innan VVB har beslutat om det.

## Ärenden, grenar och PR-flöde

1. **Beskriv:** använd fel-, förbättrings- eller uppgiftsformuläret. Beskriv behov,
   avgränsning och verifierbara acceptanskriterier. Förvaltaren prioriterar och
   utser ansvarig; säkerhetsärenden hanteras privat enligt ovan.
2. **Förankra:** större arkitektur-, data- och beroendeändringar godkänns i ärendet
   innan implementation. Håll ett huvudsyfte per PR.
3. **Skapa gren:** utgå från aktuell standardgren. Använd
   `feat/<issue>-<kort-namn>`, `fix/<issue>-<kort-namn>`,
   `docs/<issue>-<kort-namn>` eller `chore/<issue>-<kort-namn>`.
   Botverktygens `dependabot/...` och `copilot/...` är tillåtna undantag.
4. **Implementera:** använd små commits, exempelvis
   `fix: handle missing contact` eller `docs: clarify VVB review flow`.
   Rekommenderade typer är `feat`, `fix`, `docs`, `test`, `refactor`, `build`,
   `ci` och `chore`. Namn- och titelregler är konventioner, inte automatiska spärrar.
5. **Öppna utkast-PR:** fyll i mallen, länka ärendet (`Closes #...` endast när det
   faktiskt löses), beskriv risk och återställning samt redovisa utförda kontroller.
6. **Granska:** gör PR:n redo när bygget och egenkontrollen är klara. Kodägaren
   granskar funktion, scope, datahantering, beroenden och genererad `lib/`.
   Besvara kommentarer och lös diskussioner; nya ändringar kräver ny granskning.
7. **Slå ihop:** en behörig människa godkänner och sammanfogar med squash först när
   obligatoriska kontroller och oberoende granskning är klara. PR-författaren får
   inte godkänna sin egen PR. Radera arbetsgrenen efter merge.
8. **Följ upp:** verifiera ändringen i Mailspring. Vid regression, skapa en
   återställnings-PR som återför både källa och byggresultat; force-pusha inte
   standardgrenen.

En PR är klar när acceptanskriterierna är uppfyllda, scope och dokumentation
stämmer, relevanta manuella kontroller är redovisade, `lib/` matchar källan,
CI är grön och inga säkerhets- eller granskningsfrågor återstår.

## Ansvar och botkonton

`@kristianlyckebris` är initial kodägare enligt `CODEOWNERS`. Det innebär en
granskningsväg, inte att en VVB-organisation, ett team eller ett botkonto redan
har skapats. VVB ska utse förvaltare och minst en oberoende granskare med rätt
behörighet innan strikt kodägargranskning aktiveras.

| Roll | Ansvar och gräns |
| --- | --- |
| Förvaltare/kodägare | Prioriterar, granskar risk och beslutar om merge/release. |
| Bidragsgivare | Äger ändringen, verifierar och följer upp även AI-genererad kod. |
| Dependabot | Föreslår npm- och Actions-uppdateringar månadsvis; ingen automatisk merge. |
| Copilot/annan godkänd bot | Arbetar i separat gren och PR med samma krav som människor. |
| Administratör | Hanterar åtkomst, regler, privata rapporter och incidenter. |

- Föredra en separat GitHub App framför delade personkonton. Ge bara åtkomst
  till detta repository och de rättigheter som uppdraget kräver.
- Dokumentera ägare, syfte, åtkomst och avveckling innan ytterligare botar
  installeras. Använd inte en namngiven VVB-bot som inte faktiskt finns.
- Botar får inte kringgå grenregler, godkänna egna PR:er eller självständigt
  göra release. AI-granskning ersätter inte mänskligt godkännande.
- CI har bara `contents: read`, inga hemligheter och sparar inga Git-credentials.
  Kör aldrig obetrodd PR-kod med `pull_request_target` och skrivtoken.
- Externa bot-PR:er och workflowändringar granskas särskilt. Granska dependency-
  diff, leveranskedja och låsfil; botens identitet är inget säkerhetsbevis.

## Aktivera grenregler och GitHub-inställningar

**Administratörsåtgärder – inte automatiskt genomförda av denna PR:**

1. Kontrollera att GitHub-planen/repositorytypen stöder rulesets och att
   `CODEOWNERS` pekar på verkliga användare eller team med skrivbehörighet.
   Lägg till en oberoende kodägare: om enda kodägaren själv öppnar PR:n kan
   obligatorisk kodägargranskning annars blockera merge.
2. Kör CI på en PR så att `VVB build` finns som statuskontroll.
3. Öppna **Settings > Rules > Rulesets > New ruleset > Import a ruleset** och
   importera `.github/rulesets/vvb-default-branch.json`. Kontrollera målgrenen
   (mallen använder `~DEFAULT_BRANCH`) och spara/aktivera regeln.
4. Regeln kräver PR, minst ett kodägargodkännande, ny granskning efter ändringar,
   godkännande av någon annan än senaste pusharen, lösta diskussioner,
   aktuell gren och godkänd `VVB build`. Den blockerar radering, force-push och
   merge-commits på standardgrenen. Ingen bypass-aktör är förkonfigurerad.
5. Tillåt **Squash merging**, stäng av övriga merge-metoder och aktivera automatisk
   radering av arbetsgrenar under **Settings > General > Pull Requests**.
   Låt automatisk merge vara avstängd enligt kravet på mänsklig merge.
6. Sätt Actions standardtoken till läsbehörighet, granska tillåtna Actions och
   kräv godkännande för externa bidragsgivares workflow-körningar.
   Aktivera Dependabot alerts och privat sårbarhetsrapportering där de stöds.
7. Verifiera med en test-PR att röd CI och saknad granskning faktiskt blockerar
   merge och att en korrekt granskad PR kan squash-mergas. Dokumentera eventuella
   planbegränsningar eller avvikelser i ett uppföljningsärende.

Regler för grennamn, commit-typer, manuella kontroller och botars arbetssätt är
processkrav. Endast aktiverade GitHub-regler och checks är tekniskt tvingande.
Inför inte merge queue utan att först lägga till stöd för `merge_group` i CI.
Publicering och automatisk release ingår inte i denna grund.

## Mailspring-resurser

- [Plugin-dokumentation](https://foundry376.github.io/Mailspring/) (delvis äldre).
- [Mailsprings interna pluginer](https://github.com/Foundry376/Mailspring/tree/master/app/internal_packages)
  ger exempel på värdappens API och integrationsmönster.
