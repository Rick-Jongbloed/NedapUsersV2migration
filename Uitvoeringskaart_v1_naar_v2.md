# Nedap Ons v1 → v2 — Uitvoeringskaart
**Doelgroep:** IAM Consultant Tools4ever  
**Gebruik:** werk deze kaart stap voor stap af op migratiedag. Vink af, ga door.

> **Migratiestrategie:** Dit document beschrijft een optionele testfase (Deel 1) gevolgd door de productieomgeving (Deel 2). Wil je direct naar productie migreren zonder testomgeving, sla Deel 1 dan over en begin bij Deel 2.

---

## Voorbereiding testomgeving

> Sla deze sectie over als je de productiekoppeling migreert zonder eerst een proefmigratie op de testomgeving uit te voeren.

- [ ] **T-5** Feature flag v1 connector aangevraagd bij Rick van den Dijssel
- [ ] **T-5** Bevestig bij klant dat de testomgeving met ID **TE-XXXX** nog steeds in gebruik is (dezelfde waarvoor eerder een certificaat is aangevraagd)
- [ ] **T-5** Controleer of het huidige testcertificaat nog geldig is op het geplande uitvoermoment
- [ ] **T-5** Certificaat vernieuwen indien nodig — **volgorde: eerst upgraden naar versie 7, daarna pas verversen**; aanvraag via Remco den Elzen, klant keurt goed in Nedap Podium
- [ ] **T-5** Certificaat geplaatst in **testmap** op server (overschrijf bestaand certificaat)
- [ ] **T-5** Klant geïnformeerd: op testmigratiedag worden **geen provisioning-acties** uitgevoerd en is **beheer niet mogelijk**
- [ ] **T-3** Klant heeft testomgeving ververst (verse kopie van productie)
- [ ] **T-3** Certificaat actief op testomgeving bevestigd
- [ ] **T-1** Feature flag actief bevestigd
- [ ] **T-1** Pending actions testomgeving = 0 — controleer en los op vóór migratiedag
- [ ] PowerShell CSV-exportscript beschikbaar en aangepast voor nieuwe kolomnamen V2 *(zie apart instructiedocument PowerShell CSV-export)*
- [ ] CSV-bestand voor testomgeving klaarstaat
- [ ] V2-scripts beschikbaar (branch `nedap-new-permissions-api-standard` in de connector-repo)
- [ ] V1-scripts van de klant doorgenomen op maatwerk — zijn er grote afwijkingen van de standaard? Bespreek dit vóór de migratiedag met de klant en stem af of het maatwerk echt noodzakelijk is. Zie ook de toelichting bij Stap F, punt 4.
- [ ] **Boolean-check vóór migratie:** controleer de onderstaande twee instellingen in de V1-scripts en noteer de waarden. Ze worden straks elk afzonderlijk omgezet naar een configuratietoggle in V2.

  | Script | Wat te controleren | Configuratietoggle in V2 |
  |--------|-------------------|--------------------------|
  | Default Scope script | `$IsGrantMySelf` bovenin het script | Grant Default Scope Myself |
  | Roles Handle All Actions-script | `$myself` in functie `Merge-EntitlementToNedapRole` (standaard `$true` — check of overschreven) | Grant 'Myself' to each Role assignment |

  Noteer beide waarden — je hebt ze nodig bij Stap F, punt 4.
- [ ] Klantcontact uitgevraagd en beschikbaar op testmigratiedag — stel de volgende vragen:
  - Wie is beschikbaar als aanspreekpunt?
  - Wie heeft toegang tot het Excel-naar-CSV-exportscript en kan het opnieuw draaien? *(dit script moet opnieuw gedraaid worden na de migratie, omdat de kolomnamen wijzigen)*
  - Wordt de server beheerd door een externe partij? Zo ja: zorg dat die persoon ook beschikbaar is op de migratiedag, zodat we niet wachten op servertoegang
- [ ] **Dag van** Bij het inloggen: controleer op oranje driehoek-waarschuwing → schakel filter "In role: none" uit; zijn er groepen die niet meer in Nedap Ons bestaan, neem deze dan af vóór je begint

---

## Voorbereiding productieomgeving

- [ ] Testomgeving gevalideerd en **schriftelijk akkoord** klant ontvangen *(of: bewuste keuze gemaakt om zonder testfase te migreren)*
- [ ] Certificaat v7 aangevraagd bij Remco den Elzen *(als dit nog niet gedaan is in de testfase)*; klant keurt goed in Nedap Podium
- [ ] Certificaat geplaatst in **productiemap** op server (overschrijf bestaand certificaat)
- [ ] Feature flag actief bevestigd *(als dit nog niet gedaan is in de testfase)*
- [ ] Klant geïnformeerd: op productiemigratiedag worden **geen provisioning-acties** uitgevoerd en is **beheer niet mogelijk**
- [ ] Pending actions productieomgeving = 0 — controleer en los op vóór migratiedag
- [ ] CSV-bestand voor productieomgeving klaarstaat
- [ ] V2-scripts beschikbaar *(als dit nog niet gedaan is in de testfase)*
- [ ] Klantcontact uitgevraagd en beschikbaar op productiemigratiedag *(zie uitvraag bij voorbereiding testomgeving — bevestig opnieuw wie beschikbaar is, inclusief serverbeheerder indien van toepassing)*
- [ ] **Dag van** Bij het inloggen: controleer op oranje driehoek-waarschuwing → schakel filter "In role: none" uit; zijn er groepen die niet meer in Nedap Ons bestaan, neem deze dan af vóór je begint

---

## Deel 1 — Testomgeving

> **Let op:** Deel 1 is alleen nodig als je de migratie eerst op een testomgeving wilt valideren vóórdat je naar productie gaat. Bij een productie-only migratie sla je Deel 1 over en begin je bij Deel 2.

### A — Start van de dag

1. Log in op de HelloID-omgeving van de klant.
2. Zet alle **schedules uit** (handmatig, één voor één). *(Voorkomt ongewenste wijzigingen tijdens de migratie.)*
3. Controleer op uitgedeelde entitlements die niet meer bestaan in Nedap Ons: ga naar **Business Rules** → filter op Nedap Ons Users → schakel "In role: none" uit en zoek naar entitlements met een waarschuwing of ontbrekende koppeling. Los gevonden issues op vóór je verdergaat. *(Een schone uitgangssituatie voorkomt vervuiling tijdens de migratie.)*

---

### A2 — Testconnector inrichten

1. Ga naar **Provisioning → Systems → Nedap Ons - Users** (productieconnector).
2. Kopieer alle **scripts** (Create, Update, Delete, Data Import, Permission, Resources, Handle All Actions) en de volledige **configuratie** naar de **Nedap Ons - Users Test** connector. *(Je maakt hiermee een exacte kopie van de productieconnector voor de testomgeving.)* Let daarbij op:
   - Controleer dat de **naam van de Default Scope entitlement** exact overeenkomt met de productieconnector — inclusief hoofdlettergebruik.
   - Controleer dat de **correlatie-instellingen** (Tab Correlation) identiek zijn aan de productieconnector.
3. Maak op de server een **testmap** aan (als die er nog niet is) en kopieer daarin:
   - Het **certificaatbestand** (`.pfx`)
   - De **locations mapping** (`locations.csv`)
   - De **teams mapping** (`teams.csv`)
   - De **cache-map** (of maak een lege map aan als startpunt voor de cache)
4. Pas in de testconnector de **configuratie** aan zodat alle paden naar de testmap verwijzen:

   | Parameter | Aanpassen naar |
   |-----------|----------------|
   | Environment / baseUrl | `https://api-staging.ons.io` |
   | Certificaatpad | Pad naar `.pfx` in de **testmap** |
   | mappingLocations | Pad naar `locations.csv` in de **testmap** |
   | mappingTeams | Pad naar `teams.csv` in de **testmap** |
   | DirectoryCacheLocationsTeams | Pad naar de **cache-map** in de testmap |

   > Laat alle **thresholds** op de testconnector staan zoals ze zijn — deze staan standaard al op 1.

---

### B — Testomgeving gelijkstellen aan productie

1. Ga naar **Business Rules** → filter op Nedap Ons Users **en** Nedap Ons Users Test, status "Draft" + "Published" aan, "None" uit. *(Dit geeft overzicht van alle uitgedeelde entitlements. Noteer het huidige aantal.)*
2. Koppel elk entitlement dat in de productieregel staat ook aan de **testconnector**. Doe dit voor alle Nedap Ons-entitlements. Na voltooiing moet het totaal aantal entitlements **precies het dubbele** zijn van het beginaantal — elk entitlement staat nu zowel op de productie- als de testconnector.
3. Draai een **Enforcement +** om alles gelijk te trekken en de cachebestanden te genereren. Alle geblokkeerde entitlements kunnen worden doorgezet — komen er fouten uit, dan ligt de oorzaak waarschijnlijk in een verkeerd pad of ontbrekend bestand uit stap A2, punt 4. Controleer in dat geval de configuratie van de testconnector.
4. Controleer de auditlog op fouten na de Enforcement+. Los op wat mogelijk is en **documenteer elke fix** — dezelfde fouten zitten ook in productie. Niet alles hoeft nu opgelost te zijn; pending actions worden afgehandeld in Stap D.
5. Verifieer dat de testconnector gelijk is aan de productieconnector: scripts zijn identiek en alle entitlements in business rules staan op zowel Nedap Ons Users als Nedap Ons Users Test. Dit is het enige criterium — als dit klopt, is de testomgeving gereed voor migratie.

---

### D — Pending actions controleren en oplossen

> Dit geldt voor de gehele HelloID-omgeving — niet alleen de Nedap Ons connector. Alle pending actions in alle connectoren moeten opgelost zijn voordat je verdergaat.

1. Ga naar **Business → Evaluation** — stop alle enforcements met de status *Running* via de gele **Cancel**-knop.
2. Ga naar **Business → Entitlements → Actions** en los alle openstaande acties op:
   - Testaccounts → Unmanage
   - Excluded accounts met account → Unmanage
   - Groeptoewijzing zonder account: de conditie die in de account business rule staat ontbreekt in de bijbehorende permissie business rule — voeg dezelfde conditie toe. Ga naar **Business → Rules → Entitlements**, zoek het betreffende entitlement op en pas de relevante business rule(s) aan.
   - Andere → klant raadplegen en samen kijken naar een oplossing
3. Controleer: het overzicht onder **Business → Entitlements → Actions** is leeg voordat je verdergaat.

---

### E — Pre-migratie Reference Cleaner check (op V1)

1. Open de Reference Cleaner: `https://[klantnaam].helloid.com/provisioning/#/reference-cleaner/overview`
2. Lees de waarschuwing en klik dan op de gele knop **Start cleaner**.
3. Wacht tot de statusindicatoren zijn bijgewerkt en beoordeel het resultaat:
   - Alle vinkjes **groen** → ga door naar Stap F.
   - **Enforcement Runs** blijft draaien → er zijn nog pending actions; ga terug naar Stap D, los ze op en keer terug naar Stap E.
   - Enforcement Runs is groen maar **andere indicatoren** staan niet op groen → los de gemelde issues op aan de hand van de Reference Cleaner handleiding.
4. Klik **Stop cleaner** zodra je klaar bent. *(Browsertab sluiten stopt de Cleaner NIET — expliciet stoppen verplicht.)*

---

### F — Migratie uitvoeren (scripts en configuratie)

1. Ga naar **Provisioning → Systems → [Nedap Ons Users Test]**.
2. Klik **Migrate system** → bevestig met **Confirm**.

   > ⚠️ Vanaf dit moment is de migratie onomkeerbaar. Schedules worden automatisch stilgezet. Er kunnen geen schedules of handmatige acties worden uitgevoerd totdat de migratie volledig is afgerond (Stap I).

   > HelloID maakt automatisch een read-only kopie van de volledige connector aan. Je kunt deze kopie in een apart venster openen om scripts van de oude V1-connector te vergelijken met de nieuwe versie.

3. **Tab Fields**
   - Vink **Field Configuration** aan in de migratieview.
   - Klik **Delete all** (verwijder huidige field mapping).
   - Download de field mapping van GitHub (branch `nedap-new-permissions-api-standard`).
   - Importeer de field mapping.

4. **Tab Account**
   > ⚠️ Controleer vóór het overschrijven of het accountscript klantspecifiek is aangepast. Zo ja: kopieer de klantspecifieke mapping over naar het nieuwe script. Gebruik de read-only V1-kopie (zie stap 2) om de scripts naast elkaar te vergelijken.

   > **Advies bij maatwerk:** Ontdek je grote verschillen tussen de V1-scripts en de V2-standaardscripts? Klus deze dan niet direct in — begrijp eerst goed waar de verschillen liggen en bespreek dit met de klant. Stuur aan op de standaardconnector; zeker bij een complexe connector als Nedap Ons levert maatwerk op de lange termijn risico's op.
   >
   > Mocht maatwerk toch noodzakelijk zijn, voer dan eerst de migratie uit met de onbewerkte standaardscripts en rond deze volledig af vóórdat je aanpassingen doorvoert. Het account Create- en Update-script migreren namelijk éénmalig de in HelloID opgeslagen referenties naar de nieuwe versie — het is essentieel dat deze scripts onbewerkt worden uitgevoerd.
   >
   > Twijfels? Neem contact op met Tools4ever vóór je verdergaat.

   - Vervang **Create script** (uit repo).
   - Vervang **Update script** (uit repo).
   - Vervang **Delete script** (uit repo).
   - Vervang **Data Import script** (uit repo).
   - Vervang **Configuration** — bij een standaard V1→V2 migratie wijzigen de meeste configuratiesleutels niet. Let op de volgende twee uitzonderingen:

     1. **Environment (Rest)** — staat standaard ingesteld op *Production*. Verander dit voor de testflow naar **Acceptance** (dropdown).
     2. Er zijn nu twee aparte toggles voor "myself" — stel beide in op basis van de boolean-check die je hebt uitgevoerd in de Voorbereiding:

        | Configuratieoptie | Gebaseerd op | Zet aan als |
        |-------------------|--------------|-------------|
        | **Grant Default Scope Myself** | `$IsGrantMySelf` bovenin het Default Scope script | waarde = `$true` |
        | **Grant 'Myself' to each Role assignment** | `$myself` in `Merge-EntitlementToNedapRole` in het Roles Handle All Actions-script | waarde = `$true` |

        Stel beide opties in vóórdat je op Apply drukt.

     Druk daarna altijd éénmaal op **Apply**, ook als er niets is aangepast.

5. **Tab Permissions — Default Scope**
   - Laad het **Permission script** (uit repo).
   - Zet **"Use script to import permissions"** aan.
   - Klik **Eval** → daarna **Apply**.
   - Zet de toggle **Contact Data Storage** aan.
   - Controleer of de klant afwijkt van de onderstaande Tools4ever-standaarden. Gebruik de read-only V1-kopie (zie stap 2) om te vergelijken. Wijkt de klant af, pas het script dan aan vóór je verdergaat.

   | Parameter | Standaardwaarde | Toelichting |
   |-----------|-----------------|-------------|
   | Nedap Ons Identification ID | `Custom.NedapOnsIdentificationNo` | Was configureerbaar in V1, nu hardcoded in script |
   | Team primary lookup key | `{ $_.Department.ExternalId }` | Verplicht — dit veld mag niet leeg zijn in de mapping CSV |
   | Team secondary lookup key | `{ $_.Title.ExternalId }` | Niet verplicht — dit veld mag leeg zijn in de mapping CSV |
   | Location primary lookup key | `{ $_.Department.ExternalId }` | Verplicht — dit veld mag niet leeg zijn in de mapping CSV |
   | Location secondary lookup key | `{ $_.Title.ExternalId }` | Niet verplicht — dit veld mag leeg zijn in de mapping CSV |

6. **Tab Permissions — Roles**
   - Laad het **Roles permission script** (uit repo).
   - Controleer of de klant afwijkt van dezelfde standaarden als in stap 5. Pas aan indien nodig.
   - Klik **Preview** om te controleren.

7. **Tab Permissions — Handle All Actions**
   - Laad het **Handle All Actions script** (uit repo).
   - Controleer in de functie `Merge-EntitlementToNedapRole` of de `$myself`-parameter is overschreven of op de standaardwaarde (`$true`) staat. Zorg dat deze waarde consistent is met de `$IsGrantMySelf`-instelling in de Default Scope scripts (zie boolean-check in Voorbereiding).

8. **Tab Resources**
   - Vervang het **Resources script** (`resources.ps1`, uit repo).
   - Klik **Preview** om te controleren.

9. **Tab Correlation**

   | Instelling | Waarde |
   |------------|--------|
   | Enable correlation | `True` |
   | Person correlation field | `ExternalId` |
   | Account correlation field | Nedap Ons Identification Number |

10. Controleer of je alle vinkjes hebt gezet: Create ✓ Update ✓ Delete ✓ Permission ✓ Default Scope ✓ Role ✓ Resource Cache ✓
11. Klik **Complete Migration** → bevestig.

---

### G — Reference Cleaner (post-migratie)

1. Open de Reference Cleaner: `https://[klantnaam].helloid.com/provisioning/#/reference-cleaner/overview`
2. Klik **Start cleaner**.
3. Selecteer de **Roles** Permission Configuration.
4. Klik **Determine differences** — controleer dat `DisplayName` en `DisplayNameFull` in de "to remove"-lijst staan.
5. Klik **Remove fields**.
6. Controleer de **History** rechts — actie gelogd?
7. Klik **Stop cleaner**.

---

### H — Default Scope legacy instellen

> Voer de onderstaande stappen uit voor **alle business rules** waarin de oude Default Scope entitlement is opgenomen. Controleer eerst hoeveel business rules de Default Scope bevatten — bij klanten met meerdere bedrijven kan dit meer dan één zijn.

Voer de stappen in exact deze volgorde uit, voor elke business rule afzonderlijk:

1. Ga naar alle business rules met de **oude Default Scope** entitlement.
2. Vink de oude Default Scope **uit**.
3. Draai een **Sync** op de entitlements.
4. Vink de nieuwe **Default Scope (legacy)** entitlement **aan**.
5. Publiceer de business rule — kies bij het publiceren voor **Unmanage** voor het oude Default Scope entitlement.

   > ⚠️ Sla de Unmanage-stap niet over. Zonder Unmanage probeert het systeem de oude permissie alsnog in te trekken.

---

### I — Afronden en valideren

1. Draai een **Sync** op de entitlements.
2. Draai een **Force update** op alle accounts.
3. Draai een **Enforcement**.
4. Controleer:
   - Geen errors in de audit log
   - Pending actions = 0
   - Permissions Preview voor een testpersoon geeft correct resultaat
   - Rollen en entitlements correct zichtbaar in de business rules
5. Zet de **schedules weer aan** — pas nadat Reference Cleaner succesvol is afgerond en alle validaties zijn doorlopen.
6. Laat de klant functioneel valideren (accounts, rollen, bereik correct?).
7. Klant geeft **schriftelijk akkoord** (mail of Topdesk-ticket).

---

## Deel 2 — Productieomgeving

> Voer productie pas uit als de klant de testomgeving heeft gevalideerd en akkoord heeft gegeven. Business rules zijn al gecontroleerd in Deel 1 — die stap sla je hier over.

### Pre-flight check productie

- [ ] Voorbereiding productieomgeving volledig afgevinkt (zie boven)
- [ ] Pending actions productie nog steeds = 0 (bevestig vlak voor je begint)

---

### A — Start van de dag

1. Log in op de HelloID-omgeving van de klant (productie-URL).
2. Controleer op oranje driehoek-waarschuwing (zie Voorbereiding) — los niet-bestaande groepen op indien aanwezig.
3. Zet alle **schedules uit**.
4. Ga naar **Provisioning → Systems → Nedap Ons - Users** (productieconnector) → noteer de huidige **threshold-waarden** (bewaar deze) → stel in op **1**.

---

### B — Pending actions controleren

1. Ga naar **Provisioning → Actions → Pending**.
2. Los alle openstaande pending actions op (zie Deel 1 — Stap D). Controleer: **Pending actions = 0** voordat je verdergaat.

---

### C — Pre-migratie Reference Cleaner check

1. Draai de Reference Cleaner op de **V1-productieconnector** (zie Deel 1 — Stap E). Geen blocking taken vóór je verdergaat.

---

### D — Migratie uitvoeren

1. Ga naar **Provisioning → Systems → [Nedap Ons Users]** (productieconnector).
2. Klik **Migrate system** → bevestig.

   > ⚠️ Vanaf dit moment is de migratie onomkeerbaar.

3. Vervang field mapping, account-scripts, permission-scripts en configuratie exact zoals in Deel 1 — Stap F (punt 3–8). Gebruik de volgende **productiewaarden** in de configuratie:

   | Parameter | Productiewaarde |
   |-----------|-----------------|
   | Environment / baseUrl | `https://api.ons.io` |
   | Certificaatpad | Pad naar `.pfx` in de **productiemap** op de server |
   | Certificaatwachtwoord | Wachtwoord productiecertificaat |
   | Alle overige parameters | Identiek aan testconfiguratie |

4. Stel **Correlation** in (zie Deel 1 — Stap F, punt 9).
5. Controleer alle vinkjes: Create ✓ Update ✓ Delete ✓ Permission ✓ Default Scope ✓ Role ✓ Resource Cache ✓
6. Klik **Complete Migration** → bevestig.

---

### E — Reference Cleaner (post-migratie)

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
8. Klik **Stop cleaner**. *(Browsertab sluiten stopt de Cleaner NIET — expliciet stoppen verplicht.)*

---

### F — Default Scope legacy instellen

> Voer de onderstaande stappen uit voor **alle business rules** waarin de oude Default Scope entitlement is opgenomen. Controleer eerst hoeveel business rules de Default Scope bevatten — bij klanten met meerdere bedrijven kan dit meer dan één zijn.

Voer de stappen in exact deze volgorde uit, voor elke business rule afzonderlijk:

1. Ga naar alle business rules met de **oude Default Scope** entitlement.
2. Vink de oude Default Scope **uit**.
3. Draai een **Sync** op de entitlements.
4. Vink de nieuwe **Default Scope (legacy)** entitlement **aan**.
5. Publiceer de business rule — kies bij het publiceren voor **Unmanage** voor het oude Default Scope entitlement.

   > ⚠️ Sla de Unmanage-stap niet over. Zonder Unmanage probeert het systeem de oude permissie alsnog in te trekken.

---

### G — Afronden en valideren

1. Draai een **Sync** op de entitlements.
2. Draai een **Force update** op alle accounts.
3. Draai een **Enforcement**.
4. Controleer:
   - Geen errors in de audit log
   - Pending actions = 0
   - Permissions Preview voor een testpersoon geeft correct resultaat
5. Zet de **schedules weer aan** — pas nadat Reference Cleaner succesvol is afgerond en alle validaties zijn doorlopen.
6. Herstel de **thresholds** van de Nedap Ons - Users connector naar de oorspronkelijke waarden (genoteerd in Stap A, punt 4).
7. Laat de klant functioneel valideren (accounts, rollen, bereik correct?).
8. Klant geeft **schriftelijk akkoord** (mail of Topdesk-ticket).

---

### Afsluiting

- [ ] Migratie melden aan Remco den Elzen (Topdesk major ticket)
- [ ] Rick van den Dijssel informeren: feature flag v1 mag uit voor deze klant
- [ ] Afwijkingen en tijdsduur documenteren in klantdossier
- [ ] Handleiding bijwerken indien nodig

---

## Snelle referentie — Issues

| Symptoom | Oplossing |
|----------|-----------|
| Pending actions blokkeren Reference Cleaner | Oplossen (zie Stap D), controleer tot = 0 |
| Default Scope wordt niet ingetrokken | Force update niet gedraaid — draai Stap I, punt 2 opnieuw |
