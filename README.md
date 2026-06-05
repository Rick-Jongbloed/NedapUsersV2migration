# Nedap Ons v1 → v2 Migratie

Documentatie voor het migreren van klanten van de Nedap Ons v1-connector naar de v2-connector in HelloID Provisioning.

## Inhoud

| Bestand | Omschrijving |
|---------|-------------|
| [Uitvoeringskaart_Testmigratie.md](Uitvoeringskaart_Testmigratie.md) | Stap-voor-stap uitvoeringskaart voor de testmigratie — voorbereiding, connector inrichten, migratie uitvoeren, validatie |
| [Uitvoeringskaart_Productiemigratie.md](Uitvoeringskaart_Productiemigratie.md) | Stap-voor-stap uitvoeringskaart voor de productiemigratie — inclusief [Productie-only] labels voor stappen die alleen gelden bij productie zonder testfase |
| [Migratiehandleiding_v1_naar_v2.md](Migratiehandleiding_v1_naar_v2.md) | Uitgebreide achtergrondhandleiding — context, beslissingen, technische toelichting |
| [Klantcommunicatie_Migratie_Aankondiging.md](Klantcommunicatie_Migratie_Aankondiging.md) | Sjabloon voor klantcommunicatie voorafgaand aan de migratiedag |
| [Testmigratie_Vughterstede_Dag1_Bevindingen.md](Testmigratie_Vughterstede_Dag1_Bevindingen.md) | Sessienota testmigratie Vughterstede (18 mei 2026) — bugs, workarounds, procesobservaties |

## Gebruik

Begin met de **Uitvoeringskaart Testmigratie** als je eerst op een testomgeving migreert. Ga daarna verder met de **Uitvoeringskaart Productiemigratie**. Wil je direct naar productie zonder testfase, gebruik dan alleen de productiemigratie-kaart en voer alle stappen uit die zijn gemarkeerd met **[Productie-only]**.

De uitvoeringskaarten verwijzen naar de connector-repo voor scripts en field mapping: branch [`Nedap-new-permissions-api-standard`](https://github.com/Tools4everBV/HelloID-Conn-Prov-Target-NedapOns-Users/tree/Nedap-new-permissions-api-standard).

## Status

Gevalideerd op basis van pilotmigraties bij Vughterstede (test 18 mei 2026, productie 28 mei 2026).

## Doelgroep

IAM Consultants en partners die de Nedap Ons v1 → v2 migratie uitvoeren.

## Contact

Rick Jongbloed — projectleider pilotfase
