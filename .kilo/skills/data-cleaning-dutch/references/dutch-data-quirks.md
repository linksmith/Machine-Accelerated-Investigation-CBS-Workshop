# Dutch Data Quirks Reference

Extended reference for working with Dutch government open data files.

---

## 1. Missing value markers — complete list

| Marker | Betekenis | Bronnen |
|--------|-----------|---------|
| `""` (leeg) | Niet beschikbaar of niet gemeten | Algemeen |
| `-` | Niet van toepassing of nul waarnemingen | CBS en verwante bronnen |
| `.` | Niet beschikbaar (te oud of niet gemeten) | CBS StatLine |
| `x` | Onderdrukt — reden niet openbaar | CBS |
| `..` | Niet beschikbaar | CBS StatLine |
| `...` | Niet van toepassing | CBS StatLine |
| `geheim` | Privacy: minder dan 5 waarnemingen | CBS |
| `n.v.t.` | Niet van toepassing | Veel overheidsbronnen |
| `n.b.` | Niet bekend / niet gemeten | Veel overheidsbronnen |
| `z.o.z.` | Zie ommezijde — verwijst naar voetnoot | Oudere datasets |
| `?` | Onbekend | Gemeentelijke bronnen |
| `*` | Voorlopig cijfer | CBS |
| `**` | Nader voorlopig (herzien voorlopig) | CBS |

**Definitieve code om alles te vervangen:**

```python
import pandas as pd

DUTCH_MISSING = [
    '', '-', '.', 'x', '..', '...', 'geheim',
    'n.v.t.', 'n.b.', 'z.o.z.', '?'
]
df.replace(DUTCH_MISSING, pd.NA, inplace=True)
```

**Over "geheim"**: CBS onderdrukt cellen wanneer er minder dan 5 individuele waarnemingen zijn die zichtbaar zouden worden. Dit beschermt de privacy van kleine bedrijven en huishoudens. Dat een gemeente "geheim" heeft bij een bepaalde indicator, kan op zichzelf al een verhaalhaak zijn (wat verbergt deze gemeente?).

---

## 2. Nederlandse overheidsdata: standaard exportformaten per bron

| Bron | Scheidingsteken | Decimaalteken | Encoding | Bijzonderheden |
|------|-----------------|---------------|----------|----------------|
| CBS StatLine (CSV-download) | `;` (puntkomma) | `,` (komma) | `utf-8-sig` | BOM aanwezig bij Excel-exports |
| CBS microdata (RA-omgeving) | variabel | `,` | `utf-8` | Vaste breedte of pipe-separated; documentatie in datamap |
| Kadaster (transactiedata) | `\t` (tab) | `.` (punt) | `utf-8` | GML of WKT voor geodata |
| RVO (subsidiedata) | `\|` (pipe) | `,` | `utf-8` of `latin-1` | Controleer per exporttype |
| RIVM | `,` of `;` | `,` of `.` | wisselend | Verifieer altijd bij download |
| DUO (onderwijs) | `;` | `,` | `utf-8-sig` | Meerdere tabbladen in Excel; check leerjaarstructuur |
| PBL (planbureau) | `,` | `.` | `utf-8` | Doorgaans schoner dan directe overheidsbronnen |
| Gemeentelijke open data | wisselend | wisselend | wisselend | Elke gemeente heeft eigen conventies |
| CBS Statline OData v4 (API) | JSON | n.v.t. | n.v.t. | Gebruik `cbs-statline-skill` voor API-toegang |

---

## 3. Nederlands getalformaat

Nederland gebruikt het continentale Europese getalformaat:

- **Decimaalteken**: `,` (komma) — bijv. `3,14`
- **Duizendtalteken**: `.` (punt) — bijv. `1.234.567`
- **Gecombineerd**: `1.234.567,89`

### Parse Nederlandse getalstrings

```python
def herstel_nl_getal(waarde):
    """Converteer Nederlands getalformaat (string) naar float."""
    import pandas as pd
    if pd.isna(waarde) or str(waarde).strip() in ['', '-', '.', '..']:
        return pd.NA
    return float(str(waarde).strip().replace('.', '').replace(',', '.'))

# Op een kolom toepassen:
df['bedrag'] = df['bedrag'].apply(herstel_nl_getal)
# Of vectorized:
df['bedrag'] = (
    df['bedrag'].astype(str).str.strip()
    .str.replace('.', '', regex=False)
    .str.replace(',', '.', regex=False)
    .pipe(pd.to_numeric, errors='coerce')
)
```

### Weergave in Nederlandstalige artikelen

```python
def format_nl(getal, decimalen=1):
    """Formatteer getal voor weergave in Nederlandstalig artikel."""
    # Eerst Engels formaat, dan omzetten naar Nederlands
    engels = f"{getal:,.{decimalen}f}"           # "1,234.5"
    return engels.replace(',', 'X').replace('.', ',').replace('X', '.')  # "1.234,5"

print(format_nl(1234567.89))     # "1.234.567,9"
print(format_nl(0.034, decimalen=1))  # "0,0"
print(format_nl(34.2, decimalen=1))   # "34,2"
```

---

## 4. Gemeentelijke herindelingen (herindelingen) 2020–2025

Nederlandse gemeenten fuseren regelmatig. GM-codes veranderen bij een fusie. Dit veroorzaakt problemen bij het koppelen van data uit verschillende jaren.

### Belangrijke herindelingen

| Jaar | Oude gemeente(n) | Nieuwe gemeente | Oude code(s) | Nieuwe code |
|------|-----------------|-----------------|--------------|-------------|
| 2021 | Eemnes + Laren + Blaricum | Blijven apart (BEL-samenwerking, geen fusie) | — | — |
| 2022 | Beemster | Purmerend | GM0370 | GM0439 |
| 2022 | Boxmeer, Cuijk, Mill en Sint Hubert, Sint Anthonis, Grave | Land van Cuijk | GM0756, GM0leiding ... | GM1982 |
| 2022 | Bergen op Zoom, Steenbergen, Woensdrecht | — (geen fusie dit jaar voor deze) | — | — |
| 2023 | Ten Boer, Haren, Groningen | Groningen (uitgebreid) | GM0009, GM0021, GM0017 | GM0014 |
| 2023 | Appingedam, Delfzijl, Loppersum | Eemsdelta | GM0007, GM0010, GM0024 | GM1979 |
| 2024 | Renswoude + Rhenen + Veenendaal | — (geen fusie) | — | — |

*Raadpleeg de CBS-herindelingstabel (`84721NED`) voor een complete en actuele lijst.*

### Aanbevolen aanpak

```python
# Gebruik altijd geodata uit hetzelfde jaar als je statistische data.
# PDOK biedt grenzen voor elk jaar vanaf 2013.
#
# Controleer bij een slechte koppeling:
ongekoppeld = samengevoegd[samengevoegd['jouw_waarde_kolom'].isna()]
print(f"Ongekoppelde gebieden: {len(ongekoppeld)}")
print(ongekoppeld[['statcode', 'statnaam']].to_string())
# Als dit gemeenten zijn die zijn gefuseerd, is een jaarmismatch de oorzaak.
```

---

## 5. Postcode ↔ wijk/buurt koppeling

### CBS Postcode-huisnummertabel

CBS koppelt elk adres (postcode + huisnummer) aan wijk- en buurtcodes. De tabel wordt jaarlijks bijgewerkt.

Zoek de huidige tabel via `cbs-statline-skill`: zoek op "postcode huisnummer wijk buurt".

```python
# Nadat je de postcodetabel hebt gedownload:
df_met_wijk = df.merge(
    postcode_tabel[['postcode', 'wijk_code', 'buurt_code', 'gemeente_code']],
    on='postcode',
    how='left'
)

# Controleer koppelkwaliteit
n_gekoppeld = df_met_wijk['wijk_code'].notna().sum()
print(f"Gekoppeld: {n_gekoppeld}/{len(df)} ({n_gekoppeld/len(df)*100:.1f}%)")
```

### Van 4-cijferig postcode naar gemeente

```python
# 4-cijferige postcodes zijn grotendeels binnen één gemeente
# Maar postcodes over gemeentegrenzen bestaan — controleer altijd!
df['pc4'] = df['postcode'].str[:4]

# Gebruik de CBS-tabel voor een betrouwbare mapping
```

---

## 6. Tekenencodering — spiekbriefje

| Encoding | Gebruik | Signalen dat je dit nodig hebt |
|----------|---------|-------------------------------|
| `utf-8` | Standaard voor moderne data en API's | Werkt voor de meeste nieuwe bestanden |
| `utf-8-sig` | Excel/Windows UTF-8-exports | Bestand begint met BOM (`\ufeff`); pandas leest `ï»¿` aan het begin |
| `latin-1` | Oude Nederlandse overheidsbest anden | `é`, `ë`, `ü`, `ij` verschijnen als `Ã©`, `Ã«` |
| `cp1252` | Windows West-Europees | Vergelijkbaar met `latin-1`, verwerkt ook `€`-teken |
| `iso-8859-1` | Alias voor `latin-1` | |

### Automatische detectie

```python
# Installeer: pip install chardet
import chardet

with open('data.csv', 'rb') as f:
    resultaat = chardet.detect(f.read(10000))  # Eerste 10 KB bemonsteren
print(resultaat)  # Bijv.: {'encoding': 'ISO-8859-1', 'confidence': 0.73}
```

---

## 7. Nederlandse datumformaten

| Formaatstring | Voorbeeld | Context |
|---------------|-----------|---------|
| `%d-%m-%Y` | `15-03-2024` | Meest voorkomend in NL overheidsdata |
| `%d/%m/%Y` | `15/03/2024` | Minder gangbaar |
| `%d %m %Y` | `15 03 2024` | Zeldzaam |
| `%Y-%m-%d` | `2024-03-15` | ISO-formaat, moderne API's |
| `%d %B %Y` | `15 maart 2024` | Nederlandse maandnamen |

### Nederlandse maandnamen

pandas kan Nederlandse maandnamen niet automatisch parsen. Gebruik deze mapping:

```python
MAANDEN_NL = {
    'januari': 1, 'februari': 2, 'maart': 3, 'april': 4,
    'mei': 5, 'juni': 6, 'juli': 7, 'augustus': 8,
    'september': 9, 'oktober': 10, 'november': 11, 'december': 12
}

def parse_nl_datum(s):
    """Parseer 'dag maandnaam jaar' naar datetime."""
    import pandas as pd
    if pd.isna(s):
        return pd.NaT
    onderdelen = str(s).strip().lower().split()
    if len(onderdelen) != 3:
        return pd.NaT
    dag, maand_naam, jaar = onderdelen
    maand = MAANDEN_NL.get(maand_naam)
    if maand is None:
        return pd.NaT
    return pd.Timestamp(year=int(jaar), month=maand, day=int(dag))

df['datum'] = df['datum_tekst'].apply(parse_nl_datum)
```

---

## 8. Veelvoorkomende Excel-exportproblemen

| Probleem | Symptoom | Oplossing |
|---------|----------|-----------|
| Samengevoegde cellen in headers | Kolommen krijgen "Unnamed: 2"-namen | `pd.read_excel(..., header=None)`, daarna handmatig `df.columns` instellen |
| Meerdere headerrijen | MultiIndex-kolommen | `pd.read_excel(..., header=[0, 1])` of `skiprows=N` |
| Voetnoten onderaan | Laatste rijen bevatten tekst i.p.v. data | `pd.read_excel(..., nrows=N)` of `skipfooter=N` |
| Verborgen tabbladen | Data staat niet op het eerste tabblad | `pd.ExcelFile('data.xlsx').sheet_names` voor overzicht |
| Valutaformattering | `€ 1.234,56` opgeslagen als string | Strip `€ ` en verwijder voor gebruik van `herstel_nl_getal()` |
| Datumkolommen als float | Excel-datums worden getallen (bijv. `45000`) | `pd.to_datetime(df['datum'], origin='1899-12-30', unit='D')` |
| Lege rijen tussen data | Lege rijen onderbreken de dataset | `df.dropna(how='all')` om volledig lege rijen te verwijderen |
