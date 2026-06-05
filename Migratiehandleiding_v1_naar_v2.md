# Nedap Ons v1 → v2 — Migratiehandleiding

> **Status:** gevalideerd op testomgeving Vughterstede — 18 mei 2026  
> **Doelgroep:** IAM Consultants Tools4ever  
> **Doel:** 4 uur per klant op productie, zonder testomgeving (vanaf klant 4+)

> 📋 **Uitvoeringskaarten:** voor de stap-voor-stap uitvoering op de migratiedag, gebruik:
> - [Uitvoeringskaart Testmigratie](Uitvoeringskaart_Testmigratie.md) — voor de testomgeving
> - [Uitvoeringskaart Productiemigratie](Uitvoeringskaart_Productiemigratie.md) — voor de productieomgeving
>
> Dit document bevat de achtergrond, technische context en besluitvorming.

---

## Inhoudsopgave

1. [Achtergrond en scope](#1-achtergrond-en-scope)
2. [Rollen en contacten](#2-rollen-en-contacten)
3. [Fase 0 — Testrun op eigen omgeving (Rick)](#3-fase-0--testrun-op-eigen-omgeving-rick)
4. [Fase 1 — Voorbereiding per klant](#4-fase-1--voorbereiding-per-klant)
5. [Fase 2 — Pending actions: check en oplossing](#5-fase-2--pending-actions-check-en-oplossing)
6. [Fase 3 — Scope assessment: migreren of opnieuw opbouwen?](#6-fase-3--scope-assessment-migreren-of-opnieuw-opbouwen)
7. [Fase 4 — Technische achtergrond migratiestappen](#7-fase-4--technische-achtergrond-migratiestappen)
8. [Fase 5 — Productieomgeving](#8-fase-5--productieomgeving)
9. [Bijlage A — Entitlement conversie](#9-bijlage-a--entitlement-conversie)
10. [Bijlage B — Troubleshooting](#10-bijlage-b--troubleshooting)
11. [Bijlage C — Contacten en escalatie](#11-bijlage-c--contacten-en-escalatie)

---

## 1. Achtergrond en scope

### Wat is dit?

Nedap Ons stapt over op een nieuw autorisatiemodel. Tools4ever heeft hiervoor een nieuwe v2-connector ontwikkeld. Dit document is de migratiehandleiding voor het migreren van ~65 klanten van de v1- naar de v2-connector.

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
| Rutger Scholte Lubberink | Consultant opschaling | R.ScholteLubberink@tools4ever.com | Betrekken na pilotfase |
| Remco Houthuijzen | Consultant opschaling | r.houthuijzen@tools4ever.com | Betrekken na pilotfase; auteur Reference Cleaner manual |
| André Boonstra | Consultant opschaling | a.boonstra@tools4ever.com | Betrekken na pilotfase |
| Jeroen Smit | Business Consultant | J.Smit@tools4ever.com | Autorisatiematrix (indien wijziging nodig) |
| Ron Kuper | Manager | R.Kuper@tools4ever.com | Beslissingen billing/budget |
| Ronald Kamerbeek | Manager delivery | R.Kamerbeek@tools4ever.com | Billing-beslissing vervolgklanten |
| Farid Ouachour | Manager sales | F.Ouachour@tools4ever.com | Billing-beslissing vervolgklanten |

---

## 3. Fase 0 — Testrun op eigen omgeving (Rick)

> **Van toepassing op:** Rick Jongbloed, vóór uitvoering bij klant 1 (Vughterstede)  
> **Status: ✅ Afgerond — testmigratie uitgevoerd op testomgeving Vughterstede, 18 mei 2026**  
> Zie `Testmigratie_Vughterstede_Dag1_Bevindingen.md` voor de volledige sessienota.

- [x] Productieomgeving van testklant Vughterstede nagebouwd op testomgeving (zelfde dag, 07:30)
- [x] V2-scripts beschikbaar en gevalideerd (branch `Nedap-new-permissions-api-standard`)
- [x] Volledig doorlopen van Fase 4 (migratiestappen testomgeving) — stappen vastgelegd in dit document
- [x] **Open beslispunt beantwoord:** Reference Cleaner draait zowel vóór de migratie (pre-check op V1, om blocking taken te vinden) als ná de migratie (op V2, om `DisplayName`/`DisplayNameFull` te verwijderen). Beide runs zijn aparte stappen.
- [x] **Open vraag beantwoord:** Reference Cleaner werkt voor Nedap (array of objects) — bevestigd in praktijk.
- [x] `DirectoryCacheLocationsTeams` parameter ongewijzigd gebleven — bevestigd.
- [x] Entitlement-conversie gecontroleerd: statische mappings correct overgekomen.
- [x] Standaardbereik-gedrag gevalideerd.
- [x] Bevindingen verwerkt in dit document.
- [x] Screen recording beschikbaar (Vughterstede dag 1).

---

## 4. Fase 1 — Voorbereiding per klant

> 📋 Gebruik de [Uitvoeringskaart Testmigratie](Uitvoeringskaart_Testmigratie.md) voor de volledige voorbereidingschecklist met tijdlabels. Onderstaande tijdlijn en toelichting geven de context.

### Tijdlijn

| Wanneer | Actie | Wie |
|---------|-------|-----|
| T-5 werkdagen | Feature flag aanvragen bij Rick van den Dijssel | Consultant |
| T-5 werkdagen | Certificaatversie v7 aanvragen in Nedap Podium — support voert uit, klant moet goedkeuren in Podium | Consultant → Remco den Elzen (support) |
| T-5 werkdagen | Maintenance window communiceren aan klant | Consultant → klant |
| T-3 werkdagen | Klant verzoeken testomgeving te vernieuwen (refresh van productie) | Consultant → klant |
| T-3 werkdagen | Klant bevestigt: testomgeving wordt t/m dag 2 niet handmatig overschreven | Consultant → klant |
| T-3 werkdagen | Bevestigen dat certificaat v7 actief is op de testomgeving | Consultant / Remco |
| T-1 werkdag | Feature flag actief bevestigen bij Rick van den Dijssel | Consultant |
| T-1 werkdag | Pending actions controleren en oplossen (zie Fase 2) | Consultant |
| Ochtend dag 1 | Pre-flight check (zie uitvoeringskaart) | Consultant |

### Maintenance window — communicatie aan klant

> ⚠️ **HelloID Provisioning gaat in maintenance mode voor de volledige duur van de migratie.** Tijdens die periode worden geen provisioning-acties uitgevoerd: geen accounts aangemaakt of bijgewerkt, geen permissies uitgedeeld of ingetrokken.

Dit geldt voor **zowel dag 1 (testomgeving) als dag 2 (productieomgeving)**. Klant moet dit weten vóórdat de migratie gepland wordt.

Communiceer aan de klant:
- Wanneer de migratie plaatsvindt (dag + tijdvenster)
- Dat HelloID tijdens die periode geen acties uitvoert
- Dat nieuwe medewerkers, functiewijzigingen of uitdiensttredingen die dag niet automatisch worden verwerkt
- Of ze hiermee akkoord gaan en of er een rustiger moment gewenst is (bijv. buiten piekperiodes)

Zie ook: [Klantcommunicatie_Migratie_Aankondiging.md](Klantcommunicatie_Migratie_Aankondiging.md)

### Benodigde informatie van klant

- Naar welke testomgeving mag de connector schrijven?
- Wie is beschikbaar voor validatie na de migratie (dag 1 + dag 2)?
- Zijn er bekende openstaande of gefaalde synchronisaties?

---

## 5. Fase 2 — Pending actions: check en oplossing

> 📋 De uitvoering van de pending actions-check staat in de [Uitvoeringskaart Testmigratie](Uitvoeringskaart_Testmigratie.md) (Stap D) en [Uitvoeringskaart Productiemigratie](Uitvoeringskaart_Productiemigratie.md) (Stap B). Onderstaande toelichting geeft de achtergrond.

### Wat zijn pending actions?

HelloID voert account- en permissie-updates uit met afhankelijkheden. De logica is:

1. **Account vóór permissie:** een permissie mag pas uitgedeeld worden als het account in het doelsysteem succesvol aangemaakt is.
2. **Primair vóór secundair:** een accountupdate in een secundair doelsysteem mag pas worden uitgevoerd als de update in het primaire doelsysteem succesvol was.

Als een schakel in die keten is mislukt of overgeslagen, staat de taak als pending actions. Voorbeelden:

- Account uitgedeeld in Nedap Ons, maar bijbehorend HelloID-account niet aangemaakt
- Permissie in wacht omdat het account niet (succesvol) bestaat
- Medewerker uit dienst, maar provisioning-taak nog openstaand
- Secundair systeem wacht op primair systeem dat nooit succesvol was

> ⛔ **Gate:** De Reference Cleaner (eerste stap van de migratie) kan **niet starten** als er pending actions zijn. Dit is een harde blokkering.

### Oplossingsrichtingen

| Situatie | Aanpak |
|----------|--------|
| Account in secundair systeem, maar niet in primair | Handmatig account aanmaken in primair systeem, dan taak opnieuw verwerken |
| Permissie in wacht, account bestaat niet | Account aanmaken óf taak verwijderen als account niet meer nodig is |
| Medewerker uit dienst, taak is verouderd | Taak verwijderen, account intrekken |
| Afhankelijkheid onduidelijk | Klant raadplegen; indien nodig escaleren naar Rick Jongbloed |

### Pre-assessment — hoeveel werk is het?

Voordat je een migratie plant, is het waardevol om te weten hoeveel pending actions een klant heeft. Dit bepaalt mede of migreren of opnieuw opbouwen de beste aanpak is.

> 📊 **Rapportage: voorlopig niet haalbaar.** De data zit niet in Elastic en is niet makkelijk beschikbaar te stellen. **Check handmatig per klant voordat je de migratie inplant.**

### Drempelwaarde — wanneer opnieuw opbouwen?

Als er tientallen pending actions zijn die structureel niet op te lossen zijn (bijv. door jarenlange data-problemen), overweeg dan of **opnieuw opbouwen** sneller en schoner is dan migreren. Zie Fase 3.

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
- [ ] Er zijn veel pending actions die niet realistisch op te lossen zijn vóór de migratie
- [ ] De configuratie wijkt sterk af van de standaard ("Frankenstein-configuratie")
- [ ] Opschonen van de bestaande configuratie kost meer tijd dan een herstart
- [ ] De klant wil van de gelegenheid gebruikmaken om de autorisatiematrix opnieuw in te richten

> **Beslisser:** Pilotfase → Rick Jongbloed. Na pilotfase wordt dit een afrekenbare beslisboom voor andere consultants.

> ⚠️ Bij opnieuw opbouwen geldt een andere werkinstructie (nog op te stellen na pilotervaring). Meld dit aan Rick Jongbloed zodat de tijdsplanning aangepast kan worden.

---

## 7. Fase 4 — Technische achtergrond migratiestappen

> 📋 **Uitvoering:** gebruik de [Uitvoeringskaart Testmigratie](Uitvoeringskaart_Testmigratie.md) voor de stap-voor-stap uitvoering op de migratiedag. Dit hoofdstuk bevat de technische achtergrond en referentie-informatie.

> 🚫 **Er is geen rollback.** Zodra de migratie gestart is, moet je hem afmaken. Zorg vóór de start dat alle pre-flight checks groen zijn.

> 📄 **Bron:** gebaseerd op de officiële `Migration.md` in de v2 connector-repository ([Tools4everBV/HelloID-Conn-Prov-Target-NedapOns-Users](https://github.com/Tools4everBV/HelloID-Conn-Prov-Target-NedapOns-Users/tree/Nedap-new-permissions-api-standard), branch `Nedap-new-permissions-api-standard`).

---

### Configuratieparameters (referentie)

Bij het vervangen van de configuratie in de migration wizard zijn dit de parameters en hun verwachte waarden:

| Parameter | Omschrijving | Testomgeving | Productieomgeving |
|-----------|-------------|--------------|-------------------|
| `baseUrl` (Environment) | Nedap API-omgeving | `https://api-staging.ons.io` | `https://api.ons.io` |
| `certificatePath` | Volledig pad naar het `.pfx` certificaatbestand | Pad in testmap | Pad in productiemap |
| `certificatePassword` | Wachtwoord van het certificaat | *(geheim)* | *(geheim)* |
| `grantDefaultScopeMyself` | Standaardbereik op 'de medewerker zelf' | Zie boolean-check voorbereiding | Zie boolean-check voorbereiding |
| `mappingLocations` | Pad naar de locaties-mapping CSV | Pad in testmap | Pad in productiemap |
| `mappingTeams` | Pad naar de teams-mapping CSV | Pad in testmap | Pad in productiemap |
| `csvDelimiter` | CSV-scheidingsteken | `;` | `;` |
| `explicitMapping` | Expliciete mapping (dept + functie samen) | `false` | `false` |
| `ValidateTeamAndLocation` | Valideer teams/locaties tegen Nedap Ons | `false` | `false` |
| `DirectoryCacheLocationsTeams` | Pad voor cache van locaties en teams | Pad in testmap | Pad in productiemap |
| `ImportOnlyActiveEmployees` | Alleen actieve medewerkers importeren | `false` | `false` |
| `daysBeforeContractStartDate` | Dagen vóór contractstart | `0` | `0` |
| `daysAfterContractEndDate` | Dagen ná contracteinde | `0` | `0` |

### Mapping-CSVs — kolomnamen v2

De mapping-CSVs hebben in v2 nieuwe kolomnamen:

| CSV | Verplichte kolommen |
|-----|-------------------|
| `mappingLocations` | `HelloIDPrimaryLookupKey`, `HelloIDSecondaryLookupKey`, `NedapLocationIds` |
| `mappingTeams` | `HelloIDPrimaryLookupKey`, `HelloIDSecondaryLookupKey`, `NedapTeamIds` |

> **Verwachting:** ~70% van de klanten heeft de kolomnamen al correct staan. Voor de overige 30% is de exportscript-aanpassing voldoende — zie de uitvoeringskaart voor de instructies.

---

### Accountreferentie (ARef) — technische achtergrond

Na de migration wizard moet de accountreferentie gecorrigeerd worden. In v1 werd de referentie opgeslagen als een array van objecten; v2 verwacht een ander formaat. De fix-code hiervoor is standaard opgenomen in de v2-scripts in de connector-repo — er is geen handmatige actie nodig.

#### DefaultScope referentie

> ⚠️ In v1 werd de referentie van de DefaultScope-permissie niet gebruikt. In v2 **wordt deze wél gebruikt** en moet de naam exact `DefaultScope` zijn (voor de legacy-permissie).

De overige DefaultScope-permissies hebben de volgende vaste referenties:
- `DefaultScopeLocations`
- `DefaultScopeTeams`
- `DefaultScopeAllClients`
- `DefaultScopeAllEmployees`

---

### Reference Cleaner — achtergrond

> ✅ **Volgorde bevestigd (pilot 18 mei 2026):** de Reference Cleaner draait op twee momenten: (1) vóór de migratie als pre-check op V1 om blocking taken te identificeren, en (2) ná de migratie op V2 om `DisplayName` en `DisplayNameFull` te verwijderen. Dit zijn twee aparte runs met een ander doel.

> ✅ **Bevestigd:** de Reference Cleaner werkt correct voor Nedap (array of objects).

**Waarom de post-migratie Reference Cleaner noodzakelijk is:**
In v1 sloegen de Roles-permissies meerdere velden op in de Identification, waaronder de mutabele velden `DisplayName` en `DisplayNameFull`. Als deze niet verwijderd worden uit de database, falen alle permissiescripts, mislukt de subpermissie-berekening, kloppen datastorage-ID's niet, en toont de audit log onjuiste wijzigingen.

Het v2 Roles-permissiescript heeft dit al correct — `DisplayName` en `DisplayNameFull` zijn uitgecommentarieerd:

```powershell
# Correct (v2): alleen immutabele velden
Identification = @{
    id    = $permission.uuid
    $Type = $($Value)
    # DisplayName     = uitgecommentarieerd / verwijderd
    # DisplayNameFull = uitgecommentarieerd / verwijderd
}
```

> ⚠️ **Bij fouten in de Reference Cleaner:** niet herstarten — dit overschrijft de backup. Noteer de foutmelding en neem contact op met Rudolf Amersfoort.

---

### Validatie testomgeving

Na uitvoering van de uitvoeringskaart, voer de volgende technische validatie uit:

- [ ] Audit log doorlopen — zijn de verwachte wijzigingen doorgevoerd?
- [ ] Accountreferenties correct (format v2, geen array meer)
- [ ] Roles-permissies: geen `DisplayName`/`DisplayNameFull` meer in de referenties
- [ ] DefaultScope referentie = `DefaultScope` (legacy), overige referenties kloppen
- [ ] DefaultScope permissies aanwezig in datastorage
- [ ] Geen onverwachte errors in de HelloID audit log
- [ ] Permissions preview voor een testpersoon geeft correct resultaat
- [ ] Klant heeft functioneel gevalideerd en schriftelijk akkoord gegeven

---

## 8. Fase 5 — Productieomgeving

> 📋 **Uitvoering:** gebruik de [Uitvoeringskaart Productiemigratie](Uitvoeringskaart_Productiemigratie.md) voor de volledige stap-voor-stap uitvoering. Voer productie pas uit als de klant de testomgeving heeft gevalideerd en schriftelijk akkoord heeft gegeven.

De technische achtergrond uit Fase 4 (configuratieparameters, ARef-fix, Reference Cleaner) is van toepassing op zowel de testomgeving als de productieomgeving. Het enige verschil is dat `baseUrl` voor productie ingesteld wordt op `https://api.ons.io` en alle paden naar de **productiemap** verwijzen in plaats van de testmap.

### Validatie productie

Na uitvoering van de uitvoeringskaart:

- [ ] Audit log doorlopen — zijn wijzigingen conform de testrun?
- [ ] Spot-check: controleer minimaal 3 medewerkers met bekende rollen en permissies
- [ ] Geen onverwachte fouten in de HelloID audit log
- [ ] Automatische synchronisatie loopt correct
- [ ] Klant bevestigt: accounts, rollen en permissies correct op productie
- [ ] Klant geeft schriftelijk akkoord (e-mail of Topdesk)
- [ ] Migratie gemeld aan Remco den Elzen (Topdesk major ticket)

---

## 9. Bijlage A — Entitlement conversie

### Standaardbereik (default scope) — gedragswijziging en migratieaanpak

**v1:** één DefaultScope-permissie, uitgedeeld via één entitlement (my locations + my teams gecombineerd).  
**v2:** DefaultScope wordt gesplitst in twee afzonderlijke permissies: *DefaultScope my locations* en *DefaultScope my teams*.

Een **'DefaultScope (legacy)'** permissie is beschikbaar in het nieuwe permission import script en combineert beide — vergelijkbaar met de huidige werkwijze. De permission reference wijzigt echter sowieso (naamgeving in v2 wijkt af van v1), waardoor business rules bij de meeste opties alsnog moeten worden aangepast.

**Backward compatibiliteit via de migratietool:**  
De migratietool voegt `all_employees` en `all_clients` direct toe aan de betreffende rollen, zodat roluitgifte ongewijzigd blijft. Dit hoef je niet handmatig te doen.

**Let op voor handmatig aangemaakte rollen na de migratie:**  
Nieuwe rollen krijgen niet automatisch het standaardbereik. Informeer de klant hier proactief over.

---

#### Migratieaanpak DefaultScope — BESLOTEN: legacy entitlement vervangen (Optie A)

> **Besluit (meeting 13 mei 2026):** business rules aanpassen, legacy entitlement 1-op-1 vervangen. Werking blijft functioneel identiek — alleen de naamgeving wijzigt. Zie `DefaultScope_Beslisdocument.docx`.

**Waarom Optie A:** de nieuwe PowerShell v2 connector levert permission references in CamelCase aan, afwijkend van bestaande implementaties. Dit maakt een gewijzigde permission reference onvermijdelijk — en dus een BR-aanpassing sowieso nodig. De legacy entitlement houdt rekening met de CSV-matrix (functie/afdeling/all clients/all employees); de alternatieve opties niet.

**Afgewezen alternatieven:**
- *Configuratie-hack:* laat technische schuld achter in connector; riskant bij latere configuratiewijzigingen
- *Contains-check in script:* te risicovol bij toekomstige connector-uitbreidingen
- *Twee nieuwe entitlements direct:* vereist volledig redesign van de CSV-matrix logica met de klant

**Praktisch:** meestal 1 business rule per klant. ~20% heeft meerdere bedrijven en dus meer business rules. Het entitlement-overzicht in HelloID maakt identificatie eenvoudig (uitroepteken = permissiedefinitie verwijderd).

---

**Checklist DefaultScope na migratie:**

- [ ] DefaultScope permissies aanwezig in datastorage (force update uitgevoerd)
- [ ] DefaultScope referentie correct: `DefaultScope` (legacy), `DefaultScopeLocations`, `DefaultScopeTeams`, `DefaultScopeAllClients`, `DefaultScopeAllEmployees`
- [ ] Alle business rules bijgewerkt: legacy entitlement actief, geen uitroeptekens meer
- [ ] Klant geïnformeerd: nieuwe rollen krijgen niet automatisch standaardbereik

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
| Pending actions niet op te lossen | Klant raadplegen over oorsprong taak | Rick Jongbloed |
| Force update / enforcement geeft errors | Script opnieuw draaien (is idempotent) | Rudolf Amersfoort bij aanhoudend falen |
| Permissiescripts falen na migratie (alle) | Reference Cleaner is niet correct uitgevoerd — DisplayName/DisplayNameFull nog aanwezig in referenties | Rudolf Amersfoort |
| Subpermissie-berekening mislukt | Zelfde oorzaak als boven — Reference Cleaner opnieuw uitvoeren | Rudolf Amersfoort |
| DefaultScope-permissie kan niet worden ingetrokken | Force update DefaultScope is niet uitgevoerd → datastorage mist de permissies | Rudolf Amersfoort |
| Audit log toont onjuiste permissiewijzigingen | Reference Cleaner niet correct uitgevoerd | Rudolf Amersfoort |
| Accountreferentie-fout na migratie | ARef fix-code ontbreekt in `update.ps1`, of Update All Accounts nog niet uitgevoerd | Rudolf Amersfoort |
| Connector schrijft naar verkeerde omgeving | `baseUrl` in configuratie controleren: staging vs. production | Consultant (self-fix) |
| Certificaatfout | Controleer of certificaat v7 actief is in Nedap Podium en door klant goedgekeurd | Remco den Elzen (support) |
| Mapping-CSV geeft kolomfout | Kolomnamen controleren: `HelloIDPrimaryLookupKey`, `HelloIDSecondaryLookupKey`, `NedapLocationIds`/`NedapTeamIds` | Consultant (self-fix) |

---

## 11. Bijlage C — Contacten en escalatie

| Naam | Rol | Contact |
|------|-----|---------|
| Rick Jongbloed | Projectleider, eindverantwoordelijke pilotfase | R.Jongbloed@tools4ever.com |
| Rudolf Amersfoort | Connector-ontwikkelaar, technische vragen | r.amersfoort@tools4ever.com |
| Rick van den Dijssel | Feature flag, product owner | R.vdDijssel@tools4ever.com |
| Remco den Elzen | Topdesk, support, certificaatupgrade Nedap Podium | R.denElzen@tools4ever.com |
| Jeroen Smit | Autorisatiematrix (indien aanpassing nodig) | J.Smit@tools4ever.com |
| Ron Kuper | Escalatie management / billing | R.Kuper@tools4ever.com |

---

## Wijzigingslog

| Versie | Datum | Wijziging | Door |
|--------|-------|-----------|------|
| 0.1 | 2026-05-01 | Eerste opzet op basis van gesprek Rudolf + pilotplanning | Rick Jongbloed / Claude |
| 0.2 | 2026-05-01 | Reference Cleaner stappen uitgewerkt op basis van officiële manual v1.2 | Rick Jongbloed / Claude |
| 0.3 | 2026-05-01 | Migratiestappen volledig herschreven op basis van Migration.md in connector-repo | Rick Jongbloed / Claude |
| 0.4 | 2026-05-01 | Volgorde gecorrigeerd: Reference Cleaner verplaatst naar vóór [Migrate] | Rick Jongbloed / Claude |
| 0.5 | 2026-05-01 | Volgorde teruggedraaid naar Rudolf's gevalideerde procedure; open vraag RC + Nedap array-of-objects toegevoegd | Rick Jongbloed / Claude |
| 0.6 | 2026-05-01 | Rollback-waarschuwing; RC pre-check als verplichte stap; script backup; maintenance window communicatie | Rick Jongbloed / Claude |
| 0.7 | 2026-05-08 | Stap 2 volledig uitgewerkt; maatwerk check; RC pre-check GO/NOGO; correlatie-instellingen | Rick Jongbloed / Claude |
| 0.8 | 2026-05-13 | Bijlage A DefaultScope herschreven: besluit vastgelegd (Optie A), stappen uitgewerkt | Rick Jongbloed / Claude |
| 0.9 | 2026-05-22 | Verwerkt op basis van testmigratie Vughterstede 18 mei | Rick Jongbloed / Claude |
| 1.0 | 2026-06-05 | Uitvoeringskaarten afgesplitst; handleiding herschreven als achtergrond-/referentiedocument; uitvoering-stappen vervangen door verwijzingen naar uitvoeringskaarten | Rick Jongbloed / Claude |

---

*Dit document wordt opgeslagen in de GitHub-repository: [NedapUsersV2migration](https://github.com/Rick-Jongbloed/NedapUsersV2migration)*
