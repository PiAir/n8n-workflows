Deze repository bevat een verzameling n8n workflows voor verschillende automatiseringstaken.

## 📋 Overzicht Workflows

### 1. Energie Update
**Locatie**: `Energie Update/`

Een workflow die dagelijks P1 meter data synchroniseert naar Google Sheets voor energiemonitoring.

**Functionaliteit**:
- Ophalen van dagelijkse stroom-, gas- en weerdata van lokale API
- Berekening van gasverbruik per graaddag
- Maandoverzicht van totaal energie- en gasverbruik
- Automatische synchronisatie naar Google Sheets

**Trigger**: Dagelijks om 01:00 uur + handmatige trigger

**Documentatie**: Zie [Energie Update README](Energie%20Update/README.md)

---

### 2. Gmail Scholar Alerts to Sheets
**Locatie**: `Gmail Scholar Alerts to Sheets/`

Workflow voor het automatisch verwerken van Google Scholar alerts vanuit Gmail en deze opslaan in een Google Sheet.

**Functionaliteit**:
- Ophalen van scholar alert mails
- Extractie van artikel informatie
- Opslag in Google Sheet "Aanbevolen Artikelen"
- Automatische sheet creatie indien nodig

**Trigger**: Dagelijks om 05:00 uur

**Bestanden**:
- `Gmail Scholar Alerts to Sheets.json` - Standaard versie - gemaakt met VSCode
- `Gmail Scholar Alerts to Sheets - Antigravity version.json` - Alternatieve versie gemaakt door Google Antigravity

Beide werkwijzen maakten gebruik van de N8N MCP Server om verbinding te maken met een N8N server op het lokale netwerk.

---

### 3. Signal Chat with NOLAI Server
**Locatie**: `Signal chat with NOLAI server/`

Een chatbot interface die Signal berichten verbindt met een lokale Ollama AI server.

**Functionaliteit**:
- Ontvangen van Signal berichten
- Doorsturen naar lokale Ollama server (Mistral model)
- Terugsturen van AI responses via Signal
- Webhooks voor real-time chat

**API Endpoints**:
- Ollama: `https://chat.nolai.nl/ollama/api/chat`
- Signal: `http://192.168.0.83:9999/v2/send`

---

## 🚀 Gebruik

Elk workflow bestand (`.json`) kan worden geïmporteerd in n8n via:
1. Open n8n interface
2. Klik op "Import from File"
3. Selecteer het gewenste `.json` bestand
4. Configureer credentials en parameters indien nodig

## 📝 Notities

- Alle workflows zijn ontwikkeld in n8n
- Google Sheets integratie vereist OAuth2 authenticatie
- Lokale API's vereisen netwerk toegang binnen het LAN
- Zie individuele workflow mappen voor gedetailleerde documentatie

## 🔧 Vereisten

- n8n installatie (self-hosted of cloud)
- Google Sheets API credentials
- Lokale API toegang (voor Energie Update en Signal workflows)
- Ollama server (voor NOLAI chat)" 
