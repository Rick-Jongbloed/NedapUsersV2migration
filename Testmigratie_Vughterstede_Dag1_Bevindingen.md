# Testmigratie Vughterstede — Dag 1 Bevindingen
**Datum:** 18 mei 2026  
**Uitvoerder:** Rick Jongbloed  
**Omgeving:** Testomgeving Vughterstede (vughterstede.helloid.com)  
**Duur sessie:** ~3 uur  
**Status:** Migratie uitgevoerd, schedules hersteld, 2 bugs gerapporteerd aan development

---

## Voorbereiding — wat ging vooraf anders dan gepland

| Punt | Wat er gebeurde | Actie |
|------|-----------------|-------|
| Testomgeving | Niet klaargezet vóór vakantie | Om 07:30 dezelfde ochtend alsnog opgezet |
| Certificaat testomgeving | Niet aangevraagd | André gevraagd → certificaat aangeleverd en geplaatst tijdens sessie |
| Rooster-sync | Bleek al **niet te draaien sinds 24 april** | Direct toegevoegd, geïmporteerd en enforcement gedraaid |

> **Checklist-aanvulling:** Voeg toe aan voorbereiding: controleer of rooster-synchronisatie actief is vóór de migratiedag.

---

## Stap 0 — Testomgeving gelijkstellen aan productie

1. Vanochtend verse kopie van productie → testomgeving gemaakt (zelfde dag, 07:30).
2. **Schedules uitgeschakeld** om ongewenste wijzigingen tijdens migratie te voorkomen.
3. Na het kopiëren twee problemen opgelost:
   - Eén rol die al niet meer in NEDAP bestond, stond nog uitgedeeld → afgenomen.
   - Naamswijziging in NEDAP zorgde voor een dubbele rol in HelloID → afgenomen, opnieuw gekoppeld en uitgedeeld.
4. Rooster-sync toegevoegd (zie voorbereiding). Één medewerker met aggregatieprobleem (geboortedatum ontbrak tussen AFAS en AFAS-multi) → AFAS-bron als leading gekozen en snapshot aangemaakt.

---

## Stap 1 — Alle NEDAP-entitlements dupliceren naar testkoppeling

- In HelloID → Business Rules → filter op NEDAP Ons Users en NEDAP Ons Users Test, status "Draft" + "Published" aan, "None" uit.
- Elk entitlement dat in de productieregel zat, ook aan de testconnector gekoppeld.
- Na voltooiing: enforcement gedraaid om alles gelijk te trekken.

---

## Stap 2 — Business rules controleren op inconsistenties

AI-check uitgevoerd na het exporteren van de business rules. Resultaat:

### Gevonden probleem: ontbrekende conditie "druppelnummer"
- **Business rule:** `BR Toegangsgroep Medewerker Medicatie Elisabeth`
- **Probleem:** Geen conditie aanwezig om te controleren of medewerker een geldig druppelnummer heeft.
- **Gevolg:** 5 medewerkers zonder druppelnummer kregen groepen uitgedeeld terwijl ze geen account hebben.
- **Actie:** Conditie toegevoegd (druppelnummer verplicht) → business rule bijgewerkt.

### Overige inconsistenties (AI-melding, ter info voor klant)
- Naamconflicten in entitlements (bijv. "kwaliteitsverpleegkundige" vs. "nieuw kwaliteitsverpleegkundige") — zie Bijlage A.
- Manager Zorg & Welzijn: naam gewijzigd in NEDAP, inconsistentie opgelost.

> **Aanbeveling:** Maak na de migratie een business rule die medewerkers **zonder geldig druppelnummer** identificeert, zodat de klant dit structureel kan bewaken.

---

## Stap 3 — Pre-migratie: Reference Cleaner draaien op V1-systeem (check op blocking issues)

Vóór de eigenlijke migratie naar V2 is de Reference Cleaner **eerst op de bestaande V1-koppeling gedraaid** om te controleren of er blokkerende issues zijn.

**Resultaat:** 3 wachtende acties gevonden:
1. Testaccount → unmanage uitgevoerd.
2. Excluded account met gekoppeld entry-account → unmanage + uit excluded gehaald.
3. Medewerkers zonder druppelnummer die toch groepen kregen → zie Stap 2 (business rule gecorrigeerd).

Na het oplossen van de wachtende acties: Reference Cleaner gestopt, migratie naar V2 gestart (Stap 4).

> **Handleiding-aanvulling:** Zorg dat het scherm "Waiting Actions" leeg is vóór je de Reference Cleaner start. Controleer ook de achterliggende oorzaak (business rule-fout vs. tijdelijk synchronisatieprobleem). De pre-migratie check op V1 is een waardevolle stap om verrassingen na de migratie te minimaliseren.

---

## Stap 4 — Migratie naar V2-connector uitvoeren

### 4a. Field mapping
1. In HelloID → systeem → Delete All (bestaande mapping verwijderen).
2. Field mapping downloaden van GitHub (branch: `nedap-new-permission-permissions-api-standards`).
3. Importeren → geen verdere instellingen nodig.

### 4b. Account-scripts
- Account script kopiëren vanuit repo (geen klantspecifieke mapping aanwezig → kan direct over).
- Import-script plaatsen.
- **Let op bij klanten met aangepaste mapping in het accountscript:** controleer of de versie klantspecifiek is vóórdat je overschrijft.

### 4c. Connector-configuratie
Handmatig vergelijken (JSON-volgorde verschilt → automatische diff gaf geen goed resultaat). Controleer per parameter:

| Parameter | Waarde / Actie |
|-----------|----------------|
| Certificaatpad | Controleren op juistheid |
| Certificaatwachtwoord | Controleren |
| **CSV delimiter** | In V1 heet dit `csv delimiter`; in V2 was het per abuis `mapping csv delimiter` → **Rudolf heeft dit teruggedraaid naar `csv delimiter`** zodat configuratie meemigreeert |
| **Grant default scope myself** | Instellen op **FALSE** (tenzij klant dit expliciet anders wil) |
| Environment | Wijzigen naar `Acceptatie` / `Staging` voor test |

> **Aandachtspunt voor handleiding:** Noteer expliciet dat `Grant default scope myself` op **FALSE** moet staan. Verifieer dit bij iedere klant.

### 4d. Certificaat
- Staging-certificaat bij André opvragen → per mail ontvangen.
- `.pfx`-bestand plaatsen in de `test/certificaat`-map op de server (bestaand bestand overschrijven).
- Preview → verbinding succesvol.

### 4e. Permission-scripts (Default Scope)
1. Permission-script downloaden vanuit repo.
2. In HelloID: "Use script to import permissions" aanvinken.
3. Eval → apply.
4. **Default Scope legacy instellen:**
   - Oude Default Scope entitlement **uitvinken** in business rules.
   - Sync uitvoeren op de entitlements.
   - Nieuwe `Default Scope (legacy)` entitlement **aanvinken** in business rules.
   - **Volgorde is belangrijk** — doe sync vóór publish om te voorkomen dat de Reference Cleaner problemen krijgt.
5. Permission definitie bijwerken: Contact Data Storage inschakelen.

### 4f. Overige scripts
- Resources-script plaatsen en preview draaien.
- Role-script plaatsen.
- Handle All Actions-script plaatsen.
- Correlation instellen: **External ID → NEDAP Ons Identification Number**.

### 4g. Migratie afronden
- Controleren of alle vinkjes staan: Create, Update, Delete (Account), Permission, Default Scope, Role, Resource Cache.
- "Complete Migration" bevestigen.
- **Schedules blijven handmatig uitgeschakeld** tot Reference Cleaner is gedraaid.

---

## Stap 5 — Reference Cleaner draaien

### Bug #1 — Reference Cleaner hangt op "Finalizing"

**Symptoom:** Reference Cleaner start, detecteert differences, maar blijft hangen op "Finalizing" en komt nooit verder.

**Oorzaak:** Bij het migreren maakt HelloID een kopie van de V1-connector (de "disabled copy"). Deze disabled kopie heeft **dezelfde Permission Definition ID's** als de nieuwe V2-connector. De Reference Cleaner stuit op deze dubbele ID's en loopt vast.

**Workaround (bevestigd werkend):**
1. Ga naar Provisioning → zoek de disabled V1-kopie (staat als "read-only / disabled").
2. Verwijder deze disabled kopie handmatig.
3. Start daarna de Reference Cleaner opnieuw.

> **Let op:** De V1-backup is toch al als read-only beschikbaar voor 90 dagen. De disabled kopie in dit scherm is een ándere, technische kopie die probleem veroorzaakt.

> **Fix in volgende release:** Dit wordt in een komende release (verwacht ~3–4 weken na 18 mei) structureel opgelost. Na de fix hoeft de disabled kopie niet meer handmatig verwijderd te worden.

### Reference Cleaner succesvol gedraaid

Na verwijderen van de disabled kopie:
- Determine Differences → resultaat: alleen `Discipline Name` en `Discipline Full ID` worden retained (verwacht gedrag).
- Stop Cleaner.
- Sync gedraaid → uitroeptekentjes verdwenen.

---

## Bekende bugs (gerapporteerd aan development)

### Bug #1 — Reference Cleaner hangt op "Finalizing" na migratie
- **Status:** Gemeld, workaround beschikbaar (zie Stap 5).
- **Verwachte fix:** Komende release (~3–4 weken).

### Bug #2 — Nieuwe Entitlement View toont gemigreerde permissies als leeg / "isNull"
- **Symptoom:** Na migratie en Reference Cleaner staan entitlements in het nieuwe Entitlement Overview als "isNull" of zonder gekoppelde business rules.
- **Werkelijke situatie:** Business rules zijn wél correct gekoppeld. Dit is uitsluitend een weergaveprobleem in de nieuwe UI.
- **Verificatie:** Ga naar de business rule zelf → entitlement staat correct aangekruist. Of: ga naar een specifiek entitlement → business rules zijn zichtbaar.
- **Oorzaak:** Het Entitlement Overview wordt alleen geüpdatet wanneer een business rule handmatig wordt bewerkt. Bij een migratie (die een kopie maakt) wordt dit overzicht niet automatisch vernieuwd.
- **Workaround:** Geen actie nodig — alles werkt correct. Het overzicht corrigeert zichzelf zodra een business rule wordt bewerkt.
- **Status:** Gemeld bij development, fix in komende release (~3–4 weken).

### Bug #3 — Case sensitivity mismatch in permission reference-code (C#)
- **Symptoom:** Als de permission reference in de code niet exact dezelfde hoofdlettergebruik heeft als in HelloID (bijv. `defaultscope` in code vs. `DefaultScope` in HelloID), kunnen **zowel een revoke als een grant** worden getriggerd voor dezelfde permissie.
- **Oorzaak:** De code doet een case-insensitive vergelijking, maar HelloID beschouwt permissie-references als case-sensitive.
- **Impact voor migratie:** Treedt op wanneer bij de Default Scope-stap de oude permissie niet eerst via "Unmanage" is afgehaald vóór publicatie.
- **Fix:** Rudolf implementeert `StringComparison.OrdinalIgnoreCase` (of equivalent) in alle switch statements en contains-checks in de grant- én revoke-logica van het permission-script.
- **Status:** Rudolf is hiervan op de hoogte. Fix in komende release.

---

## Belangrijke procesobservaties voor de handleiding

### Unmanage vóór publicatie bij wijzigen permissie-definitie
Wanneer je een permissie-definitie vervangt (bijv. Default Scope → Default Scope Legacy):
1. Vink de oude permissie **uit** in alle relevante business rules.
2. Voer een **Unmanage** uit op de permissie (niet alleen uitvinken).
3. Voeg daarna de nieuwe permissie toe.
4. Publiceer.

Als je Unmanage vergeet, probeert het systeem de oude permissie alsnog in te trekken — met onvoorspelbare resultaten zolang Bug #3 nog niet gefixed is.

### Force update accounts vóór permissie-updates
Bij een nieuwe run na de migratie: draai eerst een **force update op alle accounts** vóórdat permissie-acties worden verwerkt. Anders probeert het systeem permissies te verwerken voor accounts waarvan de referentie nog niet is bijgewerkt.

### Schedules
- Uitschakelen bij start migratie.
- Pas weer inschakelen **nadat** Reference Cleaner succesvol is afgerond.
- Bevestig herinschakeling aan de klant.

---

## Actiepunten na dag 1

| # | Actie | Eigenaar | Status |
|---|-------|----------|--------|
| 1 | Groepen "kwaliteitsverpleegkundige" afnemen en opnieuw uitdelen (naamconflict oplossen) | Frank (klant) | Open |
| 2 | Business rule aanmaken voor medewerkers zonder geldig druppelnummer | Frank (klant) | Open |
| 3 | Case-sensitive checks implementeren in permission-reference code | Rudolf | In behandeling |
| 4 | Bug ticket aanmaken voor case sensitivity issue | Rudolf | In behandeling |
| 5 | PowerShell CSV-exportscript aanpassen — kolomnamen zijn gewijzigd in V2 | Rick | Open |
| 6 | CSV-bestanden splitsen (test vs. productie) vóór productiedatum 28 mei | Rick | Open |
| 7 | Verificatie uitvoeren na release van fix voor Bug #1 en #2 | Rick | Open |
| 8 | TeamViewer configureren op server van Linda voor remote toegang | Linda / Rick | Open |
| 9 | Schedules herinschakelen op testomgeving → **gedaan einde dag** | Rick | ✅ Gedaan |
| 10 | Migratiehandleiding aanvullen met bevindingen uit dag 1 | Rick | Open |

---

## Bijlage A — HelloID waarschuwingen bij start sessie

Bij het openen van HelloID stond een **oranje driehoek-waarschuwing**. Aanpak:
- Filter op "In role: none" uitschakelen → waarschuwing verdwijnt.
- Er bleken twee groepen in HelloID te staan die niet meer bestonden in NEDAP, doordat de naam was gewijzigd.

**Achtergrond (relevant voor andere klanten):**  
De oude V1-connector slaat het groeps-ID op als `displaynaam + nummer`. Wanneer de naam in NEDAP wordt gewijzigd, verandert het ID in HelloID mee — waardoor bestaande koppelingen breken. Dit probleem speelt bij veel klanten. De V2-connector heeft dit niet meer (ID is onafhankelijk van de naam).

**Tijdelijke oplossing (V1):**
1. Niet-bestaande groepen afnemen in HelloID.
2. Enforcement draaien.
3. Nieuwe groepen (met gewijzigde naam) uitdelen.
4. Force update gebruiken.

**Definitieve oplossing:** Migreren naar V2-connector (dit project).

---

## Volgende stappen

- **Woensdag (21 mei):** Migratieprocedure nog één keer doorlopen op testomgeving ter verificatie; procedure aanscherpen.
- **28 mei:** Productieomgeving Vughterstede migreren — zelfde dag test én productie.
- **Na release (~3–4 weken):** Bug #1 en #2 verifiëren in preview-omgeving.
