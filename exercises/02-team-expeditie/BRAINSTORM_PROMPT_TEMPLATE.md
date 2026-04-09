# Opdracht 2: Brainstorm met de AI Agent — Template

Gebruik dit template in **Stap 1** om samen met de AI agent een invalshoek te vinden.
Kopieer de ingevulde prompt naar je coding agent (VS Code / Kilo) en laat je inspireren.

---

## Hoe gebruik je dit?

1. Vul de onderdelen tussen `[...]` in op basis van jullie teamvoorkeur
2. Plak de volledige prompt in de agent
3. Bespreek de suggesties met je team
4. Kies één invalshoek en ga verder met `PROMPT_TEMPLATE.md`

---

## De Brainstorm Prompt

> Kopieer het blok hieronder naar je agent:

---

Wij zijn een team van datajournalisten en doen mee aan een workshop over Machine Accelerated
Investigation met CBS-data. We willen een interessante datavraag onderzoeken binnen het thema
**[wonen / energietransitie / beide]**.

**Onze voorkeur / richting** (optioneel — laat leeg als je open staat voor alles):
[bijv. "We zijn geïnteresseerd in ongelijkheden tussen rijke en arme gemeenten" /
"We willen iets doen met energielabels en koopwoningen" /
"Geen voorkeur, verras ons"]

**Wat we zoeken:**
Stel ons 4–6 concrete onderzoekshoeken voor die:
- Beantwoordbaar zijn met CBS StatLine-data (OData API op opendata.cbs.nl)
- Journalistiek interessant zijn (ongelijkheid, trend, verrassing, of beleidsgevolg)
- Haalbaar zijn in ~45 minuten met een coding agent

Geef per invalshoek:
1. Een pakkende onderzoeksvraag (één zin)
2. Welke CBS-tabel(len) je zou gebruiken (tabel-ID + naam)
3. Waarom dit een interessant verhaal kan zijn
4. Wat het verwachte eindproduct zou zijn (grafiek, kaart, tabel, etc.)

Gebruik de beschikbare CBS StatLine tabelregisters en OData API-documentatie om de
tabel-IDs te verifiëren voordat je ze noemt.

---

## Na de brainstorm: kies je invalshoek

Noteer hier jullie keuze zodat iedereen op dezelfde pagina zit:

**Gekozen invalshoek:** [vul in na de brainstorm]

**Reden voor deze keuze:** [bijv. "Meest verrassend", "Politiek actueel", "Data is beschikbaar"]

**Verantwoordelijke voor de prompt:** [naam van degene die de agent aanstuurt]

Ga daarna naar `PROMPT_TEMPLATE.md` om de volledige briefing uit te schrijven.

---

## Volledig ingevuld voorbeeld

Hieronder een voorbeeld van de brainstorm prompt volledig ingevuld en klaar om te plakken in de agent:

---

Wij zijn een team van datajournalisten en doen mee aan een workshop over Machine Accelerated
Investigation met CBS-data. We willen een interessante datavraag onderzoeken binnen het thema
**energietransitie**.

**Onze voorkeur / richting:**
We zijn geïnteresseerd in ongelijkheden: welke gemeenten of regio's blijven achter bij de
energietransitie? We denken aan het verschil tussen rijke en arme gemeenten, of tussen
stedelijke en landelijke gebieden. We willen liefst iets dat politiek of beleidsmatig
relevant is — iets wat een minister zou moeten weten.

**Wat we zoeken:**
Stel ons 4–6 concrete onderzoekshoeken voor die:
- Beantwoordbaar zijn met CBS StatLine-data (OData API op opendata.cbs.nl)
- Journalistiek interessant zijn (ongelijkheid, trend, verrassing, of beleidsgevolg)
- Haalbaar zijn in ~45 minuten met een coding agent

Geef per invalshoek:
1. Een pakkende onderzoeksvraag (één zin)
2. Welke CBS-tabel(len) je zou gebruiken (tabel-ID + naam)
3. Waarom dit een interessant verhaal kan zijn
4. Wat het verwachte eindproduct zou zijn (grafiek, kaart, tabel, etc.)

Gebruik de beschikbare CBS StatLine tabelregisters en OData API-documentatie om de
tabel-IDs te verifiëren voordat je ze noemt.

---

**Gekozen invalshoek:** Aandeel woningen met slecht energielabel (D–G) per gemeente, afgezet tegen het landelijk gemiddelde

**Reden voor deze keuze:** Politiek actueel (isolatiesubsidies), data beschikbaar via CBS, verrassende regionale uitkomsten verwacht

**Verantwoordelijke voor de prompt:** Sara
