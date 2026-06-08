# Nedap Ons v1 → v2 — Uitvoeringskaart Testmigratie
**Doelgroep:** IAM Consultant Tools4ever en Tools4ever-partners met ervaring in het implementeren van Nedap Ons connectoren  
**Gebruik:** werk deze kaart stap voor stap af op de testmigratiedag. Vink af, ga door.

---

<details open>
<summary>

## Voorbereiding testomgeving

</summary>

### Minimaal 5 dagen voor start migratie

- [ ] Feature flag v1 connector aangevraagd bij product owner Tools4ever of support Tools4ever
- [ ] Bevestig bij klant dat de testomgeving met ID **TE-XXXX** nog steeds in gebruik is (dezelfde waarvoor eerder een certificaat is aangevraagd)
- [ ] Controleer of het huidige testcertificaat nog geldig is op het geplande uitvoermoment
- [ ] Certificaat vernieuwen indien nodig — **volgorde: eerst upgraden naar versie 7, daarna pas verversen**; aanvraag via **Tools4ever Support**, klant keurt goed in Nedap Podium
- [ ] Certificaat geplaatst in **testmap** op server (overschrijf bestaand certificaat)
- [ ] Klant geïnformeerd: op testmigratiedag worden **geen provisioning-acties** uitgevoerd en is **beheer niet mogelijk** — gebruik hiervoor het [aankondigingssjabloon](Klantcommunicatie_Migratie_Aankondiging.md)
- [ ] **Maatwerk-check** — doorloop de V1-scripts van de klant en beoordeel of er grote afwijkingen zijn van de standaard:
  - Bespreek eventueel maatwerk vóór de migratiedag met de klant. Benadruk het voordeel van de standaardconnector: toekomstige updates zijn direct toepasbaar zonder maatwerk opnieuw te hoeven doorvoeren.
  - Intern bij Tools4ever: maatwerk vereist overleg met de **manager delivery** en mogelijk de **product owner connectoren** vóór uitvoering.
- [ ] Klantcontact uitgevraagd en beschikbaar op testmigratiedag — stel de volgende vragen:
  - Wie is beschikbaar als aanspreekpunt?
  - Wie heeft toegang tot het Excel-naar-CSV-exportscript en kan het opnieuw draaien? *(dit script moet opnieuw gedraaid worden na de migratie, omdat de kolomnamen wijzigen)*
  - Wordt de server beheerd door een externe partij? Zo ja: zorg dat die persoon beschikbaar is op de migratiedag. Geef daarbij aan dat mogelijk op de server ingelogd moet worden om scripts aan te passen.

### Minimaal 3 dagen voor start migratie

- [ ] Klant heeft Nedap Ons testomgeving ververst (verse kopie van Nedap Ons productie naar Nedap Ons testomgeving)
- [ ] Klant bevestigt testomgeving niet te overschrijven totdat de acties van HelloID zijn gecontroleerd
- [ ] Certificaat actief op testomgeving bevestigd

### Minimaal 1 dag voor start migratie

- [ ] Feature flag actief bevestigd
- [ ] Pending actions testomgeving = 0 — controleer en los pending actions op vóór migratiedag
- [ ] PowerShell CSV-exportscript controleren op kolomnamen — controleer of de `locations.csv` en `teams.csv` al de kolommen `HelloIDPrimaryLookupKey` en `HelloIDSecondaryLookupKey` bevatten:
  - **Kolommen al correct** → niets te doen.
  - **Kolommen heten nog `Department.ExternalId` en `Title.ExternalId`** → maak een kopie van het exportscript in de **testmap** op de server en pas dáár de volgende regels aan. ⚠️ Pas het originele script niet aan — de productieconnector draait nog op V1 en heeft dat script nodig.

    **Mapping teams-sectie:**
    ```powershell
    # Vervang:
    Select-Object Department.ExternalId, Title.ExternalId, NedapTeamIds, AllEmployees |
    # Door:
    Select-Object @{Name='HelloIDPrimaryLookupKey'; Expression='Department.ExternalId'}, @{Name='HelloIDSecondaryLookupKey'; Expression='Title.ExternalId'}, NedapTeamIds, AllEmployees |
    ```

    **Mapping locaties-sectie:**
    ```powershell
    # Vervang:
    Select-Object Department.ExternalId, Title.ExternalId, NedapLocationIds, AllClients |
    # Door:
    Select-Object @{Name='HelloIDPrimaryLookupKey'; Expression='Department.ExternalId'}, @{Name='HelloIDSecondaryLookupKey'; Expression='Title.ExternalId'}, NedapLocationIds, AllClients |
    ```

    Zorg daarna dat de uitvoerpaden in het script naar de **testmap** verwijzen.

  > **Als het exportscript in HelloID staat (Admin dashboard → Automation → Tasks):** maak een kopie van het script en pas in de kopie de kolomnamen aan zoals hierboven beschreven. Pas de uitvoerpaden aan zodat de CSV-bestanden in de **testmap** terechtkomen, niet in de productiemap.
- [ ] Zorg dat je toegang hebt tot de connector-repo: [`Nedap-new-permissions-api-standard`](https://github.com/Tools4everBV/HelloID-Conn-Prov-Target-NedapOns-Users/tree/Nedap-new-permissions-api-standard)
- [ ] **Myself-check** — controleer de onderstaande twee instellingen in de V1-scripts en noteer de waarden. Ze worden straks elk afzonderlijk omgezet naar een configuratietoggle in V2.

  | Script | Wat te controleren | Configuratietoggle in V2 |
  |--------|-------------------|--------------------------|
  | Default Scope script | `$IsGrantMySelf` bovenin het script | Grant Default Scope Myself |
  | Roles Handle All Actions-script | `$myself` in functie `Merge-EntitlementToNedapRole` (standaard `$true` — check of overschreven) | Grant 'Myself' to each Role assignment |

  Noteer beide waarden — je hebt ze nodig bij Stap F, punt 4.

### Op migratiedag

- [ ] Draai het exportscript om verse CSV-bestanden te genereren voor de testomgeving en controleer of de bestanden in de testmap staan.

</details>

---

<details open>
<summary>

## A — Start van de dag

</summary>

1. Log in op de HelloID-omgeving van de klant.
2. Zet alle **schedules uit** (handmatig, één voor één). *(Voorkomt ongewenste wijzigingen tijdens de migratie.)*
3. Controleer op uitgedeelde entitlements die niet meer bestaan in Nedap Ons: ga naar **Business → Rules → Entitlements** → filter op Nedap Ons Users → open het filter en zet **In rule → None** en **In target system → No** uit. Zoek naar entitlements met een waarschuwing en los gevonden issues op vóór je verdergaat. *(Bij de klant kan het filter er iets anders uitzien — geen geel uitroepteken en andere systeemnamen. De filterlogica is hetzelfde. Een schone uitgangssituatie voorkomt vervuiling tijdens de migratie.)*

</details>

---

<details open>
<summary>

## B — Testconnector inrichten

</summary>

1. Maak een nieuwe **PowerShell V1 connector** aan via **Provisioning → Systems → Add** en geef deze de naam **Nedap Ons - Users Test** (sla deze stap over als de connector al bestaat).
2. Open de productieconnector **Provisioning → Systems → Nedap Ons - Users**.
3. Kopieer alle **scripts** (Create, Update, Delete, Permission Default Scope, Permission Roles, Resources) en de volledige **configuratie** naar de **Nedap Ons - Users Test** connector via copy-paste per script in de script-editor. *(Je maakt hiermee een exacte kopie van de productieconnector voor de testomgeving.)* Let daarbij op:
   - Controleer dat de **naam van de Default Scope entitlement** exact overeenkomt met de productieconnector — inclusief hoofdlettergebruik.
   - Controleer dat de **correlatie-instellingen** (Tab Correlation) identiek zijn aan de productieconnector.
4. Maak op de server een **testmap** aan (als die er nog niet is) en kopieer daarin:
   - Het **certificaatbestand** (`.pfx`)
   - De **locations mapping** (`locations.csv`)
   - De **teams mapping** (`teams.csv`)
   - De **cache-map** (of maak een lege map aan als startpunt voor de cache)
5. Pas in de testconnector de **configuratie** aan zodat alle paden naar de testmap verwijzen:

   | Parameter | Aanpassen naar |
   |-----------|----------------|
   | Environment / baseUrl | `https://api-staging.ons.io` |
   | Certificaatpad | Pad naar `.pfx` in de **testmap** |
   | mappingLocations | Pad naar `locations.csv` in de **testmap** |
   | mappingTeams | Pad naar `teams.csv` in de **testmap** |
   | DirectoryCacheLocationsTeams | Pad naar de **cache-map** in de testmap |

   > Laat alle **thresholds** op de testconnector staan zoals ze zijn — deze staan standaard al op 1.

</details>

---

<details open>
<summary>

## C — Testomgeving gelijkstellen aan productie

</summary>

1. Ga naar **Business Rules** → filter op Nedap Ons Users **en** Nedap Ons Users Test, status "Draft" + "Published" aan, "None" uit. *(Dit geeft overzicht van alle uitgedeelde entitlements. Noteer het huidige aantal.)*
2. Koppel elk entitlement dat in de productieregel staat ook aan de **testconnector**. Doe dit voor alle Nedap Ons-entitlements. Na voltooiing moet het totaal aantal entitlements **precies het dubbele** zijn van het beginaantal — elk entitlement staat nu zowel op de productie- als de testconnector.
3. Draai een **Enforce +** om alles gelijk te trekken en de cachebestanden te genereren: klik op het pijltje rechts van de **Enforce**-knop en kies **>>+ Enforce** — dit voert de enforcement direct uit. Alle geblokkeerde entitlements kunnen worden doorgezet — komen er fouten uit, dan ligt de oorzaak waarschijnlijk in een verkeerd pad of ontbrekend bestand uit Stap B, punt 5. Controleer in dat geval de configuratie van de testconnector.
4. Controleer de auditlog op fouten na de Enforcement+. Los op wat mogelijk is en **documenteer elke fix** — dezelfde fouten zitten ook in productie. Niet alles hoeft nu opgelost te zijn; pending actions worden afgehandeld in Stap D.
5. Verifieer dat de testconnector gelijk is aan de productieconnector: scripts zijn identiek en alle entitlements in business rules staan op zowel Nedap Ons Users als Nedap Ons Users Test. Dit is het enige criterium — als dit klopt, is de testomgeving gereed voor migratie.

</details>

---

<details open>
<summary>

## D — Pending actions controleren en oplossen

</summary>

> Dit geldt voor de gehele HelloID-omgeving — niet alleen de Nedap Ons connector. Alle pending actions in alle connectoren moeten opgelost zijn voordat je verdergaat.

1. Ga naar **Business → Evaluation** — stop alle enforcements met de status *Running* via de gele **Cancel**-knop.
2. Ga naar **Business → Entitlements → Actions** en los alle openstaande (pending) acties op. Wij adviseren het volgende voor de onderstaande situaties:
   - Testpersonen met accounts → Unmanage
   - Excluded personen met account → Unmanage
   - Groeptoewijzing zonder account: de conditie die in de account business rule staat ontbreekt in de bijbehorende permissie business rule — voeg dezelfde conditie toe. Ga naar **Business → Rules → Entitlements**, zoek het betreffende entitlement op en pas de relevante business rule(s) aan.
   - Andere → klant raadplegen en samen kijken naar een oplossing
3. Controleer: het overzicht onder **Business → Entitlements → Actions** is leeg voordat je verdergaat.

</details>

---

<details open>
<summary>

## E — Pre-migratie Reference Cleaner check (op V1)

</summary>

1. Open de Reference Cleaner: `https://[klantnaam].helloid.com/provisioning/#/reference-cleaner/overview`
2. Lees de waarschuwing en klik dan op de gele knop **Start cleaner**.
3. Wacht tot de statusindicatoren zijn bijgewerkt en beoordeel het resultaat:
   - Alle vinkjes **groen** → ga door naar Stap F.
   - **Enforcement Runs** blijft draaien → er zijn nog pending actions. Klik **Stop cleaner**, los de pending actions op via Stap D en voer daarna Stap E opnieuw uit vanaf het begin.
   - Enforcement Runs is groen maar **andere indicatoren** staan niet op groen → los de gemelde issues op aan de hand van de Reference Cleaner handleiding.
4. Klik **Stop cleaner**. *(Browsertab sluiten stopt de Cleaner NIET — expliciet stoppen verplicht, als je de cleaner niet stopt, zijn alle business rules read-only.)*

</details>

---

<details open>
<summary>

## F — Migratie uitvoeren (scripts en configuratie)

</summary>

> ⚠️ Vanaf dit moment is de migratie onomkeerbaar. Schedules worden automatisch stilgezet. Er kunnen geen schedules of handmatige acties worden uitgevoerd totdat de migratie volledig is afgerond (Stap J).

> HelloID maakt automatisch een read-only kopie van de volledige connector aan. Je kunt deze kopie in een apart venster openen om scripts van de oude V1-connector te vergelijken met de nieuwe versie.

1. Ga naar **Provisioning → Systems → [Nedap Ons Users Test]**.
2. Klik **Migrate system** → bevestig met **Confirm**.
3. **Tab Fields**
   - Vink **Field Configuration** aan in de migratieview.
   - Klik **Delete all** (verwijder huidige field mapping).
   - Download de field mapping van GitHub (branch [`Nedap-new-permissions-api-standard`](https://github.com/Tools4everBV/HelloID-Conn-Prov-Target-NedapOns-Users/tree/Nedap-new-permissions-api-standard)).
   - Importeer de field mapping.

4. **Tab Account**
   > ⚠️ Controleer vóór het overschrijven of het accountscript klantspecifiek is aangepast. Zo ja: kopieer de klantspecifieke mapping over naar het nieuwe script. Gebruik de read-only V1-kopie (zie stap 2) om de scripts naast elkaar te vergelijken.
   >
   > De volgende instellingen waren in V1 hardcoded als variabelen in het script, 