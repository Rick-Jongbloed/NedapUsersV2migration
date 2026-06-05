# Nedap Ons v1 → v2 — Uitvoeringskaart Productiemigratie
**Doelgroep:** IAM Consultant Tools4ever  
**Gebruik:** werk deze kaart stap voor stap af op de productiemigratiedag. Vink af, ga door.

<details open>
<summary>

## Voorbereiding productieomgeving

</summary>

### Minimaal 5 dagen voor start migratie

- [ ] Als gekozen is eerst op testomgeving te migreren: testomgeving gevalideerd en **schriftelijk akkoord** klant ontvangen
- [ ] Klant geïnformeerd: op productiemigratiedag worden **geen provisioning-acties** uitgevoerd en is **beheer niet mogelijk**
- [ ] Bevestig dat de certificaatupgrade naar versie 7 is afgerond: aanvraag door Tools4ever Support afgerond én klant heeft goedgekeurd in Nedap Podium
- [ ] Klantcontact uitgevraagd en beschikbaar op productiemigratiedag — stel de volgende vragen:
  - Wie is beschikbaar als aanspreekpunt?
  - Wie kan inloggen op de server, heeft toegang tot het CSV-exportscript en kan het opnieuw uitvoeren?
  - Wie kan het certificaatbestand op de server plaatsen?
  - Wordt de server beheerd door een externe partij? Zo ja: zorg dat die persoon beschikbaar is op de migratiedag. Geef daarbij aan dat op de server ingelogd moet worden om het certificaat te plaatsen en mogelijk scripts aan te passen.
- [ ] Controleer of het CSV-exportscript op de server staat of in HelloID (Admin dashboard → Automation → Tasks) — dit bepaalt hoe je het script aanpast (zie Op migratiedag)

### Minimaal 1 dag voor start migratie

- [ ] Pending actions productieomgeving = 0 — controleer en los pending actions op vóór migratiedag

### Op migratiedag

- [ ] ⚠️ Schakel de schedules uit **voordat** je het certificaat plaatst — zet daarna het nieuwe certificaatbestand in de **productiemap** op de server (overschrijf het bestaande bestand)
- [ ] CSV-exportscript controleren en CSV-bestanden genereren voor de productieomgeving:

  **Als het script op de server staat:**
  - Controleer dat de uitvoerpaden in het script verwijzen naar de **productiemap**.
  - **Als testmigratie al is uitgevoerd** → kolomnamen zijn al correct. Draai het script opnieuw.
  - **Geen testmigratie uitgevoerd** → controleer of de kolommen `HelloIDPrimaryLookupKey` en `HelloIDSecondaryLookupKey` al aanwezig zijn. Zo niet, vervang:

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

    Draai het script opnieuw.

  **Als het script in HelloID staat (Admin dashboard → Automation → Tasks):**
  - Pas het script direct aan — wijzig de kolomnamen zoals hierboven en zorg dat de uitvoerpaden naar de **productiemap** verwijzen.
  - Draai het script opnieuw.

- [ ] Zorg dat je toegang hebt tot de connector-repo: [`Nedap-new-permissions-api-standard`](https://github.com/Tools4everBV/HelloID-Conn-Prov-Target-NedapOns-Users/tree/Nedap-new-permissions-api-standard)
- [ ] **Myself-check** — controleer de onderstaande twee instellingen in de V1-scripts en noteer de waarden. Je hebt ze nodig tijdens de migratie (Stap D, punt 4).

  | Script | Wat te controleren | Configuratietoggle in V2 |
  |--------|-------------------|--------------------------|
  | Default Scope script | `$IsGrantMySelf` bovenin het script | Grant Default Scope Myself |
  | Roles Handle All Actions-script | `$myself` in functie `Merge-EntitlementToNedapRole` (standaard `$true` — check of overschreven) | Grant 'Myself' to each Role assignment |

</details>

---

<details open>
<summary>

## A — Start van de dag

</summary>

1. Log in op de HelloID-omgeving van de klant (productie-URL).
2. Zet alle **schedules uit**.
3. Controleer op uitgedeelde entitlements die niet meer bestaan in Nedap Ons: ga naar **Business → Rules → Entitlements** → filter op Nedap Ons Users → schakel **In rule: None** en **target system: Yes** uit. Zoek naar entitlements met een waarschuwing en los gevonden issues op vóór je verdergaat. *(Een schone uitgangssituatie voorkomt vervuiling tijdens de migratie.)*
4. Ga naar **Provisioning → Systems → Nedap Ons - Users** (productieconnector) → noteer de huidige **threshold-waarden** (bewaar deze) → stel in op **1**.

</details>

---

<details open>
<summary>

## B — Pending actions controleren en oplossen

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

## C — Pre-migratie Reference Cleaner check (op V1)

</summary>

1. Open de Reference Cleaner: `https://[klantnaam].helloid.com/provisioning/#/reference-cleaner/overview`
2. Lees de waarschuwing en klik dan op de gele knop **Start cleaner**.
3. Wacht tot de statusindicatoren zijn bijgewerkt en beoordeel het resultaat:
   - Alle vinkjes **groen** → ga door naar Stap D.
   - **Enforcement Runs** blijft draaien → er zijn nog pending actions; ga terug naar Stap B, los ze op en keer terug naar Stap C.
   - Enforcement Runs is groen maar **andere indicatoren** staan niet op groen → los de gemelde issues op aan de hand van de Reference Cleaner handleiding.
4. Klik **Stop cleaner** zodra je klaar bent. *(Browsertab sluiten stopt de Cleaner NIET — expliciet stoppen verplicht, als je de cleaner niet stopt, zijn alle business rules read-only.)*

</details>

---

<details open>
<summary>

## D — Migratie uitvoeren

</summary>

1. Ga naar **Provisioning → Systems → [Nedap Ons Users]** (productieconnector).
2. Klik **Migrate system** → bevestig.

   > ⚠️ Vanaf dit moment is de migratie onomkeerbaar. Schedules worden automatisch stilgezet. Er kunnen geen schedules of handmatige acties worden uitgevoerd totdat de migratie volledig is afgerond (Stap G).

   > HelloID maakt automatisch een read-only kopie van de volledige connector aan. Je kunt deze kopie in een apart venster openen om scripts van de oude V1-connector te vergelijken met de nieuwe versie.

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
   > **Advies bij maatwerk:** Ontdek je behalve in de mapping (grote) verschillen tussen de V1-scripts en de V2-standaardscripts? Klus deze dan niet direct in — begrijp eerst goed waar de verschillen liggen en bespreek dit met de klant. Stuur aan op de standaardconnector; zeker bij een complexe connector als Nedap Ons levert maatwerk op de lange termijn risico's op.
   >
   > Mocht maatwerk toch noodzakelijk zijn, voer dan eerst de migratie uit met de onbewerkte standaardscripts en rond deze volledig af vóórdat je aanpassingen doorvoert. Het account Create- en Update-script migreren namelijk éénmalig de in HelloID opgeslagen referenties naar de nieuwe versie — het is essentieel dat deze scripts onbewerkt worden uitgevoerd.
   >
   > Twijfels? Neem contact op met Tools4ever support vóór je verdergaat.

   - Vervang **Create script** (uit repo).
   - Vervang **Update script** (uit repo).
   - Vervang **Delete script** (uit repo).
   - Vervang **Data Import script** (uit repo).
   - Vervang **Configuration** — bij een standaard V1→V2 migratie wijzigen de meeste configuratiesleutels niet. Let op de volgende uitzonderingen:

     1. **Environment / baseUrl** — stel in op **Production** (dropdown); de URL wordt `https://api.ons.io`.
     2. **Certificaatpad** — pas het pad aan naar het `.pfx`-bestand in de **productiemap** op de server.
     3. **Certificaatwachtwoord** — vul het wachtwoord van het productiecertificaat in.
     4. Er zijn nu twee aparte toggles voor "myself" — stel beide in op basis van de boolean-check uit de Voorbereiding:

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

## E — Reference Cleaner (post-migratie)

</summary>

1. Open de Reference Cleaner: `https://[klantnaam].helloid.com/provisioning/#/reference-cleaner/overview`
2. Lees de waarschuwing en klik op de gele knop **Start cleaner**.
3. Wacht tot de statusindicatoren zijn bijgewerkt en beoordeel het resultaat:
   - Alle vinkjes **groen** → ga door naar stap 4.
   - **Enforcement Runs** blijft draaien → er zijn nog pending actions; ga terug naar Stap B, los ze op en keer terug naar Stap E.
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

## F — Default Scope legacy instellen

</summary>

> Voer de onderstaande stappen uit voor **alle business rules** waarin de oude Default Scope entitlement is opgenomen. Controleer eerst hoeveel business rules de Default Scope bevatten.

Voer de stappen in exact deze volgorde uit, voor elke business rule afzonderlijk:

1. Zoek de business rules met de **oude Default Scope** entitlement:
   - Ga naar **Business → Rules → tab Entitlements**.
   - Zoek op de naam van de Default Scope entitlement (bijv. "DefaultScope" — afhankelijk van hoe de permissiedefinitie is ingericht).
   - Selecteer het entitlement — rechts onder **Details** verschijnen alle business rules waarin dit entitlement is opgenomen.
   - Open de betreffende business rules