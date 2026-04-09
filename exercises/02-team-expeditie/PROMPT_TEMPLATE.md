# Opdracht 2: Team Expeditie — Briefing Template

Gebruik dit template om je onderzoeksopdracht te schrijven voor de coding agent.
Vul alle secties in voordat je de prompt uitvoert. Hoe specifieker, hoe beter het resultaat.

---

## [1] Context over het probleem

> Geef de achtergrond van het onderwerp. Waarom is dit relevant? Wat speelt er in de samenleving
> of het beleid rondom dit thema? Schrijf 2–4 zinnen die de agent voorzien van journalistieke context.

_Voorbeeld: "In Nederland staat de energietransitie onder druk: veel woningen zijn slecht geïsoleerd
terwijl de gasprijzen stijgen. CBS publiceert jaarlijks data over energielabels per gemeente,
maar deze data wordt zelden systematisch geanalyseerd op ongelijkheden tussen regio's."_

**[Vul hier jouw context in]**

---

## [2] Databronnen

> Geef concrete verwijzingen naar de data die de agent moet gebruiken.
> Gebruik CBS OData API-referenties of tabelidentifiers uit de cbs-statline-skill.
> Vermeld ook lokale bestanden als die beschikbaar zijn.

**CBS-tabel(len):**
- Tabel ID: `[bijv. 85039NED]` — [omschrijving]
- Tabel ID: `[optioneel tweede tabel]` — [omschrijving]

**Lokale bestanden (indien van toepassing):**
- `[pad naar bestand, bijv. exercises/01-lokaal-woningdruk/kerncijfers_wijken_buurten_2025_wide.csv]`

**Aanvullende bronnen:**
- [optioneel: links naar andere datasets of documentatie]

---

## [3] Onderzoeksvraag

> Formuleer één scherpe, beantwoordbare vraag. Dit is het hart van de opdracht.
> Een goede onderzoeksvraag is specifiek, meetbaar en journalistiek relevant.

**Onderzoeksvraag:** [Schrijf hier je vraag, bij voorkeur vetgedrukt in de uiteindelijke prompt]

_Voorbeeld: "Welke gemeenten hebben de grootste kloof tussen het aandeel slecht geïsoleerde
woningen en het gebruik van duurzame energie?"_

---

## [4] Gewenst eindresultaat

> Beschrijf zo concreet mogelijk wat de agent moet opleveren.
> Denk aan: type output (tabel, grafiek, kaart, rapport), format, en eventuele filtermogelijkheden.

Het eindresultaat moet bevatten:
- [ ] [bijv. Een overzichtstabel met de top-10 gemeenten gesorteerd op ...]
- [ ] [bijv. Een visualisatie (staafdiagram / choropleth kaart) van ...]
- [ ] [bijv. Een CSV-export met de onderliggende data]
- [ ] [bijv. Korte tekstuele samenvatting van de belangrijkste bevindingen]

**Technisch format:** [Python-script / Jupyter notebook / HTML-rapport / anders]

---

## [5] Verificatiecriteria

> Hoe weet je of het resultaat klopt? Definieer concrete checks die je handmatig kunt uitvoeren.
> Dit dwingt je om vooraf na te denken over wat je verwacht te vinden.

- [ ] [bijv. Het totaal aantal gemeenten in de output is 342 (alle Nederlandse gemeenten)]
- [ ] [bijv. De gebruikte perioden komen overeen met de meest recente CBS-release]
- [ ] [bijv. Bekende uitschieters (bijv. Amsterdam, Groningen) staan op een logische plek in de ranking]
- [ ] [bijv. Geen lege waarden voor de kernvariabelen in de top-20]
- [ ] [bijv. De bron en peildatum zijn zichtbaar in het eindproduct]

---

## Volledig ingevuld voorbeeld

Hieronder een voorbeeld van dit template volledig ingevuld. Gebruik dit als referentie.

---

### [1] Context over het probleem

In Nederland staat de energietransitie onder druk: het kabinet wil dat woningen voor 2030
gemiddeld energielabel C hebben, maar uit CBS-cijfers blijkt dat meer dan 40% van de
woningvoorraad nog label D of lager heeft. De verdeling is sterk regionaal: krimpgebieden
en gemeenten met oude mijnbouwgeschiedenis scoren structureel slechter dan stedelijke
gemeenten met veel nieuwbouw. Inzicht in deze ongelijkheid helpt beleidsmakers subsidies
gerichter in te zetten.

---

### [2] Databronnen

**CBS-tabel(len):**
- Tabel ID: `85039NED` — Energielabels woningen; gemeente (2023)
- Tabel ID: `82931NED` — Woningkenmerken per gemeente (eigendom, bouwjaar, type)

**Lokale bestanden:**
- Geen — alles wordt via de CBS OData API opgehaald

**Aanvullende bronnen:**
- CBS StatLine tabelregister via de cbs-statline-skill voor verificatie van tabel-IDs

---

### [3] Onderzoeksvraag

**Welke gemeenten hebben de grootste achterstand in woningisolatie — gemeten als het
aandeel woningen met energielabel D t/m G — en hoe verhoudt dit zich tot het landelijk
gemiddelde?**

---

### [4] Gewenst eindresultaat

Het eindresultaat moet bevatten:
- [x] Een tabel met de 15 gemeenten met het hoogste aandeel slecht geïsoleerde woningen (label D–G), gesorteerd aflopend
- [x] Een horizontaal staafdiagram van deze top-15 met het landelijk gemiddelde als referentielijn
- [x] Een CSV-export `woningisolatie_gemeenten_2023.csv` met alle 342 gemeenten en hun labelscores
- [x] Een korte samenvatting (3–5 zinnen) met de belangrijkste bevinding

**Technisch format:** Python-script dat draait in de terminal (geen Jupyter notebook vereist)

---

### [5] Verificatiecriteria

- [ ] De output bevat precies 342 gemeenten (geen dubbele, geen missende)
- [ ] De gebruikte peildatum is 2023 of het meest recente beschikbare jaar
- [ ] Amsterdam, Rotterdam en Den Haag staan niet in de top-15 (verwacht: stedelijke gemeenten scoren beter)
- [ ] Het landelijk gemiddelde aandeel label D–G ligt tussen de 35% en 50%
- [ ] De bronvermelding "CBS StatLine, tabel 85039NED" is zichtbaar in de grafiek of het rapport
