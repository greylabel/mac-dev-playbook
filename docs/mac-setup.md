# Mac Setup Process (macOS Ventura 13.x)

This document describes the full process of setting up a fresh macOS installation per my particular preferences and requirements — hopefully will serve as a helpful reference for anyone interested.

## macOS installation and initiation setup
### macOS

> See: [Create a bootable installer for macOS
](https://support.apple.com/en-us/HT201372)

First and foremost, macOS needs to be installed.  Then, complete Apple's mandatory macOS setup wizard, creating a local admin user account.

### iCloud and App Store
Sign in to iCloud and the App Store.

### Software Update
Install any available macOS and App Store updates.

### Xcode
Install Xcode command line tools.

```bash
xcode-select --install
```

Agree To Xcode (only with a full Xcode installation).

```bash
sudo xcrun cc
```

### Rosetta
Install Rosetta (Apple silicon only).
```bash
softwareupdate --install-rosetta --agree-to-license
```

### Manually installed apps
Install applications that cannot be installed by Homebrew, through the App Store, or via another easy and Ansible-friendly scriptable process. This is done before the main playbook is run to ensure apps are in place for tasks that depend on their presence, like configuring the Dock.


## Python and Ansible
Ensure Python 3 is installed and note the version.

```bash
which python3 && python3 --version
```

Upgrade essential packages. Note that running `pip3` without `sudo` with default to a user installation in `~/Library/Python/`.

```bash
python3 -m pip install --user --upgrade pip setuptools virtualenv ipython
```
Install bcrypt — required for generating and securing SSH keys with ansible.
```bash
python3 -m pip install --user bcrypt
```

Install Ansible.
```bash
python3 -m pip install --user ansible
```

Add Python 3 to `PATH` environment variable temporarily.

> Note that the locations of Python 3 binaries and libraries may be different depending on the version of Python installed. The following example assumes Python 3.9. The actual path can be determined by running `which python3 && python3 --version` and `python3 -m site --user-base` in the terminal and adjusting the below accordingly.

```
export PATH="$HOME/Library/Python/3.9/bin:$PATH"
```

>  _If desired, Ansible and Python 3 can be (re)installed by Homebrew when the main playbook is run, and thus managed by `brew` going forward._

## Mac Development Ansible Playbook
### Get the playbook
Clone or download (this) [mac-dev-playbook](https://github.com/greylabel/mac-dev-playbook) git repo to a temporary location, or wherever you prefer to store source code checkouts, e.g. `/tmp` or `~/Projects`.

#### Clone with Git
```bash
git clone git@github.com:greylabel/mac-dev-playbook.git
```

#### Download as a zip file
```bash
curl -LJO https://github.com/greylabel/mac-dev-playbook/archive/refs/heads/main.zip
```
```bash
unzip mac-dev-playbook-main.zip && rm mac-dev-playbook-main.zip
```

### Requirements
>The below commands are run from the cloned `mac-dev-playbook` directory.
 
Install required roles.
```bash
ansible-galaxy install -r requirements.yml
```

### Additional configuration
Optionally, copy a `config.yml` file to he cloned `mac-dev-playbook` directory,  if configuration overrides are required.

#### Ansible Vault
> If encrypted content is required and will be supplied with Ansible Vault, adjust the commands below accordingly to include when running the various plays.

### Plays, roles, tasks, and tags

```bash
ansible-playbook main.yml --ask-become-pass --ask-vault-pass --tags "pre"
ansible-playbook main.yml --ask-become-pass --ask-vault-pass --skip-tags "post, post"
ansible-playbook main.yml --ask-become-pass --ask-vault-pass --tags "post"
```

#### Pre-provision pre_tasks

```bash
ansible-playbook main.yml --ask-become-pass --ask-vault-pass --tags "pre"
```

##### Directories for source code
Create `Projects` and `Sites` home directories. These directories will be used later in the process and are not present by default in macOS. They generally will contain source code and websites, respectively.

##### SSH key pair

##### User specific Git config
Create a `~/.gitconfig.local` file for username / github token / etc.

#### Roles

```bash
ansible-playbook main.yml --ask-become-pass --ask-vault-pass --tags "homebrew"
ansible-playbook main.yml --ask-become-pass --ask-vault-pass --tags "dotfiles"
ansible-playbook main.yml --ask-become-pass --ask-vault-pass --tags "mas"
ansible-playbook main.yml --ask-become-pass --ask-vault-pass --tags "dock"
```


##### Command line tools

##### Homebrew
Homebrew is installed by Ansible when the main playbook is run. Add Homebrew's `bin` directory to `PATH` environment variable, if needed for temporary use. This will be properly persisted when Dotfiles are installed later.

> @TODO: Consider using [Homebrew Bundle](https://github.com/Homebrew/homebrew-bundle) to manage package list and apps with a Brewfile, and potentially move back to Dotfiles repo.

#### Dotfiles
Dotfiles can be installed by Ansible when the main playbook is run. My [Dotfiles](https://github.com/greylabel/dotfiles/tree/main) include configuration for many of the packages installed by Homebrew, as well as an assortment of other tools and aliases.

> Note: macOS scriptable settings are stored in the Dotfiles repo, as is custom with other dotfiles setups around the web, but can be applied by Ansible when the main playbook is run.

##### Mac App Store

##### Dock

#### Tasks

ansible-playbook main.yml --ask-become-pass --ask-vault-pass --tags "extra-packages"
ansible-playbook main.yml --ask-become-pass --ask-vault-pass --tags "osx"

##### extra-packages

##### osx

##### Post-provision tasks

tag `post`

ansible-playbook main.yml --ask-become-pass --ask-vault-pass --tags "post"

> @TODO: Note about this is where tasks that are not idempotent live.

###### Activate utils installed by Homebrew

## Manual App configuration

See [Manual macOS and Application Configuration](macos-and-app-manual-config.md) for detailed configuration guide.

## Additional assets

#### Fonts
Sync fonts from Dropbox or another location.

##### Dropbox
```bash
cp -R ~/Dropbox/Apps/Config/Fonts/* ~/Library/Fonts/
```

##### iCloud Drive.
```bash
cp -R ~/Library/Mobile\ Documents/com~apple~CloudDocs/Config/fonts/* ~/Library/Fonts/
```

#### Source code and websites
Optionally, copy contents of `~/Projects` and/or `~/Sites` folder(s) from another Mac (to save time).

## Temporary artifacts
Remove or move this repo from its temporary location and use a proper git stored with other source code files.
