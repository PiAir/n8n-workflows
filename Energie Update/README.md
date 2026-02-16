# Energie Update - Marktstraat 61

n8n workflow voor automatische synchronisatie van P1 meter data naar Google Sheets voor energiemonitoring.

## 📊 Overzicht

Deze workflow haalt dagelijks energie- en weerdata op van een lokale P1 meter API en synchroniseert deze naar Google Sheets voor tracking en analyse van energieverbruik.

## 🎯 Functionaliteit

### Branch 1: Graaddagen (Degree Days)
Berekent het gasverbruik in relatie tot buitentemperatuur:

**Data bronnen**:
- Power/Gas API: `http://192.168.0.11/api/v1/powergas/day`
- Weather API: `http://192.168.0.11/api/v1/weather/day`

**Berekende velden**:
- **Dag**: Datum in DD-MM-YYYY formaat
- **Graaddag**: Verwarmingsbehoefte indicator (laatste waarde uit weather API)
- **m³**: Gasverbruik in kubieke meter (Index 9 uit PowerGas)
- **m³ / GD**: Efficiëntie ratio (gas ÷ graaddag)
- **max m³**: Theoretisch maximum (graaddag × 0.65)

**Doeltabel**: Google Sheet "Graaddagen"  
**Update methode**: Append or Update (op basis van datum)

### Branch 2: Verbruik per maand
Maandelijks overzicht van totaal energieverbruik:

**Data bron**:
- Power/Gas API: `http://192.168.0.11/api/v1/powergas/month`

**Berekende velden**:
- **Maand**: Nederlandse maandnaam
- **Jaar**: Jaartal
- **Gas**: Gasverbruik (m³) - Index 9
- **Stroom**: Elektriciteitsverbruik (kWh) - Index 6
- **Teruglevering**: Teruggleverde elektriciteit (kWh) - Index 7
- **Totaal**: Netto verbruik (Stroom - Teruglevering)

**Doeltabel**: Google Sheet "Verbruik per maand"  
**Update methode**: Append (nieuwe rijen toevoegen)

## ⚙️ Configuratie

### Triggers
1. **Schedule Trigger**: Automatisch elke dag om 01:00 uur
2. **Manual Trigger**: Handmatige uitvoering voor testen

### Data Parameters
- **Start datum**: `2024-01-01 00:00:00` (aanpasbaar in HTTP nodes)
- **Sorteer volgorde**: Oplopend (ASC)
- **Test limiet**: Momenteel ingesteld op 50 items (zie limitatie hieronder)

### API Configuratie
Alle API endpoints gebruiken het lokale IP-adres: `192.168.0.11`

### Google Sheets
- **Spreadsheet ID**: Geconfigureerd in de workflow
- **Credentials**: Google OAuth2 vereist
- **Tabellen**: "Graaddagen" en "Verbruik per maand"

## 🚦 Data Flow

```
Trigger (Schedule/Manual)
    ↓
[Branch 1: Graaddagen]          [Branch 2: Maandverbruik]
    ↓                                  ↓
HTTP Weather Day               HTTP PowerGas Month
    ↓                                  ↓
HTTP PowerGas Day              Code Maand
    ↓                                  ↓
Code Graaddagen                Google Sheets Maand
    ↓
Google Sheets Graaddagen
```

## 📈 Conditionele Opmaak (Google Sheets)

Voor visuele indicatie van gasverbruik efficiëntie in de "Graaddagen" sheet:

1. Selecteer **Kolom D** (`m³ / GD`)
2. Format → Conditional formatting → Color scale
3. Configureer kleuren:
   - **Minpoint**: 0 (Groen) - Efficiënt/warme dag
   - **Midpoint**: 0.5 (Geel) - Normaal
   - **Maxpoint**: 1.0 (Rood) - Hoog verbruik/koude dag

**Target waarden**:
- `< 0.3`: Zeer efficiënt / warme dag
- `0.3 - 0.6`: Normaal gebruik
- `> 0.6`: Hoog verbruik relatief aan temperatuur

## 🔧 Data Limitatie (Testen)

De workflow is momenteel ingesteld om **50 items** te verwerken om rate limiting te voorkomen.

**Volledige historische sync herstellen**:
1. Open de `Code Graaddagen` node
2. Verwijder: `return results.slice(0, 50);`
3. Vervang door: `return results;`

⚠️ **Let op**: Bij >1000 rijen, overweeg de `SplitInBatches` loop te activeren om API rate limits te respecteren.

## 📚 Documentatie

### Beschikbare Documenten
- **[task.md](task.md)**: Complete implementatie checklist (alle taken afgerond)
- **[implementation_plan.md](implementation_plan.md)**: Technisch ontwerp en data mapping
- **[walkthrough.md](walkthrough.md)**: Setup instructies en configuratie
- **[n8n-Google-Sheets/SKILL.md](n8n-Google-Sheets/SKILL.md)**: Expert guide voor n8n Google Sheets integratie - door Antigravity gemaakt naar aanleiding van de ontwikkelsessie. Deze SKILL zou moeten helpen bij het maken van n8n workflows die Google Sheets bewerken, inclusief best practices voor rate limiting, data synchronisatie en troubleshooting.

### Data Mapping Referentie

**PowerGas API Array Indices**:
- Index 0: Datum/tijd
- Index 6: Stroom verbruik (kWh)
- Index 7: Teruglevering (kWh)
- Index 9: Gas verbruik (m³)

**Weather API Array**:
- Index 0: Datum/tijd
- Laatste index (19): Graaddagen waarde

## ⚡ Rate Limiting

Google Sheets API heeft strikte quota (60 writes/minuut):

**Huidige bescherming**:
- Data gelimiteerd tot 50 items tijdens testen
- `executeOnce: true` op HTTP PowerGas Day node

**Voor grote datasets**:
1. Voeg `SplitInBatches` node toe (batch size: 50)
2. Voeg `Wait` node toe (2-5 seconden)
3. Loop terug naar batches

## 🔐 Vereisten

- n8n installatie met toegang tot:
  - Lokaal netwerk (192.168.0.11)
  - Google Sheets API
- Google OAuth2 credentials geconfigureerd
- P1 meter met REST API interface
- Weather data API op lokale server

## 🎯 Status

✅ **Operationeel**  
Alle implementatie taken zijn afgerond en getest.

## 📝 Notities

- Workflow is geconfigureerd voor **append** operatie (nieuwe rijen toevoegen)
- Datum formatting gebruikt leading zeros voor consistente matching
- Weather "Graaddagen" gebruikt laatste array waarde (magnitude based)
- Water verbruik wordt momenteel genegeerd
- Conditionele opmaak moet handmatig in Google Sheets worden ingesteld

## 🐛 Troubleshooting

**Geen data in sheets?**
- Controleer of Google Sheets operations op "Append" of "Append or Update" staan (niet "Get")
- Verificeer Google OAuth2 credentials

**Rate limit errors?**
- Verklein data vanaf test limiet (50 items)
- Voeg batching en wait nodes toe

**Datum mismatches?**
- Controleer dat datum format in Code nodes `padStart(2, '0')` gebruikt
- Sheets verwacht: `DD-MM-YYYY` met leading zeros

**API niet bereikbaar?**
- Verifieer dat n8n toegang heeft tot lokaal netwerk
- Test API endpoints handmatig: `http://192.168.0.11/api/v1/powergas/day`
