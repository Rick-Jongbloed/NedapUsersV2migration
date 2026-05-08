# Nedap Ons v1 → v2 — Uitvoeringshandleiding

> **Status:** concept — gebaseerd op gesprek met Rudolf + pilotervaring Rick Jongbloed  
> **Doelgroep:** IAM Consultants Tools4ever  
> **Doel:** 4 uur per klant op productie, zonder testomgeving (vanaf klant 4+)

---

## Inhoudsopgave

1. [Achtergrond en scope](#1-achtergrond-en-scope)
2. [Rollen en contacten](#2-rollen-en-contacten)
3. [Fase 0 — Testrun op eigen omgeving (Rick)](#3-fase-0--testrun-op-eigen-omgeving-rick)
4. [Fase 1 — Voorbereiding per klant](#4-fase-1--voorbereiding-per-klant)
5. [Fase 2 — Wachtacties: check en oplossing](#5-fase-2--wachtacties-check-en-oplossing)
6. [Fase 3 — Scope assessment: migreren of opnieuw opbouwen?](#6-fase-3--scope-assessment-migreren-of-opnieuw-opbouwen)
7. [Fase 4 — Migratiestappen (testomgeving)](#7-fase-4--migratiestappen-testomgeving)
8. [Fase 5 — Migratiestappen (productieomgeving)](#8-fase-5--migratiestappen-productieomgeving)
9. [Bijlage A — Entitlement conversie](#9-bijlage-a--entitlement-conversie)
10. [Bijlage B — Troubleshooting](#10-bijlage-b--troubleshooting)
11. [Bijlage C — Contacten en escalatie](#11-bijlage-c--contacten-en-escalatie)

---

## 1. Achtergrond en scope

### Wat is dit?

Nedap Ons stapt over op een nieuw autorisatiemodel. Tools4ever heeft hiervoor een nieuwe v2-connector ontwikkeld. Dit document is de uitvoeringshandleiding voor het migreren van ~65 klanten van de v1- naar de v2-connector.

### Uitgangspunt: technische vertaalslag

De migratie is **100% technisch** — geen functionele wijzigingen, geen verbeteringen, geen klantadvies. Als het in v1 werkte, werkt het na migratie identiek in v2. Afwijken van dit uitgangspunt kost tijd en vergroot de kans op fouten.

### Wat verandert er in v2?

| Onderdeel | v1 | v2 |
|-----------|----|----|
| Standaardbereik (default scope) | Toegepast op zowel roluitgifte als Cockpit | Alleen nog voor Cockpit/scope-instellingen. Rollen krijgen `all_employees`/`all_clients` direct toegewezen (backward compat. via migratietool) |
| API URL | Oude endpoint | Nieuw endpoint (enige configuratieparameter die wijzigt) |
| Entitlements | Eigen structuur v1 | Nieuwe endpoints; statische mapping blijft actief (zie Bijlage A) |

> ⚠️ **Belangrijk voor klanten:** In v2 krijgt een handmatig aangemaakte rol (buiten de migratietool om) **niet** automatisch het standaardbereik. Dit gedrag wijzigt. Klanten die zelf rollen aanmaken moeten dit weten.

### Wat is NIET de scope van de migratie?

- Verbeteringen of optimalisaties doorvoeren
- Autorisatiematrix aanpassen
- Nieuwe entitlements toevoegen die in v1 niet bestonden
- Advies geven over v2-functionaliteit

---

## 2. Rollen en contacten

| Naam | Rol | Betrokkenheid |
|------|-----|---------------|
| Rick Jongbloed | Projectleider, uitvoerder klant 1+2 | Actief |
| Rudolf | Connector-ontwikkelaar | Technische escalatie, levert migratiescripts |
| Rick van den Dijssel | Product Owner | Feature flag inschakelen (uiterlijk T-1) |
| Remco den Elzen | Support, Topdesk | Major ticket beheren, certificaatupgrade v7 coördineren in Nedap Podium |
| KaHo | Nedap-consultant, zelfstandig inzetbaar | Na validatie handleiding klant 1+2 |
| Jerry | Nedap-consultant, zelfstandig inzetbaar (2 vroege pilots) | Beschikbaar voor ervaringsvragen |
| Rick Nieuweveen | Nedap-consultant, zelfstandig inzetbaar | Betrekken bij opschaling |
| Rutger | Consultant opschaling (begeleiding Rick) | Betrekken na pilotfase |
| Remco Houthuijzen | Consultant opschaling (begeleiding Rick) | Betrekken na pilotfase; auteur Reference Cleaner manual |
| André | Consultant opschaling (begeleiding Rick) | Betrekken na pilotfase |
| Jeroen Smit | Business Consultant | Autorisatiematrix (indien wijziging nodig) |
| Ron | Manager | Beslissingen billing/budget |
| Ronald | Manager delivery | Billing-beslissing vervolgklanten |
| Farid | Manager sales | Billing-beslissing vervolgklanten |

---

## 3. Fase 0 — Testrun op eigen omgeving (Rick)

> **Van toepassing op:** Rick Jongbloed, vóór uitvoering bij klant 1 (Vughterstede)  
> **Doel:** stappen valideren, edge cases ontdekken, definitief stappenplan vaststellen

De volgende stappen kunnen niet volledig op papier worden vastgesteld zonder ze te hebben uitgevoerd. Voer daarom een volledige testrun uit op een eigen testomgeving vóórdat klant 1 ingepland wordt.

- [ ] Productieomgeving van een testklant nabouwen op eigen testomgeving
- [ ] Controleer of Rudolf de connector scripts al heeft bijgewerkt (vereiste vóór Reference Cleaner)
- [ ] Volledig doorlopen van Fase 4 (migratiestappen testomgeving) — noteer elke exacte scriptnaam, klik en volgorde
- [ ] **Beantwoord open beslispunt:** kan de Reference Cleaner draaien op v1-scripts (pre-check vóór migratie werkt dan als volledige run), of heeft hij v2-scripts nodig? → volgorde definitief vastleggen
- [ ] **Beantwoord open vraag:** werkt de Reference Cleaner voor Nedap (array of objects)? → antwoord Remco Houthuijzen verwerken
- [ ] Documenteer het aantal Permission Configurations bij de testklant
- [ ] Valideer dat de `DirectoryCacheLocationsTeams` parameter ongewijzigd is gebleven (verwacht: ja)
- [ ] Noteer foutmeldingen die optreden en hoe je die oplost
- [ ] Controleer entitlement-conversie: zijn statische mappings correct overgekomen?
- [ ] Controleer standaardbereik-gedrag na migratie
- [ ] Verwerk bevindingen in dit document (Fase 4 aanvullen met exacte stappen)
- [ ] Screen recording maken van de volledige run als naslagmateriaal

> ✏️ **Na de testrun:** Vul de exacte scriptnamen en klikpaden in in Fase 4. Pas dit document aan waar nodig. Dat is de definitieve handleiding voor klant 1.

---

## 4. Fase 1 — Voorbereiding per klant

### Tijdlijn

| Wanneer | Actie | Wie |
|---------|-------|-----|
| T-5 werkdagen | Feature flag aanvragen bij Rick van den Dijssel | Consultant |
| T-5 werkdagen | Certificaatversie v7 aanvragen in Nedap Podium — support voert uit, klant moet goedkeuren in Podium | Consultant → Remco den Elzen (support) |
| T-5 werkdagen | Maintenance window communiceren aan klant (zie toelichting hieronder) | Consultant → klant |
| T-3 werkdagen | Klant verzoeken testomgeving te vernieuwen (refresh van productie) | Consultant → klant |
| T-3 werkdagen | Klant bevestigt: testomgeving wordt t/m dag 2 niet handmatig overschreven | Consultant → klant |
| T-3 werkdagen | Bevestigen dat certificaat v7 actief is op de testomgeving | Consultant / Remco |
| T-1 werkdag | Feature flag actief bevestigen bij Rick van den Dijssel | Consultant |
| T-1 werkdag | Wachtacties controleren en oplossen (zie Fase 2) | Consultant |
| Ochtend dag 1 | Pre-flight check (zie Fase 4) | Consultant |

### Checklist voorbereiding

- [ ] Feature flag v1 connector aangevraagd en bevestigd (Rick van den Dijssel)
- [ ] Certificaatversie v7 aangevraagd bij Remco den Elzen (support) en goedgekeurd door klant in Nedap Podium
- [ ] Klant heeft testomgeving ververst (T-3)
- [ ] Klant heeft bevestigd testomgeving niet te overschrijven tot na dag 2
- [ ] Certificaat v7 actief op testomgeving bevestigd
- [ ] Wachtacties = 0 (zie Fase 2)
- [ ] Klantcontact beschikbaar op migratiedag voor vragen en validatie

### Maintenance window — communicatie aan klant

> ⚠️ **HelloID Provisioning gaat in maintenance mode voor de volledige duur van de migratie.** Tijdens die periode worden geen provisioning-acties uitgevoerd: geen accounts aangemaakt of bijgewerkt, geen permissies uitgedeeld of ingetrokken.

Dit geldt voor **zowel dag 1 (testomgeving) als dag 2 (productieomgeving)**. Klant moet dit weten vóórdat de migratie gepland wordt.

Communiceer aan de klant:
- Wanneer de migratie plaatsvindt (dag + tijdvenster)
- Dat HelloID tijdens die periode geen acties uitvoert
- Dat dit betekent dat nieuwe medewerkers, functiewijzigingen of uitdiensttredingen die dag niet automatisch worden verwerkt
- Of ze hiermee akkoord gaan en of er een rustiger moment gewenst is (bijv. buiten piekperiodes)

> ✏️ **TODO:** Remco een concepttekst aanleveren voor de standaard klantcommunicatie (sjabloon nog op te stellen).

### Benodigde informatie van klant

- Naar welke testomgeving mag de connector schrijven?
- Wie is beschikbaar voor validatie na de migratie (dag 1 + dag 2)?
- Zijn er bekende openstaande of gefaalde synchronisaties?

---

## 5. Fase 2 — Wachtacties: check en oplossing

### Wat zijn wachtacties?

HelloID voert account- en permissie-updates uit met afhankelijkheden. De logica is:

1. **Account vóór permissie:** een permissie mag pas uitgedeeld worden als het account in het doelsysteem succesvol aangemaakt is.
2. **Primair vóór secundair:** een accountupdate in een secundair doelsysteem mag pas worden uitgevoerd als de update in het primaire doelsysteem succesvol was.

Als een schakel in die keten is mislukt of overgeslagen, staat de taak in status **Waiting**. Voorbeelden:

- Account uitgedeeld in Nedap Ons, maar bijbehorend HelloID-account niet aangemaakt
- Permissie in wacht omdat het account niet (succesvol) bestaat
- Medewerker uit dienst, maar provisioning-taak nog openstaand
- Secundair systeem wacht op primair systeem dat nooit succesvol was

> ⛔ **Gate:** De Reference Cleaner (eerste stap van de migratie) kan **niet starten** als er wachtacties zijn. Dit is een harde blokkering.

### Stap 1: Controleren op wachtacties

1. Ga in HelloID naar **Provisioning → Entitlements**
2. Open de tab **Actions**
3. Filter op status: **Waiting**
4. Noteer het aantal en de aard per wachtactie (account? permissie? welk systeem? origin toont "Process blocked")

### Stap 2: Oplossen

| Situatie | Aanpak |
|----------|--------|
| Account in secundair systeem, maar niet in primair | Handmatig account aanmaken in primair systeem, dan taak opnieuw verwerken |
| Permissie in wacht, account bestaat niet | Account aanmaken óf taak verwijderen als account niet meer nodig is |
| Medewerker uit dienst, taak is verouderd | Taak verwijderen, account intrekken |
| Afhankelijkheid onduidelijk | Klant raadplegen; indien nodig escaleren naar Rick Jongbloed |

### Stap 3: Hercontrole

Na oplossen opnieuw controleren: **wachtacties = 0** voordat je verdergaat.

### Stap 4: Pre-assessment — hoeveel werk is het?

Voordat je een migratie plant, is het waardevol om te weten hoeveel wachtacties een klant heeft. Dit bepaalt mede of migreren of opnieuw opbouwen de beste aanpak is.

> 📊 **Rapportage: voorlopig niet haalbaar.** Rick Jongbloed heeft dit aangevraagd bij Rick van den Dijssel (PO Provisioning). Antwoord: de data zit niet in Elastic en is niet makkelijk beschikbaar te stellen — dit zou door het dev team gebouwd moeten worden en via de PM lopen. Voor de pilotfase (eerste 3 klanten) is dit niet nodig. **Check handmatig per klant voordat je de migratie inplant.**

### Drempelwaarde — wanneer opnieuw opbouwen?

Als er tientallen wachtacties zijn die structureel niet op te lossen zijn (bijv. door jarenlange data-problemen), overweeg dan of **opnieuw opbouwen** sneller en schoner is dan migreren. Zie Fase 3.

---

## 6. Fase 3 — Scope assessment: migreren of opnieuw opbouwen?

### Standaard: gebruik de migratietool

Gebruik altijd de migratietool, tenzij een van de situaties hieronder van toepassing is. De tool:
- Bewaart bestaande business rules
- Zet entitlement-koppelingen correct om
- Handelt de backward-compatibiliteit van het standaardbereik af

### Wanneer opnieuw opbouwen?

Overweeg opnieuw opbouwen als één of meer van onderstaande punten van toepassing zijn:

- [ ] De connector heeft structurele, langdurige fouten die niet opgelost zijn
- [ ] Er zijn veel wachtacties die niet realistisch op te lossen zijn vóór de migratie
- [ ] De configuratie wijkt sterk af van de standaard ("Frankenstein-configuratie")
- [ ] Opschonen van de bestaande configuratie kost meer tijd dan een herstart
- [ ] De klant wil van de gelegenheid gebruikmaken om de autorisatiematrix opnieuw in te richten

> **Beslisser:** Pilotfase → Rick Jongbloed. Na pilotfase wordt dit een afrekenbare beslisboom voor andere consultants.

> ⚠️ Bij opnieuw opbouwen geldt een andere werkinstructie (nog op te stellen na pilotervaring). Meld dit aan Rick Jongbloed zodat de tijdsplanning aangepast kan worden.

---

## 7. Fase 4 — Migratiestappen (testomgeving)

> Voer deze stappen **altijd eerst op de testomgeving** uit. Ga pas naar productie als alles succesvol is en de klant heeft gevalideerd.

> 📄 **Bron:** gebaseerd op de officiële `Migration.md` in de v2 connector-repository (Tools4everBV/HelloID-Conn-Prov-Target-NedapOns-Users, branch `Nedap-new-permissions-api-standard`). Dit document is als draft gemarkeerd; validatie door de consultancy-tak is vereist.

> 🚫 **Er is geen rollback.** Zodra de migratie gestart is, moet je hem afmaken. Zorg vóór de start dat alle pre-flight checks groen zijn. Twijfel je? Stop en overleg met Rick Jongbloed.

---

### Pre-flight check

- [ ] Wachtacties = 0 (harde check — stop als er nog wachtacties zijn, zie Fase 2)
- [ ] Reference Cleaner pre-check uitgevoerd (zie hieronder)
- [ ] Feature flag actief (Rick van den Dijssel bevestigd)
- [ ] Testomgeving is actueel (recent ververst door klant)
- [ ] Certificaat testomgeving geldig + versie v7
- [ ] Nieuwe v2 scripts klaarstaan (van Rudolf / connector-repo)
- [ ] Nieuwe `configuration.json` klaarstaat (v2 versie)
- [ ] Klantcontact beschikbaar
- [ ] Huidige v1 scripts gebackupt (zie stap 0)

### Reference Cleaner pre-check (vóór de migratie)

> 💡 **Doel:** controleren of de Reference Cleaner straks werkt en of er blocking issues zijn — zonder al iets te wijzigen. Dit geeft zekerheid vóórdat je aan de migratie begint die niet meer teruggedraaid kan worden.

1. Open de Reference Cleaner: `https://[klantnaam].helloid.com/provisioning/#/reference-cleaner/overview`
2. Klik **Start cleaner**
3. Selecteer de **Roles** Permission Configuration
4. Klik **Determine differences**
5. Bekijk de **Reference details** — staan er velden klaar om te verwijderen? Werkt het?
6. Klik **NIET** op Remove fields
7. Klik **Stop cleaner**

> Als de Cleaner start en velden toont → geen blocking issues, je kunt door met de migratie.  
> Als de Cleaner niet start of een fout geeft → stop, overleg met Rudolf. Ga niet verder.

---

### Stap 0 — Scripts backuppen (baseline)

Sla alle huidige v1-scripts op vóórdat je ze vervangt. Dit is de basis voor een vergelijking achteraf.

1. Open de connector in HelloID
2. Kopieer alle scriptinhoud naar een lokale map, bijv. `backup_[klantnaam]_v1_[datum]\`
3. Sla op: `create.ps1`, `update.ps1`, `delete.ps1`, `import.ps1`, alle permissions- en subpermissions-scripts

> 💡 **Toekomstig idee:** tooling bouwen die automatisch een diff maakt tussen de v1-baseline en de nieuwe v2-scripts en per wijziging advies geeft. Nog niet beschikbaar — handmatig opslaan voor nu.

✅ Backup aanwezig → ga door naar stap 1.

---

### Stap 1 — Baseline: force update accounts + permissies

Voer vóór de migratie een volledige force update uit zodat alle accounts en permissies in een schone, actuele staat zijn. Geen errors mogen aanwezig zijn.

1. `[TODO: exacte naam/locatie force update custom event in HelloID invullen na testrun]`
2. Wacht op afronding
3. Controleer: **geen errors**

> *Waarom:* Als er errors zijn vóór de migratie, worden die meegenomen en kan de migratie vastlopen of onjuiste data produceren.

✅ Gate: geen errors → ga door naar stap 2.

---

### Stap 2 — [Migrate]: HelloID connector migreren naar v2

> ✏️ `[TODO: exacte stappen voor het uitvoeren van de HelloID-migratie (connector upgrade v1 → v2) invullen na testrun. Dit is de stap waarmee HelloID de connector formeel omzet naar PowerShell v2-formaat.]`

---

### Stap 4 — Connector-scripts vervangen door v2-versies

Na de migratie moeten alle scripts in de connector overschreven worden met de v2-versies uit de connector-repository.

#### 4a — Configuration.json vervangen

De v2 connector heeft een nieuw `configuration.json` met andere parameters. **Alle velden opnieuw invullen.**

| Parameter | Omschrijving | Voorbeeld/opties |
|-----------|-------------|-----------------|
| `baseUrl` (Environment) | Nedap API-omgeving | `https://api-staging.ons.io` (test) / `https://api.ons.io` (productie) |
| `certificatePath` | Volledig pad naar het `.pfx` certificaatbestand | `C:\...\Nedap-cert.pfx` |
| `certificatePassword` | Wachtwoord van het certificaat | *(geheim)* |
| `grantDefaultScopeMyself` | Standaardbereik instellen op 'de medewerker zelf' | `false` (standaard) |
| `mappingLocations` | Pad naar de locaties-mapping CSV | `C:\...\locations.csv` |
| `mappingTeams` | Pad naar de teams-mapping CSV | `C:\...\teams.csv` |
| `mappingCsvDelimiter` | CSV-scheidingsteken | `;` |
| `explicitMapping` | Expliciete mapping (dept + functie samen) | `false` |
| `ValidateTeamAndLocation` | Valideer teams/locaties tegen Nedap Ons | `false` |
| `DirectoryCacheLocationsTeams` | Pad voor cache van locaties en teams | `C:\...\cache\` |
| `ImportOnlyActiveEmployees` | Alleen actieve medewerkers importeren | `false` |
| `daysBeforeContractStartDate` | Dagen vóór contractstart (bij bovenstaande optie) | `0` |
| `daysAfterContractEndDate` | Dagen ná contracteinde (bij bovenstaande optie) | `0` |

> ⚠️ **Omgeving instelling:** zet `baseUrl` op testomgeving (`https://api-staging.ons.io`) voor dag 1, en op productie (`https://api.ons.io`) voor dag 2.

#### 4b — Scripts vervangen

Overschrijf de volgende scripts met de v2-versies uit de connector-repo:

- [ ] `create.ps1`
- [ ] `update.ps1` *(let op: accountreferentie-fix toevoegen in stap 5)*
- [ ] `delete.ps1`
- [ ] `import.ps1`
- [ ] `permissions/Roles/permissions.ps1`
- [ ] `permissions/Roles/subPermissions.ps1`
- [ ] `permissions/defaultscope/permissions.ps1`
- [ ] `permissions/defaultscope/subPermissions.ps1`
- [ ] `resources/resources.ps1`

#### 4c — Account Import script toevoegen

De v2-connector vereist een Account Import script dat in v1 niet bestond.

- [ ] Account Import script toevoegen (script uit v2 connector-repo)

#### 4d — Mapping-CSVs controleren op nieuwe headers

De mapping-CSVs hebben in v2 nieuwe kolomnamen voor DefaultScope en Roles.

Vereiste kolomnamen:

| CSV | Verplichte kolommen |
|-----|-------------------|
| `mappingLocations` | `HelloIDPrimaryLookupKey`, `HelloIDSecondaryLookupKey`, `NedapLocationIds` |
| `mappingTeams` | `HelloIDPrimaryLookupKey`, `HelloIDSecondaryLookupKey`, `NedapTeamIds` |

- [ ] Controleer de bestaande klant-CSVs: kloppen de kolomnamen?
- [ ] Zo niet: excel2csv script aanpassen met `Select-Object` en een alias voor de kolomnamen om ze automatisch te hernoemen

> **Verwachting:** ~70% van de klanten heeft de kolomnamen al correct staan. Voor de overige 30% is de excel2csv aanpassing voldoende.

✅ Gate: alle scripts en configuratie bijgewerkt → ga door naar stap 5.

---

### Stap 5 — Accountreferentie (ARef) fixen

Na de scriptvervanging moet de accountreferentie gecorrigeerd worden. In v1 werd de referentie opgeslagen als een array van objecten; v2 verwacht een ander formaat.

#### 5a — Fix-code toevoegen aan `update.ps1`

Voeg onderstaande code toe aan het begin van de `update.ps1` (accountreferentie-conversie):

```powershell
if (-not [string]::IsNullOrEmpty($($actionContext.References.Account))) {
    if ($actionContext.References.Account.GetType().fullname -eq 'System.Object[]') {
        $accountReferences = [PSCustomObject]@{}
        foreach ($account in $actionContext.References.Account) {
            $accountReferences | Add-Member @{
                $account.IdentificationNo = @{
                    Uuid             = $account.userUuid
                    IdentificationNo = $account.IdentificationNo
                }
            }
        }
        $actionContext.References.Account = $accountReferences
    }
}
```

#### 5b — Update All Accounts uitvoeren

1. Voer "Update All Accounts" uit `[TODO: exacte locatie/naam in HelloID invullen]`
2. Wacht op afronding
3. Controleer output op errors

#### 5c — Enforcement uitvoeren

1. Voer een enforcement run uit
2. Controleer dat de run succesvol afrondt

#### 5d — Accountreferentie testen

1. Open een testpersoon in HelloID
2. Bekijk de permissions via Preview/DryRun
3. Controleer dat de accountreferentie correct is (format: `IdentificationNo → { Uuid, IdentificationNo }`)

#### 5e — DefaultScope referentie controleren

> ⚠️ In v1 werd de referentie van de DefaultScope-permissie niet gebruikt. In v2 **wordt deze wél gebruikt** en moet de naam exact `DefaultScope` zijn (voor de legacy-permissie).

- [ ] Controleer in de database/audit log dat de DefaultScope-referentie de waarde `DefaultScope` heeft
- [ ] De overige DefaultScope-permissies hebben de volgende vaste referenties:
  - `DefaultScopeLocations`
  - `DefaultScopeTeams`
  - `DefaultScopeAllClients`
  - `DefaultScopeAllEmployees`

✅ Gate: accountreferentie correct, enforcement succesvol → ga door naar stap 5.

---

### Stap 5 — Reference Cleaner (alleen voor Roles)

> 📋 **Gevalideerde procedure:** deze volgorde (migratie eerst, Reference Cleaner daarna) is in de praktijk uitgevoerd door Rudolf en Mauro.

> ❓ **Open beslispunt — volgorde Reference Cleaner:** als de Reference Cleaner ook op v1-scripts kan draaien, is het logischer om hem vóór de migratie uit te voeren (schonere volgorde, minder risico halverwege vast te lopen). Als hij de v2-scripts nodig heeft, moet hij ná de migratie. Remco Houthuijzen (auteur Reference Cleaner manual) is gevraagd om dit te verduidelijken. Definitieve volgorde wordt vastgesteld tijdens de pilot op 18 mei bij Vughterstede. **Pas dit document daarna aan.**

> ⚠️ **Open vraag:** Rick van den Dijssel denkt dat de Reference Cleaner mogelijk niet werkt voor Nedap (array of objects). Rudolf heeft het getest en zegt van wel. Rick Jongbloed onderzoekt dit nog. **Bij twijfel: overleg met Rudolf vóór je verdergaat.**

> *Waarom:* In v1 sloegen de Roles-permissies meerdere velden op in de Identification, waaronder de mutabele velden `DisplayName` en `DisplayNameFull`. Als deze niet verwijderd worden uit de database, falen alle permissiescripts, mislukt de subpermissie-berekening, kloppen datastorage-ID's niet, en toont de audit log onjuiste wijzigingen.

#### 5a — Roles permissions.ps1 controleren

Verifieer dat `DisplayName` en `DisplayNameFull` **niet meer aanwezig** zijn in het `Identification`-object van `permissions/Roles/permissions.ps1`. De v2-versie uit de repo heeft dit al correct:

```powershell
# Correct (v2): alleen immutabele velden
Identification = @{
    id    = $permission.uuid
    $Type = $($Value)
    # DisplayName     = uitgecommentarieerd / verwijderd
    # DisplayNameFull = uitgecommentarieerd / verwijderd
}
```

- [ ] Bevestigd: `DisplayName` en `DisplayNameFull` niet aanwezig in actieve Roles permissions.ps1

#### 5b — Reference Cleaner starten

1. Log in op de HelloID-omgeving van de klant
2. Open in een **tweede browsertab**:
   `https://[klantnaam].helloid.com/provisioning/#/reference-cleaner/overview`
3. Klik op **Start cleaner**

> Terwijl de Cleaner actief is: business rules, target systems, enforcements, entitlement import en reconciliation zijn tijdelijk uitgeschakeld.

> ⚠️ De Cleaner maakt bij elke start een backup. Herstarten **overschrijft** de backup. Bij fouten: **niet herstarten**, Rudolf bellen.

#### 5c — Roles Permission Configuration opruimen

1. Selecteer de **Roles** Permission Configuration (linkerpaneel)
2. Klik op **Determine differences**
3. Controleer de **Reference details**: `DisplayName` en `DisplayNameFull` moeten in de "to remove"-lijst staan
4. Klik op **Remove fields**
5. Controleer de **History** rechts — actie gelogd?

#### 5d — Reference Cleaner stoppen

Klik op **Stop cleaner** rechts bovenaan.

> ⚠️ Browsertab sluiten stopt de Cleaner **niet**. Explicit stoppen verplicht.

#### 5e — Bij fouten

Bij fouten: **niet herstarten** (dit overschrijft de backup). Noteer de foutmelding en neem contact op met Rudolf.

✅ Gate: Reference Cleaner succesvol gestopt, Roles-referenties gecorrigeerd → ga door naar stap 6.

---

### Stap 6 — DefaultScope: force update permissies

> *Waarom:* De DefaultScope-permissies moeten opgeslagen worden in de datastorage van HelloID. Als dit niet gedaan wordt, kunnen DefaultScope-permissies bij een toekomstige revoke **niet** worden ingetrokken, omdat ze niet in de datastorage staan.

1. Voer een force update uit specifiek voor de DefaultScope-permissies `[TODO: exacte naam/locatie in HelloID invullen]`
2. Wacht op afronding
3. Controleer: permissies zijn correct opgeslagen in datastorage

✅ Gate: DefaultScope permissies opgeslagen in datastorage → ga door naar stap 7 (validatie).

---

### Stap 7 — Validatie testomgeving

#### Technische validatie (door consultant)

- [ ] Audit log doorlopen — zijn de verwachte wijzigingen doorgevoerd?
- [ ] Accountreferenties correct (format v2, geen array meer)
- [ ] Roles-permissies: geen `DisplayName`/`DisplayNameFull` meer in de referenties
- [ ] DefaultScope referentie = `DefaultScope` (legacy), overige referenties kloppen
- [ ] DefaultScope permissies aanwezig in datastorage
- [ ] Geen onverwachte errors in de HelloID audit log
- [ ] Permissions preview (DryRun) voor een testpersoon geeft correct resultaat

#### Functionele validatie (door klant)

- [ ] Klant controleert: accounts correct aanwezig?
- [ ] Klant controleert: rollen en permissies correct toegewezen?
- [ ] Klant controleert: bereik (locaties/teams) werkt zoals verwacht?
- [ ] Klant geeft schriftelijk akkoord (e-mail of Topdesk)

✅ Gate: technische én functionele validatie akkoord → mag door naar productie (Fase 5).

---

### Documentatie na dag 1

- [ ] Screen recording beschikbaar van de volledige run
- [ ] Tijdsduur per stap genoteerd
- [ ] Afwijkingen of bijzonderheden genoteerd
- [ ] `[TODO]`-placeholders in dit document aangevuld met werkelijke scriptnamen/locaties
- [ ] Handleiding bijgewerkt waar nodig

---

## 8. Fase 5 — Migratiestappen (productieomgeving)

> Voer productie pas uit als **de klant dag 1 (testomgeving) heeft gevalideerd en akkoord heeft gegeven.**

### Pre-flight check productie

- [ ] Klant heeft testomgeving gevalideerd en schriftelijk akkoord gegeven
- [ ] Wachtacties op productie = 0 (opnieuw controleren — kan veranderd zijn)
- [ ] Klantcontact beschikbaar op migratiedag
- [ ] Klant weet: er is een kort window waarbij synchronisatie onderbroken kan zijn

---

### Stap 1 t/m 6 — Identiek aan testomgeving

Voer dezelfde stappen 1 t/m 6 uit zoals beschreven in Fase 4, nu op de **productieomgeving**.

> ⚠️ Zet bij stap 3a de `baseUrl` op **productie**: `https://api.ons.io`

---

### Stap 4 — Validatie productie

#### Technische validatie (door consultant)

- [ ] Audit log doorlopen — zijn wijzigingen conform de testrun?
- [ ] Spot-check: controleer minimaal 3 medewerkers met bekende rollen en permissies
- [ ] Geen onverwachte fouten in de HelloID audit log
- [ ] Automatische synchronisatie loopt correct

#### Functionele validatie (door klant)

- [ ] Klant bevestigt: accounts, rollen en permissies correct op productie
- [ ] Klant geeft schriftelijk akkoord (e-mail of Topdesk)

---

### Stap 5 — Afsluiting

- [ ] Migratie afsluiten in Topdesk (melden aan Remco voor het major ticket)
- [ ] Bijzonderheden en afwijkingen documenteren in klantdossier
- [ ] Tijdsduur noteren
- [ ] Handleiding bijwerken op basis van bevindingen dag 2

---

## 9. Bijlage A — Entitlement conversie

### Standaardbereik (default scope) — kritieke gedragswijziging

**v1:** standaardbereik werd automatisch toegepast op roluitgifte (alle medewerkers en/of alle cliënten).  
**v2:** standaardbereik geldt **alleen nog voor Cockpit/scope-settings**. Het wordt **niet meer** toegepast op roluitgifte.

**Backward compatibiliteit via de migratietool:**  
De migratietool voegt `all_employees` en `all_clients` direct toe aan de betreffende rollen, zodat het bestaande gedrag bewaard blijft. Dit hoef je als consultant niet handmatig te doen.

**Let op voor handmatig aangemaakte rollen na de migratie:**  
Als een klant na de migratie zelf een nieuwe rol aanmaakt, krijgt die rol niet automatisch het standaardbereik. Dit is een bewuste gedragswijziging in v2. Informeer de klant hier proactief over.

---

### Statische entitlements vs. list_entitlements

**Situatie:** de naam van een statische entitlement in de HelloID-configuratie kan afwijken van de naam die het `list_entitlements`-script teruggeeft.

**Afgesproken aanpak (afstemming Rick Jongbloed + Rudolf):**

- Rudolf voegt de mapping toe in het migratiescript (connector-side)
- Wij voegen **geen `list_entitlements`-script** toe
- De **statische entitlement-mapping blijft actief**
- Onbekende entitlement-namen vallen terug op `default_scope` (defensief gedrag)

**Checklist na migratie:**

- [ ] Zijn alle statische entitlements nog aanwezig?
- [ ] Zijn er entitlements op `default_scope` terechtgekomen die dat eigenlijk niet zouden moeten?
  - Zo ja: controleer of de naam in de statische mapping klopt met de verwachte naam
  - Escaleer naar Rudolf als de mapping ontbreekt
- [ ] Geen actief `list_entitlements`-script (tenzij dit voor deze klant expliciet gewenst is)

> **Voorstel ter bespreking met het team:** deze aanpak (statische mapping, geen `list_entitlements`, mapping in migratiescript door Rudolf) is het voorstel van Rick Jongbloed. Dit moet besproken en gevalideerd worden met KaHo, Jerry, Rutger en André vóór opschaling.

---

## 10. Bijlage B — Troubleshooting

| Symptoom | Eerste stap | Escalatie naar |
|----------|-------------|----------------|
| Reference Cleaner faalt | Foutmelding lezen, **niet herstarten** (backup wordt overschreven), Rudolf bellen | Rudolf |
| Reference Cleaner toont geen velden om te verwijderen | Scripts zijn nog niet bijgewerkt (DisplayName/DisplayNameFull nog aanwezig) → controleer permissions.ps1 | Rudolf |
| Wachtacties niet op te lossen | Klant raadplegen over oorsprong taak | Rick Jongbloed |
| Force update / enforcement geeft errors | Script opnieuw draaien (is idempotent) | Rudolf bij aanhoudend falen |
| Permissiescripts falen na migratie (alle) | Reference Cleaner is niet correct uitgevoerd — DisplayName/DisplayNameFull nog aanwezig in referenties | Rudolf |
| Subpermissie-berekening mislukt | Zelfde oorzaak als boven — Reference Cleaner opnieuw uitvoeren | Rudolf |
| DefaultScope-permissie kan niet worden ingetrokken | Force update DefaultScope is niet uitgevoerd → datastorage mist de permissies | Rudolf |
| Audit log toont onjuiste permissiewijzigingen | Reference Cleaner niet correct uitgevoerd | Rudolf |
| Accountreferentie-fout na migratie | ARef fix-code ontbreekt in `update.ps1`, of Update All Accounts nog niet uitgevoerd | Rudolf |
| Connector schrijft naar verkeerde omgeving | `baseUrl` in configuration.json controleren: staging vs. production | Consultant (self-fix) |
| Certificaatfout | Controleer of certificaat v7 actief is in Nedap Podium en door klant goedgekeurd | Remco den Elzen (support) |
| Mapping-CSV geeft kolomfout | Kolomnamen controleren: `HelloIDPrimaryLookupKey`, `HelloIDSecondaryLookupKey`, `NedapLocationIds`/`NedapTeamIds` | Consultant (self-fix) |

---

## 11. Bijlage C — Contacten en escalatie

| Naam | Rol | Contact |
|------|-----|---------|
| Rick Jongbloed | Projectleider, eindverantwoordelijke pilotfase | Intern |
| Rudolf | Connector-ontwikkelaar, technische vragen | Via Teams/intern |
| Rick van den Dijssel | Feature flag, product owner | Via Teams/intern |
| Remco den Elzen | Topdesk, support, certificaatupgrade Nedap Podium | Via Topdesk |
| Jeroen Smit | Autorisatiematrix (indien aanpassing nodig) | Via Teams/intern |
| Ron | Escalatie management / billing | Via Teams/intern |

---

## Wijzigingslog

| Versie | Datum | Wijziging | Door |
|--------|-------|-----------|------|
| 0.1 | 2026-05-01 | Eerste opzet op basis van gesprek Rudolf + pilotplanning | Rick Jongbloed / Claude |
| 0.2 | 2026-05-01 | Reference Cleaner stappen uitgewerkt op basis van officiële manual v1.2 | Rick Jongbloed / Claude |
| 0.3 | 2026-05-01 | Migratiestappen volledig herschreven op basis van Migration.md in connector-repo (branch Nedap-new-permissions-api-standard); correcte volgorde, ARef-fix, DefaultScope datastorage, configuration.json parameters | Rick Jongbloed / Claude |
| 0.4 | 2026-05-01 | Volgorde gecorrigeerd: Reference Cleaner verplaatst naar vóór [Migrate] (was erna per draft Rudolf); wachtacties pre-assessment rapportage toegevoegd | Rick Jongbloed / Claude |
| 0.5 | 2026-05-01 | Volgorde teruggedraaid naar Rudolf's gevalideerde procedure (migratie eerst, Reference Cleaner daarna); open vraag Reference Cleaner + Nedap array-of-objects toegevoegd; rapportage wachtacties gemarkeerd als niet haalbaar pilotfase; Remco Houthuijzen toegevoegd als consultant | Rick Jongbloed / Claude |
| 0.6 | 2026-05-01 | Rollback-waarschuwing toegevoegd; RC pre-check als verplichte stap vóór migratie; script backup stap 0; maintenance window communicatie aan klant; mapping CSV noot (70%/30%); Rick Nieuweveen, André, Ronald, Farid toegevoegd aan rollen | Rick Jongbloed / Claude |

---

*Dit document wordt opgeslagen in de GitHub-repository: [NedapUsersV2migration](https://github.com/Rick-Jongbloed/NedapUsersV2migration)*
