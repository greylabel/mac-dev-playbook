# Mac Development Ansible Playbook

[![CI][badge-gh-actions]][link-gh-actions]

Based on the [Mac Development Ansible Playbook](https://github.com/geerlingguy/mac-dev-playbook) by Jeff Geerling ([geerlingguy](https://github.com/geerlingguy)).

## Installation

See the full from-scratch setup document [full-mac-setup.md](docs/full-mac-setup.md) for more details.

Installation include the following steps.

#### macOS install and initiation setup
1. Install macOS
1. Sign in to iCloud and the App Store
1. Install  Xcode command line tools and agree to license.

#### Python and Ansible
1. Add `python3` to `$PATH`, update `pip`and `setuptools`, and install `ansible`
1. Add `brew` to `$PATH` (confirm if this is needed for playbook to run)
1. Clone or download the (this) Mac Development Ansible Playbook. 
   1. Copy in additional config files.
   1. Install requirements and run playbook.

#### Playbook
1. Create directories for source code and websites in home folder
1. [A] Create SSH key and add to SSH Agent, copy `config` setup git commit signing 
1. [A] Copy or symlink additional dotfiles
1. [A - post] Install non-managed software, e.g. Node
1. [A - post] Manually copy other assets, e.g. fonts, repos, documents

#### Manual configuration
1. Manually configure macOS, System Application, and User Application, Cloud Applications

#### Cleanup
1. Clean up temporary artifacts


### Running without a specific tag

    ansible-playbook main.yml --ask-become-pass --skip-tags "post"

### Running a specific set of tagged tasks

You can filter which part of the provisioning process to run by specifying a set of tags using `ansible-playbook`'s `--tags` flag. The tags available are `dotfiles`, `homebrew`, `mas`, `extra-packages` and `osx`.

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
