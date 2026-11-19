# SafetyScan Clip-Lok International Demo v1.0 — Leverance

Build: `a484b00c31` · Base: `johnfinmann-ctrl/SafetyScan` @ `c70d82b` (index.html, md5 `05b1f9cd…ec`)

## 1–2. Filer (separat projekt — upload hele mappen til et NYT repo)

```
index.html               komplet app (single-file, ingen eksterne biblioteker)
manifest.webmanifest     PWA-manifest
icons/                   apple-touch-icon.png (180), icon-192/512, icon-maskable-512, favicon-16/32, favicon.ico
INTEGRATION.md           integrationsdokumentation + kodelister
example-payload.json     eksempel på JSON-payload
LEVERANCE.md             denne fil
```
Cache-busting: alle ikon- og manifest-referencer har `?v=<build-hash>`, som skiftes automatisk ved hvert build.
localStorage bruger eget præfiks `sscli_v1_` og deler derfor ikke data med SafetyScan, SafetyScan International eller Clip-Lok-demoen (samme GitHub Pages-origin).

## 3. Sprogaudit — 10 sprog

Metode: (a) statisk audit af alle `t()`/`tf()`/`data-i18n`-nøgler + dynamiske nøglefamilier; (b) automatisk runtime-audit i Chrome: 15 skærme/dialoger pr. sprog (forside, registrering m. AI-boks, kvittering, Mine rapporter, dashboard m. filtre, rapportdetalje, statistik, admin, 3 masterdatalister, instrukser, maildialog, tildel-dialog, luk-dialog). Alle synlige tekster, `<option>` og placeholders blev sammenlignet med dansk og scannet for rå nøgler/interne koder.

| Sprog | Kode | Tale-locale | Brugte nøgler dækket | Rå nøgler/koder vist | Tekst identisk med DA* |
|---|---|---|---|---|---|
| Dansk (standard) | DA | da-DK | 477/477 | 0 | — |
| Engelsk | EN | en-GB | 477/477 | 0 | kun fælles ord (Truck, Dashboard, HSE, måneder) |
| Svensk | SV | sv-SE | 477/477 | 0 | kun fælles ord |
| Norsk | NO | nb-NO | 477/477 | 0 | kun fælles ord |
| Tysk | DE | de-DE | 477/477 | 0 | kun fælles ord (Lagerautomat, Rampe, Objekt, måneder) |
| Polsk | PL | pl-PL | 477/477 | 0 | 0 (ud over leverandørnavne/«Maj») |
| Ukrainsk | UK | uk-UA | 477/477 | 0 | 0 (ud over leverandørnavne) |
| Estisk | ET | et-EE | 477/477 | 0 | 0 (ud over fælles ord: Objekt, Kvartal, måneder) |
| Lettisk | LV | lv-LV | 477/477 | 0 | 0 (ud over «Marts») |
| Litauisk | LT | lt-LT | 477/477 | 0 | 0 (ud over leverandørnavne) |

\* Efter frasortering af data (rapport-ID'er, brugerens fritekst, navne, e-mails) og sprognavne i sprogvælgeren.

Dækket: startside, navigation, ny registrering, kamera/galleri, GPS, mikrofon/tale-til-tekst (inkl. 5 lokaliserede fejlbeskeder), hændelsestyper, objekter, lokationer, prioritet, status, kommentarer, ansvarlig, leverandør, dashboard, filtre, statistik/mønstre/trends, Mine rapporter, rapportdetalje, historik, mail, rapportdeling, instrukser (6 instrukser × 7 tekster), administration/masterdata, demo-reset, fejlbeskeder, toasts, bekræftelser, HTML-rapport/print, CSV-kolonner, demo-banner.

Bevidst ikke oversat: brugerens fritekst (kommentarer, navne), admin-oprettede stamdatanavne, brandlinjen «Spot it. Report it. Prevent it.» (fast slogan fra kravspecifikationen), «SafetyScan»-navnet.
Note: 172 ældre nøgler fra basen, som ikke vises nogen steder i UI'et, findes kun i DA/EN/SV/NO/DE (bevaret uændret som dormant kode).

Hardcode-audit: 0 hardcodede danske strenge tilbage i JS-logikken (kommentarer, legacy-mapping og nøgleordslister undtaget). Basens danske fallback-strenge (`t('x') || '…'`) er fjernet.

## 6. Kodeliste
Se `INTEGRATION.md` §3 (status, prioritet, type, objekt, funktion, handling, historik, lokation).

## 7. PWA / CL-appikon — verificeret
- Navy baggrund `#0f2340`, blåt «CL» `#2b7de9`, ingen øvrig tekst (`icons/apple-touch-icon.png` 180×180, fuld firkant — iOS runder selv).
- `apple-touch-icon`, `favicon` (16/32/ico), `manifest` (192, 512, maskable 512), `theme-color`, `apple-mobile-web-app-title` = «SafetyScan CL».
- Det gamle «S»-standardikon er væk (basen havde ingen ikon-referencer, så iOS genererede et «S»).
- Manifest valideret som JSON; ikoner ligger som filer (ikke data-URI), hvilket er den mest pålidelige løsning på iPhone/iPad.
- **Kræver at mappen `icons/` og `manifest.webmanifest` uploades sammen med `index.html`.**

## 8. Testmatrix

| Platform | Metode | Resultat |
|---|---|---|
| iPhone (Safari-profil, 390×844) | Automatisk regressionstest i Chromium med iPhone 13-emulering | 48/48 PASS |
| iPad (iPad Pro 11-profil) | Samme | 48/48 PASS |
| Android Chrome (Pixel 7-profil) | Samme | 48/48 PASS |
| Chrome desktop (1440×900) | Samme | 48/48 PASS |
| Edge desktop | Samme Chromium-motor som Chrome — ikke separat kørt | forventet = Chrome |
| Ægte iPhone/iPad Safari (WebKit) | **Ikke testet** — WebKit kunne ikke installeres i testmiljøet | skal testes manuelt |

Regressionstesten dækker: ny registrering, billede, kamera/galleri-input, GPS (frivillig), manuel hændelsestid, mikrofon (locale pr. sprog + fallback uden talegenkendelse), AI-forslag (også på polske nøgleord), brugervalg vinder over AI, lokation, prioritet, status (5 trin + historik), dashboard + KPI'er + filtre, statistik, CSV, Mine rapporter, kommentar, ansvarlig (funktionskode), leverandør, mail (modtager/CC/emne/tekst), rapport-HTML/deling, instrukser, admin/masterdata, luk-flow med efterfoto-politik, slet alle hændelser, gendan demo, sprogskift i åben detalje/tildel-dialog/maildialog/formular uden reload, åbning af gammel rapport uden schema_version, JSON-export med alle 28 felter, `sendToExternalSystem` i demo-mode, 0 JS-fejl.

Manuel tjekliste på ægte enheder: «Føj til hjemmeskærm» viser CL-ikonet (iPhone + iPad) · GPS-tilladelsesdialog (kræver https — virker på GitHub Pages, ikke via file://) · tale-til-tekst på de 10 sprog (afhænger af enhed; ved manglende understøttelse vises lokaliseret besked) · «Del rapport» via share sheet.

## 9. Original SafetyScan IKKE ændret
`johnfinmann-ctrl/SafetyScan` (`c70d82b`), `SafetyScan-International` (`394c847`) og `SafetyScan-Clip-Lok` (`e5cf9b5`) er kun klonet read-only. `git status` er tom i alle tre, og md5 af SafetyScan/index.html er uændret. Der er ikke pushet noget.

## 10. Ingen kunde-backend/integration implementeret
Ingen Supabase, intet rigtigt login, ingen backend, ingen cloud-database, ingen automatisk mailserver, ingen SAP-/Microsoft-/ERP-integration, ingen ekstern API-forbindelse. Appen laver ingen netværkskald (ud over valgfrit «Åbn kort»-link til OpenStreetMap og wa.me/mailto, som brugeren selv aktiverer — som i basen).

## Rettede fejl fra basen (nødvendige for at funktionerne virker)
- Rapport-generering/deling kastede `ReferenceError: ACTIVE_LANG` → rettet.
- «Send til»-modtagerlisten blev aldrig udfyldt → rettet.
- AI's prioritetsforslag blev aldrig gemt (manglende `band`) → rettet.
- Risikomotoren matchede ikke objekt-ID'erne (rack, gate …) → rettet.
- Gemt sprog blev ikke anvendt ved opstart; måneder, «billede/billeder» og luk-knapper viste rå nøgler → rettet.
- Mail brugte formularens modtagere i stedet for rapportens; gemt mailkladde blev vist på forkerte sager → rettet.
- Dashboard viste rå interne værdier (fx «kritisk»), KPI «I gang» var altid 0 → rettet.

## Tilføjet fordi kravspecifikationen krævede det, men basen manglede det
GPS-knap (valgfri), manuel hændelsestid, historik/audit trail, 5-trins statusflow, Lederadgang som forsidekort, AI-kategoriforslag (kun forslag), nøgleord til AI-forslag på alle 10 sprog.
