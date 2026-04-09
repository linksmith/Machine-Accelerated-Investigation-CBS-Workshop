---
name: data-cleaning-dutch
description: "Use this skill when the user has a CSV, Excel, or pandas DataFrame that needs cleaning — especially Dutch data with comma decimals, semicolon delimiters, mixed encodings, Dutch date formats (dd-mm-yyyy), missing value markers, or messy column names. Also trigger for deduplication, type conversion, outlier flagging, or general data quality checks. Do NOT use for CBS StatLine API access or CBS-specific code conventions — those belong to cbs-statline-skill."
---

## Purpose

Clean and validate tabular data for analysis. Focused on Dutch data quirks that trip up pandas defaults.

## When to use

User has data (CSV, Excel, or DataFrame) that needs cleaning before analysis. User mentions dirty data, missing values, encoding issues, or asks to "clean" or "prepare" the data.

## When NOT to use

- **CBS StatLine data** → use `cbs-statline-skill`. That skill handles CBS-specific cleaning: trailing spaces in region codes, period code parsing, code-to-label resolution.
- **Analysis / story-finding** → use `data-analysis-journalism` after cleaning is done.

## Step 0: Language

Respond in Dutch if the user writes in Dutch. Use Dutch labels in all print output and code comments unless told otherwise.

---

## Step 1: Data health check

Always start here. Run this block and share the output with the user before touching anything.

```python
import pandas as pd

# Dataconditie — altijd als eerste uitvoeren
print(f"Vorm: {df.shape}")
print(f"\nKolomtypes:\n{df.dtypes}")
print(f"\nOntbrekende waarden:\n{df.isnull().sum()}")
print(f"\nDubbele rijen: {df.duplicated().sum()}")
print(f"\nVoorbeeld (eerste 5 rijen):\n{df.head()}")
```

After showing the output, interpret it journalistically: "Kolom X heeft 14% ontbrekende waarden — dat kan een verhaal op zichzelf zijn."

---

## Step 2: Load Dutch data files

### CSV — standard Dutch government format

Dutch government CSVs almost always use **semicolon separator** and **comma decimal**:

```python
import pandas as pd

df = pd.read_csv(
    'data.csv',
    sep=';',           # Puntkomma als scheidingsteken
    decimal=',',       # Komma als decimaalteken
    encoding='utf-8-sig',  # BOM verwerken (veelvoorkomend bij Excel-exports)
    thousands='.'      # Punt als duizendtalteken
)
```

### CSV — encoding fallback

If `utf-8-sig` fails, try encodings in order:

```python
for encoding in ['utf-8-sig', 'utf-8', 'latin-1', 'cp1252']:
    try:
        df = pd.read_csv('data.csv', sep=';', decimal=',', encoding=encoding)
        print(f"Gelukt met encoding: {encoding}")
        break
    except (UnicodeDecodeError, ValueError):
        print(f"Mislukt met encoding: {encoding}")
```

### Excel

```python
df = pd.read_excel('data.xlsx', engine='openpyxl')

# Als de headers er vreemd uitzien (samengevoegde cellen in bronbestand):
df = pd.read_excel('data.xlsx', header=None, engine='openpyxl')
# Daarna handmatig: df.columns = ['col1', 'col2', ...]
```

---

## Step 3: Clean column names

```python
df.columns = (
    df.columns
    .str.strip()
    .str.lower()
    .str.replace(' ', '_', regex=False)
    .str.replace('-', '_', regex=False)
)
```

---

## Step 4: Replace Dutch missing value markers

Dutch government data uses many markers for missing or suppressed values:

```python
DUTCH_MISSING = [
    '',         # Lege string
    '-',        # Streepje
    '.',        # Punt (CBS)
    'x',        # Onderdrukt (reden onbekend)
    '..',       # Niet beschikbaar
    '...',      # Niet van toepassing
    'geheim',   # Privacy: minder dan 5 waarnemingen
    'n.v.t.',   # Niet van toepassing
    'n.b.',     # Niet bekend
]

df.replace(DUTCH_MISSING, pd.NA, inplace=True)
```

**Over "geheim"**: CBS onderdrukt waarden wanneer er minder dan 5 waarnemingen zijn, ter bescherming van de privacy. Dat een waarde geheim is, kan op zichzelf een verhaal zijn.

---

## Step 5: Parse Dutch dates

```python
# Gebruik altijd een expliciete formaatstring — dayfirst=True is onbetrouwbaar
df['datum'] = pd.to_datetime(df['datum'], format='%d-%m-%Y')

# Andere veelvoorkomende Nederlandse datumformaten:
# format='%d/%m/%Y'   → 15/03/2024
# format='%Y-%m-%d'   → 2024-03-15  (ISO, zeldzaam in NL overheidsdata)
# format='%d %B %Y'   → 15 maart 2024 (Nederlandse maandnamen)

# Onbekend formaat? Gebruik errors='coerce' en inspecteer de fouten:
df['datum'] = pd.to_datetime(df['datum'], dayfirst=True, errors='coerce')
print(f"Mislukte datumparses: {df['datum'].isna().sum()}")
```

---

## Step 6: Fix comma-decimal numbers stored as strings

Dutch numbers: puntkomma als decimaalteken, punt als duizendtalteken (1.234.567,89):

```python
def herstel_nl_getal(series):
    """Converteer Nederlands getalformaat (string) naar float.
    Verwijdert duizendtalscheidingspunten, converteert decimaalkomma naar punt.
    """
    return (
        series
        .astype(str)
        .str.strip()
        .str.replace('.', '', regex=False)   # Duizendtalpunt verwijderen
        .str.replace(',', '.', regex=False)  # Decimaalkomma → punt
        .pipe(pd.to_numeric, errors='coerce')
    )

# Toepassen op specifieke kolommen:
df['bedrag'] = herstel_nl_getal(df['bedrag'])
df['percentage'] = herstel_nl_getal(df['percentage'])
```

---

## Step 7: Validate Dutch postcodes

Dutch postcodes follow the `1234AB` format (4 digits + 2 uppercase letters):

```python
# Normaliseren: spaties verwijderen, hoofdletters
df['postcode'] = df['postcode'].str.replace(' ', '').str.upper().str.strip()

# Valideren: ongeldige postcodes markeren
patroon = r'^\d{4}[A-Z]{2}$'
geldig = df['postcode'].str.match(patroon, na=False)
print(f"Ongeldige postcodes: {(~geldig & df['postcode'].notna()).sum()}")
print(df[~geldig & df['postcode'].notna()]['postcode'].unique()[:10])
```

---

## Step 8: Deduplication

```python
# Exacte duplicaten
n_voor = len(df)
df = df.drop_duplicates()
n_na = len(df)
print(f"{n_voor - n_na} exacte duplicaten verwijderd")

# Bijna-duplicaten: zelfde sleutel, andere waarden
sleutel_kolommen = ['gemeente', 'jaar']  # Aanpassen aan jouw data
bijnadura = df[df.duplicated(subset=sleutel_kolommen, keep=False)]
if len(bijnadura) > 0:
    print(f"Waarschuwing: {len(bijnadura)} rijen delen dezelfde {sleutel_kolommen}")
    print(bijnadura.sort_values(sleutel_kolommen))
```

---

## Step 9: Check for mixed-type columns

Excel-exports from Dutch government sources sometimes mix numeric values and text in the same column:

```python
for col in df.columns:
    type_counts = df[col].dropna().apply(type).value_counts()
    if len(type_counts) > 1:
        print(f"Gemengde types in '{col}':")
        print(type_counts)
```

---

## Step 10: Before/after report

Always close with a summary showing what changed:

```python
print("=== Opschoningsoverzicht ===")
print(f"Rijen:  {n_voor} → {n_na} ({n_voor - n_na} verwijderd)")
print(f"\nOntbrekende waarden na opschoning:")
print(df.isnull().sum())
print(f"\nKolomtypes na opschoning:")
print(df.dtypes)
```

---

## Gotchas

- **Samengevoegde cellen in Excel-headers**: gebruik `header=None` en stel kolommen handmatig in.
- **Gemengde types**: check met `df[col].apply(type).value_counts()` vóór numerieke bewerkingen.
- **"geheim"-waarden**: CBS onderdrukt voor privacy. Zelf een verhaal waard.
- **Spaties in categoriekolommen**: `df['gemeente'] = df['gemeente'].str.strip()` — vergeet dit niet.
- **Nederlandse maandnamen in datumstrings**: pandas kan "15 maart 2024" niet automatisch parsen. Zie `references/dutch-data-quirks.md` voor een mapping.
- **"True"/"False" als strings uit Excel**: `df[col].map({'True': True, 'False': False})`.

---

## Extended reference

For a complete list of Dutch government missing value markers, common data source formats (CBS, Kadaster, RVO, RIVM), municipality mergers (herindelingen) 2020–2025, postcode mapping guidance, and encoding cheat sheet:

→ Read `references/dutch-data-quirks.md`
