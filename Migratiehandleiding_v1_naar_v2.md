# Nedap Ons v1 → v2 — Uitvoeringshandleiding

> **Status:** concept — gebaseerd op gesprek met Rudolf Amersfoort + pilotervaring Rick Jongbloed  
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

| Naam | Rol | E-mail | Betrokkenheid |
|------|-----|--------|---------------|
| Rick Jongbloed | Projectleider, uitvoerder klant 1+2 | R.Jongbloed@tools4ever.com | Actief |
| Rudolf Amersfoort | Connector-ontwikkelaar | r.amersfoort@tools4ever.com | Technische escalatie, levert migratiescripts |
| Rick van den Dijssel | Product Owner | R.vdDijssel@tools4ever.com | Feature flag inschakelen (uiterlijk T-1) |
| Remco den Elzen | Support, Topdesk | R.denElzen@tools4ever.com | Major ticket beheren, certificaatupgrade v7 coördineren in Nedap Podium |
| KaHo Man | Nedap-consultant, zelfstandig inzetbaar | K.Man@tools4ever.com | Na validatie handleiding klant 1+2 |
| Jerry Breek | Nedap-consultant, zelfstandig inzetbaar (2 vroege pilots) | j.breek@tools4ever.com | Beschikbaar voor ervaringsvragen |
| Rick Nieuweveen | Nedap-consultant, zelfstandig inzetbaar | R.Nieuweveen@tools4ever.com | Betrekken bij opschaling |
| Rutger Scholte Lubberink | Consultant opschaling (begeleiding Rick) | R.ScholteLubberink@tools4ever.com | Betrekken na pilotfase |
| Remco Houthuijzen | Consultant opschaling (begeleiding Rick) | r.houthuijzen@tools4ever.com | Betrekken na pilotfase; auteur Reference Cleaner manual |
| André Boonstra | Consultant opschaling (begeleiding Rick) | a.boonstra@tools4ever.com | Betrekken na pilotfase |
| Jeroen Smit | Business Consultant | J.Smit@tools4ever.com | Autorisatiematrix (indien wijziging nodig) |
| Ron Kuper | Manager | R.Kuper@tools4ever.com | Beslissingen billing/budget |
| Ronald Kamerbeek | Manager delivery | R.Kamerbeek@tools4ever.com | Billing-beslissing vervolgklanten |
| Farid Ouachour | Manager sales | F.Ouachour@tools4ever.com | Billing-beslissing vervolgklanten |

---

## 3. Fase 0 — Testrun op eigen omgeving (Rick)

> **Van toepassing op:** Rick Jongbloed, vóór uitvoering bij klant 1 (Vughterstede)  
> **Doel:** stappen valideren, edge cases ontdekken, definitief stappenplan vaststellen

De volgende stappen kunnen niet volledig op papier worden vastgesteld zonder ze te hebben uitgevoerd. Voer daarom een volledige testrun uit op een eigen testomgeving vóórdat klant 1 ingepland wordt.

- [ ] Productieomgeving van een testklant nabouwen op eigen testomgeving
- [ ] Controleer of Rudolf Amersfoort de connector scripts al heeft bijgewerkt (vereiste vóór Reference Cleaner)
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
- [ ] Nieuwe v2 scripts klaarstaan (van Rudolf Amersfoort / connector-repo)
- [ ] Klantcontact beschikbaar
- [ ] Huidige v1 scripts gebackupt (zie Stap 0a)
- [ ] Maatwerk check uitgevoerd: klant-specifieke aanpassingen in v1 geïdentificeerd (zie Stap 0b)

### Reference Cleaner pre-check (vóór de migratie)

> 💡 **Doel:** controleren of er blocking taken zijn die de migratie zouden blokkeren. Dit is een GO/NOGO check vóórdat je de onomkeerbare migratiestap start.

**Stap 1: Cleaner starten en controleren**

1. Open de Reference Cleaner: `https://[klantnaam].helloid.com/provisioning/#/reference-cleaner/overview`
2. Klik **Start cleaner**
3. Selecteer de **Roles** Permission Configuration
4. Klik **Determine differences**
5. Bekijk de **Reference details**

**Stap 2: GO/NOGO beoordelen**

| Uitkomst | Actie |
|----------|-------|
| Geen blocking taken | ✅ **GO** — ga door naar Stap 3 hieronder |
| Blocking taken aanwezig | ⛔ **NOGO** — los de taken op met de klant (zie Fase 2), doe daarna de pre-check opnieuw |

> ⚠️ Klik **NIET** op **Remove fields** tijdens de pre-check. Dat is voor de echte Reference Cleaner run ná de migratie (Stap 4).

> Als de Cleaner niet start of een fout geeft → stop, overleg met Rudolf Amersfoort. Ga niet verder.

**Stap 3: Reference Cleaner sluiten**

Klik **Stop cleaner** rechts bovenaan.

> ⚠️ Browsertab sluiten stopt de Cleaner **niet**. Expliciet stoppen is verplicht.

---

### Stap 0 — Backup en maatwerk check

#### 0a — Scripts backuppen (baseline)

Sla alle huidige v1-scripts op vóórdat je ze vervangt. Dit is de basis voor een vergelijking achteraf.

1. Open de connector in HelloID
2. Kopieer alle scriptinhoud naar een lokale map, bijv. `backup_[klantnaam]_v1_[datum]\`
3. Sla op: `create.ps1`, `update.ps1`, `delete.ps1`, `import.ps1`, alle permissions- en subpermissions-scripts

> 💡 **Toekomstig idee:** tooling bouwen die automatisch een diff maakt tussen de v1-baseline en de nieuwe v2-scripts en per wijziging advies geeft. Nog niet beschikbaar — handmatig opslaan voor nu.

#### 0b — Maatwerk check

De migratiewizard maakt automatisch een **readonly backup** van de v1-inrichting (zichtbaar in de connector na migratie). Maar controleer vóór de migratie al of er klant-specifiek maatwerk in de v1-connector zit.

- [ ] Bekijk alle scripts in de v1-connector: zijn er aanpassingen gedaan die niet in de standaard v2-repo zitten?
- [ ] Noteer afwijkingen en bespreek met Rick Jongbloed of Rudolf Amersfoort of deze meegenomen moeten worden in de v2-versie

> ⚠️ Na de migratie is de v1-inrichting als readonly backup inzichtelijk in de connector. Je kunt er dan nog in terugkijken, maar niet meer bewerken.

✅ Backup aanwezig, maatwerk geïdentificeerd → ga door naar Stap 1.

---

### Stap 1 — Baseline: force update accounts + permissies

Voer vóór de migratie een volledige force update uit zodat alle accounts en permissies in een schone, actuele staat zijn. Geen errors mogen aanwezig zijn.

1. `[TODO: exacte naam/locatie force update custom event in HelloID invullen na testrun]`
2. Wacht op afronding
3. Controleer: **geen errors**

> *Waarom:* Als er errors zijn vóór de migratie, worden die meegenomen en kan de migratie vastlopen of onjuiste data produceren.

✅ Gate: geen errors → ga door naar stap 2.

---

### Stap 2 — V2 migratie uitvoeren (HelloID migration wizard)

> ⚠️ **Geen rollback.** Zodra je "Confirm" klikt is de migratie onomkeerbaar. Zorg dat alle pre-flight checks groen zijn.

> 💡 De migratiewizard maakt automatisch een **readonly backup** van de v1-inrichting (zichtbaar in de connector na migratie).

1. Open de connector in HelloID
2. Ga naar tab **General**
3. Klik **Migrate system**
4. Bevestig de melding: *"Are you sure you want to start migrating the system?"* → klik **Confirm**

> ⛔ Vanaf dit moment is de migratie gestart en niet meer terug te draaien.

---

#### Tab Fields

- [ ] Field mapping script plaatsen (uit v2 connector-repo)

---

#### Tab Account

- [ ] Account Create script vervangen
- [ ] Account Update script vervangen *(let op: ARef-fix toevoegen ná de wizard — zie Stap 3)*
- [ ] Account Delete script vervangen
- [ ] Account Data Import script vervangen
- [ ] Custom connector configuration (configuration.json) vervangen — **alle velden opnieuw invullen**

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

> ⚠️ **Omgeving:** zet `baseUrl` op testomgeving (`https://api-staging.ons.io`) voor dag 1, en op productie (`https://api.ons.io`) voor dag 2.

---

#### Tab Permissions

**Default scope permissie-definitie:**

- [ ] Static permission: laat staan zoals het is
- [ ] Omzetten naar **Handle all actions script**
- [ ] Subpermission script plaatsen
- [ ] Toggle **Context Data storage** aanzetten

**Roles permissie-definitie:**

- [ ] Import permission script vervangen
- [ ] Handle all actions script vervangen

---

#### Tab Resource

- [ ] Cache script vervangen (resources.ps1)

---

#### Tab Correlation

Configureer de correlatie conform de [connector ReadMe](https://github.com/Tools4everBV/HelloID-Conn-Prov-Target-NedapOns-Users-ReadMe/tree/Nedap-new-permissions-api-standard#correlation-configuration):

| Setting | Value |
|---------|-------|
| Enable correlation | `True` |
| Person correlation field | `ExternalId` |
| Account correlation field | `_outputInfo.externalId` |

- [ ] Correlatie geconfigureerd en opgeslagen

---

#### Mapping-CSVs controleren op nieuwe headers

De mapping-CSVs hebben in v2 nieuwe kolomnamen.

| CSV | Verplichte kolommen |
|-----|-------------------|
| `mappingLocations` | `HelloIDPrimaryLookupKey`, `HelloIDSecondaryLookupKey`, `NedapLocationIds` |
| `mappingTeams` | `HelloIDPrimaryLookupKey`, `HelloIDSecondaryLookupKey`, `NedapTeamIds` |

- [ ] Controleer de bestaande klant-CSVs: kloppen de kolomnamen?
- [ ] Zo niet: excel2csv script aanpassen met `Select-Object` en een alias voor de kolomnamen

> **Verwachting:** ~70% van de klanten heeft de kolomnamen al correct staan. Voor de overige 30% is de excel2csv aanpassing voldoende.

✅ Gate: wizard volledig doorlopen, alle scripts geplaatst, correlatie geconfigureerd → ga door naar Stap 3.

---

### Stap 3 — Accountreferentie (ARef) fixen

Na de migration wizard moet de accountreferentie gecorrigeerd worden. In v1 werd de referentie opgeslagen als een array van objecten; v2 verwacht een ander formaat.

#### 3a — Fix-code toevoegen aan `update.ps1`

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

#### 3b — Update All Accounts uitvoeren

1. Voer "Update All Accounts" uit `[TODO: exacte locatie/naam in HelloID invullen]`
2. Wacht op afronding
3. Controleer output op errors

#### 3c — Enforcement uitvoeren

1. Voer een enforcement run uit
2. Controleer dat de run succesvol afrondt

#### 3d — Accountreferentie testen

1. Open een testpersoon in HelloID
2. Bekijk de permissions via Preview/DryRun
3. Controleer dat de accountreferentie correct is (format: `IdentificationNo → { Uuid, IdentificationNo }`)

#### 3e — DefaultScope referentie controleren

> ⚠️ In v1 werd de referentie van de DefaultScope-permissie niet gebruikt. In v2 **wordt deze wél gebruikt** en moet de naam exact `DefaultScope` zijn (voor de legacy-permissie).

- [ ] Controleer in de database/audit log dat de DefaultScope-referentie de waarde `DefaultScope` heeft
- [ ] De overige DefaultScope-permissies hebben de volgende vaste referenties:
  - `DefaultScopeLocations`
  - `DefaultScopeTeams`
  - `DefaultScopeAllClients`
  - `DefaultScopeAllEmployees`

✅ Gate: accountreferentie correct, enforcement succesvol → ga door naar Stap 4.

---

### Stap 4 — Reference Cleaner (alleen voor Roles)

> 📋 **Gevalideerde procedure:** deze volgorde (migratie eerst, Reference Cleaner daarna) is in de praktijk uitgevoerd door Rudolf Amersfoort en Mauro.

> ❓ **Open beslispunt — volgorde Reference Cleaner:** als de Reference Cleaner ook op v1-scripts kan draaien, is het logischer om hem vóór de migratie uit te voeren (schonere volgorde, minder risico halverwege vast te lopen). Als hij de v2-scripts nodig heeft, moet hij ná de migratie. Remco Houthuijzen (auteur Reference Cleaner manual) is gevraagd om dit te verduidelijken. Definitieve volgorde wordt vastgesteld tijdens de pilot op 18 mei bij Vughterstede. **Pas dit document daarna aan.**

> ⚠️ **Open vraag:** Rick van den Dijssel denkt dat de Reference Cleaner mogelijk niet werkt voor Nedap (array of objects). Rudolf Amersfoort heeft het getest en zegt van wel. Rick Jongbloed onderzoekt dit nog. **Bij twijfel: overleg met Rudolf Amersfoort vóór je verdergaat.**

> *Waarom:* In v1 sloegen de Roles-permissies meerdere velden op in de Identification, waaronder de mutabele velden `DisplayName` en `DisplayNameFull`. Als deze niet verwijderd worden uit de database, falen alle permissiescripts, mislukt de subpermissie-berekening, kloppen datastorage-ID's niet, en toont de audit log onjuiste wijzigingen.

#### 4a — Roles permissions.ps1 controleren

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

#### 4b — Reference Cleaner starten

1. Log in op de HelloID-omgeving van de klant
2. Open in een **tweede browsertab**:
   `https://[klantnaam].helloid.com/provisioning/#/reference-cleaner/overview`
3. Klik op **Start cleaner**

> Terwijl de Cleaner actief is: business rules, target systems, enforcements, entitlement import en reconciliation zijn tijdelijk uitgeschakeld.

> ⚠️ De Cleaner maakt bij elke start een backup. Herstarten **overschrijft** de backup. Bij fouten: **niet herstarten**, Rudolf Amersfoort bellen.

#### 4c — Roles Permission Configuration opruimen

1. Selecteer de **Roles** Permission Configuration (linkerpaneel)
2. Klik op **Determine differences**
3. Controleer de **Reference details**: `DisplayName` en `DisplayNameFull` moeten in de "to remove"-lijst staan
4. Klik op **Remove fields**
5. Controleer de **History** rechts — actie gelogd?

#### 4d — Reference Cleaner stoppen

Klik op **Stop cleaner** rechts bovenaan.

> ⚠️ Browsertab sluiten stopt de Cleaner **niet**. Explicit stoppen verplicht.

#### 4e — Bij fouten

Bij fouten: **niet herstarten** (dit overschrijft de backup). Noteer de foutmelding en neem contact op met Rudolf Amersfoort.

✅ Gate: Reference Cleaner succesvol gestopt, Roles-referenties gecorrigeerd → ga door naar Stap 5.

---

### Stap 5 — DefaultScope: force update permissies

> *Waarom:* De DefaultScope-permissies moeten opgeslagen worden in de datastorage van HelloID. Als dit niet gedaan wordt, kunnen DefaultScope-permissies bij een toekomstige revoke **niet** worden ingetrokken, omdat ze niet in de datastorage staan.

1. Voer een force update uit specifiek voor de DefaultScope-permissies `[TODO: exacte naam/locatie in HelloID invullen]`
2. Wacht op afronding
3. Controleer: permissies zijn correct opgeslagen in datastorage

✅ Gate: DefaultScope permissies opgeslagen in datastorage → ga door naar Stap 6 (validatie).

---

### Stap 6 — Validatie testomgeving

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

> ⚠️ Zet in Stap 2 (Tab Account → configuration.json) de `baseUrl` op **productie**: `https://api.ons.io`

---

### Stap 6 — Validatie productie

#### Technische validatie (door consultant)

- [ ] Audit log doorlopen — zijn wijzigingen conform de testrun?
- [ ] Spot-check: controleer minimaal 3 medewerkers met bekende rollen en permissies
- [ ] Geen onverwachte fouten in de HelloID audit log
- [ ] Automatische synchronisatie loopt correct

#### Functionele validatie (door klant)

- [ ] Klant bevestigt: accounts, rollen en permissies correct op productie
- [ ] Klant geeft schriftelijk akkoord (e-mail of Topdesk)

---

### Stap 7 — Afsluiting

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

- Rudolf Amersfoort voegt de mapping toe in het migratiescript (connector-side)
- Wij voegen **geen `list_entitlements`-script** toe
- De **statische entitlement-mapping blijft actief**
- Onbekende entitlement-namen vallen terug op `default_scope` (defensief gedrag)

**Checklist na migratie:**

- [ ] Zijn alle statische entitlements nog aanwezig?
- [ ] Zijn er entitlements op `default_scope` terechtgekomen die dat eigenlijk niet zouden moeten?
  - Zo ja: controleer of de naam in de statische mapping klopt met de verwachte naam
  - Escaleer naar Rudolf Amersfoort als de mapping ontbreekt
- [ ] Geen actief `list_entitlements`-script (tenzij dit voor deze klant expliciet gewenst is)

> **Voorstel ter bespreking met het team:** deze aanpak (statische mapping, geen `list_entitlements`, mapping in migratiescript door Rudolf) is het voorstel van Rick Jongbloed. Dit moet besproken en gevalideerd worden met KaHo Man, Jerry Breek, Rutger Scholte Lubberink en André Boonstra vóór opschaling.

---

## 10. Bijlage B — Troubleshooting

| Symptoom | Eerste stap | Escalatie naar |
|----------|-------------|----------------|
| Reference Cleaner faalt | Foutmelding lezen, **niet herstarten** (backup wordt overschreven), Rudolf Amersfoort bellen | Rudolf Amersfoort |
| Reference Cleaner toont geen velden om te verwijderen | Scripts zijn nog niet bijgewerkt (DisplayName/DisplayNameFull nog aanwezig) → controleer permissions.ps1 | Rudolf Amersfoort |
| Wachtacties niet op te lossen | Klant raadplegen over oorsprong taak | Rick Jongbloed |
| Force update / enforcement geeft errors | Script opnieuw draaien (is idempotent) | Rudolf Amersfoort bij aanhoudend falen |
| Permissiescripts falen na migratie (alle) | Reference Cleaner is niet correct uitgevoerd — DisplayName/DisplayNameFull nog aanwezig in referenties | Rudolf Amersfoort |
| Subpermissie-berekening mislukt | Zelfde oorzaak als boven — Reference Cleaner opnieuw uitvoeren | Rudolf Amersfoort |
| DefaultScope-permissie kan niet worden ingetrokken | Force update DefaultScope is niet uitgevoerd → datastorage mist de permissies | Rudolf Amersfoort |
| Audit log toont onjuiste permissiewijzigingen | Reference Cleaner niet correct uitgevoerd | Rudolf Amersfoort |
| Accountreferentie-fout na migratie | ARef fix-code ontbreekt in `update.ps1`, of Update All Accounts nog niet uitgevoerd | Rudolf Amersfoort |
| Connector schrijft naar verkeerde omgeving | `baseUrl` in configuration.json con