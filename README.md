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


# Vecka 6 -- Workshop 3 Blue Team

## Team 1 -- Slutdokumentation

**Kurs:** Kurs 6 -- Avancerad IT-säkerhet\
**Team:** Team 1\
**GCP-projekt:** `itsx25-lab`\
**GitHub-organisation:** `CHAS-Grupp-1-kurs-6`\
**Applikationsrepo:** `company-website`\
**Infrastrukturrepo:** `team1-infra`

------------------------------------------------------------------------

## 1. Syfte

Syftet med Workshop 3 var att få den interna webbapplikationen att
fungera i Team 1:s miljö och bygga ett fungerande deploymentflöde med
GitHub Actions, Headscale/Tailscale och K3s.

Arbetet omfattade: - nätverk, routing, DNS och NAT - primary-VM och
Terraform - installation och verifiering av K3s - GitHub Actions åtkomst
till K3s via Headscale - deployment av `company-website` - felsökning av
GHCR och container images - Kubernetes rollout och `hostPort` -
verifiering av applikationen - säkerhetstest och IDOR

------------------------------------------------------------------------

## 2. Team 1:s miljö

  Komponent             Adress
  --------------------- ---------------
  Team 1 subnet         `10.0.1.0/24`
  Jumphost intern IP    `10.0.1.2`
  Primary intern IP     `10.0.1.3`
  Jumphost tailnet      `100.64.0.1`
  Primary tailnet       `100.64.0.8`
  Mac tailnet           `100.64.0.4`
  Sharp reverse proxy   `10.0.0.2`

Primary kör Debian, maskintyp `e2-small`, och används som K3s-nod.

------------------------------------------------------------------------

## 3. IAP och åtkomst till Primary

Under arbetet fanns problem med åtkomst till primary via GCP/IAP. Dennis
hjälpte till med saknad IAM/firewall-relaterad konfiguration. Efter
korrigeringen fungerade åtkomsten.

Exempel:

``` bash
ssh malcolm@10.0.1.3
```

GCP IAP användes även för SSH och filöverföring.

------------------------------------------------------------------------

## 4. Terraform och Primary

Primary aktiverades i Terraform och konfigurationen applicerades.

Resultat:

``` text
1 added, 1 changed, 0 destroyed
```

Primary fick intern IP `10.0.1.3` och maskintyp `e2-small`.

Under arbetet var det viktigt att undvika Terraform-förändringar som
kunde ersätta jumphosten.

------------------------------------------------------------------------

## 5. NAT och internet

Primary saknade periodvis internetåtkomst. Under felsökningen användes
följande på jumphosten:

``` bash
sudo iptables -t nat -A POSTROUTING -s 10.0.1.0/24 -o ens4 -j MASQUERADE
```

Efter korrigeringen gav:

``` bash
curl -I --connect-timeout 10 https://ghcr.io/v2/
```

ett HTTP-svar i stället för timeout.

Vid kontroll av Terraform/startup-scriptet konstaterades att
NAT/MASQUERADE-logik redan finns definierad där. Den manuella regeln
användes därför under felsökningen.

------------------------------------------------------------------------

## 6. DNS på Primary

Tailscale tog över DNS på primary och `tailscale0` ägde den globala
DNS-routen. Detta bröt namnuppslagningen.

Lösning:

``` bash
sudo tailscale set --accept-dns=false
sudo systemctl restart systemd-resolved
```

Verifiering:

``` bash
getent hosts company-website.itsx25.chas-lab.dev
```

gav `10.0.0.2`.

------------------------------------------------------------------------

## 7. Headscale/Tailscale-routing

Jumphosten konfigurerades att annonsera Team 1:s subnet och reverse
proxy:

``` bash
sudo tailscale set --advertise-routes=10.0.0.2/32,10.0.1.0/24
```

På Macen:

``` bash
tailscale set --accept-routes=true
```

Routing verifierades med:

``` bash
route -n get 10.0.0.2 | grep interface
```

och gick via Tailscale-interface.

------------------------------------------------------------------------

## 8. Sharp-adressen

Från Macen:

``` bash
curl -I --connect-timeout 10 https://company-website.itsx25.chas-lab.dev/
```

gav:

``` text
HTTP/2 200
server: gunicorn
```

Webbplatsen fungerade även i Safari.

------------------------------------------------------------------------

## 9. K3s

K3s installerades på primary:

``` bash
curl -sfL https://get.k3s.io | sh -s - --disable=traefik --disable=metrics-server
```

Noden verifierades som `Ready`.

`k8s/github-permissions.yaml` applicerades för GitHub-relaterade
Kubernetes-rättigheter. En service account `github-deployer` med
tillhörande token och RBAC användes för deployment.

------------------------------------------------------------------------

## 10. Rätt applikation

I början hade fel applikation deployats. Rätt kursapplikation hämtades
från det interna repot:

``` bash
git clone ssh://git@10.0.0.2/home/git/repos/company-website.git
```

Arbetskopian blev:

``` text
~/company-website-real
```

GitHub-repot är `CHAS-Grupp-1-kurs-6/company-website`.

------------------------------------------------------------------------

## 11. Git-remotes

Två remotes användes.

`origin`:

``` text
ssh://git@10.0.0.2/home/git/repos/company-website.git
```

`github`:

``` text
git@github.com:CHAS-Grupp-1-kurs-6/company-website.git
```

Lokalt användes `master`, medan GitHub använde `main`. Push gjordes
därför med:

``` bash
git push github master:main
```

Vid en divergens mellan lokal branch och GitHub användes:

``` bash
git fetch github main
git rebase github/main
git push github master:main
```

------------------------------------------------------------------------

## 12. GitHub Actions och Headscale

GitHub Actions behövde nå K3s i det privata nätet. Headscale användes
för att ansluta GitHub-runnern till Team 1:s mesh.

Headscale-användare:

``` text
github-ci
```

User ID:

``` text
8
```

GitHub Variables:

``` text
TEAM_ID=1
HEADSCALE_USER_ID=8
```

Pipeline-stegen omfattade:

``` text
Build and push image
Generate Ephemeral Headscale Key
Connect to Headscale Mesh
Verify connectivity to K3s node
Configure kubeconfig
Apply manifests and update image
```

ACL gav runnern nödvändig åtkomst till primary och K3s API.

------------------------------------------------------------------------

## 13. Kubeconfig

GitHub Secret `KUBECONFIG` används för remote deployment.

Workflowen gör bland annat:

``` bash
mkdir -p ~/.kube
echo "${{ secrets.KUBECONFIG }}" | base64 -d > ~/.kube/config
chmod 600 ~/.kube/config
kubectl get pods -n default
```

Det verifierade att GitHub Actions kunde kommunicera med K3s.

------------------------------------------------------------------------

## 14. GHCR-problemet

Actions kunde bygga och pusha imagen till GHCR. GHCR-namnet korrigerades
till lowercase:

``` text
ghcr.io/chas-grupp-1-kurs-6/company-website:latest
```

Paketet var dock privat. K3s fick:

``` text
401 Unauthorized
```

vid anonym pull.

Workshopens tänkta lösning var ett publikt GitHub Package, men
organisationens inställningar blockerade Public-alternativet.

------------------------------------------------------------------------

## 15. Lokal image som workaround

Imagen byggdes på Mac:

``` bash
docker build --platform linux/amd64 -t company-website:latest .
docker save company-website:latest -o company-website.tar
```

Den kopierades till primary:

``` bash
gcloud compute scp company-website.tar team1-primary:.   --project=itsx25-lab   --zone=europe-north2-a   --tunnel-through-iap
```

Sedan importerades den i K3s/containerd:

``` bash
sudo k3s ctr images import company-website.tar
```

Deploymenten ändrades till:

``` yaml
image: company-website:latest
imagePullPolicy: Never
```

Detta gör att K3s använder den lokalt importerade imagen.

**Viktigt:** Detta är en fungerande workshop-workaround, men inte
fullständig produktionslik CI/CD. Workflowen bygger fortfarande
GHCR-imagen, medan deploymenten använder den lokala imagen. En permanent
lösning är publik GHCR eller autentisering mot privat GHCR med
exempelvis `imagePullSecret`.

------------------------------------------------------------------------

## 16. Applikationsverifiering

På primary:

``` bash
curl -I http://localhost
```

gav:

``` text
HTTP/1.1 200 OK
```

Health endpoint:

``` bash
curl http://localhost/healthz
```

gav:

``` json
{"db":"connected","status":"healthy"}
```

Den skarpa adressen fungerade också.

------------------------------------------------------------------------

## 17. Rollout-problemet

En tidigare Actions-körning fastnade med:

``` text
Waiting for deployment "company-website" rollout to finish:
1 old replicas are pending termination...
error: timed out waiting for the condition
```

Två ReplicaSets fanns samtidigt. Den gamla poden fungerade medan den nya
blev `Pending`.

`kubectl describe pod` visade:

``` text
FailedScheduling
0/1 nodes are available:
1 node(s) didn't have free ports for the requested pod ports
```

------------------------------------------------------------------------

## 18. HostPort-konflikten

Manifestet innehåller:

``` yaml
ports:
- containerPort: 7000
  hostPort: 80
```

Vid `RollingUpdate` försöker Kubernetes normalt starta en ny pod innan
den gamla avslutas. Eftersom båda behöver hostens port 80 och klustret
har en nod kunde den nya poden inte schemaläggas.

Detta var orsaken till `Pending` och rollout-timeout.

------------------------------------------------------------------------

## 19. Recreate-fix

I `k8s/deployment.yaml` lades:

``` yaml
spec:
  replicas: 1
  strategy:
    type: Recreate
```

till för att den gamla poden ska avslutas innan den nya skapas.

Commit:

``` text
af03af4  fix deployment rollout with hostPort
```

Actions-körningen för denna commit blev grön.

------------------------------------------------------------------------

## 20. Kontroll av Deployment-strategin

Vid en efterföljande kontroll visade det live Deployment-objekt som
inspekterades fortfarande:

``` text
RollingUpdate
```

trots att `github/main` innehöll:

``` yaml
strategy:
  type: Recreate
```

GitHub-versionen verifierades med:

``` bash
git show github/main:k8s/deployment.yaml | sed -n '1,25p'
```

och innehöll `Recreate`.

`last-applied-configuration` som inspekterades saknade samtidigt
`strategy`, vilket ledde till ytterligare diagnostik av vad Actions
genererade.

------------------------------------------------------------------------

## 21. Diagnostik i GitHub Actions

Workflowen kompletterades med:

``` bash
envsubst < k8s/deployment.yaml > /tmp/deployment.yaml
echo "=== GENERATED DEPLOYMENT ==="
cat /tmp/deployment.yaml
echo "=== END GENERATED DEPLOYMENT ==="
kubectl apply -f /tmp/deployment.yaml -f k8s/pvc.yaml -f k8s/service.yaml
```

Commit:

``` text
2a49a8b  debug generated deployment manifest
```

Den efterföljande körningen:

``` text
Deploy to K3s #5
Commit: 2a49a8b
Branch: main
Status: Success
Duration: 57s
```

slutfördes framgångsrikt.

Denna körning visar att deploymentflödet nu går igenom utan den tidigare
timeouten. Diagnostikutskriften finns kvar för att kunna verifiera det
genererade manifestet.

------------------------------------------------------------------------

## 22. Slutstatus för Kubernetes

Under slutkontrollen fanns en fungerande pod:

``` text
READY   STATUS
1/1     Running
```

Den tidigare `Pending`-poden var borta.

Deploymenten verifierades använda:

``` text
company-website:latest
imagePullPolicy: Never
```

Senaste GitHub Actions-körningen är grön.

------------------------------------------------------------------------

## 23. Säkerhetstest och IDOR

Applikationen testades även säkerhetsmässigt. Workshopens
utvecklingskonto `dev` användes.

Profilfunktionen använde URL:er:

``` text
/profiles/<id>/edit
```

Det upptäcktes att applikationen inte tillräckligt kontrollerade att den
inloggade användaren endast fick komma åt sin egen profil.

Genom att ändra ID kunde andra användares profilinformation visas.
Testningen gjordes read-only och inga ändringar sparades.

`dev` hade profil-ID `6`, och profiler med ID 1 till 6 kunde observeras.

------------------------------------------------------------------------

## 24. Exempel på information via IDOR

Alice Admin:

``` text
Password vault backup location: /opt/vault/backup.zip
```

Charlie Coder:

``` text
Secret project codename: Phoenix. Repo is hidden under /internal/phoenix.git
```

Dave Designer:

``` text
Admin panel mockups are in /shared/admin_v2.fig. Still using the old FTP server.
```

dev:

``` text
Remember to rotate the dev-secret-key before production. Also, the staging DB password is staging123.
```

Detta demonstrerade konsekvensen av bristande objektbaserad
åtkomstkontroll.

------------------------------------------------------------------------

## 25. Flagga via IDOR

I Bob Boss profil observerades följande flagga i `Internal Notes`:

``` text
ITSX25{th1s_1s_d3f1n1t3ly_n0t_my_pr0f1l3}
```

Flaggan hittades genom IDOR-sårbarheten i profilfunktionen.

------------------------------------------------------------------------

## 26. Lokala placeholder-flaggor

Under tidigare lokal felsökning observerades även:

``` text
ITSX25{this_is_a_placeholder}
ITSX25{this_is_a_second_placeholder}
```

Dessa var placeholder-data från lokal/testrelaterad applikationsdata och
ska skiljas från flaggan som observerades via IDOR.

------------------------------------------------------------------------

## 27. Problem, orsak och lösning

  ----------------------------------------------------------------------------------------
  Problem                 Orsak                       Åtgärd
  ----------------------- --------------------------- ------------------------------------
  Primary-åtkomst         IAP/IAM/firewall            Konfiguration korrigerades med hjälp
                                                      av Dennis

  Primary saknade         NAT/routing                 MASQUERADE användes/verifierades
  internet                                            

  DNS bröts               Tailscale tog över DNS      `tailscale set --accept-dns=false`

  Sharp-adress ej nåbar   Route till `10.0.0.2`       Route annonserades via jumphost
                          saknades                    

  Fel app                 Fel applikation deployad    Rätt internt repo klonades

  Actions nådde inte K3s  Privat nät                  Headscale/Tailscale

  GHCR pull misslyckades  Privat package, 401         Lokal image importerades

  Ny pod blev Pending     `hostPort: 80` upptagen     `Recreate` lades till i manifestet

  Actions timeout         Rollout blockerad av        Orsaken identifierades och flödet
                          HostPort                    korrigerades

  Profilåtkomst           Saknad                      IDOR identifierades
                          objektbehörighetskontroll   
  ----------------------------------------------------------------------------------------

------------------------------------------------------------------------

## 28. Viktiga commits

``` text
9364f22  fix: use lowercase GHCR image name
005cf25  fix deployment to use local k3s image
af03af4  fix deployment rollout with hostPort
2a49a8b  debug generated deployment manifest
```

Senaste verifierade Actions-körning:

``` text
Commit: 2a49a8b
Branch: main
Status: Success
```

------------------------------------------------------------------------

## 29. Credentials och säkerhet

API-nycklar, tokens och andra credentials ska inte sparas i Git eller
dokumentation.

Temporära Headscale-nyckelfiler bör städas:

``` bash
sudo rm -f /tmp/hskey
```

Exponerade nycklar bör roteras/inaktiveras.

Känslig information lagras som GitHub Secrets, exempelvis:

``` text
HEADSCALE_API_KEY
KUBECONFIG
```

------------------------------------------------------------------------

## 30. Teknisk skuld

### GHCR

Nuvarande lösning använder `company-website:latest` med
`imagePullPolicy: Never`. För riktig CI/CD bör K3s kunna hämta den image
som Actions precis byggt, genom publik GHCR eller autentisering mot
privat GHCR.

### Tailscale

Manuella inställningar som route advertisement och `accept-dns=false`
bör vid behov göras reproducerbara i provisioning.

### NAT

NAT-logik finns i startup-konfigurationen men bör verifieras efter
reboot/redeploy.

### Diagnostik

`cat /tmp/deployment.yaml` är felsökningsdiagnostik och kan tas bort när
manifestbeteendet är slutligt verifierat.

### Recreate

`Recreate` finns i GitHub-manifestet och den senaste Actions-körningen
är grön. Eftersom ett tidigare live-objekt fortfarande visade
`RollingUpdate` bör det genererade manifestet/live-strategin verifieras
en sista gång innan man beskriver just den detaljen som permanent
verifierad.

------------------------------------------------------------------------

-

## 31. Härdning med att ta bort gamla SSH konton

vi tog bort gamla SSH inloggningar så att dom inte kan bli utnyttjade i framtiden.

Att minska attack området är kritiskt för att ha en säker miljö

------------------------------------------------------------------------

## 32. Kort sammanfattning

Team 1 felsökte hela kedjan från GCP, NAT, DNS och Headscale-routing
till K3s och GitHub Actions.

De största problemen var nätverksåtkomst, DNS, privat GHCR, container
image-hantering och en Kubernetes-rollout där två pods konkurrerade om
`hostPort: 80`.

Applikationen körs nu i K3s, en pod har verifierats som `1/1 Running`,
webbplatsen och `/healthz` fungerar och senaste GitHub Actions-körningen
slutfördes framgångsrikt.

Säkerhetstestningen identifierade dessutom en IDOR-sårbarhet som
exponerade andra användares interna profilinformation och ledde till
workshopflaggan.

Härdade kod
### Senaste verifierade pipeline

``` text
Deploy to K3s #5
Commit: 2a49a8b
Branch: main
Status: Success
Duration: 57s
```

# Vecka 6 – CI/CD, sårbarhetsanalys och SBOM-spårbarhet

**Datum:** 15/9–19/9

## Fokus

- Få den riktiga `company-website`-applikationen (Flask) att köra i k3s-klustret, både lokalt på team1-primary och via den automatiska CI/CD-pipelinen till skarp miljö.
- Hitta och exploatera sårbarheter i applikationen för att samla in flaggor.
- Sätta upp spårbarhet för containerimages: SHA-taggning, SBOM (Dependency-Track) och signering (Cosign).

## Vårt arbete

### Vad gjorde vi?

Upptäckte att det som tidigare var deployat i klustret var en tom platshållar-app (Node.js-stub), inte den riktiga uppgiften. Klonade rätt källkod från kursens interna git-server och byggde om Docker-imagen (`--platform linux/amd64` för att matcha GCP-nodernas arkitektur). Importerade imagen i k3s, rättade portkonfigurationen (appen kör på port 7000, tidigare felkonfigurerad mot port 3000), städade bort gamla ReplicaSets som orsakade schemaläggningskonflikter, och exponerade appen via en NodePort-service.

Loggade in lokalt som `dev`/`devpass123` och började utforska applikationen för sårbarheter.

Parallellt felsökte vi varför den automatiska deploy-pipelinen (GitHub Actions → self-hosted Headscale-mesh → k3s) inte fungerade mot den skarpa miljön. Grundorsaken var att `team1-primary` aldrig hade anslutits till lagets egna Headscale-mesh, vilket gjorde att GitHub Actions-runnern inte kunde nå noden. Installerade tailscale-klienten på primary, anslöt den mot `https://team1.itsx25.chas-lab.dev` med `--advertise-routes=10.0.1.0/24`, och godkände routen på Headscale-servern. Rotera även en förlegad Headscale API-nyckel som användes av pipelinens steg för att generera ephemeral preauth-nycklar.

Utöver detta satte vi upp Dependency-Track (SBOM-verktyg) via Helm i ett eget namespace, för att kunna pusha SBOM:er genererade med `syft` och få spårbarhet på beroenden i containerimagen.

### Vilka tester genomförde vi?

- Manuell utforskning av applikationens routes (`/profile`, `/profiles/<id>`, `/profiles/<id>/edit`, `/employees`).
- Testade API-enumerering (`/api/users`, `/api/employees`, etc.) utan resultat, applikationen exponerar ingen sådan API-yta.
- Läste igenom källkoden (`routes.py`) för att identifiera saknade behörighetskontroller.
- Testade att komma åt och redigera andra användares profiler genom att ändra ID:t i URL:en.

### Vad hittade vi?

**IDOR-sårbarhet (Insecure Direct Object Reference):** routen `/profiles/<id>/edit` har `@login_required` men saknar kontroll av att den inloggade användaren faktiskt äger profilen. Vilken inloggad användare som helst kan alltså se och redigera vem som helst annans profil, inklusive fältet `internal_notes`.

Genom att läsa `users`-tabellen direkt i applikationens SQLite-databas (`/app/data/database.db`) hittade vi:

- `ITSX25{this_is_a_placeholder}` – låg felaktigt i lösenordshash-fältet för kontot `flag`
- `ITSX25{this_is_a_second_placeholder}` – i interna anteckningar för `Bob Boss` (CEO)

Ytterligare ledtrådar i databasen (ej bekräftade flaggor):
- Alice (Database Manager): referens till en password vault-backup på `/opt/vault/backup.zip`
- Charlie (Software Developer): dolt repo `/internal/phoenix.git`
- Dave (UI/UX Designer): referens till en gammal FTP-server och admin-mockups

### Vilka åtgärder genomfördes?

- Fixade portkonfiguration och stale ReplicaSets i den lokala k3s-deploymenten.
- Anslöt team1-primary till lagets Headscale-mesh och godkände dess subnet-route, vilket löste CI/CD-pipelinens anslutningsproblem.
- Roterade en förlegad Headscale API-nyckel.
- Installerade Dependency-Track via Helm, inklusive en egen Postgres-databas med persistent lagring (`PersistentVolumeClaim`), efter att en första version utan persistent volym tappade all data (inklusive adminkontot) vid varje pod-omstart.
- Dokumenterade hela felsöknings- och installationsprocessen för Dependency-Track separat, se länkar nedan.

## Ansvarsfördelning

| Person | Uppgift | Status |
|---|---|---|
| Malle | Deploy av rätt applikation, CI/CD-felsökning (Headscale), sårbarhetsanalys (IDOR), installation av Dependency-Track | Klart |
| | | |
| | | |

## Bevis / länkar

- Statusrapport: deploy och CI/CD-fix (`status-company-website-deploy.md`)
- Installationsguide och felsökning: Dependency-Track (`dependency-track-setup.md`)
- Flaggor: `ITSX25{this_is_a_placeholder}`, `ITSX25{this_is_a_second_placeholder}`
- Skärmdumpar av utnyttjad IDOR-sårbarhet och Dependency-Track-dashboard (se teamets delade mapp/Discord)


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

# Vecka 6 – CI/CD, sårbarhetsanalys och SBOM-spårbarhet

**Datum:** 15/9–19/9

## Fokus

- Få den riktiga `company-website`-applikationen (Flask) att köra i k3s-klustret, både lokalt på team1-primary och via den automatiska CI/CD-pipelinen till skarp miljö.
- Hitta och exploatera sårbarheter i applikationen för att samla in flaggor.
- Sätta upp spårbarhet för containerimages: SHA-taggning, SBOM (Dependency-Track) och signering (Cosign).

## Vårt arbete

### Vad gjorde vi?

Upptäckte att det som tidigare var deployat i klustret var en tom platshållar-app (Node.js-stub), inte den riktiga uppgiften. Klonade rätt källkod från kursens interna git-server och byggde om Docker-imagen (`--platform linux/amd64` för att matcha GCP-nodernas arkitektur). Importerade imagen i k3s, rättade portkonfigurationen (appen kör på port 7000, tidigare felkonfigurerad mot port 3000), städade bort gamla ReplicaSets som orsakade schemaläggningskonflikter, och exponerade appen via en NodePort-service.

Loggade in lokalt som `dev`/`devpass123` och började utforska applikationen för sårbarheter.

Parallellt felsökte vi varför den automatiska deploy-pipelinen (GitHub Actions → self-hosted Headscale-mesh → k3s) inte fungerade mot den skarpa miljön. Grundorsaken var att `team1-primary` aldrig hade anslutits till lagets egna Headscale-mesh, vilket gjorde att GitHub Actions-runnern inte kunde nå noden. Installerade tailscale-klienten på primary, anslöt den mot `https://team1.itsx25.chas-lab.dev` med `--advertise-routes=10.0.1.0/24`, och godkände routen på Headscale-servern. Roterade även en förlegad Headscale API-nyckel som användes av pipelinens steg för att generera ephemeral preauth-nycklar.

Utöver detta satte vi upp Dependency-Track (SBOM-verktyg) via Helm i ett eget namespace, för att kunna pusha SBOM:er genererade med `syft` och få spårbarhet på beroenden i containerimagen.

### Vilka tester genomförde vi?

- Manuell utforskning av applikationens routes (`/profile`, `/profiles/<id>`, `/profiles/<id>/edit`, `/employees`).
- Testade API-enumerering (`/api/users`, `/api/employees`, etc.) utan resultat, applikationen exponerar ingen sådan API-yta.
- Läste igenom källkoden (`routes.py`) för att identifiera saknade behörighetskontroller.
- Testade att komma åt och redigera andra användares profiler genom att ändra ID:t i URL:en.

### Vad hittade vi?

**IDOR-sårbarhet (Insecure Direct Object Reference):** routen `/profiles/<id>/edit` har `@login_required` men saknar kontroll av att den inloggade användaren faktiskt äger profilen. Vilken inloggad användare som helst kan alltså se och redigera vem som helst annans profil, inklusive fältet `internal_notes`.

Genom att läsa `users`-tabellen direkt i applikationens SQLite-databas (`/app/data/database.db`) hittade vi:

- `ITSX25{this_is_a_placeholder}` – låg felaktigt i lösenordshash-fältet för kontot `flag`
- `ITSX25{this_is_a_second_placeholder}` – i interna anteckningar för `Bob Boss` (CEO)

Ytterligare ledtrådar i databasen (ej bekräftade flaggor):
- Alice (Database Manager): referens till en password vault-backup på `/opt/vault/backup.zip`
- Charlie (Software Developer): dolt repo `/internal/phoenix.git`
- Dave (UI/UX Designer): referens till en gammal FTP-server och admin-mockups

### Vilka åtgärder genomfördes?

- Fixade portkonfiguration och stale ReplicaSets i den lokala k3s-deploymenten.
- Anslöt team1-primary till lagets Headscale-mesh och godkände dess subnet-route, vilket löste CI/CD-pipelinens anslutningsproblem.
- Roterade en förlegad Headscale API-nyckel.
- Installerade Dependency-Track via Helm, inklusive en egen Postgres-databas med persistent lagring (`PersistentVolumeClaim`), efter att en första version utan persistent volym tappade all data (inklusive adminkontot) vid varje pod-omstart.
- Dokumenterade hela felsöknings- och installationsprocessen för Dependency-Track separat, se länkar nedan.

## Ansvarsfördelning

| Person | Uppgift | Status |
|---|---|---|
| Malle | Deploy av rätt applikation, CI/CD-felsökning (Headscale), sårbarhetsanalys (IDOR), installation av Dependency-Track | Klart |
| | | |
| | | |

## Bevis / länkar

- Statusrapport: deploy och CI/CD-fix (`status-company-website-deploy.md`)
- Installationsguide och felsökning: Dependency-Track (`dependency-track-setup.md`)
- Flaggor: `ITSX25{this_is_a_placeholder}`, `ITSX25{this_is_a_second_placeholder}`
- Skärmdumpar av utnyttjad IDOR-sårbarhet och Dependency-Track-dashboard (se teamets delade mapp/Discord)
