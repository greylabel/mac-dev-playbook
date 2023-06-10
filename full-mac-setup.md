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
#### Dock
#### System Preferences
#### Time Machine
Configure Time Machine backup drive and [Time Machine Editor](https://tclementdev.com/timemachineeditor/) (if needed)
#### Vim

### Manual App configuration
#### Photos
Open Photos and make sure iCloud sync options are correct.
#### Music
Open Music, make sure computer is authorized, and set Library sync options.

#### Dropbox
Open Dropbox, sign in, and set up sync.
- Preferences -> Import -> Photos :: Enable camera uploads = unchecked
- Preferences -> Notification -> Personal :: all[] = unchecked
- Preferences -> Notification -> Business :: all[] = unchecked
- Preferences -> Sync -> Selective Sync :: [] = configure

#### iTerm
- Preferences -> Profiles :: {how to import from json?}
- Preferences -> Keys -> Hotkey :: Show/hide all windows with a... = checked
- Preferences -> Keys -> Hotkey :: Hotkey = ^`

#### Textmate
- Preferences -> Projects -> Show file browser on = Left side
- 
- Preferences -> Bundles -> [] = {manually chosen}

- Preferences -> Software Update -> Software Update :: Ask before up... = checked
- Preferences -> Software Update -> Crash Reports :: Submit to... = unchecked

- Preferences -> Terminal :: Shell support = {install at default location}
- Preferences -> Terminal :: Accept rmate connections = unchecked


### Syncing additional assets
#### Fonts
Sync fonts from Dropbox or another location.
```bash
cp -R ~/Dropbox/Apps/Config/Fonts/* ~/Library/Fonts/
```
