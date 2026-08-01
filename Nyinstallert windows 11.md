# Widnows 11 survival guide
Denne guiden er litt blanding av blog, privat og startup. Vil splittes senere, derfor skal jeg jobbe litt godt med denne først siden den blir til tre senere.


Finn iso installer uten bloat fra windows

## Update
- Windows Update
- Restart
- Windows Update
- Restart
- Windows Update


**Dette installeres på alle brukere** ADMIN
## Debloat skript
*Debloat regime: Fjerner kun programmer som* 
1. Skaper støy eller reklame.
2. Gir liten verdi.
3. Har lav risiko for å påvirke systemet.

Testes etter hver større Windows-versjon.

**Kjør som admin i powershell:**
En liste med alle pakker som skal fjernes
```powershell
$BloatwareApps = @(
    "*BingWeather*",                     # Vær-appen. Unødvendig hvis du bruker nettleser.
    "*BingNews*",                        # Nyhetsapp med Microsoft-feed.
    "*BingSearch*",                      # Bing-integrasjon i Windows-søk.
    "*Microsoft.Copilot*",               # Microsoft Copilot. Reklame/AI-assistent hvis du ikke bruker den.
    "*Microsoft.GetHelp*",               # "Få hjelp"-appen. Kan erstattes med Google.
    "*Microsoft.Getstarted*",            # Windows introduksjonsguide. Kun nyttig første gang.
    "*Microsoft.MicrosoftStickyNotes*",  # Gule klistrelapper. Fjern hvis du ikke bruker dem.
    "*Microsoft.People*",                # Kontakt-app. Lite brukt.
#        "*Microsoft.YourPhone*",             # Phone Link. Brukes for mobilintegrasjon. Beholdes midlertidig. Tester Samsung Link to Windows.
#        "*MicrosoftWindows.CrossDevice*",    # Kryssenhetsfunksjoner mellom PC og mobil. Beholdes midlertidig. Kreves trolig av Samsung Link to Windows.
#        "*Microsoft.WindowsFeedbackHub*",    # Feedback Hub. Kun for å sende tilbakemeldinger til Microsoft. Fjern senere
#        "*Microsoft.WindowsAlarms*",         # Klokke, alarmer og timer. # Jeg husker ikke hvorfor jeg tok denne på debloat listen. Hvis denne meldingen er her neste gang så har jeg ikke angret å fjerne den.
# Xbox ting er fjernet pga animasjoner, popups, og den spør om å bruke game pass på apps som ikke krever det dersom det er tilgjengelig. Popups i fullscreen mens jeg spiller, som det ikke er mulig å fjerne eller klikke på. osv osv.
    "*Microsoft.GamingApp*",             # Xbox Gaming App. Unødvendig hvis du kun bruker Steam/Battle.net.
    "*Microsoft.XboxApp*",               # Xbox-appen.
    "*Microsoft.XboxIdentityProvider*",  # Xbox-innlogging som brukes i Game Pass eller Microsoft Store-spill. 
    "*Microsoft.XboxSpeechToTextOverlay*", # Xbox tale-til-tekst.
    "*Microsoft.XboxGamingOverlay*",     # Xbox Game Bar-overlay.
    "*Xbox.TCUI*",                       # Xbox brukergrensesnitt.
    "*Microsoft.MicrosoftSolitaireCollection*" # Solitaire og andre Microsoft-spill.
)
```
Skript som sletter apper som ligger i listen
```powershell
foreach ($App in $BloatwareApps) {
    $Packages = Get-AppxPackage -AllUsers -Name $App -ErrorAction SilentlyContinue

    if (-not $Packages) {
        Write-Host "[INFO] Allerede fjernet eller ikke funnet: $App" -ForegroundColor Yellow
        continue
    }

    $Failed = $false

    foreach ($Package in $Packages) {
        try {
            $Package | Remove-AppxPackage -AllUsers -ErrorAction Stop
        }
        catch {
            $Failed = $true
            Write-Host "[FAILED] $($Package.Name): $($_.Exception.Message)" -ForegroundColor Red
        }
    }

    if (-not $Failed) {
        Write-Host "[OK] Fjernet: $App" -ForegroundColor Green
    }
}
```
## Apper som skal brukes på alle brukerkontoer

Disse kan installeres før flere brukerkontoer opprettes.

**Filosofi:** Programmer installeres med PowerShell/Winget dersom de:

1. Ikke krever manuell konfigurasjon under installasjonen.
2. Er veletablerte pakker med lav risiko for feil eller mangelfulle Winget-pakker.
3. Ikke har noen praktiske fordeler ved manuell installasjon.

```powershell
$Apps = @(
    "Discord.Discord",
    "EclipseAdoptium.Temurin.21.JDK", # Anbefalt 2026-2027, sjekk versjon støtte senere
    "Google.Chrome",              # Bytt denne om du foretrekker noe annet
    "Git.Git",                    # Versjonskontroll
    "GitHub.cli",                 # GitHub fra terminalen.
    "Microsoft.VisualStudioCode", # Must have, kode, dagbok, mm.
    "Microsoft.PowerShell",       # PowerShell 7
    "Microsoft.PowerToys",        # FancyZones, remapping, PowerRename, Always on Top osv.
    "OBSProject.OBSStudio", # # self explanatory
    "Python.Python.3.13", # Anbefalt 2026-2027, sjekk versjon støtte senere
    "Spotify.Spotify", # self explanatory
    "Tailscale.Tailscale", # Egen cloud og remote desktop osv. skalerer inn startup
    "VideoLAN.VLC",               # Mediespiller, mest av nostalgiske grunner
)
```

```powershell
foreach ($App in @($Apps)) {
    Write-Host "`n[INSTALLING] $App med machine-scope" -ForegroundColor Cyan

    winget install `
        --id $App `
        --exact `
        --scope machine `
        --silent `
        --disable-interactivity `
        --accept-package-agreements `
        --accept-source-agreements

    if ($LASTEXITCODE -ne 0) {
        Write-Host "[RETRY] Prøver $App uten machine-scope" -ForegroundColor Yellow

        winget install `
            --id $App `
            --exact `
            --silent `
            --disable-interactivity `
            --accept-package-agreements `
            --accept-source-agreements
    }

    if ($LASTEXITCODE -eq 0) {
        Write-Host "[OK] Installert: $App" -ForegroundColor Green

        # Fjern appen fra listen. $Apps vil til slutt bare inneholde feil.
        $Apps = @($Apps | Where-Object { $_ -ne $App })
    }
    else {
        Write-Host "[FAILED] $App feilet med og uten machine-scope." -ForegroundColor Red
    }
}

if ($Apps.Count -eq 0) {
    Write-Host "`n[OK] Alle apper ble installert." -ForegroundColor Green
}
else {
    Write-Host "`n[FAILED] Disse appene feilet og ligger fortsatt i `$Apps:" -ForegroundColor Red

    foreach ($App in $Apps) {
        Write-Host " - $App" -ForegroundColor Yellow
    }

    Write-Host "`nInstaller disse manuelt, eller undersøk Winget-ID og installasjonskrav." -ForegroundColor Yellow
}
```

Installer manuelt
TEX, MSI, sett opp mobil deling

## Brukere !!

Lag to eller flere brukere til dine bruk, en egen bruker for eksempel studier, en organisasjon etc.
Liste over brukere som skal opprettes:
```powershell
$Users = @(
    @{
        Name        = "Dev"
        Description = "Startup, utvikling og Reglum"
    },
    @{
        Name        = "Gaming"
        Description = "Spill"
    },
    @{
        Name        = "Student"
        Description = "UiO og OsloMet"
    },
    @{
        Name        = "Frivillig"
        Description = "NITO, HEF, fagforening og organisasjoner"
    }
)
```
Lag brukere:
```powershell
foreach ($User in $Users) {
    if (Get-LocalUser -Name $User.Name -ErrorAction SilentlyContinue) {
        Write-Host "[INFO] Brukeren finnes allerede: $($User.Name)" -ForegroundColor Yellow
        continue
    }

    $Password = Read-Host "Skriv passord for $($User.Name)" -AsSecureString

    New-LocalUser `
        -Name $User.Name `
        -Password $Password `
        -Description $User.Description `
        -AccountNeverExpires

    Write-Host "[OK] Opprettet bruker: $($User.Name)" -ForegroundColor Green
}
```


## Installere software:
Funksjon som tar inn bruker name og matcher det med bruker
```powershell
```

```powershell
$dev_apps = @(
gimp, incscape, audacity
```

```powershell
Install_dev_apps
```
Installer manuelt:

**Gaming**
- Battlenet.net -> sc2
- Steam

**Student**
- Office 365
- Zoom
- Teams
- Slack

**Frivillig**
- Office 365
- Teams

- Logg in med ulike brukere brukernavn**+student**@leverandør.xx


# Konfigurering

- Finn ut hvordan man omvender fn tasten for å få rask tilgang til f tastene.(og sc2)
- Nvidia App
- Instillinger -> skjerm -> lysstyrke og farge
- Innstillinger -> System -> Strøm og dvalemodus -> Avanserte
- Kontrollpanel → Lyd → Lyder → Ingen lyder → OK
- Kjør (Win + R) → sysdm.cpl → Avansert → Ytelse → Innstillinger → Juster for best ytelse(fjern: animasjoner, skygge og fade effects) → Bruk & OK
- Win + R -> control input.dll -> Advanced Key Settings -> Change Key Sequence -> Sett alt til Not Assigned hvis du vil deaktivere hurtigtaster for språkbytte.
- Dessverre er innstillingen for bakgrunds-kjøring inne på hver app i Windows 11, så du må gå inn i hver relevant app og deaktivere. Stort sett kan du la det være, men bakgrunnapper stjeler batterilevetid.
- Ctrl + Shift + Esc → Oppgavebehandling → Oppstart → Høyreklikk app → Deaktiver
- Innstillinger → System → Varslinger → Skru av varsler
- Instillinger -> Personvern og instillinger -> Søketilatelser -> La søkeapper vise resultater -> Av
evt innstillingsknapp i oppgavemenyen
- Slå av pekertapping på touchpad
- Fjerne faner fra alt-tab Instillinger -> System -> Fleroppgavevisning -> Skru av "vis faner fra apper når du trykker alt + tab"
- Fra oppstartsmenyen ctrl + shift + esc - deaktivere Xbox, Steam, OneDrive, Microsoft 365 Copilot, Microsoft Teams, MSI Center, Killer Intelligence Center, Terminal og eventuelt Telefonkobling.
- Instillinger - > Personalisering -> låseskjerm -> bakgrunnsbilde -> velg bilde # Gjør at windows slutter å laste end masse bilder for så å slette dem.
- Instillinger -> System -> Fleroppgavekjøring -> ikke vis faner når en trykker alt tab.



taskschd.msc -> Microsoft -> XblGameSave -> Disable #XBOX åpner seg hver gang pcen blir inaktiv for å sjekke om den skal lagre spill og er hardkodet til os'et.

- deaktiver filtertaster hotkey
- deaktiver trege taster hotkey
- slå av pekerpresisjon


Installer -> power toys -> og fjern popup spam knapper som ikke er ønskelig. Coopilot knappen som ertatter ctrl knappen.
Disable Widgets
Disable Copilot if you don't use it.
Disable Game DVR
- Settings
- Gaming
- Captures

enable system restore

enable bitlocker 

Set PowerShell 7 as default in w terminal

Enable Developer Mode -> Settings → System → For developers

Git git config --global user.name ""
git config --global user.email ""

Enhanced indexing off

OneDrive uninstall

Refreshrate check

Windows power plan -> max

NVIDIA Control Panel

For SC2 specifically I'd test:

Power Management → Prefer maximum performance
Low Latency Mode → Off / On / Ultra (benchmark all three)
V-Sync → Off
Maximum Frame Rate → Off (unless needed) -> (lapptop only have 60 fps)
G-SYNC configuration if your display supports it

Student bruker
Office 365
