# Mac Teardown Process

This document describes the full process of resetting a Mac that was set up following the [Mac Setup Process](mac-setup.md) to its factory settings — useful when preparing for a clean installation or getting ready to part ways with a computer.

## Backup files in, 1Password, iCloud Drive or elsewhere

- SSH Keys
  - (should be backed up in 1Password already)
  - `id_ed25519`
  - `id_ed25519.pub`
- Lightroom settings
  - `~Library/Application Support/Adobe/Lightroom`
- Camera Raw Settings
  - `~/Library/Application Support/Adobe/Camera Raw/Settings`

## Sign out and deauthorize apps

- Sign out of Backblaze - adjust schedule to be manual to pause
- Deauthorize computer in (Apple Music) Music App and sign out
- Sing out of iCloud, and confirm sign out for Messages, Photos, FaceTime, etc
- Sign out JetBrains Toolbox
- Sign out of Adobe Creative Cloud
- Sign out of Dropbox and delete folder
- Sign out of 1Password

## Follow Apple's guide to factory reset

- [Erase your Mac and reset it to factory settings](https://support.apple.com/en-us/HT212749)
