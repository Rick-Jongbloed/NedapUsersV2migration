# Template — Interne intake-mail Nedap Ons Users v1 → v2 migratie

Herbruikbaar sjabloon voor de interne terugkoppeling na de technische intake, in het Tools4ever-format (vgl. de Confluence-templates voor HR- en AD→Entra ID-migraties). Vul de `[...]`-placeholders in en verwijder wat niet van toepassing is. Kopieer de inhoud uit het codeblok in de mail/Topdesk.

**Aandachtspunten bij versturen:**

- Versturen naar `technischeintake@tools4ever.com`, cc de accountmanager.
- Vermeld het **incident-/ticketnummer** altijd in het onderwerp (staat in je agenda-item).
- Werk de bijbehorende melding bij met de **juiste status en behandelaar**. Moet PC iets doen → behandelaar op Projectcoördinatie, status open. Moet sales iets doen → behandelaarsgroep Accountmanager, status "wait for sales response".
- Voeg een korte samenvatting toe met de actie die van hen verwacht wordt en markeer de comment met een vlaggetje (geel).
- Placeholders staan tussen `[...]`; verwijder de haken zodra je een waarde invult.

```
***************************************************************************************************
 TO:         Technische Intake <technischeintake@tools4ever.com>, [accountmanager] <undefined>
 SUBJECT:    Terugkoppeling technische intake HelloID Nedap Ons Users v1→v2 migratie voor [klant] - [INCIDENTNR]
 ATTACHMENTS:
***************************************************************************************************

Goedemiddag allen,

Vandaag hebben we de technische intake voor de migratie van de Nedap Ons Users v1-connector naar de v2-connector voor [klant] uitgevoerd.

Hierbij een samenvatting van de besproken punten.

Status: [Positief / aandacht vereist / on-hold]

Aanwezig bij deze intake:
*	Xxx                        FB HelloID
*	Xxx                        Applicatiebeheerder Nedap Ons
*	Xxx                        System admin
*	[consultant]        Intake Consultant

Scoping — Nedap Ons Users v2
*	Migratie van de bestaande v1-provisioning connector naar de nieuwe v2-connector
*	Uitvoering via Reference Cleaner + connectormigratie (business rules blijven behouden, functioneel identiek)
*	DefaultScope entitlement komt voor in [x] business rules
	-> Business rules aanpassen: legacy entitlement 1-op-1 vervangen door 'DefaultScope (legacy)'
*	[Geen] business rules in draft — draft-wijzigingen vereisen afstemming met de klant vóór herpubliceren
*	Na de migratie wordt een force update uitgevoerd voor alle accounts en alle permissies

Actiepunten [klant]:
*	Klant dataset laten controleren
*	Testomgeving verversen vóór de testmigratie [indien testmigratie]
*	CSV-exportscript (PowerShell) opnieuw draaien na de migratie — kolomnamen wijzigen in v2; test- en productie-CSV gescheiden houden
*	Certificaat goedkeuren in Nedap Podium (na aanvraag door support)
*	Server-/remote toegang regelen voor de consultant [indien server door externe partij beheerd]
*	Business rule aanmaken voor medewerkers zonder geldig druppelnummer [indien van toepassing]

Aanpak / Planning
*	Type migratie: [productie-only / test + productie]
*	Uitgangspunt (indicatie benodigde tijd): productie-only = 8 uur; test + productie = 2× 8 uur
*	Testmigratie v2-connector gepland op [dd-MM-yyyy] (optioneel)
*	Livegang v2-connector gepland op [dd-MM-yyyy]

@Projectcoördinatie | Tools4ever
(Verwijder het blok dat niet van toepassing is op het gekozen scenario.)

Algemeen (test + productie):
*	Project kan [wel/niet] gestart worden
*	Klant heeft [wel/niet] een interne deadline: [datum]
*	Planning dient nog afgestemd te worden / is afgestemd — zie kopje 'Aanpak / Planning'
*	Intake document is al verstuurd naar de klant [ja/nee]
*	Validatiecheck [wel/niet] nodig
*	Topdesk major ticket koppelen (beheer: Remco den Elzen)

Bij testmigratie:
*	PowerShell v1-feature in HelloID laten activeren door de product owner HelloID
*	Certificaat testomgeving [test-ID]: vernieuwen en upgraden naar versie v7

Bij productiemigratie:
*	Feature flag v1-connector: na succesvolle migratie laten uitschakelen door de product owner HelloID
*	Certificaat productieomgeving [productie-ID]: upgraden naar versie v7

Als jullie nog vragen hebben, dan hoor ik het natuurlijk graag.

Met vriendelijke groeten,

[consultant]
IAM Consultant
```
