# Full Mac Setup Process (macOS Ventura 13.x)

This document describes the full process of setting up a fresh macOS installation per my particular preferences and requirements. It is likely overkill or divergent from your specific use case, but hopefully will serve as a helpful reference.

## macOS setup
### macOS
First and foremost, macOS needs to be installed. Then, complete Apple's mandatory macOS setup wizard, creating a local user account

> @TODO: Document macOS setup wizard steps.

### iCloud and App Store
Sign in to iCloud and the App Store.

#### Software Update
Install any available macOS updates.

### Xcode
Install Xcode command line tools.

```bash
xcode-select --install
```

Agree To Xcode.

```bash
sudo xcrun cc
```

#### Create directories for Projects and Sites
Create directories for Projects and Sites, and set permissions to prevent accidental deletion. These directories will be used later in the process and are not created by default. They generally will contain code and websites, respectively.

```bash
mkdir ~/Projects 
chmod +a "group:everyone deny delete" ~/Projects
chmod u+rwx,go-rwx ~/Projects
```
```bash
mkdir ~/Sites 
chmod +a "group:everyone deny delete" ~/Sites
chmod u+rwx,go-rwx ~/Sites
```
> Optionally, copy contents of `~/Projects` and/or `~/Sites` folder(s) from another Mac (to save time).

### $PATH

```bash
export PATH="$HOME/Library/Python/3.9/bin:/opt/homebrew/bin:$PATH"
```

### Python and Ansible
Ensure Python 3 is installed, and upgrade `pip` and `setuptools` packages.
```bash
pip3 install --upgrade setuptools
pip3 install --upgrade pip
pip3 install --upgrade virtualenv
pip3 install --upgrade ipython
```
- Install Ansible with `pip` for the current user.
```bash
pip3 install --upgrade ansible
```

> _If desired, Ansible and Python 3 can be (re)installed by Homebrew when the main playbook is run, and thus managed by `brew` going forward._

Add Python 3 to `PATH` environment variable.

> Note that the locations of Python 3 binaries and libraries may be different depending on the version of Python installed. The following example assumes Python 3.9. The actual path can be determined by running `which python3` and `python3 -m site --user-base` in the terminal and adjusting the below accordingly.

```bash
export PATH="$HOME/Library/Python/3.9/bin:$PATH"
```

### Mac Development Ansible Playbook
Clone or download (this) [mac-dev-playbook](https://github.com/greylabel/mac-dev-playbook) git repo to a temporary location, or wherever you prefer to store source code checkouts, e.g. `~/Projects`.

Clone the repo with Git.
```bash
git clone https://github.com/greylabel/mac-dev-playbook.git
```

Download the repo as a zip file.

```bash
curl -LJO https://github.com/greylabel/mac-dev-playbook/archive/refs/heads/main.zip
```

```bash
unzip mac-dev-playbook-main.zip && rm mac-dev-playbook-main.zip
```

#### Additional configuration

Optionally, add a `config.yml` to the playbook if configuration overrides are required (copy over the network or using a USB flash drive).

> @TODO: Note about `ansible vault` file for sensitive data.

#### Install requirements and run playbook

>The below commands are run from the cloned `mac-dev-playbook` directory.

Install required roles.
```bash
ansible-galaxy install -r requirements.yml
```
Run main playbook with 'BECOME' password and skip post tasks.
```bash
ansible-playbook main.yml --ask-become-pass --skip-tags post
```

### Homebrew

Homebrew is installed by Ansible when the main playbook is run. For reference, the official command to install Homebrew with `curl` is below.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

> Note: If some Homebrew commands fail, you might need to agree to Xcode's license or fix some other Brew issue. Run `brew doctor` to see if this is the case.

Add Homebrew's `bin` directory to `PATH` environment variable, if needed for temporary use. This will be properly persisted when Dotfiles are installed later.

```bash
export PATH="/opt/homebrew/bin:$PATH"
```

> @TODO: Consider using [Homebrew Bundle](https://github.com/Homebrew/homebrew-bundle) to manage package list and apps with a Brewfile, and potentially move back to Dotfiles repo.

#### Activate utils installed by Homebrew
```
# Save Homebrew’s installed location.
BREW_PREFIX=$(brew --prefix)

# Don’t forget to add `$(brew --prefix coreutils)/libexec/gnubin` to `$PATH`.
ln -s "${BREW_PREFIX}/bin/gsha256sum" "${BREW_PREFIX}/bin/sha256sum"

# Switch to using brew-installed bash as default shell
if ! fgrep -q "${BREW_PREFIX}/bin/bash" /etc/shells; then
  echo "${BREW_PREFIX}/bin/bash" | sudo tee -a /etc/shells;
  chsh -s "${BREW_PREFIX}/bin/bash";
fi;
```

### Dotfiles
Dotfiles can be installed by Ansible when the main playbook is run. My [Dotfiles](https://github.com/greylabel/dotfiles/tree/main) include configuration for many of the packages installed by Homebrew, as well as an assortment of other tools and aliases.

> Note: macOS scriptable settings are stored in the Dotfiles repo, as is custom with other dotfiles setups around the web, but can be applied by Ansible when the main playbook is run.

#### Git
Create and use `~/.gitconfig.local` file for username / github token / etc.

```
[user]

  name = Grant Gaudet
  email = grant@greylabel.net

[github]

  user = greylabel
```


### SSH
#### Generating a new SSH key
> @TODO: Investigate if this can be automated with Ansible and implications on remote Mac provisioning.

> See Github's [Generating a new SSH key and adding it to the ssh-agent](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/) for more information.

Create directory for SSH keys and configuration and set permissions.
```bash
mkdir ~/.ssh
chmod u+rwx,go-rwx ~/.ssh
chmod 0700 ~/.ssh
```
Generate a new SSH key.
```bash
ssh-keygen -t ed25519 -C "grant@greylabel.net"
# chmod u+rw,go-rwx ~/.ssh/*
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

#### Adding your SSH key to the ssh-agent
```bash
eval "$(ssh-agent -s)"
```

Modify the `~/.ssh/config` file to automatically load keys into the ssh-agent and store passphrases in your keychain.
```bash
IgnoreUnknown AddKeysToAgent,UseKeychain

Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
```

Set permissions on the `~/.ssh/config` file.
```bash
chmod 644 ~/.ssh/config
# chmod u+rw,go-rwx ~/.ssh/config
```

```bash
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

At this point, the new key can be added to GitHub and other cloud services.

#### Adding a passphrase to an existing SSH key

Add/replace the passphrase for an existing private key without regenerating the keypair.

```bash
ssh-keygen -p -f ~/.ssh/id_ed25519
```

### App installation
Install applications that cannot be installed by Homebrew, through the App Store, or via another easy and Ansible-friendly scriptable process. This is done before the main playbook is run to ensure apps are in place for tasks that depend on their presence, like configuring the Dock.

#### Install Apps with Homebrew Cask or from developer websites
* [1Password](https://agilebits.com/downloads)
* [Adobe Creative Cloud](http://www.adobe.com/creativecloud/desktop-app.html)
  * Adobe Acrobat DC
  * Adobe Bride
  * Adobe InDesign
  * Adobe Lightroom Classic
  * Adobe Media Encoder
  * Adobe Photoshop
  * Adobe Premier Pro
* [Airfoil Satellite](https://rogueamoeba.com/airfoil/satellite/mac/)
* [Airfoil](https://rogueamoeba.com/airfoil/mac/)
* [Backblaze](https://www.backblaze.com/mac/install_backblaze.dmg)
* [ChromeDirver](https://chromedriver.chromium.org/downloads)
* [Cyberduck](https://cyberduck.io/download)
* [Docker](https://store.docker.com/editions/community/docker-ce-desktop-mac)
* [Doxie](https://help.getdoxie.com/doxiepro/software/download/)
* [Dropbox](https://www.dropbox.com/downloading?src=index)
* [Figma](https://www.figma.com/download/desktop/mac)
* [Firefox](http://www.mozilla.org/en-US/firefox/all/)
* [HandBrake](https://handbrake.fr/rotation.php?file=HandBrake-1.6.1.dmg)
* [Garmin Express](https://www.garmin.com/en-US/software/express)
* [Google Chrome](https://www.google.com/intl/en/chrome/browser/)
* [iTerm2](https://www.iterm2.com/)
* [JetBrains Toolbox App](https://www.jetbrains.com/toolbox/app/)
  * DataGrip
  * GoLand
  * PhpStorm
  * PyCharm Professional
  * WebStorm
* [Kaleidoscope](https://www.kaleidoscopeapp.com)
* [Keep It](http://reinventedsoftware.com/keepit/downloads/)
* [NetNewsWire](http://netnewswireapp.com/mac/)
* [Notion](https://www.notion.so/desktop)
* [Raspberry Pi Imager](https://downloads.raspberrypi.org/imager/imager_latest.dmg)
* [Rectangle](https://rectangleapp.com)
* [TablePlus](https://tableplus.com/release/osx/tableplus_latest)
* [TextMate](http://macromates.com/download)
* [TrainerRoad](https://reinventedsoftware.com/keepit/)
* [Tower](https://www.git-tower.com/mac/)
* [Vagrant](https://www.vagrantup.com/downloads.html)
* [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
* [Visual Studio Code](https://code.visualstudio.com)
* [VLC](https://www.videolan.org/vlc/download-macosx.html)
* [Wireshark](https://www.wireshark.org/download.html)

#### App Store
##### macOS
* [Mela](https://apps.apple.com/us/app/mela-recipe-manager/id1568924476?mt=12)
* [Reeder](https://apps.apple.com/us/app/reeder-4/id1449412482?mt=12)
* [Xcode](https://itunes.apple.com/us/app/xcode/id497799835?ls=1&mt=12)

##### Safari extensions
[Safari Extensions](https://safari-extensions.apple.com)
* [1Password for Safari](https://apps.apple.com/us/app/1password-for-safari/id1569813296?mt=12)
* [Instapaper Save](https://apps.apple.com/us/app/instapaper-save/id1481302432?mt=12)

### Node
#### NMV

See `.exports` and `.bash_profile`.

NVM recommends installing via their own installer script. [See installation instructions here](https://github.com/creationix/nvm).

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
```

Install node with `nvm install`, then select the locally-specified version for use with nvm use. If the node version needs to be changed, this can be done in the .nvmrc file in the root of a project's repository.

You will also need to install the latest npm. Install with `npm install -g npm@latest`.

### macOS config

### Manual macOS config

#### System Preferences
##### Apple ID
> @TODO: Full accounting of all settings.

##### Wi-Fi
- Ask to join hotspots = unchecked

##### Bluetooth

##### Network

##### Firewall
- Firewall = checked
- Firewall Options -> Block all incoming connections = unchecked
- Firewall Options -> Automatically allow built-in software to receive incoming connections = unchecked
- Firewall Options -> Automatically allow signed software to receive incoming connections = unchecked
- Firewall Options -> Enable stealth mode = unchecked

##### Notification
- Show previews -> never
- Allow notifications when screen is locked = unchecked
###### Application Notifications
> @TODO: Explore how to programmatically configure these settings.

##### Sound
- Play sound on startup = unchecked
- Play user interface sound effects = unchecked

##### Focus
- Share across devices = checked
- Focus Status -> Share Focus Status = checked

##### Screen Time
- Screen Time = unchecked

##### General
###### About
- Name = {hostname}
###### Software Update
- Install macOS updates = checked
- Install application updates from the App Store = checked
###### AirDrop & Handoff
- Allow Handoff between this Mac and your iCloud devices = unchecked
- AirDrop = No One
- AirPlay Receiver = unchecked
- Allow AirPlay for = Current User
###### Login Items
Open at Login
Allow in the Background
###### Language & Region
- Preferred languages = [English (US), Italian (Italy)]
###### Date & Time
###### Sharing
- Local hostname = {hostname}
###### Time Machine
###### Transfer or Reset
###### Startup Disk

##### Appearance
- Appearance = Auto
- Accent color = green
- Highlight color = green
- Sidebar icon size = medium
- Allow wallpaper tinting in windows = unchecked
- Show scroll bars = Automatically based on mouse or trackpad
- Click in the scroll bar to = Jump to the next page

##### Accessibility

##### Control Center

###### Control Center Modules
 - Wi-Fi = Show in Menu Bar
 - Bluetooth = Show in Menu Bar
 - AirDrop = Show in Menu Bar
 - Focus = Always Show in Menu Bar
 - Stage Manager = Always Show in Menu Bar
 - Screen Mirroring = Show When Active
 - Display = Show When Active
 - Sound = Show When Active
 - Now Playing = Show When Active
###### Other Modules
  - Accessibility Shortcuts -> Show in Menu Bar = unchecked
  - Hearing -> Show in Menu Bar = unchecked
  - Fast User Switching = Don't Show
###### Menu Bar Only
  - Clock -> Time -> Use a 24-hour clock = checked
  - Clock -> Display the time with seconds = checked
  - Spotlight -> Show in Menu Bar
  - Siri -> Show in Menu Bar
  - Time Machine -> Don't Show in Menu Bar

##### Siri & Spotlight
  - Ask Siri = unchecked
  - Spotlight -> Search Results
    - Applications = checked
    - System Settings = checked
    - {all other options} = unchecked

##### Privacy & Security
###### Privacy
- Location Services -> Location Services = unchecked
###### Security
- Allow applications downloaded from = App Store and identified developers
- FileVault -> Turn On FileVault = checked
###### Others

##### Desktop & Dock

##### Displays
- Advanced -> Show resolutions as list = checked

##### Wallpaper

##### Screen Saver
###### Lock Screen
- Start Screen Saver when inactive = Never
- Turn display off when inactive = For 5 minutes
- Require password after screen saver begins or display is turned off = After 5 seconds

##### Energy Saver

##### Lock Screen

##### Login Password

##### Users & Groups

##### Passwords
###### Password Options
- AutoFill Passwords = unchecked
- Security Recommendation -> Detect Leaked Passwords = unchecked

##### Internet Accounts

##### Game Center

##### Keyboard
###### Text Input
- Input Sources -> {add Italian}
- Input Sources -> Capitalize words automatically = unchecked
- Input Sources -> Add period with double-space = unchecked

##### Trackpad
###### Point & Click
###### Scroll & Zoom
- Natural scrolling = checked
###### More Gestures

##### Printers & Scanners


### Finder configuration
#### Finder Settings
- Finder Settings -> General -> Show these items on the desktop :: [] = unchecked
- Finder Settings -> General -> New Finder windows show :: {home directory}
- Finder Settings -> General -> Open folders in tabs instead of new windows = checked
- Finder Settings -> Sidebar -> Favorites :: [Recents, Movies, Music, Pictures] = unchecked
- Finder Settings -> Sidebar -> Favorites :: [AirDrop, Applications, Desktop, Documents, Downloads, Home] = checked
- Finder Settings -> Sidebar -> Locations :: [] = unchecked
- Finder Settings -> Sidebar -> Tags :: ['Recent Tags'] = unchecked

#### View Options - use as defaults
- View Options -> Always open in list view = checked
- View Options -> Browse in list view = checked
- View Options -> Group files by = None
- View Options -> Sort by = Name
- View Options -> Icon size = {small}
- View Options -> Text size = 12
- View Options -> Show columns :: [Date Modified, Date Created, Date Added, Size, Kind, Version] = checked 

### System Applications

#### App Store
- Settings -> In-App Ratings & Reviews = unchecked

#### FaceTime
#### Find My
#### Messages
- Settings -> General -> Set up Name and Photo Sharing... = {Name and Photo}
- Settings -> General -> Auto-play message effects = unchecked
- Settings -> General -> Play sound effects = unchecked
- Settings -> iMessage -> Enable Messages in iCloud = checked
- Settings -> Shared with You -> Automatic Sharing = unchecked

#### Music
Open Music, make sure computer is authorized, and set Library sync options.
- Settings -> General -> Use Listening History = unchecked
- Settings -> General -> Notifications -> When song changes = unchecked
- Settings -> Playback -> Audio Quality -> Lossless Audio = checked
- Settings -> Playback -> Audio Quality -> Streaming = Lossless
- Settings -> Playback -> Audio Quality -> Download = Lossless
- Settings -> Advanced -> Add songs to Library when adding to playlists = unchecked

#### Photos
Open Photos and make sure iCloud sync options are correct.
- Settings -> General -> Memories -> Show Featured Content = unchecked
- Settings -> General -> Memories -> Show Memories Notification = unchecked
- Settings -> Shared Library -> Shared Library Suggestions = unchecked

#### Safari
- Settings -> General -> Remove history items = Manually
- Settings -> General -> Remove download list items = When Safari quits
- Settings -> General -> Open "safe" files after downloading = unchecked
- Settings -> AutoFill -> AutoFill web forms :: [] = checked
- Settings -> Privacy -> Apple Pay and Apple Card -> Allow websites to check for Apple Pay and Apple Card = unchecked
- Settings -> Privacy -> Web Advertising -> Allow privacy-preserving measurement of ad effectiveness = unchecked
- Settings -> Advanced -> Smart Search Field -> Show full website address = checked
- Settings -> Advanced -> Show Develop menu in menu bar = checked

#### Siri

#### Terminal
- Install theme

### User Applications

#### 1Password
Open 1Password, sign in, and set up sync using details from iPhone.
- Settings -> General -> Default Vault -> Save new items in = {Personal}
- Settings -> Appearance -> Always Show in Sidebar :: [Categories, Tags] = checked
- Settings -> Advanced -> Show debugging tools = checked
- Settings -> Developer -> Command-Line Interface -> Connect with 1Password CLI = checked
##### CLI Integration
- 1Password -> Install 1Password CLI...
  - Install following [Get started with 1Password CLI](https://developer.1password.com/docs/cli/get-started/#install)
```bash
op signin
```
###### Github
- [Use 1Password to securely authenticate the GitHub CLI](https://developer.1password.com/docs/cli/shell-plugins/github)
```bash
op plugin init gh
```
- Select an existing item
  - _select existing GitHub Login from 1Password_
- Set default credential scope -> Use as global default on my system
- Source the plugins.sh file
  - plugin.sh should already be sourced in Dotfiles `.bash_profile`

#### Adobe Creative Cloud
Open 1Password, sign in, and set up sync.
Apps to install:
- Photoshop
- Illustrator
- Acrobat DC
- InDesign
- Premiere Pro
- Lightroom Classic

##### Lightroom Classic

Copy settings from another computer
```bash
~/Library/Application Support/Adobe/CameraRaw/Settings/{profile_name}
~/Library/Application Support/Adobe/Lightroom
```

- Settings -> General -> Default Catalog -> When starting up use this catalog = Prompt me when starting Lightroom
- Settings -> Presets -> Override global settings for specific cameras = checked
- Settings -> Presets -> Camera -> {find a way to import}
- Settings -> External Editing -> Edit in Adobe Photoshop 2023 -> File Format = PSD
- Settings -> External Editing -> Edit in Adobe Photoshop 2023 -> Resolution = 600
- Settings -> External Editing -> Additional External Editor -> Resolution = 600

#### Docker
Open and 'Use recommended settings' to complete installation.
- Settings -> General -> Send usage statistics = unchecked

#### Doxie
- Settings -> General -> Play sound effects = unchecked

#### Dropbox
Open Dropbox, sign in, and set up sync.
- Settings -> Backup -> Show setup notifications when new external drives are plugged-in = unchecked
- Settings -> Backup -> Enable camera uploads for = unchecked
- Settings -> Notifications -> Notify me about :: [] = 
- checked

#### Firefox 
Open app and do not import and setting or set as default browser.

#### Google Chrome
Open app and do not import and setting or set as default browser. Do not send usage statistics.

#### iTerm
- iTerm2 -> Make iTerm2 Default Term
- iTerm2 -> Install Shell Integration
- Preferences -> Profiles :: {import profile}
- Preferences -> Keys -> Hotkey :: Show/hide all windows with a... = checked
- Preferences -> Keys -> Hotkey :: Hotkey = ^`

#### JetBrains Toolbox
Open app and do not send usage statistics.
- Settings -> Login
Apps to install:
- GoLand
- PyCharm Professional Edition
- WebStorm
- PhpStorm
- DataGrip

#### NetNewsWire

#### Notion

#### Rectangle
Open app and grant system permissions.

#### Textmate
- Settings -> Files -> With no open documents -> Create one at startup = unchecked
- Settings -> Projects -> File browser location -> Folders on top = checked
- Settings -> Projects -> Show file browser on = Left side
- Settings -> Bundles -> [] = {manually chosen}
- Settings -> Software Update -> Software Update :: Ask before up... = checked
- Settings -> Software Update -> Crash Reports :: Submit to... = unchecked
- Settings -> Terminal :: Shell support = {install at default location}
- Settings -> Terminal :: Accept rmate connections = unchecked


### Cloud Services


### Syncing additional assets
#### Fonts
Sync fonts from Dropbox or another location.
```bash
cp -R ~/Dropbox/Apps/Config/Fonts/* ~/Library/Fonts/
```
