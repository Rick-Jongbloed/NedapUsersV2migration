# Nedap Ons v1 → v2 — Uitvoeringskaart Productiemigratie
**Doelgroep:** IAM Consultant Tools4ever  
**Gebruik:** werk deze kaart stap voor stap af op de productiemigratiedag. Vink af, ga door.

> Stappen en checklistitems gemarkeerd met **[Productie-only]** zijn alleen verplicht als er geen testmigratie is uitgevoerd. Bij een migratie ná testfase sla je deze over.

---

<details open>
<summary>

## Voorbereiding productieomgeving

</summary>

- [ ] Testomgeving gevalideerd en **schriftelijk akkoord** klant ontvangen *(of: bewuste keuze gemaakt om zonder testfase te migreren)*
- [ ] **[Productie-only]** Certificaat v7 aangevraagd bij Remco den Elzen; klant keurt goed in Nedap Podium
- [ ] Certificaat geplaatst in **productiemap** op server (overschrijf bestaand certificaat)
- [ ] **[Productie-only]** Feature flag actief bevestigd
- [ ] Klant geïnformeerd: op productiemigratiedag worden **geen provisioning-acties** uitgevoerd en is **beheer niet mogelijk**
- [ ] Pending actions productieomgeving = 0 — controleer en los op vóór migratiedag
- [ ] CSV-exportscript controleren en CSV-bestand voor productieomgeving genereren:
  - **Als testmigratie al is uitgevoerd** → gebruik het script dat je tijdens de testfase hebt aangepast. Draai het opnieuw om een verse CSV te genereren.
  - **Geen testmigratie uitgevoerd** → volg dezelfde stappen als bij de testmigratie: controleer of de kolommen `HelloIDPrimaryLookupKey` en `HelloIDSecondaryLookupKey` al aanwezig zijn. Zo niet, pas het script aan volgens de instructies in het testdocument en draai het opnieuw.

  > **Als het exportscript in HelloID staat (Admin dashboard → Automation → Tasks):** pas het script direct aan — wijzig de kolomnamen zoals hierboven beschreven en draai het script opnieuw om de CSV-bestanden te genereren.
- [ ] **[Productie-only]** V2-scripts beschikbaar — branch [`Nedap-new-permissions-api-standard`](https://github.com/Tools4everBV/HelloID-Conn-Prov-Target-NedapOns-Users/tree/Nedap-new-permissions-api-standard) in de connector-repo
- [ ] **[Productie-only] Boolean-check vóór migratie** *(gebruik eerder genoteerde waarden als de testfase al is uitgevoerd):* controleer de onderstaande twee instellingen in de V1-scripts en noteer de waarden. Ze worden straks elk afzonderlijk omgezet naar een configuratietoggle in V2.

  | Script | Wat te controleren | Configuratietoggle in V2 |
  |--------|-------------------|--------------------------|
  | Default Scope script | `$IsGrantMySelf` bovenin het script | Grant Default Scope Myself |
  | Roles Handle All Actions-script | `$myself` in functie `Merge-EntitlementToNedapRole` (standaard `$true` — check of overschreven) | Grant 'Myself' to each Role assignment |

  Noteer beide waarden — je hebt ze nodig bij Stap D, punt 4.
- [ ] Klantcontact uitgevraagd en beschikbaar op productiemigratiedag — stel (opnieuw) de volgende vragen:
  - Wie is beschikbaar als aanspreekpunt?
  - Wie heeft toegang tot het Excel-naar-CSV-exportscript en kan het opnieuw draaien? *(dit script moet opnieuw gedraaid worden na de migratie, omdat de kolomnamen wijzigen)*
  - Wordt de server beheerd door een externe partij? Zo ja: zorg dat die persoon ook beschikbaar is op de migratiedag, zodat we niet wachten op servertoegang

### Pre-flight check

- [ ] Voorbereiding productieomgeving volledig afgevinkt (zie boven)
- [ ] Pending actions productie nog steeds = 0 (bevestig vlak voor je begint)

</details>

---

<details open>
<summary>

## A — Start van de dag

</summary>

1. Log in op de HelloID-omgeving van de klant (productie-URL).
2. Controleer op oranje driehoek-waarschuwing → schakel filter "In role: none" uit; zijn er entitlements die niet meer in Nedap Ons bestaan, neem deze dan af vóór je verdergaat.
3. **[Productie-only]** Ga naar **Business → Rules → Entitlements** → filter op Nedap Ons Users → schakel "In role: none" uit en los entitlements met een waarschuwing op. Los gevonden issues op vóór je verdergaat. *(Een schone uitgangssituatie voorkomt vervuiling tijdens de migratie.)*
4. Zet alle **schedules uit**.
5. Ga naar **Provisioning → Systems → Nedap Ons - Users** (productieconnector) → noteer de huidige **threshold-waarden** (bewaar deze) → stel in op **1**.

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
   - Open de betreffende business rules via **Ctrl+klik** of **middlemuisklik** op het moersleuteltje om ze in een nieuw venster te openen.
2. Vink de oude Default Scope **uit**.
3. Draai een **Sync** op de entitlements van Nedap Ons Users.
4. Vink de nieuwe **Default Scope (legacy)** entitlement **aan**.
5. Publiceer de business rule — kies bij het publiceren voor **Unmanage removed entitlement(s)** voor het oude Default Scope entitlement.

   > ⚠️ Sla de Unmanage-stap niet over. Zonder Unmanage probeert het systeem de oude permissie alsnog in te trekken en genereren de acties straks een error.

</details>

---

<details open>
<summary>

## G — Afronden en valideren

</summary>

1. Draai een **Sync** op de entitlements.
2. Forceer update van alle accounts via **Update all accounts**.
3. Forceer