# MediFlow 🏥

![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat&logo=typescript&logoColor=white)
![Next.js](https://img.shields.io/badge/Next.js-000000?style=flat&logo=nextdotjs&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-3ECF8E?style=flat&logo=supabase&logoColor=white)
![n8n](https://img.shields.io/badge/n8n-EA4B71?style=flat&logo=n8n&logoColor=white)
![Claude AI](https://img.shields.io/badge/Claude_AI-D4A843?style=flat&logo=anthropic&logoColor=white)
![WhatsApp](https://img.shields.io/badge/WhatsApp_API-25D366?style=flat&logo=whatsapp&logoColor=white)
![Google Sheets](https://img.shields.io/badge/Google_Sheets-34A853?style=flat&logo=googlesheets&logoColor=white)
![Vercel](https://img.shields.io/badge/Vercel-000000?style=flat&logo=vercel&logoColor=white)

**KI-gestützte Terminverwaltung per WhatsApp für Gesundheitseinrichtungen.**

Patienten buchen Arzttermine direkt über WhatsApp — keine App, kein Formular. MediBot übernimmt die gesamte Konversation, von der Begrüßung bis zur Terminbestätigung, inklusive automatischer Erinnerungen am Vorabend.

🔗 **Live Demo:** [mediflow-two.vercel.app](https://mediflow-two.vercel.app)

---

## Architektur

```
Patient (WhatsApp)
       │
       ▼
Evolution API (Webhook)
       │
       ▼
n8n Workflow Engine
  ├── 🔍 Nachricht parsen & validieren
  ├── 👤 Patienten-Lookup (Google Sheets)
  ├── 📋 Gesprächszustand laden
  │
  ├── [Neuer Patient] → Registrierung + Begrüßung
  │
  └── [Bekannt] → Claude AI analysiert Intention
        ├── PRENDRE_RDV  → Verfügbare Slots anzeigen → Bestätigung
        ├── ANNULER_RDV  → Terminsuche → Stornierung
        ├── URGENCE      → Notfallnachricht + SAMU-Hinweis
        └── AUTRE        → Allgemeine Antwort
       │
       ▼
Google Sheets (Patienten / Termine / Slots)
       │
       ▼ (täglich 18:00 Uhr, Cron)
Claude AI generiert personalisierte Erinnerung
       │
       ▼
WhatsApp Rappel J-1
```

---

## Funktionen

- **Automatische Patientenerkennung** — Neuer Patient wird automatisch registriert (ID: `PAT_YYYYMMDDHHMMSS`)
- **Persistenter Gesprächszustand** — `LIBRE` / `ATTENTE_CRENEAU` / `ATTENTE_SPECIALITE` gespeichert in Google Sheets
- **KI-Intentionserkennung** via Claude Haiku — erkennt Buchung, Stornierung, Urgenz, Smalltalk
- **Slot-Management** — verfügbare Termine werden gesperrt sobald gebucht (`DISPONIBLE` → `RESERVE`)
- **Automatische Erinnerungen** — Cron-Job täglich 18 Uhr, Claude generiert personalisierten Text pro Patient
- **Next.js Dashboard** — Verwaltungsoberfläche für die Klinik (Vercel-deployed)

---

## n8n Workflows

Die Automatisierungsschicht besteht aus zwei voneinander getrennten Workflows:

| Datei | Beschreibung | Trigger |
|---|---|---|
| [`mediflow-rdv-whatsapp.json`](./docs/workflows/mediflow-rdv-whatsapp.json) | Hauptworkflow — Terminbuchung per WhatsApp | Webhook (Evolution API) |
| [`mediflow-cron-rappels.json`](./docs/workflows/mediflow-cron-rappels.json) | Cron-Workflow — Tägliche Erinnerungen | Schedule (18:00 täglich) |

### Import in n8n

1. In n8n: **Settings → Import Workflow**
2. JSON-Datei hochladen
3. Alle `YOUR_*` Platzhalter durch eigene Credentials ersetzen (siehe unten)

---

## Google Sheets Datenstruktur

Das System nutzt drei Sheets in einer Google Spreadsheet:

**PATIENTS**
```
patient_id | telephone | prenom | nom_complet | etat_conversation | creneaux_proposes | date_inscription
```

**CRENEAUX**
```
creneau_id | medecin_nom | specialite | date | heure_debut | statut | rdv_id_associe
```

**RENDEZ-VOUS**
```
rdv_id | patient_id | patient_telephone | medecin_nom | specialite | date_rdv | heure_rdv | statut | rappel_j1_envoye | rappel_j1_date
```

---

## Setup

### Voraussetzungen

- n8n (self-hosted oder Cloud)
- Evolution API Instanz (WhatsApp)
- Anthropic API Key
- Google Sheets API (OAuth2)
- Node.js 18+

### Konfiguration

Ersetze in den Workflow-JSONs folgende Platzhalter:

```
YOUR_GOOGLE_SHEET_ID      → ID deiner Google Spreadsheet
YOUR_EVOLUTION_API_URL    → URL deiner Evolution API Instanz
YOUR_INSTANCE_NAME        → Name deiner WhatsApp-Instanz
YOUR_CREDENTIAL_ID        → n8n Credential ID (nach Import vergeben)
```

### Next.js Dashboard

```bash
git clone https://github.com/guilloulearnlife/mediflow.git
cd mediflow
cp .env.example .env.local
# .env.local ausfüllen
npm install
npm run dev
```

---

## Technischer Stack

| Komponente | Technologie |
|---|---|
| Frontend | Next.js 14 (App Router), TypeScript, Tailwind CSS |
| Datenbank | Supabase (PostgreSQL) |
| Automatisierung | n8n (self-hosted) |
| WhatsApp | Evolution API |
| KI | Anthropic Claude Haiku |
| Datenspeicher | Google Sheets |
| Deployment | Vercel |

---

## Projektkontext

Entwickelt als Praxisprojekt im Rahmen von **TGM Automation** — eine B2B-Automatisierungsagentur für den frankophonen Markt. MediFlow zeigt, wie WhatsApp-basierte KI-Assistenten Klinikprozesse automatisieren können, ohne dass Patienten eine App installieren müssen.

---

## Lizenz

MIT — freie Verwendung mit Quellenangabe.
