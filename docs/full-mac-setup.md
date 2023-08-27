# Full Mac Setup Process (macOS Ventura 13.x)

This document describes the full process of setting up a fresh macOS installation per my particular preferences and requirements. It is likely overkill or divergent from your specific use case, but hopefully will serve as a helpful reference.

## macOS setup
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

### SSH
#### Generating a new SSH key
> @TODO: Investigate if this can be automated with Ansible and implications on remote Mac provisioning.

> See Github's [Generating a new SSH key and adding it to the ssh-agent](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/) for more information.

Create directory for SSH keys and configuration and set permissions.
```bash
mkdir ~/.ssh && chmod 0700 ~/.ssh
```
Generate a new SSH key.
```bash
ssh-keygen -t ed25519 -C "grant@greylabel.net"
```

```bash
chmod 600 ~/.ssh/id_ed25519 && chmod 644 ~/.ssh/id_ed25519.pub
```

At this point, the new key can be added to GitHub and other cloud services.

#### SSH Configuration

Create and set permissions on the `~/.ssh/config` file.

```bash
touch ~/.ssh/config && chmod 644 ~/.ssh/config
```

Modify the `~/.ssh/config` file to automatically load keys into the ssh-agent and store passphrases in your keychain.

```bash
IgnoreUnknown AddKeysToAgent,UseKeychain

Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
```

#### Adding your SSH key to the ssh-agent
```bash
eval "$(ssh-agent -s)"
```

```bash
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

### Allowed Signers
> See: https://docs.gitlab.com/ee/user/project/repository/ssh_signed_commits/

Create and set permissions on the `allowed_signers` file.

```bash
touch ~/.ssh/allowed_signers && chmod 644 ~/.ssh/allowed_signers
```

Add an entry with a public key.

```bash
grant@greylabel.net namespaces="git" <public_key>
```

#### Adding a passphrase to an existing SSH key

Add/replace the passphrase for an existing private key without regenerating the keypair.

```bash
ssh-keygen -p -f ~/.ssh/id_ed25519
```

## Install Python and Ansible
Ensure Python 3 is installed and note the version.

```bash
which python3 && python3 --version
```

Upgrade essential packages. Note that running `pip3` without `sudo` with default to a user installation in `~/Library/Python/`.

```bash
pip3 install --upgrade pip setuptools virtualenv ipython
```

```bash
python3 -m pip install bcrypt
```

Install Ansible with `pip` for the current user.
```bash
pip3 install --upgrade ansible
```
Add Python 3 to `PATH` environment variable.
> Note that the locations of Python 3 binaries and libraries may be different depending on the version of Python installed. The following example assumes Python 3.9. The actual path can be determined by running `which python3` and `python3 -m site --user-base` in the terminal and adjusting the below accordingly.
```bash
export PATH="$HOME/Library/Python/3.9/bin:$PATH"
```
> _If desired, Ansible and Python 3 can be (re)installed by Homebrew when the main playbook is run, and thus managed by `brew` going forward._

### $PATH
> @TODO: Does brew need to be in `$PATH` for ansible to install its packages?
```bash
export PATH="$HOME/Library/Python/3.9/bin:/opt/homebrew/bin:$PATH"
```

## Other manually installed items
Install applications that cannot be installed by Homebrew, through the App Store, or via another easy and Ansible-friendly scriptable process. This is done before the main playbook is run to ensure apps are in place for tasks that depend on their presence, like configuring the Dock.

## Mac Development Ansible Playbook
### Get the playbook
Clone or download (this) [mac-dev-playbook](https://github.com/greylabel/mac-dev-playbook) git repo to a temporary location, or wherever you prefer to store source code checkouts, e.g. `~/Projects`.

#### Clone with Git
```bash
git clone git@github.com:greylabel/mac-dev-playbook.git

git clone https://github.com/greylabel/mac-dev-playbook.git
```

#### Download as a zip file
```bash
curl -LJO https://github.com/greylabel/mac-dev-playbook/archive/refs/heads/main.zip
```
```bash
unzip mac-dev-playbook-main.zip && rm mac-dev-playbook-main.zip
```

### Install requirements
>The below commands are run from the cloned `mac-dev-playbook` directory.
Install required roles.
```bash
ansible-galaxy install -r requirements.yml
```

### Additional configuration
Optionally, add a `config.yml` to the playbook if configuration overrides are required (store in iCloud, or copy over the network or using a USB flash drive).

> @TODO: Note about `ansible vault` file for sensitive data.

### Run the playbook
Run main playbook with 'BECOME' password and skip post tasks.
```bash
ansible-playbook main.yml --ask-become-pass --skip-tags post
```
> @TODO: Note about `post` and `never`.


#### Pre-provision pre_tasks

##### Directories for source code
Create directories for Projects and Sites. These directories will be used later in the process and are not present by default in macOS. They generally will contain source code and websites, respectively.
```bash
mkdir ~/Projects 
chmod u+rwx,go-rwx ~/Projects
```
```bash
mkdir ~/Sites 
chmod u+rwx,go-rwx ~/Sites
```

##### User specific Git config
Create and use `~/.gitconfig.local` file for username / github token / etc.

```
[user]

  name = Grant Gaudet
  email = grant@greylabel.net
  signingkey = ~/.ssh/id_ed25519.pub
  
[github]

  user = greylabel

```

#### Roles

##### Command line tools

##### Homebrew
Homebrew is installed by Ansible when the main playbook is run. For reference, the official command to install Homebrew with `curl` is below.
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

> Note: If some Homebrew commands fail, you might need to agree to Xcode's license or fix some other Brew issue. Run `brew doctor` to see if this is the case.

> @TODO: Do we need `brew` in `$PATH` at here?

Add Homebrew's `bin` directory to `PATH` environment variable, if needed for temporary use. This will be properly persisted when Dotfiles are installed later.

```bash
export PATH="/opt/homebrew/bin:$PATH"
```

> @TODO: Consider using [Homebrew Bundle](https://github.com/Homebrew/homebrew-bundle) to manage package list and apps with a Brewfile, and potentially move back to Dotfiles repo.


#### Dotfiles
Dotfiles can be installed by Ansible when the main playbook is run. My [Dotfiles](https://github.com/greylabel/dotfiles/tree/main) include configuration for many of the packages installed by Homebrew, as well as an assortment of other tools and aliases.

> Note: macOS scriptable settings are stored in the Dotfiles repo, as is custom with other dotfiles setups around the web, but can be applied by Ansible when the main playbook is run.

##### Mac App Store

##### Dock

#### Tasks

##### osx

##### extra-packages

##### Post-provision tasks
> @TODO: Note about this is where tasks that are not idempotent live.

###### ACLs for Projects and Sites
Set ACLs for Projects and Sites to prevent accidental deletion.
```bash 
chmod +a "group:everyone deny delete" ~/Projects
```
```bash
chmod +a "group:everyone deny delete" ~/Sites
```

###### Activate utils installed by Homebrew

> @TODO: Turn into a `post` task

```bash
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


## Manual App configuration

See [Manual macOS and Application Configuration](manual-macos-and-app-config.md) for detailed configuration guide.


## Syncing additional assets

#### Fonts
Sync fonts from Dropbox or another location.
```bash
cp -R ~/Dropbox/Apps/Config/Fonts/* ~/Library/Fonts/
```

#### Source code and websites
Optionally, copy contents of `~/Projects` and/or `~/Sites` folder(s) from another Mac (to save time).

## Cleanup
Remove or move this repo from its temporary location.
