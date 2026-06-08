# Klantcommunicatie — Aankondiging migratie Nedap Ons-koppeling

Gebruik deze mail om de klant te informeren vóór de migratiedag.  
Het blok over de testomgeving is optioneel — laat het weg bij een productie-only migratie.

---

**Onderwerp:** Migratie Nedap Ons-koppeling — [datum testomgeving] / [datum productie]

Beste [naam],

In het kader van de migratie van de Nedap Ons-koppeling in HelloID naar de nieuwe versie plannen we de volgende werkzaamheden in:

---

*[Onderstaand blok alleen opnemen bij migratie met testomgeving — weglaten bij productie-only]*

**Testomgeving — [datum]**  
Op [datum] voeren we de migratie uit op jullie testomgeving. De migratie duurt naar verwachting [X uur]. We vragen je om vóór [datum, uiterlijk de dag ervoor] een verse kopie van de productieomgeving naar de testomgeving te zetten, zodat we met actuele data kunnen werken.

Na afloop van de testmigratie ontvang je van ons een verzoek om de testomgeving kort te valideren. We vragen je daarna om schriftelijk akkoord te geven vóór we verder gaan naar productie.

- Er worden **geen provisioning-acties** uitgevoerd (nieuwe accounts, wijzigingen en intrekkingen worden uitgesteld tot na de migratie).
- **Beheer via HelloID** is tijdelijk niet beschikbaar.

> **Belangrijk:** zet de testomgeving na dag 1 niet opnieuw handmatig over vanuit productie. We hebben de gemigreerde testomgeving intact nodig voor de verificatie vóór dag 2.

---

**Productieomgeving — [datum]**  
Op [datum] voeren we de migratie uit op de productieomgeving. De migratie duurt naar verwachting [X uur]. Gedurende deze periode is het volgende niet mogelijk:

- Er worden **geen provisioning-acties** uitgevoerd (nieuwe accounts, wijzigingen en intrekkingen worden uitgesteld tot na de migratie).
- **Beheer via HelloID** is tijdelijk niet beschikbaar.

We raden aan om medewerkers die afhankelijk zijn van toegangswijzigingen hiervan vooraf op de hoogte te stellen.

Na afloop van de productiemigratie vragen we je opnieuw om een korte functionele validatie en schriftelijke bevestiging.

---

**Wat we van jullie vragen op de migratiedag**

Om de migratie soepel te laten verlopen, hebben we jullie hulp nodig op de volgende punten:

**Nedap Ons Podium — certificaatverzoek goedkeuren**  
Een paar dagen voor de migratie sturen wij een verzoek via Nedap Ons Podium voor het vernieuwen van het koppelcertificaat. We vragen je om Nedap Ons Podium die dag in de gaten te houden en het verzoek zo snel mogelijk goed te keuren zodra je het ontvangt.

**Beheerder beschikbaar**  
We vragen je om een beheerder beschikbaar te houden die toegang heeft tot:

- **Nedap Ons Podium**, om de certificaatverzoeken te kunnen goedkeuren (zie hierboven).
- **De server waarop het CSV-exportscript draait**, als dit script lokaal op een server staat (en niet in HelloID zelf). Door de migratie wijzigen de kolomnamen in het exportbestand, waardoor het script op de migratiedag aangepast moet worden. We hebben hiervoor remote toegang tot die server nodig, of een beheerder die het script ter plekke kan aanpassen.

**Eventuele opens