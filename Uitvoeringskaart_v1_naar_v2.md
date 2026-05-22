# Nedap Ons v1 → v2 — Uitvoeringskaart
**Doelgroep:** IAM Consultant Tools4ever  
**Gebruik:** werk deze kaart stap voor stap af op migratiedag. Vink af, ga door.

> **Migratiestrategie:** Dit document beschrijft een optionele testfase (Deel 1) gevolgd door de productieomgeving (Deel 2). Wil je direct naar productie migreren zonder testomgeving, sla Deel 1 dan over en begin bij Deel 2.

---

## Voorbereiding (vóór migratiedag)

- [ ] **T-5** Feature flag v1 connector aangevraagd bij Rick van den Dijssel
- [ ] **T-5** Certificaat v7 aangevraagd bij Remco den Elzen; klant keurt goed in Nedap Podium
- [ ] **T-5** Certificaat geplaatst op server: testmap én productiemap (overschrijf bestaand certificaat)
- [ ] **T-5** Klant geïnformeerd: op migratiedag worden **geen provisioning-acties** uitgevoerd en is **beheer niet mogelijk** — zowel voor de testomgeving als de productieomgeving
- [ ] **T-3** *(alleen bij testfase)* Klant heeft testomgeving ververst (verse kopie van productie)
- [ ] **T-3** *(alleen bij testfase)* Certificaat v7 actief op testomgeving bevestigd
- [ ] **T-1** Feature flag actief bevestigd
- [ ] **T-1** Wachtacties (Waiting Actions) = 0 — controleer en los op vóór migratiedag
- [ ] PowerShell CSV-exportscript beschikbaar en aangepast voor nieuwe kolomnamen V2 *(zie apart instructiedocument PowerShell CSV-export)*
- [ ] CSV-bestanden klaarstaan voor testomgeving én productieomgeving (apart)
- [ ] V2-scripts beschikbaar (branch `nedap-new-permissions-api-standard` in de connector-repo)
- [ ] Klantcontact beschikbaar op migratiedag

---

## Deel 1 — Testomgeving

> **Let op:** Deel 1 is alleen nodig als je de migratie eerst op een testomgeving wilt valideren vóórdat je naar productie gaat. Bij een productie-only migratie sla je Deel 1 over en begin je bij Deel 2.

### A — Start van de dag

1. Log in op de HelloID-omgeving van de klant.
2. Zet alle **schedules uit** (handmatig, één voor één). *(Voorkomt ongewenste wijzigingen tijdens de migratie.)*
3. Ga naar **Provisioning → Systems** → noteer de huidige **threshold-waarden** per connector (bewaar deze).
4. Stel alle thresholds in op **1**. *(Dit zorgt dat bij een onverwacht grote actie de connector stopt na de eerste uitvoering.)*
5. Controleer: staat de **rooster-synchronisatie** aan? Zo nee: toevoegen en eenmalig draaien vóór je verdergaat.
6. Vernieuw de testomgeving als dat nog niet is gedaan (kopie van productie → testomgeving).

---

### B — Testomgeving gelijkstellen aan productie

7. Ga naar **Business Rules** → filter op NEDAP Ons Users **en** NEDAP Ons Users Test, status "Draft" + "Published" aan, "None" uit.
8. Koppel elk entitlement dat in de productieregel staat ook aan de **testconnector**. Doe dit voor alle NEDAP-entitlements.
9. Draai een **Enforcement** om alles gelijk te trekken.
10. Controleer op eventuele fouten die door het kopiëren zijn ontstaan (bijv. verwijderde rollen of hernoemde groepen in NEDAP) en los ze op.

---

### C — Business rules controleren

11. Exporteer de business rules naar CSV/JSON.
12. Controleer handmatig (of met AI) op inconsistenties:
    - Entitlements gekoppeld zonder bijbehorende conditie (bijv. druppelnummer)
    - Naamconflicten tussen HelloID en NEDAP
13. Los gevonden inconsistenties op in de business rules.

---

### D — Wachtacties controleren en oplossen

14. Ga naar **Provisioning → Actions → Waiting**.
15. Los alle wachtacties op:
    - Testaccounts → Unmanage
    - Excluded accounts met entry-account → Unmanage + uit Excluded halen
    - Andere → klant raadplegen of verwijderen
16. Controleer: **Waiting Actions = 0** voordat je verdergaat.

---

### E — Pre-migratie Reference Cleaner check (op V1)

17. Open de Reference Cleaner: `https://[klantnaam].helloid.com/provisioning/#/reference-cleaner/overview`
18. Klik **Start cleaner**.
19. Selecteer de **Roles** Permission Configuration.
20. Klik **Determine differences** — controleer of er geen blocking taken zijn.
21. Klik **Stop cleaner**. *(Browsertab sluiten stopt de Cleaner NIET — expliciet stoppen verplicht.)*

> ⛔ Zijn er blocking taken? Los ze op (zie stap 14–16), herhaal dan stap 17–21.

---

### F — Migratie uitvoeren (scripts en configuratie)

22. Ga naar **Provisioning → Systems → [NEDAP Ons Users Test]**.
23. Klik **Migrate system** → bevestig met **Confirm**.

    > ⚠️ Vanaf dit moment is de migratie onomkeerbaar. Schedules worden automatisch stilgezet.

24. **Tab Fields**
    - Klik **Delete all** (verwijder huidige field mapping).
    - Download de field mapping van GitHub (branch `nedap-new-permissions-api-standard`).
    - Importeer de field mapping.

25. **Tab Account**
    - Vervang **Create script** (uit repo).
    - Vervang **Update script** (uit repo).
    - Vervang **Delete script** (uit repo).
    - Vervang **Data Import script** (uit repo).
    - Vervang **Configuration** — stel alle parameters in (gebruik onderstaande standaardwaarden als uitgangspunt):

      | Parameter | Standaard | Vul in voor klant |
      |-----------|-----------|-------------------|
      | Environment / baseUrl | `https://api-staging.ons.io` (test) | — |
      | Certificaatpad | — | Pad naar `.pfx` op server (testmap) |
      | Certificaatwachtwoord | — | Wachtwoord certificaat |
      | grantDefaultScopeMyself | `false` | Standaard niet wijzigen |
      | mappingLocations | — | Pad naar `locations.csv` |
      | mappingTeams | — | Pad naar `teams.csv` |
      | csvDelimiter | `;` | Standaard niet wijzigen |
      | explicitMapping | `false` | Standaard niet wijzigen |
      | ValidateTeamAndLocation | `false` | Standaard niet wijzigen |
      | DirectoryCacheLocationsTeams | — | Pad naar cache-map |
      | ImportOnlyActiveEmployees | `false` | Standaard niet wijzigen |
      | daysBeforeContractStartDate | `0` | Alleen relevant als bovenstaande `true` is |
      | daysAfterContractEndDate | `0` | Alleen relevant als bovenstaande `true` is |

26. **Tab Permissions — Default Scope**
    - Laad het **Permission script** (uit repo).
    - Zet **"Use script to import permissions"** aan.
    - Klik **Eval** → daarna **Apply**.
    - Zet de toggle **Contact Data Storage** aan.

27. **Tab Permissions — Roles**
    - Laad het **Roles permission script** (uit repo).
    - Klik **Preview** om te controleren.

28. **Tab Permissions — Handle All Actions**
    - Laad het **Handle All Actions script** (uit repo).

29. **Tab Resources**
    - Vervang het **Resources script** (`resources.ps1`, uit repo).
    - Klik **Preview** om te controleren.

30. **Tab Correlation**

    | Instelling | Waarde |
    |------------|--------|
    | Enable correlation | `True` |
    | Person correlation field | `ExternalId` |
    | Account correlation field | NEDAP Ons Identification Number |

31. Controleer of je alle vinkjes hebt gezet: Create ✓ Update ✓ Delete ✓ Permission ✓ Default Scope ✓ Role ✓ Resource Cache ✓
32. Klik **Complete Migration** → bevestig.

---

### G — Reference Cleaner (post-migratie)

33. Open de Reference Cleaner: `https://[klantnaam].helloid.com/provisioning/#/reference-cleaner/overview`
34. Klik **Start cleaner**.
35. Selecteer de **Roles** Permission Configuration.
36. Klik **Determine differences** — controleer dat `DisplayName` en `DisplayNameFull` in de "to remove"-lijst staan.
37. Klik **Remove fields**.
38. Controleer de **History** rechts — actie gelogd?
39. Klik **Stop cleaner**.

---

### H — Default Scope legacy instellen

> Voer de onderstaande stappen uit voor **alle business rules** waarin de oude Default Scope entitlement is opgenomen. Controleer eerst hoeveel business rules de Default Scope bevatten — bij klanten met meerdere bedrijven kan dit meer dan één zijn.

Voer de stappen in exact deze volgorde uit, voor elke business rule afzonderlijk:

40. Ga naar alle business rules met de **oude Default Scope** entitlement.
41. Vink de oude Default Scope **uit**.
42. Draai een **Sync** op de entitlements.
43. Vink de nieuwe **Default Scope (legacy)** entitlement **aan**.
44. Publiceer de business rule — kies bij het publiceren voor **Unmanage** voor het oude Default Scope entitlement.

    > ⚠️ Sla de Unmanage-stap niet over. Zonder Unmanage probeert het systeem de oude permissie alsnog in te trekken.

---

### I — Afronden en valideren

45. Draai een **Sync** op de entitlements.
46. Draai een **Force update** op alle accounts.
47. Draai een **Enforcement**.
48. Controleer:
    - Geen errors in de audit log
    - Wachtacties = 0
    - Permissions Preview voor een testpersoon geeft correct resultaat
    - Rollen en entitlements correct zichtbaar in de business rules
49. Zet de **schedules weer aan**.
50. Herstel de **thresholds** naar de oorspronkelijke waarden (genoteerd in stap 3).
51. Laat de klant functioneel valideren (accounts, rollen, bereik correct?).
52. Klant geeft **schriftelijk akkoord** (mail of Topdesk-ticket).

---

## Deel 2 — Productieomgeving

> Voer productie pas uit als de klant de testomgeving heeft gevalideerd en akkoord heeft gegeven.

### Pre-flight check productie

- [ ] Testomgeving gevalideerd en akkoord klant ontvangen
- [ ] Wachtacties op productie = 0 (opnieuw controleren)
- [ ] Klantcontact beschikbaar
- [ ] Productiecertificaat staat klaar op server (productiemap)

### Uitvoering

53. Voer stappen **A t/m I** opnieuw uit, nu op de **productieconnector**.
54. Wijzig in stap 25 de Environment naar: `https://api.ons.io`
55. Gebruik in stap 25 het pad naar het **productiecertificaat** (productiemap op server).

### Afsluiting

- [ ] Migratie melden aan Remco den Elzen (Topdesk major ticket)
- [ ] Rick van den Dijssel informeren: feature flag v1 mag uit voor deze klant
- [ ] Afwijkingen en tijdsduur documenteren in klantdossier
- [ ] Handleiding bijwerken indien nodig

---

## Snelle referentie — Issues

| Symptoom | Oplossing |
|----------|-----------|
| Wachtacties blokkeren Reference Cleaner | Oplossen (stap 14–16), controleer tot = 0 |
| Default Scope wordt niet ingetrokken | Force update niet gedraaid — draai stap 46 opnieuw |
| Rooster niet gesynchroniseerd | Rooster-sync toevoegen en eenmalig draaien (stap 5) |
