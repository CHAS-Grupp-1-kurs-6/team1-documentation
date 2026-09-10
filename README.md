# Lag 1 – Blue Team Documentation

Dokumentation för Lag 1 under **Kurs 6 – Avancerad IT-säkerhet**.

Detta repository används för att samla dokumentation från Blue Team-arbetet under vecka 4–8. Här dokumenterar vi vad vi har gjort, vilka tester vi har genomfört, vilka findings vi har identifierat och hur problemen har hanterats.

---

## 📌 Projektinformation

| Information | Värde |
|---|---|
| Kurs | Kurs 6 – Avancerad IT-säkerhet |
| Lag | Lag 1 |
| GCP-projekt | `itsx25-lab` |
| GitHub-organisation | `CHAS-Grupp-1-kurs-6` |
| Infrastrukturrepo | `team1-infra` |
| Dokumentationsrepo | `team1-documentation` |

---

# 📁 Dokumentstruktur

Vi använder följande struktur:

```text
team1-documentation/
│
├── README.md
│
├── week-04/
│   └── README.md
│
├── week-05/
│   └── README.md
│
├── week-06/
│   └── README.md
│
├── week-07/
│   └── README.md
│
└── week-08/
    └── README.md
```

Varje vecka dokumenteras separat. README-filen fungerar som översikt och index för hela Blue Team-arbetet.

---

# 🔵 Blue Team – Vecka 4–8

Blue Team-arbetet omfattar vecka 4–8 och fokuserar på att analysera, skydda, övervaka och härda infrastrukturen och applikationsmiljön.

---

# Vecka 4 – Blue Team Start

**Datum:** 7/9–11/9

## Fokus

- Terraform
- Workload Identity Federation
- Nätverkshärdning
- Deployment
- Infrastrukturgranskning
- Identifiering av initiala brister
- Sårbarhetsrapportering

## Planerade moment

### Pass 1 – Workshop
**7/9 13:00–15:00**

Genomgång av:
- Terraform-mall
- Workload Identity Federation
- Nätverkshärdning

### Pass 2 – Labb / handledning
**8/9 13:00–17:00**

- Deploya miljön
- Analysera infrastrukturkoden
- Identifiera initiala brister

### Pass 3 – Checkpoint
**10/9 13:00–15:00**

- Första avstämningen
- Genomgång av sårbarhetsrapportering

## Vårt arbete

### Vad gjorde vi?

> Skriv här.

### Vilka tester genomförde vi?

> Skriv här.

### Vad hittade vi?

> Skriv här.

### Vilka säkerhetsissues skapades?

> Skriv här.

### Vilka åtgärder genomfördes?

> Skriv här.

## Ansvarsfördelning

| Person | Uppgift | Status |
|---|---|---|
| | | ☐ Klar |
| | | ☐ Klar |
| | | ☐ Klar |
| | | ☐ Klar |
| | | ☐ Klar |

## Bevis / länkar

- GitHub PR:
- GitHub Issue:
- Commit:
- Screenshot:
- Logg:
- Annat:

---

# Vecka 5 – Säker åtkomst & Zero Trust Network Access

**Datum:** 14/9–18/9

## Fokus

- Zero Trust
- Säker åtkomst
- Headscale
- Tailscale
- Policies
- Tags
- ACL
- Least privilege

## Planerade moment

### Pass 1 – Workshop
**14/9 09:00–13:00**

Workshop kring:
- Headscale
- Tailscale
- Zero Trust-principer
- Policies
- Tags

### Pass 2 – Labb / handledning
**15/9 13:00–17:00**

- Konfigurera Headscale
- Ansluta noder
- Rota subnät
- Tillämpa Tailscale ACL

### Pass 3 – Checkpoint & Live-validering
**17/9 13:00–15:00**

- Demonstrera säker åtkomst
- Demonstrera principen om minsta behörighet

## Vårt arbete

### Vad gjorde vi?

> Skriv här.

### Vilka konfigurationer ändrade vi?

> Skriv här.

### Vilka tester genomförde vi?

> Skriv här.

### Vad hittade vi?

> Skriv här.

### Vilka issues skapades?

> Skriv här.

### Vilka åtgärder genomfördes?

> Skriv här.

## Ansvarsfördelning

| Person | Uppgift | Status |
|---|---|---|
| | | ☐ Klar |
| | | ☐ Klar |
| | | ☐ Klar |
| | | ☐ Klar |
| | | ☐ Klar |

## Bevis / länkar

- PR:
- Issue:
- Commit:
- Screenshot:
- Konfiguration:
- Logg:

---

# Vecka 6 – DevSecOps, applikationssäkerhet & Supply Chain

**Datum:** 21/9–25/9

## Fokus

- DevSecOps
- CI/CD
- GitHub Organisation
- SBOM
- Dependency-Track
- Cosign
- Image Signing
- Kubernetes
- Applikationssäkerhet
- Supply Chain Security

## Planerade moment

### Pass 1 – Workshop
**21/9 09:00–13:00**

- CI/CD i GitHub Organisation
- SBOM
- Dependency-Track
- Cosign Image Signing

### Pass 2 – Labb / handledning
**22/9 13:00–17:00**

- CI/CD-deploy av webbapplikationen i Kubernetes
- Analysera applikationsbrister
- Åtgärda brister
- Etablera grundläggande pipelinesäkerhet

### Pass 3 – Checkpoint & Code Review
**24/9 13:00–15:00**

- Live-granskning av CI/CD-pipelines
- Applikationssäkerhet
- Leveransskydd

## Vårt arbete

### Vad gjorde vi?

> Skriv här.

### Vilka säkerhetskontroller genomförde vi?

> Skriv här.

### Vilka sårbarheter hittade vi?

> Skriv här.

### Vilka issues skapades?

> Skriv här.

### Hur åtgärdades problemen?

> Skriv här.

## Ansvarsfördelning

| Person | Uppgift | Status |
|---|---|---|
| | | ☐ Klar |
| | | ☐ Klar |
| | | ☐ Klar |
| | | ☐ Klar |
| | | ☐ Klar |

## Bevis / länkar

- PR:
- Issue:
- Commit:
- CI/CD-run:
- Screenshot:
- Logg:

---

# Vecka 7 – SOC, Systemaudit & Blue Team-avslutning

**Datum:** 28/9–2/10

## Fokus

- SOC
- Systemaudit
- Wazuh
- SIEM
- Logginsamling
- Säkerhetsgranskning
- Sårbarheter
- Slutrapport

## Planerade moment

### Pass 1 – Workshop
**28/9 09:00–13:00**

- Wazuh som SIEM
- Implementering av logginsamling
- Säkerhetsgranskning

### Pass 2 – Labb & rapporthandledning
**29/9 13:00–17:00**

- Slutföra Wazuh-konfiguration
- Stänga sista sårbarheterna
- Sammanställa samtliga fynd
- Förbereda slutrapport

### Pass 3 – Blue Team-inlämning & Retro
**1/10 13:00–15:00**

- Redovisning av slutlig säkerhetsstatus
- Härdningsgrad
- Retrospektiv

## Vårt arbete

### Vad implementerade vi?

> Skriv här.

### Vad övervakade vi?

> Skriv här.

### Vilka findings identifierade vi?

> Skriv här.

### Vilka issues återstår?

> Skriv här.

### Vilka problem har åtgärdats?

> Skriv här.

## Ansvarsfördelning

| Person | Uppgift | Status |
|---|---|---|
| | | ☐ Klar |
| | | ☐ Klar |
| | | ☐ Klar |
| | | ☐ Klar |
| | | ☐ Klar |

## Bevis / länkar

- Wazuh:
- PR:
- Issue:
- Commit:
- Screenshot:
- Logg:
- Slutrapport:

---

# Vecka 8 – Threat Intelligence & Halvtidsavstämning

**Datum:** 5/10–9/10

## Fokus

- Threat Intelligence
- MITRE ATT&CK
- Mapping av findings
- Threat Intelligence-analys
- TLP-klassificering
- Actionable Threat Intelligence

## Planerade moment

### Pass 1 – Workshop
**5/10 09:00–13:00**

- MITRE ATT&CK-mapping av findings från vecka 4–7
- Threat Intelligence-analys
- TLP-klassificering

### Pass 2 – Labb & handledning
**6/10 13:00–17:00**

- Skapa en Actionable Threat Intelligence-rapport
- Utgå från vår miljö

### Pass 3 – Individuellt utvecklingssamtal
**8/10 13:00–15:00**

- Halvtidsavstämning
- Progression
- Arbetssätt
- AI-användning

## Vårt arbete

### Vilka findings mappar vi mot MITRE ATT&CK?

> Skriv här.

### Vilka hot har vi identifierat?

> Skriv här.

### Vilken Threat Intelligence har vi tagit fram?

> Skriv här.

### Vilka åtgärder rekommenderas?

> Skriv här.

## Ansvarsfördelning

| Person | Uppgift | Status |
|---|---|---|
| | | ☐ Klar |
| | | ☐ Klar |
| | | ☐ Klar |
| | | ☐ Klar |
| | | ☐ Klar |

## Bevis / länkar

- MITRE ATT&CK:
- Threat Intelligence-rapport:
- Issue:
- PR:
- Commit:
- Annat:

---

# 🔎 Findings

Alla verifierade säkerhetsfynd sammanställs här.

| ID | Finding | Vecka | Severity | Status | GitHub Issue |
|---|---|---|---|---|---|
| F-001 | | | | | |
| F-002 | | | | | |
| F-003 | | | | | |
| F-004 | | | | | |
| F-005 | | | | | |
| F-006 | | | | | |
| F-007 | | | | | |
| F-008 | | | | | |
| F-009 | | | | | |
| F-010 | | | | | |

## För varje finding dokumenterar vi

1. Vad vi hittade
2. Hur vi hittade det
3. Bevis
4. Påverkan och risk
5. Severity
6. Rekommenderad åtgärd
7. Status
8. GitHub Issue

---

# 🛡️ Security Issues

Säkerhetsproblem och tekniska förbättringsområden dokumenteras i:

`team1-infra/SECURITY-ISSUES.md`

Ett fynd ska verifieras innan det klassificeras som ett bekräftat säkerhetsproblem.

## Status

- ☐ Att verifiera
- 🔎 Under undersökning
- 🔴 Bekräftat
- 🟢 Åtgärdat
- ⚪ Ej ett problem

## Viktigt

CTF-flaggor och flaggrelaterade resultat dokumenteras separat och ska inte användas som Security Issues.

---

# 🔗 Relaterade repositories

## Infrastruktur

`team1-infra`

Här finns bland annat:

- Terraform
- GCP-konfiguration
- CI/CD
- Workload Identity Federation
- Infrastruktur-logg
- Security Issues

## Dokumentation

`team1-documentation`

Här finns:

- Veckodokumentation
- Workshop-dokumentation
- Findings
- Tester
- Bevis
- Sammanfattningar
- Blue Team-arbete

---

# 👥 Teamets arbetsfördelning

| Person | Ansvarsområde | Aktuell uppgift |
|---|---|---|
| Malcolm | | |
| Samuel | | |
| Abdulghani | | |
| Mert | | |
| Kristoffer | | |

---

# 📊 Övergripande status

| Vecka | Område | Status |
|---|---|---|
| Vecka 4 | Blue Team Start | 🔎 Pågår |
| Vecka 5 | Zero Trust | ☐ Ej startad |
| Vecka 6 | DevSecOps | ☐ Ej startad |
| Vecka 7 | SOC / Wazuh | ☐ Ej startad |
| Vecka 8 | Threat Intelligence | ☐ Ej startad |

---

# 📝 Dokumentationsprincip

Vi dokumenterar endast sådant som faktiskt har genomförts eller verifierats.

För varje tekniskt fynd ska vi om möjligt kunna svara på:

> **Vad gjorde vi → Vad observerade vi → Vad betyder det → Hur bevisar vi det → Vad bör åtgärdas?**

På så sätt blir dokumentationen spårbar och användbar för både säkerhetsrapporteringen och den slutliga Blue Team-redovisningen.

---

# 📌 Checklista inför slutrapport

- [ ] Alla veckor dokumenterade
- [ ] Alla relevanta tester dokumenterade
- [ ] Alla verifierade findings dokumenterade
- [ ] Security Issues länkade
- [ ] Bevis och screenshots sparade
- [ ] PR:er och commits dokumenterade
- [ ] Åtgärder dokumenterade
- [ ] MITRE ATT&CK-mapping genomförd
- [ ] Threat Intelligence dokumenterad
- [ ] Slutlig säkerhetsstatus sammanställd
- [ ] Retrospektiv genomförd
