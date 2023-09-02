# Mac Development Ansible Playbook

[![CI][badge-gh-actions]][link-gh-actions]

Based on the [Mac Development Ansible Playbook](https://github.com/geerlingguy/mac-dev-playbook) by Jeff Geerling ([geerlingguy](https://github.com/geerlingguy)).

## Installation

See the full from-scratch setup document [full-mac-setup.md](docs/mac-setup.md) for more details.

Setup goes through the following automated [A] and manual [M] steps.

#### macOS installation and initiation setup
1. [M] Install macOS
1. [M] Sign in to iCloud and the App Store and run software update
1. [M] Install Xcode command line tools and agree to license
1. [M] Install Rosetta (Apple silicon)
1. [M] Install apps that won't be handled through automation

#### Python and Ansible
1. [M] Add `python3` to `$PATH`, update packages, and install `ansible`
1. [M] Clone or download the (this) Mac Development Ansible Playbook. 
   1. [M] Copy in additional config files and vaults.
   1. [M] Install requirements and run playbook.

#### Ansible playbook
1. [A] Ensure OS X Command Line Tools are installed
1. [A] Create home directories for source code and websites
1. [A] Create SSH key and config, add key to SSH Agent, setup git commit signing
2. [A] Install packages and apps with Homebrew
3. [A] Install and symlink dotfiles
4. [A] Install apps from the Mac App Store
5. [A] Configure macOS Dock
6. [A] Install packages from other package managers, e.g. composer, pip
7. [A] Configure masOS

#### Manual configuration
1. [M] Install non-managed software, e.g. Node
1. [M] Configure macOS, System Application, User and Cloud Applications
1. [M] Manually copy other assets, e.g. fonts, repos, documents

#### Cleanup
1. [M] Clean up temporary artifacts


### Running without a specific tag

    ansible-playbook main.yml --ask-become-pass --skip-tags "post"

### Running a specific set of tagged tasks

You can filter which part of the provisioning process to run by specifying a set of tags using `ansible-playbook`'s `--tags` flag. The tags available are `pre`, `post`, `dotfiles`, `homebrew`, `mas`, `extra-packages` and `osx`.

    ansible-playbook main.yml -K --tags "dotfiles,homebrew"

### Use with a remote Mac

You can use this playbook to manage other Macs as well; the playbook doesn't even need to be run from a Mac at all! If you want to manage a remote Mac, either another Mac on your network, or a hosted Mac like the ones from [MacStadium](https://www.macstadium.com), you just need to make sure you can connect to it with SSH:

Enable _Remote Login_ on the Mac you want to connect to:
- System Settings -> General -> Sharing -> Remote Login = On

> You can also enable remote login on the command line:
>
>     `sudo systemsetup -setremotelogin on`

Then edit the `inventory` file in this repository and change the line that starts with `127.0.0.1` to:

```
[ip address or hostname of mac]  ansible_user=[mac ssh username]
```

If you need to supply an SSH password (if you don't use SSH keys), make sure to pass the `--ask-pass` parameter to the `ansible-playbook` command.
