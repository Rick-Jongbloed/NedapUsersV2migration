# Nedap Ons v1 → v2 — Uitvoeringskaart Testmigratie
**Doelgroep:** IAM Consultant Tools4ever  
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
- [ ] Certificaat vernieuwen indien nodig — **volgorde: eerst upgraden naar versie 7, daarna pas verversen**; aanvraag via Remco den Elzen, klant keurt goed in Nedap Podium
- [ ] Certificaat geplaatst in **testmap** op server (overschrijf bestaand certificaat)
- [ ] Klant geïnformeerd: op testmigratiedag worden **geen provisioning-acties** uitgevoerd en is **beheer niet mogelijk**

### Minimaal 3 dagen voor start migratie

- [ ] Klant heeft Nedap Ons testomgeving ververst (verse kopie van Nedap Ons productie naar Nedap Ons testomgeving)
- [ ] Certificaat actief op testomgeving bevestigd

### Minimaal 1 dag voor start migratie

- [ ] Feature flag actief bevestigd
- [ ] Pending actions testomgeving = 0 — controleer en los pending actions op vóór migratiedag

### Op migratiedag

- [ ] PowerShell CSV-exportscript controleren op kolomnamen — controleer of de `locations.csv` en `teams.csv` al de kolommen `HelloIDPrimaryLookupKey` en `HelloIDSecondaryLookupKey` bevatten:
  - **Kolommen al correct** → niets te doen, sla dit item over.
  - **Kolommen heten nog `Department.ExternalId` en `Title.ExternalId`** → maak een kopie van het exportscript in de **testmap** op de server en pas dáár de volgende regel aan, in zowel de mapping teams- als de mapping locaties-sectie. ⚠️ Pas het originele script niet aan — de productieconnector draait nog op V1 en heeft dat script nodig.
    ```powershell
    # Vervang:
    Select-Object Department.ExternalId, Title.ExternalId, NedapTeamIds, AllEmployees |
    # Door:
    Select-Object @{Name='HelloIDPrimaryLookupKey'; Expression='Department.ExternalId'}, @{Name='HelloIDSecondaryLookupKey'; Expression='Title.ExternalId'}, NedapTeamIds, AllEmployees |
    ```
    Draai het aangepaste script daarna vanuit de testmap om een CSV-bestand met de juiste kolomnamen te genereren.

  > **Als het exportscript in HelloID staat (Admin dashboard → Automation → Tasks):** maak een kopie van het script en pas in de kopie de kolomnamen aan zoals hierboven beschreven. Pas daarnaast de uitvoerpaden aan zodat de gegenereerde CSV-bestanden in de **testmap** terechtkomen, niet in de productiemap. Draai daarna de kopie om de CSV-bestanden te genereren.
- [ ] CSV-bestand voor testomgeving staat klaar
- [ ] V2-scripts beschikbaar — branch [`Nedap-new-permissions-api-standard`](https://github.com/Tools4everBV/HelloID-Conn-Prov-Target-NedapOns-Users/tree/Nedap-new-permissions-api-standard) in de connector-repo
- [ ] V1-scripts van de klant doorgenomen op maatwerk — zijn er grote afwijkingen van de standaard? Bespreek dit vóór de migratiedag met de klant en stem af of het maatwerk echt noodzakelijk is.
- [ ] Controleer de onderstaande twee instellingen in de V1-scripts en noteer de waarden. Ze worden straks elk afzonderlijk omgezet naar een configuratietoggle in V2.

  | Script | Wat te controleren | Configuratietoggle in V2 |
  |--------|-------------------|--------------------------|
  | Default Scope script | `$IsGrantMySelf` bovenin het script | Grant Default Scope Myself |
  | Roles Handle All Actions-script | `$myself` in functie `Merge-EntitlementToNedapRole` (standaard `$true` — check of overschreven) | Grant 'Myself' to each Role assignment |

  Noteer beide waarden — je hebt ze nodig bij Stap F, punt 4.
- [ ] Klantcontact uitgevraagd en beschikbaar op testmigratiedag — stel de volgende vragen:
  - Wie is beschikbaar als aanspreekpunt?
  - Wie heeft toegang tot het Excel-naar-CSV-exportscript en kan het opnieuw draaien? *(dit script moet opnieuw gedraaid worden na de migratie, omdat de kolomnamen wijzigen)*
  - Wordt de server beheerd door een externe partij? Zo ja: zorg dat die persoon ook beschikbaar is op de migratiedag, zodat we niet wachten op servertoegang

</details>

---

<details open>
<summary>

## A — Start van de dag

</summary>

1. Log in op de HelloID-omgeving van de klant.
2. Zet alle **schedules uit** (handmatig, één voor één). *(Voorkomt ongewenste wijzigingen tijdens de migratie.)*
3. Controleer op uitgedeelde entitlements die niet meer bestaan in Nedap Ons: ga naar **Business → Rules → Entitlements** → filter op Nedap Ons Users → schakel "In role: none" uit en los entitlements met een waarschuwing op. Los gevonden issues op vóór je verdergaat. *(Een schone uitgangssituatie voorkomt vervuiling tijdens de migratie.)*

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
3. Draai een **Enforcement +** om alles gelijk te trekken en de cachebestanden te genereren. Alle geblokkeerde entitlements kunnen worden doorgezet — komen er fouten uit, dan ligt de oorzaak waarschijnlijk in een verkeerd pad of ontbrekend bestand uit Stap B, punt 5. Controleer in dat geval de configuratie van de testconnector.
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
   - **Enforcement Runs** blijft draaien → er zijn nog pending actions; ga terug naar Stap D, los ze op en keer terug naar Stap E.
   - Enforcement Runs is groen maar **andere indicatoren** staan niet op groen → los de gemelde issues op aan de hand van de Reference Cleaner handleiding.
4. Klik **Stop cleaner**. *(Browsertab sluiten stopt de Cleaner NIET — expliciet stoppen verplicht, als je de cleaner niet stopt, zijn alle business rules read-only.)*

</details>

---

<details open>
<summary>

## F — Migratie uitvoeren (scripts en configuratie)

</summary>

> ⚠️ Vanaf dit moment is de migratie onomkeerbaar. Schedules worden automatisch stilgezet. Er kunnen geen schedules of handmatige acties worden uitgevoerd totdat de migratie volledig is afgerond (Stap I).

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
   > Dit stukje gaat van code in het script naar de mapping:
   > contractRequiredAtLogin = $true
   > ssoEnabled              = $true
   > limitLocationView       = $true
   > passwordChange          = $true
   >
   > **Advies bij maatwerk:** Ontdek je behalve in de mapping (grote) verschillen tussen de V1-scripts en de V2-standaardscripts? Klus deze dan niet direct in — begrijp eerst goed waar de verschillen liggen en bespreek dit met de klant. Stuur aan op de standaardconnector; zeker bij een complexe connector als Nedap Ons levert maatwerk op de lange termijn risico's op.
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

        | Configuratieoptie | Gebaseerd op | Zet aan als |
        |-------------------|--------------|-------------|
        | **Grant Default Scope Myself** | `$IsGrantMySelf` bovenin de Default Scope grant en update permission scripts | waarden `$true` of `$false` |
        | **Grant 'Myself' to each Role assignment** | `$myself` in functieaanroep van `Merge-EntitlementToNedapRole` in het Roles Handle All Actions-script | waarden `$true` of `$false` |

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
   | Account correlation field | Nedap Ons Identification Number |

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
3. Selecteer de **Roles** Permission Configuration.
4. Klik **Determine differences** — controleer dat `DisplayName` en `DisplayNameFull` in de "to remove"-lijst staan.
5. Klik **Remove fields**.
6. Controleer de **History** rechts — actie gelogd?
7. Klik **Stop cleaner**.