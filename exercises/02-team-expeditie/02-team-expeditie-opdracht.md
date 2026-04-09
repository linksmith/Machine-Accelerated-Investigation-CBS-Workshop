# Team opdracht 
Maak 4 teams (15 min prompt schrijven, 30 min uitvoeren, 15 min presenteren)

Opdracht: Vind een interessante CBS-dataset over wonen, schrijf een prompt en voer uit met VS / Kilo.

Stap 1: Kies samen een invalshoek binnen het thema wonen of energietransitie. Je kunt zelf een invalshoek bedenken (ga direct naar stap 2) of brainstormen samen met de AI agent. Gebruik daarvoor exercises/02-team-expeditie/BRAINSTORM_PROMPT_TEMPLATE.md.

Stap 2: Schrijf een briefing met onderzoeksvraag, gewenst eindresultaat en verificatiecriteria. Gebruik het template in exercises/02-team-expeditie/PROMPT_TEMPLATE.md.

Stap 3: Voer de prompt uit met je coding agent. Eén persoon stuurt aan, de rest denkt mee over prompts en verificatie.

Presentatie: wat was jullie vraag, wat hebben jullie gebouwd, wat ging goed en wat niet?

---

## Voorbeeld eindprompt

Hieronder een voorbeeld van een volledig uitgeschreven opdracht-prompt die je direct kunt uitvoeren:

---

**Context:**
In Nederland heeft meer dan 40% van de woningvoorraad nog een energielabel D of lager. De
verdeling is ongelijk: krimpgemeenten en gebieden met oude woningbouw scoren structureel
slechter. Het kabinet wil dat alle woningen voor 2030 gemiddeld label C halen, maar de
subsidie-inzet is verspreid over alle gemeenten zonder onderscheid naar urgentie.

**Databron:**
Gebruik CBS-tabel `85039NED` (Energielabels woningen per gemeente, 2023) via de CBS OData
API. Gebruik de cbs-statline-skill om de tabel op te halen en de juiste kolomnamen te
achterhalen.

**Onderzoeksvraag:**
**Welke 15 gemeenten hebben het hoogste aandeel woningen met energielabel D t/m G, en hoe
verhoudt dit zich tot het landelijk gemiddelde?**

**Gewenst eindresultaat:**
1. Een tabel met de top-15 gemeenten: naam, aandeel label D–G in %, verschil met landelijk gemiddelde
2. Een horizontaal staafdiagram van de top-15 met een referentielijn op het landelijk gemiddelde
3. Een CSV-export met alle gemeenten: `woningisolatie_gemeenten_2023.csv`
4. Een tekstuele samenvatting van 3–5 zinnen met de hoofdbevinding

Schrijf een Python-script dat alles uitvoert. Geen Jupyter notebook nodig.

**Verificatie:**
- De output bevat precies 342 gemeenten
- De peildatum is 2023 of het meest recent beschikbare jaar
- Amsterdam en Utrecht staan niet in de top-15
- De grafiek heeft een duidelijke titel, aslabels en bronvermelding