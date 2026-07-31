Finn iso installer uten bloat fra windows på lenken:

Lag to brukere:
Dev og student.

Chrome -> Logg in på alt -> gå til alle sider som har cookie spørsmål.

Finn ut hvordan man omvender fn tasten for å få rask tilgang til f tastene.(og sc2)
Ingen cuda i Winblows, har det i NIX

Nvidia App

Instillinger -> skjerm -> lysstyrke og farge
Innstillinger -> System -> Strøm og dvalemodus -> Avanserte
Fjern bloatware og unødvendige apper
Kontrollpanel → Lyd → Lyder → Ingen lyder → OK
Kjør (Win + R) → sysdm.cpl → Avansert → Ytelse → Innstillinger → Juster for best ytelse(fjern: animasjoner, skygge og fade effects) → Bruk & OK
Dessverre er innstillingen for bakgrunds-kjøring inne på hver app i Windows 11, så du må gå inn i hver relevant app og deaktivere.
Ctrl + Shift + Esc → Oppgavebehandling → Oppstart → Høyreklikk app → Deaktiver
Innstillinger → System → Varslinger → Skru av varsler
Innstillinger → Tilpasning → Farger → Gjennomsiktighet av
Installer: Java → LaTeX → VS Code → Python → Git → Spotify → Discord → Steam → Battle.net → sc2 -> Zoom → Teams → Slack → OBS → VLC → Blender → GIMP → Audacity -> MSI Center
Instillinger -> Personvern og instillinger -> Søketilatelser -> La søkeapper vise resultater -> Av
evt innstillingsknapp i oppgavemenyen
evt slå på bedre indeksering
slå av pekertapping på touchpad
Fjerne faner fra alt-tab Instillinger -> System -> Fleroppgavevisning -> Skru av "vis faner fra apper når du trykker alt + tab"
Fra oppstartsmenyen ctrl + shift + esc deaktivere Xbox, Steam, OneDrive, Microsoft 365 Copilot, Microsoft Teams, MSI Center, Killer Intelligence Center, Terminal og eventuelt Telefonkobling.






EVT:
https://www.android.com/better-together/nearby-share-app/ istedenfor Microsoft Your Phone
Google meldinger på telefon og nettside meldinger.google.com.
Instillinger - > Personalisering -> låseskjerm -> bakgrunnsbilde -> velg bilde # Gjør at windwos slutter å laste end masse bilder for så å slette dem.

Kjør som admin i powershell:
$BloatwareApps = @(
    "*Xbox.TCUI*", "*XboxSpeechToTextOverlay*", "*XboxGamingOverlay*", "*Microsoft.GamingApp*",
    "*BingWeather*", "*BingNews*", "*BingSearch*",
    "*Microsoft.WindowsAlarms*", "*Microsoft.MicrosoftStickyNotes*",
    "*Microsoft.Copilot*",
    "*Microsoft.YourPhone*", "*MicrosoftWindows.CrossDevice*"
)
foreach ($App in $BloatwareApps) { Get-AppxPackage $App | Remove-AppxPackage }

taskschd.msc -> Microsoft -> XblGameSave -> Disable #XBOX åpner seg hver gang pcen blir inaktiv for å sjekke om den skal lagre spill og er hardkodet til os'et.

deaktiver filtertaster hotkey
deaktiver trege taster hotkey
slå av pekerpresisjon

Instillinger -> System -> Fleroppgavekjøring -> ikke vis faner når en trykker alt tab.

Installer -> power toys -> og fjern popup spam knapper som ikke er ønskelig. Coopilot knappen som ertatter ctrl knappen.
Disable Widgets
Disable Copilot if you don't use it.
Disable Game DVR
- Settings
- Gaming
- Captures

enable system restore

enable bitlocker 

Tailscale

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
Maximum Frame Rate → Off (unless needed)
G-SYNC configuration if your display supports it

Student bruker
Office 365
