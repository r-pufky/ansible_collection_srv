Debian 12
#########
From docker build https://gist.github.com/OnniSaarni/a3f840cef63335212ae085a3c6c10d5c

Requirements:
RAM: ~1GB
DISK: ~500MB (~8GB during build process)
CPU: ~1 core

#. Add packages, users, directories

   apt update && apt dist-upgrade
   apt install sudo vim ssh git git-lfs
   adduser {USER} sudo
   adduser eft

   sudo mkdir /opt/eft
   sudo chown eft:eft /opt/eft
   sudo su - eft

   export SPT_TAG=3.9.8
   export FIKA_TAG=v2.2.8
   export NODE_VERSION=20.11.1

#. Setup NPM/NVM in current environment

   git clone https://github.com/nvm-sh/nvm ~/.nvm
   . ~/.nvm/nvm.sh
   source ~/.bashrc
   nvm install $NODE_VERSION

#. Build server from source

   # Large repository causes clone issues occasionally; only grab latest files
   git clone --depth 1 --branch $SPT_TAG https://dev.sp-tarkov.com/SPT/Server ~/srv
   cd ~/srv/project
   # SPT=use head from current branch
   git checkout HEAD^
   git lfs fetch --all
   git lfs pull
   # Remove AKI encoding
   sed -i '/setEncoding/d' ~/srv/project/src/Program.ts
   npm install
   # ignore modFilePath warnings, these do not exist during build.
   npm run build:release -- --arch=x64 --platform=linux
   mv build/* /opt/eft/
   rm -rfv ~/srv

#. Build FIKA server mod

   git clone --branch $FIKA_TAG https://github.com/project-fika/Fika-Server /opt/eft/user/mods/fika-server
   cd /opt/eft/user/mods/fika-server
   git checkout HEAD^
   npm install

#. Generate initial configuratiion files and bind to all interfaces

   cd /opt/eft
   # Generate initial configs and kill after 25 seconds.
   nohup timeout --preserve-status 25s ./SPT.Server.exe >/dev/null 2>&1
   # Wait 30 seconds to shutdown
   sed -i 's/127.0.0.1/0.0.0.0/g' /opt/eft/SPT_Data/Server/configs/http.json

#. Set FIKA server settings to preferred options

   `/opt/eft/user/mods/fika-server/assets/configs/fika.jsonc`
   ```json
   {
     "client": {
       "useBtr": true,
       "friendlyFire": true,
       "dynamicVExfils": false,
       "allowFreeCam": false,
       "allowSpectateFreeCam": true,
       "allowItemSending": true,
       "blacklistedItems": [],
       "forceSaveOnDeath": false,
       "mods": {
         "required": [],
         "optional": []
       },
       "useInertia": true,
       "sharedQuestProgression": true
     },
     "server": {
       "giftedItemsLoseFIR": false,
       "launcherListAllProfiles": false,
       "sessionTimeout": 5,
       "showDevProfile": false,
       "showNonStandardProfile": false
     },
     "natPunchServer": {
       "enable": false,
       "port": 6790,
       "natIntroduceAmount": 1
     }
     "dedicated": {
       "profiles": {
         "amount": 0
       },
       "scripts": {
         "generate": true,
         "forceIp": ""
       }
     },
     "background": {
       "enable": true,
       "easteregg": false
     }
   }
   ```

#. Enable Open flea market for all items
   https://old.reddit.com/r/SPTarkov/comments/sibdoz/how_to_enable_all_items_on_the_flea/

   Allows traditional flea market usage and quest progression.

   sed -i 's/CanRequireOnRagfair\":\ false/CanRequireOnRagfair\":\ true/g' /opt/eft/SPT_Data/Server/database/templates/items.json
   sed -i 's/CanSellOnRagfair\":\ false/CanSellOnRagfair\":\ true/g' /opt/eft/SPT_Data/Server/database/templates/items.json

   `/opt/eft/SPT_Data/Server/database/templates/items.json`

#. Migrate from SIT to FIKA (server profiles)
   https://old.reddit.com/r/SPTarkov/comments/1chnvrs/guide_how_to_port_your_spt_profile_to_fika_or_sit/

   * Copy `/opt/eft/user/profiles` from SIT to FIKA (same location)
   * Remove `password` line in each profile, under `info` section

#. Create and start systemd service

   `/etc/systemd/system/eft.service`
   ```bash
   [Unit]
   Description=Escape from Tarkov (Coop) SPT/FIKA Server.

   [Service]-
   Type=exec
   WorkingDirectory=/opt/eft
   ExecStart=/opt/eft/SPT.Server.exe
   Restart=on-failure
   User=eft
   Group=eft

   [Install]
   WantedBy=default.target
   ```

   systemctl daemon-reload
   systemctl enable eft
   systemctl start eft

Upgrading
#########
Just repeat the process above to rebuild from head and update server binaries
with the new versions.

#. systemctl stop eft

#. mv /opt/eft /opt/{OLD SPT}-eft

#. Build process above

#. cp -av /opt/{OLD SPT}-eft/user/profiles/* /opt/eft/user/profiles/

#. systemctl start eft

Profile Conversions
###################
User profiles may need to be converted. Always double check profiles load
correctly after upgrades.

## 3.8.3 to 3.9.0
  * https://github.com/MadBender/spt-profile-converter
  * https://github.com/ArchangelWTF/SPT.ProfileConverter

Dedicated Server Ports
######################
https://old.reddit.com/r/SPTarkov/comments/1ckuwmv/project_fika_on_a_dedicated_machine/

#. 'Dedicated server' syncs state, profiles, and enforces settings.
   This is the linux server.

   Ports: `TCP 6969 in/out`

#. 'Host server' actually hosts the game using game assets.
   This is the client with the most powerful machine.

   Ports: `UDP 25565 in/out`

#. FIKA will force clients to locally process hit registration, culling, etc.
   If you are running a 'dedicated server' separately from the 'host server'
   you will need to forward the following ports to your machine. The dedicated
   server will redirect clients automatically (clients specify the dedicated
   server to connect to):
