# Nedap Ons v1 → v2 Migratie

Documentatie voor het migreren van klanten van de Nedap Ons v1-connector naar de v2-connector in HelloID Provisioning.

## Inhoud

| Bestand | Omschrijving |
|---------|-------------|
| [Proces migratie - Testomgeving](Proces%20Migratie%20Nedap%20Ons%20Users%20v1%20naar%20v2%20-%20Testomgeving.md) | Stap-voor-stap procesbeschrijving voor de testmigratie |
| [Proces migratie - Productieomgeving](Proces%20migratie%20Nedap%20Ons%20Users%20v1%20naar%20v2%20-%20Productieomgeving.md) | Stap-voor-stap procesbeschrijving voor de productiemigratie — stappen die alleen gelden bij productie zonder testfase zijn gemarkeerd met **[Productie-only]** |
| [Klantcommunicatie — aankondiging](Klantcommunicatie_Migratie_Aankondiging.md) | Sjabloon voor klantcommunicatie voorafgaand aan de migratiedag |

> De migratiehandleiding (achtergrond, beslissingen, technische toelichting) en klantspecifieke sessienota's worden lokaal beheerd en staan niet in deze repo.

## Gebruik

Start met de procesbeschrijving **Testomgeving** als je eerst op een testomgeving migreert, gevolgd door de procesbeschrijving **Productieomgeving**. Bij een productie-only migratie gebruik je alleen de procesbeschrijving productieomgeving en voer je alle **[Productie-only]** stappen uit.

De uitvoeringskaarten verwijzen naar de connector-repo voor scripts en field mapping: branch [`Nedap-new-permissions-api-standard`](https://github.com/Tools4everBV/HelloID-Conn-Prov-Target-NedapOns-Users/tree/Nedap-new-permissions-api-standard).

## Status

Gevalideerd op basis van pilotmigraties bij Vughterstede (test 18 mei 2026, productie 28 mei 2026).

## Doelgroep

IAM Consultants en partners die de Nedap Ons v1 → v2 migratie uitvoeren.

## Contact

Rick Jongbloed — projectleider pilotfase
