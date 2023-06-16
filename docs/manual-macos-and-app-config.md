
## macOS config

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
- Settings -> Extensions -> 1Password for Safari = checked
- Settings -> Extensions -> (NetNewsWire) Subscribe to Feed = checked
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
###### SSH
Import any new SSH keys into 1Password.
Turn on the 1Password SSH Agent.
- Settings -> Developer -> SSH Agent -> Setup SSH Agent...

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

#### Tower
- Settings -> General -> Terminal Application -> iTerm2
- Settings -> General -> Automatically fetch remote repositories = Never (only manually)
- Settings -> General -> Show Reflog in sidebar = checked
- Settings -> Git Config -> Git binary = {homebrew}
- Settings -> Git Config -> Global signing key = (greyed out) ~/.ssh/id_ed25519.pub
- Settings -> Git Config -> Allowed signers file = ~/.ssh/allowed_signers
- Settings -> Git Config -> Sign commits = checked
- Settings -> Advanced -> Automatically index Git repositories = unchecked

### Cloud Services
#### Github
Visit the [GitHub SSH key settings page](https://github.com/settings/ssh/new) to upload your public key as both an _Authentication Key_ and a _Signin Key_.

#### Gitlab
Visit the [User settings -> SSH Keys page](https://gitlab.com/-/profile/keys) to upload your public key with Usage type = Authentication & Signing
