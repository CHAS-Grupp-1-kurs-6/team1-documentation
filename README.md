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
# Vecka 5 -- Säker åtkomst & Zero Trust Network Access

**Datum:** 14/9--18/9

## Fokus

-   Zero Trust Network Access
-   Säker administrativ åtkomst
-   Google Cloud OS Login
-   Service Accounts
-   Secret Manager
-   Headscale
-   Tailscale
-   Split DNS och dnsmasq
-   Subnet routing
-   IP forwarding och NAT
-   Policies, tags och ACL
-   Least Privilege
-   Terraform
-   Felsökning och verifiering

## Planerade moment

### Pass 1 -- Workshop

**14/9 09:00--13:00**

-   Headscale och Tailscale
-   Zero Trust-principer
-   Policies och tags
-   Säker administrativ åtkomst

### Pass 2 -- Labb / handledning

**15/9 13:00--17:00**

-   Konfigurera Headscale
-   Ansluta noder
-   Rota subnät
-   Tillämpa och verifiera åtkomstkontroller
-   Felsöka DNS, routing och NAT

### Pass 3 -- Checkpoint & Live-validering

**17/9 13:00--15:00**

-   Demonstrera säker åtkomst
-   Verifiera DNS och routing
-   Demonstrera principen om minsta behörighet

## Vårt arbete

### Vad gjorde vi?

Under veckan byggde och verifierade vi en säkrare administrativ åtkomst
till Team 1:s Google Cloud-miljö. Arbetet började med Google Cloud OS
Login, Service Accounts och Secret Manager och fortsatte därefter med
Headscale/Tailscale för privat nätverksåtkomst, subnet routing och Split
DNS.

Jumphosten konfigurerades som central åtkomstpunkt mellan
Tailscale-nätet, Team 1:s GCP-subnät och interna tjänster. Målet var att
den normala åtkomstvägen skulle vara:

`Tailscale-klient → Headscale/Split DNS → dnsmasq → jumphost → routing/NAT → Spectre`

Under den första felsökningen fungerade inte hela den avsedda vägen till
Spectre. För att kunna fortsätta arbetet använde vi därför en
SSH-baserad SOCKS-proxy på lokal port `1080` som fallback. Lösningen
fungerade och gjorde det möjligt att fortsätta testa Spectre medan DNS-,
routing- och NAT-problemen undersöktes.

Efter vidare felsökning identifierades grundorsakerna. Vi korrigerade
dnsmasq, Headscale Split DNS och NAT-konfigurationen och flyttade de
fungerande ändringarna till Terraform. Därefter kunde Spectre nås direkt
genom Tailscale utan SOCKS-proxy.

### OS Login

Jumphosten migrerades från traditionella SSH-nycklar i instance metadata
till Google Cloud OS Login.

Aktiv Terraform-konfiguration använder:

``` hcl
metadata = {
  enable-oslogin         = "TRUE"
  block-project-ssh-keys = true
}
```

OS Login-behörighet hanteras med IAM-rollen
`roles/compute.osAdminLogin`. Äldre SSH-nyckelhantering togs bort från
den aktiva jumphost-konfigurationen efter att OS Login hade verifierats.

Teamets användare testade därefter SSH med sina CHAS-identiteter.

**Resultat:** OS Login verifierades och fungerade.

### Instance Service Account

Jumphosten använder ett dedikerat Service Account:

`team1-jumphost@itsx25-lab.iam.gserviceaccount.com`

Service Account verifierades även från Compute Engine Metadata Service.

**Resultat:** Instance Service Account verifierat.

### Secret Manager

Åtkomst till Google Cloud Secret Manager testades från jumphosten med
instansens Service Account. Testet visade att en secret kunde läsas från
instansen med den tilldelade identiteten.

**Resultat:** Secret Manager-access verifierad.

### Headscale och Tailscale

Headscale installerades på jumphosten och version `0.29.3` användes
under arbetet. Tjänsten verifierades med:

``` bash
sudo systemctl status headscale
```

Tailscale-klienter anslöts till Headscale. Jumphosten använde
Tailscale-adressen `100.64.0.1`.

Vi arbetade även med annonsering och godkännande av routes. Verifierade
routes omfattade:

``` text
10.0.1.0/24
10.0.0.2/32
```

### Första fungerande fallback-lösningen

När den normala vägen till Spectre ännu inte fungerade användes en
SSH-baserad SOCKS-proxy:

``` bash
gcloud compute ssh team1-jumphost \
  --zone=europe-north2-a \
  --project=itsx25-lab \
  -- -D 1080
```

Det skapade en lokal SOCKS-proxy på `127.0.0.1:1080`.

Detta var en fungerande workaround och gjorde att vi kunde fortsätta
testa tjänster samtidigt som grundproblemet felsöktes. Tunneln är
fortfarande användbar som fallback vid framtida felsökning, men behövs
inte längre för normal åtkomst.

### DNS och dnsmasq

Vid fortsatt felsökning identifierades en konflikt i
dnsmasq-konfigurationen. Workshoplösningen använde:

``` text
interface=tailscale0
bind-dynamic
server=169.254.169.254
```

Samtidigt fanns en äldre konfiguration med `bind-interfaces`.
Kombinationen gjorde att dnsmasq inte kunde starta korrekt.

Startup-scriptet använde dessutom `set -e`, vilket innebar att ett
dnsmasq-fel stoppade senare steg i scriptet, bland annat
NAT-konfigurationen.

Den gamla `bind-interfaces`-inställningen togs därför bort innan dnsmasq
startades. Konfigurationen verifierades med:

``` bash
dnsmasq --test
```

GCP:s interna DNS `169.254.169.254` användes som upstream resolver.

### Split DNS i Headscale

Vi identifierade även att `split` låg på fel nivå i
Headscale-konfigurationens YAML-struktur. Den korrigerades så att den
placerades under `dns.nameservers.split`.

Efter korrigeringen verifierades att:

``` text
itsx25.chas-lab.dev → 100.64.0.1
spectre.itsx25.chas-lab.dev → 10.0.0.2
```

**Resultat:** Split DNS fungerade korrekt.

### IP forwarding, routing och NAT

Jumphosten behövde fungera som router mellan Tailscale och
GCP-nätverket. IP forwarding aktiverades:

``` text
net.ipv4.ip_forward=1
```

NAT/MASQUERADE användes för lagets subnät och en specifik NAT-regel
lades till för Spectre `10.0.0.2/32`.

För att undvika duplicerade regler när startup-scriptet körs flera
gånger gjordes NAT-konfigurationen idempotent:

``` bash
iptables -t nat -C ... || iptables -t nat -A ...
```

Terraform innehåller även routes för den aktuella nätverksarkitekturen,
bland annat `0.0.0.0/0` och `100.64.0.0/10` via jumphosten för resurser
med relevant taggning.

### Slutlig lösning utan SOCKS

Efter korrigering av Split DNS, dnsmasq, routing och NAT kunde den
avsedda nätverksvägen användas direkt.

Den lokala SOCKS-proxyn kontrollerades med:

``` bash
lsof -nP -iTCP:1080 -sTCP:LISTEN
```

Ingen output returnerades, vilket visade att ingen SOCKS-listener var
aktiv.

Spectre testades därefter direkt:

``` bash
curl -I https://spectre.itsx25.chas-lab.dev/
```

Resultat:

``` text
HTTP/2 200
```

Spectre kunde även öppnas direkt i webbläsaren.

**Resultat:** Normal åtkomst fungerade utan SOCKS-proxy.

### Terraform och beständig konfiguration

De fungerande DNS- och NAT-ändringarna lades in i Terraform så att
lösningen återställs genom jumphostens startup-script och inte enbart
bygger på manuella kommandon.

Startup-scriptet hanterar bland annat:

-   IP forwarding
-   installation och konfiguration av dnsmasq
-   GCP intern DNS som upstream resolver
-   borttagning av konflikt med `bind-interfaces`
-   `dnsmasq --test` före restart
-   NAT för lagets subnät
-   NAT mot Spectre
-   idempotenta iptables-regler

OS Login/IAM-konfigurationen från den senaste versionen av `main`
bevarades samtidigt.

Efter apply verifierades Terraform och den slutliga planen visade:

``` text
No changes. Your infrastructure matches the configuration.
```

### Git, code review och Pull Request

Eftersom `main` hade ändrats av andra teammedlemmar behövde
DNS/NAT-ändringarna rebasas mot senaste `main`. Konflikten löstes
samtidigt som teamets OS Login-konfiguration bevarades.

Den slutliga ändringen levererades genom:

-   Branch: `fix-jumphost-dns-nat`
-   Commit: `3ca9192`
-   Pull Request: `#16`
-   PR-titel: `fix jumphost DNS, NAT and OS Login configuration`

PR #16 granskades och mergades till `main`.

### Säkerhetsgranskning

Vid Terraform-granskningen identifierades även en bred GCP
firewall-regel:

``` hcl
allow {
  protocol = "all"
}

source_ranges = ["0.0.0.0/0"]
```

Regeln identifierades som ett separat område för fortsatt härdning
enligt Least Privilege. Den ändrades inte som en del av DNS/NAT-fixen
eftersom säkerhetsförändringen behöver hanteras och verifieras separat.

## Vilka konfigurationer ändrade vi?

-   Google Cloud OS Login aktiverades och äldre SSH-nyckelhantering togs
    bort från aktiv jumphost-konfiguration.
-   IAM-behörigheter för OS Login verifierades.
-   Instance Service Account verifierades.
-   Secret Manager-access verifierades.
-   Headscale/Tailscale konfigurerades och routes verifierades.
-   Split DNS korrigerades i Headscale.
-   dnsmasq konfigurerades på `tailscale0`.
-   GCP intern DNS `169.254.169.254` sattes som upstream resolver.
-   Konflikten mellan `bind-interfaces` och `bind-dynamic` löstes.
-   IP forwarding verifierades.
-   NAT/MASQUERADE konfigurerades för subnätet och Spectre.
-   NAT-regler gjordes idempotenta.
-   DNS/NAT-fixen gjordes beständig genom Terraform startup-script.

## Vilka tester genomförde vi?

-   OS Login via teamets CHAS-identiteter
-   Instance Service Account via Compute Engine Metadata Service
-   Secret Manager-access från jumphosten
-   Headscale service status
-   Tailscale-noder och routes
-   Split DNS med Tailscale
-   DNS-uppslag mot Spectre
-   `dnsmasq --test`
-   kontroll av IP forwarding
-   kontroll av iptables/NAT
-   åtkomst via SSH SOCKS-proxy som fallback
-   kontroll att port `1080` inte längre hade en aktiv listener
-   direkt åtkomst till Spectre utan proxy
-   HTTP-test som gav `HTTP/2 200`
-   Terraform state-granskning
-   Terraform validering och plan
-   Git rebase och code review före merge

## Vad hittade vi?

1.  **dnsmasq-konflikt:** `bind-interfaces` kolliderade med
    `bind-dynamic` och hindrade dnsmasq från att starta korrekt.
2.  **Startup-scriptets felkedja:** eftersom `set -e` användes stoppade
    dnsmasq-felet senare konfiguration, bland annat NAT.
3.  **Felplacerad Split DNS-konfiguration:** `split` låg på fel nivå i
    Headscale YAML.
4.  **NAT behövdes mot Spectre:** jumphosten behövde korrekt forwarding
    och MASQUERADE för trafik mot `10.0.0.2/32`.
5.  **NAT-regler behövde vara idempotenta:** kontroll med `iptables -C`
    infördes före `iptables -A`.
6.  **Bred firewall-regel:** `protocol = "all"` från `0.0.0.0/0`
    identifierades som ett separat område för fortsatt Least
    Privilege-härdning.

## Vilka issues skapades?

Verifierade säkerhetsproblem och förbättringsområden dokumenteras
separat i `team1-infra/SECURITY-ISSUES.md`.

CTF-flaggor och flaggrelaterade resultat dokumenteras separat och
klassificeras inte som Security Issues.

## Vilka åtgärder genomfördes?

-   OS Login infördes och verifierades.
-   Äldre aktiv SSH-nyckelhantering togs bort.
-   Service Account och Secret Manager-access verifierades.
-   Headscale/Tailscale och routes verifierades.
-   SOCKS-proxy användes som fungerande fallback under felsökningen.
-   Headscale Split DNS korrigerades.
-   dnsmasq-konflikten löstes.
-   IP forwarding och NAT verifierades.
-   NAT mot Spectre lades till.
-   iptables-regler gjordes idempotenta.
-   DNS/NAT-konfigurationen gjordes beständig i Terraform.
-   Direkt åtkomst till Spectre utan SOCKS verifierades.
-   Ändringarna rebasades mot senaste `main`, granskades och mergades
    genom PR #16.

## Resultat och lärdomar

Veckans arbete gav en fungerande Zero Trust-baserad åtkomstväg till den
interna miljön och en tydligare förståelse för hur flera komponenter
måste samverka:

`Headscale → Tailscale → Split DNS → dnsmasq → GCP DNS → routing → IP forwarding → NAT → Spectre`

Den första SOCKS-lösningen var användbar och fungerade som fallback. Den
gjorde det möjligt att fortsätta arbetet medan grundorsakerna
analyserades. Den slutliga lösningen åtgärdade däremot problemen i den
normala infrastrukturen och gjorde SOCKS-proxyn onödig för normal
åtkomst.

Vi fick praktisk erfarenhet av hur Terraform, GCP IAM, OS Login, Service
Accounts, VPC-routing, DNS, Headscale/Tailscale och NAT påverkar
varandra.

## Ansvarsfördelning

  ------------------------------------------------------------------------
  Person                  Uppgift                  Status
  ----------------------- ------------------------ -----------------------
  Malcolm                 Felsökning och           ☑ Klar
                          korrigering av DNS,      
                          dnsmasq, Split DNS,      
                          routing/NAT,             
                          Terraform-integration,   
                          verifiering och PR #16   

  Samuel                  Teamarbete, testning och ☑ Genomfört
                          granskning under         
                          workshop/labb            

  Abdulghani              Infrastrukturändringar   ☑ Genomfört
                          och teamarbete via egen  
                          branch/PR                

  Mert                    Teamarbete, testning och ☑ Genomfört
                          granskning               

  Kristoffer              Teamarbete,              ☑ Genomfört
                          Git/GitHub-stöd och      
                          granskning               
  ------------------------------------------------------------------------

## Bevis / länkar

-   Infrastrukturrepo: `team1-infra`
-   Dokumentationsrepo: `team1-documentation`
-   Pull Request: `team1-infra` PR #16
-   Branch: `fix-jumphost-dns-nat`
-   Commit: `3ca9192`
-   Terraform-konfiguration: `main.tf`
-   Headscale: `v0.29.3`
-   Spectre DNS: `spectre.itsx25.chas-lab.dev → 10.0.0.2`
-   HTTP-verifiering: `HTTP/2 200`
-   Terraform slutkontroll:
    `No changes. Your infrastructure matches the configuration.`


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
