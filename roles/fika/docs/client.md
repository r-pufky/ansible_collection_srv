FIKA -- Tarkov COOP
###################################
https://github.com/project-fika/Fika-Documentation

Requirements:
CPU: i7 8700k / Ryzen 7 2700x
GPU: GTX 1060 / RX 580
Memory: 16 GB minimum, 32 GB highly recommended
Storage: ~42GB SSD

Install
#######
https://github.com/project-fika/Fika-Documentation?tab=readme-ov-file#installation

Use the automated SPT installer and install to a new coop location.
Settings will be copied over.

## Upgrading SPT Tarkov
Upgrades require a complete reinstall. Save your current user settings, then
delete the existing install.

#. Backup c:/tarkov-coop-fika/user to a separate location

#. Remove *ALL* files in c:/tarkov-coop-fika

#. Continue to `Base Tarkov Install`

#. Remember to copy `user` back to the updated installation!

## Base Tarkov Install
Only if Tarkov is not already installed on the machine (or updating to latest
release).

#. Install Escape from Tarkov using the included BSG Launcher:

   `BsgLauncher.*.exe`

#. Run the game once with latest patches. Login to your account character and
   exit the game.

#. Disable auto-updates:

   user -> launcher settings -> updates -> disable automatic game updates

## Install SPT (Single Player Tarkov)
https://sp-tarkov.com/#download

#. Create FIKA Directory structure OUTSIDE of tarkov folder

   c:/tarkov-coop-fika

#. Copy `SPTInstaller.exe` to `c:/tarkov-coop-fika` and run it

   * Run through all the prechecks and install missing dependencies
   * Click the 'bug' to refresh the checks; all must be green
   * If you get a 'patcher not found' error; then the installer doesn't have a
     current downgrade patch for the release to get EFT to a supported EFT
     version by SPT. Try updating the game (SPT usually releases downgrade
     patches within a day of a EFT release).

   This will take a few minutes.

#. Install FIKA Mod
   https://github.com/project-fika/Fika-Plugin/releases/latest

   * Extract inlcuded `Fika.Release.*.zip` to `c:/tarkov-coop-fika`
   * `Fika.Core.dll` should be in `c:/tarkov-coop-fika/BepInEx/plugins`

## Migrate config and set mod manager menu

#. Switch mod manager hotkey to `F11`
   F12 conflicts with steam overlay settings.

   `c:/tarkov-coop-fika/BepInEx/config/com.bepis.bepinex.configurationmanager.cfg`
   ```json
     Show config manager = F11 
     Show config manager = F11
   ```

   There are TWO options to set.

#. Copy existing tarkov settings to SPT

   `user` -> `c:/tarkov-coop-fika/user`

#. Setup steam shorcut

   * steam -> games -> add non-steam game to library
   * Target: `c:/tarkov-coop-fika/SPT.Launcher.exe`
   * Icon (click it): select `EscapeFromTarkov.exe`
   * Name: 'Escape from Tarkov (Coop)'
   * Start In: `c:/tarkov-coop-fika`
   * Enable steam overlay while in game
   * controller -> disable steam input
   * controller -> use desktop configruation in launcher

#. Launch 'Escape from Tarkov (Coop)'
   Ignore 'copy game settings' dialogs and any errors on inital launch (these
   are because the default server is not running).

   * settings -> Developer Mode -> Enable
   * settings -> URL-> `http://{IP}:6969`
   * settings -> On Game Start -> Close Launcher
   * settings -> SPT Game Path -> `c:/tarkov-coop-fika`
   * click `->` to launch

   Your previous account (if migrated) will be available with no password.
   
   On first run, YOU MUST WAIT 30 SECONDS FOR THE TOS ACCEPTANCE OR GAME WILL
   EXIT.

#. Set FIKA Options
   Press `F11` During game to pull up mod overlay; click on `FIKA Core` to
   expand mod options. Note: The UI is confusing. The verbage next to the
   checkbox represents the current state, *NOT* what ticking it does.
  
   * Show Feed: Enabled (checked)
   * Auto Extract: Enabled (checked)
   * Show Extract Message: Enabled (checked)
   * Extract Key: F8
   * Ping System: Enabled (checked)
   * Ping Button: {SET}
   * Free Camera Button: F9
   * Show Player Name Plates: Enabled (checked)
   * Show Player Faction Icon: Disabled (unchecked)
   * Hide Name Plate In Optic: Enabled (checked)
   * Dynamic AI: Enabled (checked)
   * Dynamic AI Range: 150.0m (double check if sniping and AI not moving)
   * Culling System: Enabled (checked)
   * Culling Range: 30m
   * Enforced Spawn Limits: Disabled (unchecked)

#. You are ready to play. 

Creating Host Server
####################
#. 'Dedicated server' syncs state, profiles, and enforces settings.
   This is the linux server. See debian-12-server-build.md.

   Ports: `TCP 6969 in/out`

#. 'Host server' actually hosts the game using game assets.
   This is the client with the most powerful machine.

   Ports: `UDP 25565 in/out`

#. FIKA will force clients to locally process hit registration, culling, etc.
   If you are running a 'dedicated server' separately from the 'host server'
   you will need to forward the following ports to your machine. The dedicated
   server will redirect clients automatically (clients specify the dedicated
   server to connect to):

#. Starting a Game
   Certain game options will be enforce server side.

   * pick raid map to play
   * pick the same SLOT number (1 or 2)
   * player with the strongest machine should HOST the raid
   * `COOP` mode and `Practice Mode` will be force enabled with FIKA
   * after insurance click `host raid`
     * Pick number of players; game will NOT start until everyone has joined
   * start

Moving Install Location
#######################
Registry keys must be updated to point to new binary locations.

#. Find EFT Uninstall registry keys (regedit)

   Computer\HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\{B0FDA062-7581-4D67-B085-C4E7C358037F}_is1

#. Upload binary locations for each key (do not change additional options)

   * DisplayIcon
   * Inno Setup: App Path
   * InstallLocation
   * QuietUninstallString
   * UninstallString

#. Update BsgLauncher Paths

   `%appdata%/Roaming/Battlestate Games/BsgLauncher/settings`

Upgrading SPT Minor Releases
############################
Minor releases only require in-place replace of existing SPT files.

#. Download the specified version from `versions.md`

#. Backup replaced files (prepend with date `YYYY-MM-DD.{FILE}`)

   * BepInEx
   * SPT_Data
   * SPT.Server.exe
   * SPT.Launcher.exe
   * doorstop_config.ini
   * winhttp.dll
   
#. Extract archive into `c:/tarkov-coop-fika`

#. Update plugin configuration files

   `c:/tarkov-coop-fika/BepInEx/config`