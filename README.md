# Mac Development Ansible Playbook

[![CI][badge-gh-actions]][link-gh-actions]

Based on the [Mac Development Ansible Playbook](https://github.com/geerlingguy/mac-dev-playbook) by Jeff Geerling ([geerlingguy](https://github.com/geerlingguy)).

## Installation

See the full from-scratch setup document [full-mac-setup.md](full-mac-setup.md) for more details.

### Run without the `post` tag

    ansible-playbook main.yml --ask-become-pass --skip-tags post

### Running a specific set of tagged tasks

You can filter which part of the provisioning process to run by specifying a set of tags using `ansible-playbook`'s `--tags` flag. The tags available are `dotfiles`, `homebrew`, `mas`, `extra-packages` and `osx`.

    ansible-playbook main.yml -K --tags "dotfiles,homebrew"

### Use with a remote Mac

You can use this playbook to manage other Macs as well; the playbook doesn't even need to be run from a Mac at all! If you want to manage a remote Mac, either another Mac on your network, or a hosted Mac like the ones from [MacStadium](https://www.macstadium.com), you just need to make sure you can connect to it with SSH:

1. (On the Mac you want to connect to:) Go to System Preferences > Sharing.
2. Enable 'Remote Login'.

> You can also enable remote login on the command line:
>
>     `sudo systemsetup -setremotelogin on`

Then edit the `inventory` file in this repository and change the line that starts with `127.0.0.1` to:

```
[ip address or hostname of mac]  ansible_user=[mac ssh username]
```

If you need to supply an SSH password (if you don't use SSH keys), make sure to pass the `--ask-pass` parameter to the `ansible-playbook` command.