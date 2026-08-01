# Widnows 11 survival guide
Denne guiden er litt blanding av blog, privat og startup. Vil splittes senere, derfor skal jeg jobbe litt godt med denne først siden den blir til tre senere.
Husk å restart etter å ha kjørt kode herifra. Nesten alltid, så bare alltid gjør det.

Finn iso installer uten bloat fra windows

## Update
- Windows Update
- Restart
- Windows Update
- Restart
- Windows Update

## Maskinomfattende oppsett

Kjøres som administrator før de øvrige brukerkontoene konfigureres.

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
    "EclipseAdoptium.Temurin.21.JDK", # Anbefalt 2026-2027, sjekk versjon støtte senere
    "Google.Chrome",              # Bytt denne om du foretrekker noe annet
    "Git.Git",                    # Versjonskontroll
    "GitHub.cli",                 # GitHub fra terminalen.
    "Microsoft.VisualStudioCode", # Must have, kode, dagbok, mm.
    "Microsoft.PowerShell",       # PowerShell 7
    "Microsoft.PowerToys",        # FancyZones, remapping, PowerRename, Always on Top osv.
    "OBSProject.OBSStudio", # # self explanatory
    "Python.Python.3.13", # Anbefalt 2026-2027, sjekk versjon støtte senere
    "Tailscale.Tailscale", # Egen cloud og remote desktop osv. skalerer inn startup
    "VideoLAN.VLC"               # Mediespiller, mest av nostalgiske grunner
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

## Strømisntillinger og annet
```powershell
# Slå aldri av skjermen
powercfg /change monitor-timeout-ac 0
powercfg /change monitor-timeout-dc 0

# Gå aldri i dvale
powercfg /change standby-timeout-ac 0
powercfg /change standby-timeout-dc 0

# Sikkre at hybernate er tilgjengelig
powercfg /hibernate on

# Når lokket lukkes ac/dc:
# 0 = Gjør ingenting
# 1 = Sleep
# 2 = Hibernate
# 3 = Shut down

powercfg /setacvalueindex SCHEME_CURRENT SUB_BUTTONS LIDACTION 2
powercfg /setdcvalueindex SCHEME_CURRENT SUB_BUTTONS LIDACTION 2
powercfg /setactive SCHEME_CURRENT

# Slå av System Restore (snapshot på C:)
Disable-ComputerRestore -Drive "C:\"
```



## Globale instillinger
```powershell
$MachineRegistry = @{
    "HKLM:\SOFTWARE\Policies\Microsoft\Dsh" = @{
        AllowNewsAndInterests = 0 # Widgets av
    }

    "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Authentication\LogonUI\BootAnimation" = @{
        DisableStartupSound = 1 # Oppstartslyd av
    }
}
```

```powershell
foreach ($Path in $MachineRegistry.Keys) {
    if (-not (Test-Path $Path)) {
        New-Item -Path $Path -Force | Out-Null
    }

    foreach ($Property in $MachineRegistry[$Path].GetEnumerator()) {
        try {
            $PropertyType = if ($Property.Value -is [string]) {
                "String"
            }
            else {
                "DWord"
            }

            New-ItemProperty `
                -Path $Path `
                -Name $Property.Key `
                -Value $Property.Value `
                -PropertyType $PropertyType `
                -Force `
                -ErrorAction Stop |
                Out-Null

            Write-Host `
                "[OK] $Path\$($Property.Key) = $($Property.Value) [$PropertyType]" `
                -ForegroundColor Green
        }
        catch {
            Write-Host `
                "[FAILED] $Path\$($Property.Key): $($_.Exception.Message)" `
                -ForegroundColor Red
        }
    }
}
```
## Kan ikke automatiseres (tilganger evt annet)
Developer Mode -> Settings → System → For developers → Developer Mode

- MSI Center (<https://www.msi.com/Landing/msi-center/download>)
- Nvidia App (<https://www.nvidia.com/software/nvidia-app/>)


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
Funksjon som installer alle apps men bruker spesifikt.
**Dev**

```powershell
runas /user:.\Dev pwsh
```

```powershell  
$Apps = @(
    "GIMP.GIMP",
    "Inkscape.Inkscape",
    "Audacity.Audacity",
    "Spotify.Spotify", # er brukerspesifikk og installeres i locale, må kanskje derfor ligge i alle
    "WhatsApp.WhatsApp", # er brukerspesifikk og installeres i locale, må kanskje derfor ligge i alle
    "Discord.Discord" # er brukerspesifikk og installeres i locale, må kanskje derfor ligge i alle
)
```
```powershell  
foreach ($App in @($Apps)) {

    Write-Host "`n[INSTALLING] $App" -ForegroundColor Cyan

    winget install `
        --id $App `
        --exact `
        --scope user `
        --silent `
        --disable-interactivity `
        --accept-package-agreements `
        --accept-source-agreements

    if ($LASTEXITCODE -eq 0) {
        Write-Host "[OK] Installert: $App" -ForegroundColor Green
    }
    else {
        Write-Host "[FAILED] $App" -ForegroundColor Red
    }
}
```
```powershell  


$Registry = @{
    "HKCU:\Control Panel\Mouse" = @{
        MouseSensitivity = "13" # Ca. 65 til 70 % av Windows-slideren
        MouseSpeed       = "0"  # Museakselerasjon av
        MouseThreshold1  = "0"  # Akselerasjonsterskel av
        MouseThreshold2  = "0"  # Akselerasjonsterskel av
    }

    "HKCU:\Control Panel\Accessibility\StickyKeys" = @{
        Flags = "26" # Sticky Keys og tilhørende hurtigtast, ikon, låsing og lyd av
    }

    "HKCU:\Control Panel\Accessibility\Keyboard Response" = @{
        AutoRepeatDelay       = "0"
        AutoRepeatRate        = "0"
        BounceTime            = "0"
        DelayBeforeAcceptance = "0"
        Flags                 = "26" # Filter Keys og hurtigtasten av
    }
    "HKCU:\Software\Microsoft\Windows\CurrentVersion\PushNotifications" = @{
        ToastEnabled           = 0 # Deaktiver varsler
        LockScreenToastEnabled = 0 # Ingen varsler på låseskjermen
    }
    "HKCU:\Software\Microsoft\Windows\CurrentVersion\SearchSettings" = @{
        IsGlobalWebSearchProviderToggleEnabled = 0 # Deaktiver webresultater fra søkeapper
    }
    "HKCU:\Software\Microsoft\Windows\CurrentVersion\PrecisionTouchPad" = @{
        LeaveOnWithMouse           = [uint32]::MaxValue
        PanEnabled                 = [uint32]::MaxValue
        RightClickZoneEnabled      = [uint32]::MaxValue
        TapAndDrag                 = [uint32]::MaxValue
        TapsEnabled                = 0
        TwoFingerTapEnabled       = 0
        ZoomEnabled                = [uint32]::MaxValue
    
        FeedbackIntensity          = 50
        ClickForceSensitivity      = 50
        FeedbackEnabled            = [uint32]::MaxValue
        HonorMouseAccelSetting     = 0
    
        RightClickZoneWidth        = 0
        RightClickZoneHeight       = 0
        EdgeAutoPanningEnabled     = 1
        BoostedPanningEnabled      = 1
        PanSensitivity             = 50
        ZoomSensitivity            = 50
        SingleFingerPanningMode    = 2
        PressureAutoPanningEnabled = 1
    
        ThreeFingerSlideEnabled    = 65535
        FourFingerSlideEnabled     = 65535
    
        ThreeFingerTapEnabled      = 0
        CustomThreeFingerTap       = 0
        ThreeFingerUp              = 0
        ThreeFingerRight           = 0
        ThreeFingerDown            = 0
        ThreeFingerLeft            = 0
    
        FourFingerTapEnabled       = 0
        CustomFourFingerTap        = 0
        FourFingerUp               = 0
        FourFingerRight            = 0
        FourFingerLeft             = 0
        FourFingerDown             = 0
    }
    "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced" = @{
        MultiTaskingAltTabFilter = 3 # Alt+Tab viser kun åpne vinduer, aldri faner
    }
}
```  

```powershell
foreach ($Path in $Registry.Keys) {
    if (-not (Test-Path $Path)) {
        New-Item -Path $Path -Force | Out-Null
    }

    foreach ($Property in $Registry[$Path].GetEnumerator()) {
        try {
            $PropertyType = if ($Property.Value -is [string]) {
                "String"
            }
            else {
                "DWord"
            }

            New-ItemProperty `
                -Path $Path `
                -Name $Property.Key `
                -Value $Property.Value `
                -PropertyType $PropertyType `
                -Force `
                -ErrorAction Stop |
                Out-Null

            Write-Host `
                "[OK] $Path\$($Property.Key) = $($Property.Value) [$PropertyType]" `
                -ForegroundColor Green
        }
        catch {
            Write-Host `
                "[FAILED] $Path\$($Property.Key): $($_.Exception.Message)" `
                -ForegroundColor Red
        }
    }
}
```


```powershell  
# Funksjoner som har egne komandoer eller av andre grunne er bedre å endre med skript


# Kontrollpanel → Lyd → Lyder → Ingen lyder → OK
Set-Item `
    -Path "HKCU:\AppEvents\Schemes" `
    -Value ".None"

```  
# Kan ikke automatiseres eller det er ikke verdt det
# Innstillinger -> skjerm -> lysstyrke og farge
git config --global user.name ""
git config --global user.email ""
**Gaming**

```powershell
runas /user:.\Gaming pwsh
```

```powershell  
$Apps = @(
    "Valve.Steam",
    "Blizzard.BattleNet",
    "Spotify.Spotify", # er brukerspesifikk og installeres i locale, må kanskje derfor ligge i alle
    "WhatsApp.WhatsApp," # er brukerspesifikk og installeres i locale, må kanskje derfor ligge i alle
    "Discord.Discord" # er brukerspesifikk og installeres i locale, må kanskje derfor ligge i alle
)
```
```powershell  
foreach ($App in @($Apps)) {

    Write-Host "`n[INSTALLING] $App" -ForegroundColor Cyan

    winget install `
        --id $App `
        --exact `
        --scope user `
        --silent `
        --disable-interactivity `
        --accept-package-agreements `
        --accept-source-agreements

    if ($LASTEXITCODE -eq 0) {
        Write-Host "[OK] Installert: $App" -ForegroundColor Green
    }
    else {
        Write-Host "[FAILED] $App" -ForegroundColor Red
    }
}
```
```powershell  


$Registry = @{
    "HKCU:\Control Panel\Mouse" = @{
        MouseSensitivity = "13" # Ca. 65 til 70 % av Windows-slideren
        MouseSpeed       = "0"  # Museakselerasjon av
        MouseThreshold1  = "0"  # Akselerasjonsterskel av
        MouseThreshold2  = "0"  # Akselerasjonsterskel av
    }

    "HKCU:\Control Panel\Accessibility\StickyKeys" = @{
        Flags = "26" # Sticky Keys og tilhørende hurtigtast, ikon, låsing og lyd av
    }

    "HKCU:\Control Panel\Accessibility\Keyboard Response" = @{
        AutoRepeatDelay       = "0"
        AutoRepeatRate        = "0"
        BounceTime            = "0"
        DelayBeforeAcceptance = "0"
        Flags                 = "26" # Filter Keys og hurtigtasten av
    }
    "HKCU:\Software\Microsoft\Windows\CurrentVersion\PushNotifications" = @{
        ToastEnabled           = 0 # Deaktiver varsler
        LockScreenToastEnabled = 0 # Ingen varsler på låseskjermen
    }
    "HKCU:\Software\Microsoft\Windows\CurrentVersion\SearchSettings" = @{
        IsGlobalWebSearchProviderToggleEnabled = 0 # Deaktiver webresultater fra søkeapper
    }
    "HKCU:\Software\Microsoft\Windows\CurrentVersion\PrecisionTouchPad" = @{
        LeaveOnWithMouse           = [uint32]::MaxValue
        PanEnabled                 = [uint32]::MaxValue
        RightClickZoneEnabled      = [uint32]::MaxValue
        TapAndDrag                 = [uint32]::MaxValue
        TapsEnabled                = 0
        TwoFingerTapEnabled       = 0
        ZoomEnabled                = [uint32]::MaxValue
    
        FeedbackIntensity          = 50
        ClickForceSensitivity      = 50
        FeedbackEnabled            = [uint32]::MaxValue
        HonorMouseAccelSetting     = 0
    
        RightClickZoneWidth        = 0
        RightClickZoneHeight       = 0
        EdgeAutoPanningEnabled     = 1
        BoostedPanningEnabled      = 1
        PanSensitivity             = 50
        ZoomSensitivity            = 50
        SingleFingerPanningMode    = 2
        PressureAutoPanningEnabled = 1
    
        ThreeFingerSlideEnabled    = 65535
        FourFingerSlideEnabled     = 65535
    
        ThreeFingerTapEnabled      = 0
        CustomThreeFingerTap       = 0
        ThreeFingerUp              = 0
        ThreeFingerRight           = 0
        ThreeFingerDown            = 0
        ThreeFingerLeft            = 0
    
        FourFingerTapEnabled       = 0
        CustomFourFingerTap        = 0
        FourFingerUp               = 0
        FourFingerRight            = 0
        FourFingerLeft             = 0
        FourFingerDown             = 0
    }
    "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced" = @{
        MultiTaskingAltTabFilter = 3 # Alt+Tab viser kun åpne vinduer, aldri faner
    }


}
```  

```powershell
foreach ($Path in $Registry.Keys) {
    if (-not (Test-Path $Path)) {
        New-Item -Path $Path -Force | Out-Null
    }

    foreach ($Property in $Registry[$Path].GetEnumerator()) {
        try {
            $PropertyType = if ($Property.Value -is [string]) {
                "String"
            }
            else {
                "DWord"
            }

            New-ItemProperty `
                -Path $Path `
                -Name $Property.Key `
                -Value $Property.Value `
                -PropertyType $PropertyType `
                -Force `
                -ErrorAction Stop |
                Out-Null

            Write-Host `
                "[OK] $Path\$($Property.Key) = $($Property.Value) [$PropertyType]" `
                -ForegroundColor Green
        }
        catch {
            Write-Host `
                "[FAILED] $Path\$($Property.Key): $($_.Exception.Message)" `
                -ForegroundColor Red
        }
    }
}
```


```powershell  
# Funksjoner som har egne komandoer eller av andre grunne er bedre å endre med skript


# Kontrollpanel → Lyd → Lyder → Ingen lyder → OK
Set-Item `
    -Path "HKCU:\AppEvents\Schemes" `
    -Value ".None"

```  
# Kan ikke automatiseres eller det er ikke verdt det
# Innstillinger -> skjerm -> lysstyrke og farge
git config --global user.name ""
git config --global user.email ""

**Student**

```powershell
runas /user:.\Student pwsh
```

```powershell  
$Apps = @(
    "Microsoft.Office",
    "Microsoft.OneDrive",
    "Microsoft.Teams",
    "Zoom.Zoom",
    "SlackTechnologies.Slack",
    "Zotero.Zotero",
    "Spotify.Spotify", # er brukerspesifikk og installeres i locale, må kanskje derfor ligge i alle
    "WhatsApp.WhatsApp", # er brukerspesifikk og installeres i locale, må kanskje derfor ligge i alle
    "Discord.Discord" # er brukerspesifikk og installeres i locale, må kanskje derfor ligge i alle
)
```
```powershell  
foreach ($App in @($Apps)) {

    Write-Host "`n[INSTALLING] $App" -ForegroundColor Cyan

    winget install `
        --id $App `
        --exact `
        --scope user `
        --silent `
        --disable-interactivity `
        --accept-package-agreements `
        --accept-source-agreements

    if ($LASTEXITCODE -eq 0) {
        Write-Host "[OK] Installert: $App" -ForegroundColor Green
    }
    else {
        Write-Host "[FAILED] $App" -ForegroundColor Red
    }
}
```
```powershell  


$Registry = @{
    "HKCU:\Control Panel\Mouse" = @{
        MouseSensitivity = "13" # Ca. 65 til 70 % av Windows-slideren
        MouseSpeed       = "0"  # Museakselerasjon av
        MouseThreshold1  = "0"  # Akselerasjonsterskel av
        MouseThreshold2  = "0"  # Akselerasjonsterskel av
    }

    "HKCU:\Control Panel\Accessibility\StickyKeys" = @{
        Flags = "26" # Sticky Keys og tilhørende hurtigtast, ikon, låsing og lyd av
    }

    "HKCU:\Control Panel\Accessibility\Keyboard Response" = @{
        AutoRepeatDelay       = "0"
        AutoRepeatRate        = "0"
        BounceTime            = "0"
        DelayBeforeAcceptance = "0"
        Flags                 = "26" # Filter Keys og hurtigtasten av
    }
    "HKCU:\Software\Microsoft\Windows\CurrentVersion\PushNotifications" = @{
        ToastEnabled           = 0 # Deaktiver varsler
        LockScreenToastEnabled = 0 # Ingen varsler på låseskjermen
    }
    "HKCU:\Software\Microsoft\Windows\CurrentVersion\SearchSettings" = @{
        IsGlobalWebSearchProviderToggleEnabled = 0 # Deaktiver webresultater fra søkeapper
    }
    "HKCU:\Software\Microsoft\Windows\CurrentVersion\PrecisionTouchPad" = @{
        LeaveOnWithMouse           = [uint32]::MaxValue
        PanEnabled                 = [uint32]::MaxValue
        RightClickZoneEnabled      = [uint32]::MaxValue
        TapAndDrag                 = [uint32]::MaxValue
        TapsEnabled                = 0
        TwoFingerTapEnabled       = 0
        ZoomEnabled                = [uint32]::MaxValue
    
        FeedbackIntensity          = 50
        ClickForceSensitivity      = 50
        FeedbackEnabled            = [uint32]::MaxValue
        HonorMouseAccelSetting     = 0
    
        RightClickZoneWidth        = 0
        RightClickZoneHeight       = 0
        EdgeAutoPanningEnabled     = 1
        BoostedPanningEnabled      = 1
        PanSensitivity             = 50
        ZoomSensitivity            = 50
        SingleFingerPanningMode    = 2
        PressureAutoPanningEnabled = 1
    
        ThreeFingerSlideEnabled    = 65535
        FourFingerSlideEnabled     = 65535
    
        ThreeFingerTapEnabled      = 0
        CustomThreeFingerTap       = 0
        ThreeFingerUp              = 0
        ThreeFingerRight           = 0
        ThreeFingerDown            = 0
        ThreeFingerLeft            = 0
    
        FourFingerTapEnabled       = 0
        CustomFourFingerTap        = 0
        FourFingerUp               = 0
        FourFingerRight            = 0
        FourFingerLeft             = 0
        FourFingerDown             = 0
    }
    "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced" = @{
        MultiTaskingAltTabFilter = 3 # Alt+Tab viser kun åpne vinduer, aldri faner
    }


}
```  

```powershell
foreach ($Path in $Registry.Keys) {
    if (-not (Test-Path $Path)) {
        New-Item -Path $Path -Force | Out-Null
    }

    foreach ($Property in $Registry[$Path].GetEnumerator()) {
        try {
            $PropertyType = if ($Property.Value -is [string]) {
                "String"
            }
            else {
                "DWord"
            }

            New-ItemProperty `
                -Path $Path `
                -Name $Property.Key `
                -Value $Property.Value `
                -PropertyType $PropertyType `
                -Force `
                -ErrorAction Stop |
                Out-Null

            Write-Host `
                "[OK] $Path\$($Property.Key) = $($Property.Value) [$PropertyType]" `
                -ForegroundColor Green
        }
        catch {
            Write-Host `
                "[FAILED] $Path\$($Property.Key): $($_.Exception.Message)" `
                -ForegroundColor Red
        }
    }
}
```


```powershell  
# Funksjoner som har egne komandoer eller av andre grunne er bedre å endre med skript


# Kontrollpanel → Lyd → Lyder → Ingen lyder → OK
Set-Item `
    -Path "HKCU:\AppEvents\Schemes" `
    -Value ".None"

```  
# Kan ikke automatiseres eller det er ikke verdt det
# Innstillinger -> skjerm -> lysstyrke og farge
git config --global user.name ""
git config --global user.email ""

**Frivillig**

```powershell
runas /user:.\Frivillig pwsh
```

```powershell  
$Apps = @(
    "Microsoft.Office",
    "Microsoft.Teams",
    "Zoom.Zoom",
    "Spotify.Spotify", # er brukerspesifikk og installeres i locale, må kanskje derfor ligge i alle
    "WhatsApp.WhatsApp", # er brukerspesifikk og installeres i locale, må kanskje derfor ligge i alle
    "Discord.Discord" # er brukerspesifikk og installeres i locale, må kanskje derfor ligge i alle
)
```
```powershell  
foreach ($App in @($Apps)) {

    Write-Host "`n[INSTALLING] $App" -ForegroundColor Cyan

    winget install `
        --id $App `
        --exact `
        --scope user `
        --silent `
        --disable-interactivity `
        --accept-package-agreements `
        --accept-source-agreements

    if ($LASTEXITCODE -eq 0) {
        Write-Host "[OK] Installert: $App" -ForegroundColor Green
    }
    else {
        Write-Host "[FAILED] $App" -ForegroundColor Red
    }
}
```
```powershell  


$Registry = @{
    "HKCU:\Control Panel\Mouse" = @{
        MouseSensitivity = "13" # Ca. 65 til 70 % av Windows-slideren
        MouseSpeed       = "0"  # Museakselerasjon av
        MouseThreshold1  = "0"  # Akselerasjonsterskel av
        MouseThreshold2  = "0"  # Akselerasjonsterskel av
    }

    "HKCU:\Control Panel\Accessibility\StickyKeys" = @{
        Flags = "26" # Sticky Keys og tilhørende hurtigtast, ikon, låsing og lyd av
    }

    "HKCU:\Control Panel\Accessibility\Keyboard Response" = @{
        AutoRepeatDelay       = "0"
        AutoRepeatRate        = "0"
        BounceTime            = "0"
        DelayBeforeAcceptance = "0"
        Flags                 = "26" # Filter Keys og hurtigtasten av
    }
    "HKCU:\Software\Microsoft\Windows\CurrentVersion\PushNotifications" = @{
        ToastEnabled           = 0 # Deaktiver varsler
        LockScreenToastEnabled = 0 # Ingen varsler på låseskjermen
    }
    "HKCU:\Software\Microsoft\Windows\CurrentVersion\SearchSettings" = @{
        IsGlobalWebSearchProviderToggleEnabled = 0 # Deaktiver webresultater fra søkeapper
    }
    "HKCU:\Software\Microsoft\Windows\CurrentVersion\PrecisionTouchPad" = @{
        LeaveOnWithMouse           = [uint32]::MaxValue
        PanEnabled                 = [uint32]::MaxValue
        RightClickZoneEnabled      = [uint32]::MaxValue
        TapAndDrag                 = [uint32]::MaxValue
        TapsEnabled                = 0
        TwoFingerTapEnabled       = 0
        ZoomEnabled                = [uint32]::MaxValue
    
        FeedbackIntensity          = 50
        ClickForceSensitivity      = 50
        FeedbackEnabled            = [uint32]::MaxValue
        HonorMouseAccelSetting     = 0
    
        RightClickZoneWidth        = 0
        RightClickZoneHeight       = 0
        EdgeAutoPanningEnabled     = 1
        BoostedPanningEnabled      = 1
        PanSensitivity             = 50
        ZoomSensitivity            = 50
        SingleFingerPanningMode    = 2
        PressureAutoPanningEnabled = 1
    
        ThreeFingerSlideEnabled    = 65535
        FourFingerSlideEnabled     = 65535
    
        ThreeFingerTapEnabled      = 0
        CustomThreeFingerTap       = 0
        ThreeFingerUp              = 0
        ThreeFingerRight           = 0
        ThreeFingerDown            = 0
        ThreeFingerLeft            = 0
    
        FourFingerTapEnabled       = 0
        CustomFourFingerTap        = 0
        FourFingerUp               = 0
        FourFingerRight            = 0
        FourFingerLeft             = 0
        FourFingerDown             = 0
    }
    "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced" = @{
        MultiTaskingAltTabFilter = 3 # Alt+Tab viser kun åpne vinduer, aldri faner
    }


}
```  

```powershell
foreach ($Path in $Registry.Keys) {
    if (-not (Test-Path $Path)) {
        New-Item -Path $Path -Force | Out-Null
    }

    foreach ($Property in $Registry[$Path].GetEnumerator()) {
        try {
            $PropertyType = if ($Property.Value -is [string]) {
                "String"
            }
            else {
                "DWord"
            }

            New-ItemProperty `
                -Path $Path `
                -Name $Property.Key `
                -Value $Property.Value `
                -PropertyType $PropertyType `
                -Force `
                -ErrorAction Stop |
                Out-Null

            Write-Host `
                "[OK] $Path\$($Property.Key) = $($Property.Value) [$PropertyType]" `
                -ForegroundColor Green
        }
        catch {
            Write-Host `
                "[FAILED] $Path\$($Property.Key): $($_.Exception.Message)" `
                -ForegroundColor Red
        }
    }
}
```


```powershell  
# Funksjoner som har egne komandoer eller av andre grunne er bedre å endre med skript


# Kontrollpanel → Lyd → Lyder → Ingen lyder → OK
Set-Item `
    -Path "HKCU:\AppEvents\Schemes" `
    -Value ".None"

```  
# Kan ikke automatiseres eller det er ikke verdt det
# Innstillinger -> skjerm -> lysstyrke og farge
git config --global user.name ""
git config --global user.email ""


Installer manuelt:
**Dev**
Fjern onedrive hvis det finnes.
- Logg inn med riktig mail+name@gmail.com
Power Management → konservativ, batteritid

**Gaming**
- sc2
- Logg inn med riktig mail+name@gmail.com
```
Power Management → Prefer maximum performance
Low Latency Mode → Off / On / Ultra (benchmark all three)
V-Sync → Off
Maximum Frame Rate → Off (unless needed) -> (lapptop only have 60 fps)
G-SYNC configuration if your display supports it
```
Refreshrate check overclock monitor !!

**Student**
- Logg inn med riktig mail+name@gmail.com

**Frivillig**
- Logg inn med riktig mail+name@gmail.com


# After setting up dualboot
enable bitlocker 
Enhanced indexing off - of by deafult 
Fix bios settings

# Todo
## Konfigurering
- TeX Live (<https://www.tug.org/texlive/>)
- Sett opp mobil deling, (<https://support.microsoft.com/windows/apps/phonelink/phone-link-requirements-and-setup>)

- Finn ut hvordan man omvender fn tasten for å få rask tilgang til f tastene.(og sc2) - Bios
- Kjør (Win + R) → sysdm.cpl → Avansert → Ytelse → Innstillinger → Juster for best ytelse(fjern: animasjoner, skygge og fade effects) → Bruk & OK
- Win + R -> control input.dll -> Advanced Key Settings -> Change Key Sequence -> Sett alt til Not Assigned hvis du vil deaktivere hurtigtaster for språkbytte.
- Ctrl + Shift + Esc → Oppgavebehandling → Oppstart → Høyreklikk app → Deaktiver
- Sett pwsh 7 som default i terminal -> Åpne Windows Terminal → Klikk pilen ved siden av ny fane-knappen → Settings → Startup → Default profile → PowerShell
