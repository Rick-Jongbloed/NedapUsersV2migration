# Nedap Ons v1 → v2 — Proces migratie Nedap Ons Users v1 naar v2 - Testomgeving
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
3. Controleer op uitgedeelde entitlements die niet meer bestaan in Nedap Ons: ga naar **Business → Rules → Entitlements** → filter op Nedap Ons Users → open het filter en zet **In rule → None** en **In target system → No** uit. Zoek naar entitlements met een waarschuwing en los gevonden issues op vóór je verdergaat.

</details>

---

<details open>
<summary>

## B — Testconnector inrichten

</summary>

1. Maak een nieuwe **PowerShell V1 connector** aan via **Provisioning → Systems → Add** en geef deze de naam **Nedap Ons - Users Test**. Als de connector al bestaat, sla de aanmaak dan over — controleer in dat geval wel dat: (a) er geen entitlements uit deze connector zijn gekoppeld in business rules, en (b) alle scripts en configuratie gelijk zijn aan de productieconnector.
2. Open de productieconnector **Provisioning → Systems → Nedap Ons - Users**.
3. Kopieer alle **scripts** (Create, Update, Delete, Permission Default Scope, Permission Roles, Resources) en de volledige **configuratie** naar de **Nedap Ons - Users Test** connector via copy-paste per script in de script-editor. *(Je maakt hiermee een exacte kopie van de productieconnector voor de testomgeving.)* Let daarbij op:
   - Controleer dat de **naam van de Default Scope entitlement** exact overeenkomt met de productieconnector — inclusief hoofdlettergebruik.
4. Maak op de server een **testmap** aan (als die er nog niet is) en kopieer daarin:
   - Het **certificaatbestand** (`.pfx`) van de testomgeving
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

1. Ga naar **Business → Rules → tab Entitlements** → filter op Nedap Ons Users **en** Nedap Ons Users Test, status "Draft" + "Published" aan, "None" uit. *(Dit geeft overzicht van alle uitgedeelde entitlements op basis van de filternamen. Noteer het huidige aantal.)*
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
   - Testaccounts → Unmanage
   - Excluded accounts met account → Unmanage
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
4. Klik **Stop cleaner** zodra alle vinkjes op groen staan. *(Browsertab sluiten stopt de Cleaner NIET — expliciet stoppen verplicht, als je de cleaner niet stopt, zijn alle business rules read-only.)*

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
   > De volgende instellingen waren in V1 hardcoded als variabelen in het script, maar worden in V2 beheerd via de connector-configuratie. Neem deze regels **niet** over uit het V1-script — controleer wel de waarden en stel de bijbehorende toggles in via de configuratie (zie Configuratie hieronder).
   >
   > ```powershell
   > contractRequiredAtLogin = $true
   > ssoEnabled              = $true
   > limitLocationView       = $true
   > passwordChange          = $true
   > ```
   >
   > **Advies bij maatwerk:** Ontdek je behalve in de mapping (grote) verschillen? De beslissing hierover is al gemaakt in de Voorbereiding (Maatwerk-check). Klopt er iets niet of is er iets gemist, overleg dan alsnog vóór je verdergaat.
   >
   > Mocht maatwerk toch noodzakelijk zijn, voer dan eerst de migratie uit met de onbewerkte standaardscripts en rond deze volledig af vóórdat je aanpassingen doorvoert. Het account Create- en Update-script migreren namelijk éénmalig de in HelloID opgeslagen referenties naar de nieuwe versie — het is essentieel dat deze scripts onbewerkt worden uitgevoerd.
   >
   > Twijfels? Neem contact op met Tools4ever support vóór je verdergaat.

   - Vervang **Create script** (uit repo).
   - Vervang **Update script** (uit repo).
   - Vervang **Delete script** (uit repo).
   - Vervang **Data Import script** (uit repo).
   - Vervang **Configuration** — bij een standaard V1→V2 migratie wijzigen de meeste configuratiesleutels niet. Let op de volgende twee uitzonderingen:

     1. **Environment (Rest)** — staat standaard ingesteld op *Production*. Verander dit voor de testflow naar **Acceptance** (dropdown).
     2. Er zijn nu twee aparte toggles voor "myself" — stel beide in op basis van de boolean-check die je hebt uitgevoerd in de Voorbereiding:

        | Configuratieoptie | Gebaseerd op | Instelling |
        |-------------------|--------------|------------|
        | **Grant Default Scope Myself** | `$IsGrantMySelf` bovenin de Default Scope grant en update permission scripts | Aan als `$true`, uit als `$false` |
        | **Grant 'Myself' to each Role assignment** | `$myself` in functieaanroep van `Merge-EntitlementToNedapRole` in het Roles Handle All Actions-script | Aan als `$true`, uit als `$false` |

        Stel beide opties in vóórdat je op Apply drukt.

     Druk daarna altijd éénmaal op **Apply**, ook als er niets is aangepast.

5. **Tab Permissions — Default Scope**
   - Controleer of de logica van het default scope script afwijkt van de onderstaande Tools4ever-standaarden. Gebruik de read-only V1-kopie (zie stap 2) om te vergelijken. Wijkt de klant af, pas het script dan aan vóór je verdergaat.

   | Parameter | Standaardwaarde | Toelichting |
   |-----------|-----------------|-------------|
   | Nedap Ons Identification ID | `Custom.NedapOnsIdentificationNo` | Was configureerbaar in V1, nu hardcoded in script |
   | Team primary lookup key | `{ $_.Department.ExternalId }` | Verplicht — dit veld mag niet leeg zijn in de mapping CSV |
   | Team secondary lookup key | `{ $_.Title.ExternalId }` | Niet verplicht — dit veld mag leeg zijn in de mapping CSV |
   | Location primary lookup key | `{ $_.Department.ExternalId }` | Verplicht — dit veld mag niet leeg zijn in de mapping CSV |
   | Location secondary lookup key | `{ $_.Title.ExternalId }` | Niet verplicht — dit veld mag leeg zijn in de mapping CSV |

   - Zet **Use script to import permissions** aan.
   - Plaats het **Import permission script** (uit repo).
   - Zet **Use separate script for each action** uit.
   - Plaats het **Handle all actions script** (uit repo).
   - Zet **Contact Data Storage** aan.

6. **Tab Permissions — Roles**
   - Plaats het **Import permission script** (uit repo).
   - Klik **Preview** om te controleren, je test nu meteen de certificaatconfiguratie, dan **Apply**.
   - Plaats het **Handle all actions script** (uit repo).

7. **Tab Resources**
   - Vervang het **Resources script** (`resources.ps1`, uit repo).
   - Klik **Preview** om te controleren, dan **Apply**.

8. **Tab Correlation**

   | Instelling | Waarde |
   |------------|--------|
   | Enable correlation | `True` |
   | Person correlation field | `ExternalId` |
   | Account correlation field | `_outputInfo.externalId` |

9. Controleer of je alle vinkjes hebt gezet: Field Configuration ✓ Create ✓ Update ✓ Delete ✓ Permission Default Scope ✓ Permission Role ✓ Resource Cache ✓
10. Klik **Complete Migration** → bevestig.

</details>

---

<details open>
<summary>

## G — Reference Cleaner (post-migratie)

</summary>

1. Open de Reference Cleaner: `https://[klantnaam].helloid.com/provisioning/#/reference-cleaner/overview`
2. Klik **Start cleaner**.
3. Wacht tot de statusindicatoren zijn bijgewerkt en beoordeel het resultaat:
   - Alle vinkjes **groen** → ga door naar stap 4.
   - **Enforcement Runs** blijft draaien → er zijn nog pending actions. Klik **Stop cleaner**, los de pending actions op via Stap D en voer daarna Stap G opnieuw uit vanaf het begin.
   - Enforcement Runs is groen maar **andere indicatoren** staan niet op groen → los de gemelde issues op aan de hand van de Reference Cleaner handleiding.
4. Selecteer de **Roles** Permission Configuration.
5. Klik **Determine differences** — controleer dat `DisplayName` en `DisplayNameFull` in de "to remove"-lijst staan.
6. Klik **Remove fields**.
7. Controleer de **History** rechts — actie gelogd?
8. Klik **Stop cleaner**. *(Browsertab sluiten stopt de Cleaner NIET — expliciet stoppen verplicht, als je de cleaner niet stopt, zijn alle business rules read-only.)*

</details>

---

<details open>
<summary>

## H — Default Scope (legacy) Entitlement instellen

</summary>

> Voer de onderstaande stappen uit voor **alle business rules** waarin de oude Default Scope entitlement is opgenomen. Controleer eerst hoeveel business rules de Default Scope bevatten.

Voer de stappen in exact deze volgorde uit, voor elke business rule afzonderlijk:

1. Zoek de business rules met de **oude Default Scope** entitlement:
   - Ga naar **Business → Rules → tab Entitlements**.
   - Zoek op de naam van de Default Scope entitlement (bijv. "DefaultScope" — afhankelijk van hoe de permissiedefinitie is ingericht).
   - Selecteer het entitlement — rechts onder **Details** verschijnen alle business rules waarin dit entitlement is opgenomen.
   - Open de betreffende business rules via **Ctrl+klik** of **middlemuisklik** op het moersleuteltje om ze in een nieuw venster te openen.
2. Vink de oude Default Scope **uit**.
3. Draai een **Sync** op de entitlements van Nedap Ons Users Test.
4. Vink de nieuwe **Default Scope (legacy)** entitlement **aan**.
5. Publiceer de business rule — kies bij het publiceren voor **Unmanage removed entitlement(s)** voor het oude Default Scope entitlement.

   > ⚠️ Sla de Unmanage-stap niet over. Zonder Unmanage probeert het systeem de oude permissie alsnog in te trekken en genereren de acties straks een error.

</details>

---

<details open>
<summary>

## I — CSV-bestanden controleren

</summary>

De V2-connector gebruikt nieuwe kolomnamen in de mapping-bestanden. Als de `locations.csv` en `teams.csv` nog de oude kolomnamen bevatten, zal de connector fouten geven bij de enforcement in Stap J.

Controleer of de bestanden in de **testmap** de volgende kolomnamen bevatten:
- `HelloIDPrimaryLookupKey`
- `HelloIDSecondaryLookupKey`

Zo niet: genereer nu nieuwe CSV-bestanden via het exportscript (zie Voorbereiding) voordat je verdergaat met Stap J.

</details>

---

<details open>
<summary>

## J — Afronden en valideren

</summary>

1. Draai een **Sync** op de entitlements.
2. Forceer update van alle accounts via **Update all accounts**.
3. Forceer update van alle Default Scope permissies: ga naar de permissiedefinitie van Default Scope en klik het gele knopje met het refresh-icoon (twee ronde pijltjes) rechts van het rode delete-icoon — dit is **Update in permission in definition**.
4. Herhaal dit voor de permissiedefinitie van Roles.
5. Draai een **Enforcement**.
6. Ga naar **Business → Entitlements → tab Blocked** — zet de Blocked actions voor accounts door en wacht totdat deze allemaal zijn uitgevoerd.
7. Zet de resterende Blocked actions voor de default scope en roles door en wacht totdat deze allemaal zijn uitgevoerd.
8. Controleer:
   - Geen nieuwe errors in de audit log
   - Pending actions = 0
9. Laat de klant valideren of alle accounts en rollen nog correct zijn.
10. Klant geeft **schriftelijk akkoord** (mail of Topdesk-ticket).
11. Zet **alle schedules** weer aan.
12. Verwijder de **migration reference** connector: dit is een automatisch aangemaakte, disabled en read-only connector met de naam van de originele connector gevolgd door "**- migration reference**". Deze bevat de volledige V1-configuratie en scripts zoals die waren op het moment dat de migratie werd gestart. Verwijder deze connector nadat de migratie volledig is gevalideerd.

</details>

---
