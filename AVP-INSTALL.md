# Installing Portal 2 VR on Apple Vision Pro

Portal 2 VR is by [portal-in-the-visor](https://github.com/portal-in-the-visor/portal2-visionos), who archived it and asked
the community to fork it and take it further. This repository is a copy of theirs with this guide added, so it can be
listed in the [AVP Ports Index](https://github.com/edgytoast/avp-ports-index); [Trevorbilt](https://trevorbilt.com) wrote
the guide. It's built on [nxnative](https://github.com/seedds/nxnative) by seedds.

*Portal 2* (Nintendo Switch, v1.0.0) runs as a native visionOS app with stereo rendering, head tracking, a hand-aimed
portal gun and Joy-Con controls. None of the game's files are here. It has been tested on one M2 Vision Pro.

## Requirements

- A Mac with Apple silicon, macOS 26.6 or later, and Xcode 27.1 with the visionOS platform
- [Homebrew](https://brew.sh) and Python 3.12 or newer
- Apple Vision Pro in Developer Mode, paired with Xcode
- An Apple ID signed in to Xcode (Xcode › Settings › Accounts)
- Your own Portal 2 for Nintendo Switch, version 1.0.0 (title ID `0100ABD01785C000`), and `prod.keys` from your own Switch
- A pair of Joy-Cons or another game controller
- Free disk space of about twice the game's size plus 10 GB

## Game files

You need to supply your own game files. This repository contains none. Use only files dumped from a game you bought and
from your own Switch.

1. Dump Portal 2 as an `.nsp`, `.xci`, `.nsz` or `.xcz` (base game 1.0.0; no update has been tested), and dump `prod.keys`.
2. Put the game file in a folder of its own, for example `~/Games/Portal2/`.
3. The build writes everything to `0100ABD01785C000_build` inside that folder, including the game image it copies to the
   headset later (`extract/<version>/RomFS.img`).

## Build

Start from a checkout of this repository.

1. Install the tools (once):

   ```sh
   xcodebuild -downloadComponent MetalToolchain
   brew install xcodegen zstd glslang
   brew install --cask dotnet-sdk
   python3 -m pip install lz4 capstone fonttools nsz "backports.zstd; python_version < '3.14'"
   ```

2. Prepare the game. This extracts and rewrites the game code and prepares the shaders, then generates an Xcode project
   instead of an IPA. `--romfs separate` keeps the game files out of the app, so installs stay quick. The first run takes
   30 to 60 minutes:

   ```sh
   python3 scripts/build_ipa_from_nsp.py ~/Games/Portal2 -k ~/keys/prod.keys --prepare-xcode --romfs separate
   ```

3. Generate the visionOS project from that build folder:

   ```sh
   python3 dm/visionos/tools/gen_visionos.py ~/Games/Portal2/0100ABD01785C000_build
   ```

4. Open `~/Games/Portal2/0100ABD01785C000_build/xcode-visionos/DmVision.xcodeproj`. Under Signing & Capabilities, choose
   your team and change the bundle identifier to one of your own.

## Install on Apple Vision Pro

1. Select the scheme **Portal 2 visionOS** and your Vision Pro, and press Run. Trust your developer certificate in
   Settings › General › VPN & Device Management if the headset asks.
2. With the app installed, copy the game files into its Documents folder once, using your headset's identifier
   (`xcrun devicectl list devices`) and the bundle identifier you chose:

   ```sh
   scripts/push_romfs.sh <device UDID> <your bundle ID> ~/Games/Portal2/0100ABD01785C000_build/extract/<version>/RomFS.img
   ```

   Step 2 of the build prints this command with your paths. Run it again only after a new extraction or game version.
3. Launch the app, pair a controller, and press **Enter VR**. Allow hand tracking and world sensing when asked.

## Controls and known issues

- Head: look around; walking goes where you look. Right hand: aims the portal gun. Controller: stick to walk, triggers
  to fire, buttons as in the Switch game.
- Settings such as field of view and hand aim can be changed with an `env.txt` in the app's Documents folder; see the
  [README](README.md#tuning).
- Heavier areas can drop below 60 fps (judder); the portal gun's glow is missing; objects carried close, like Wheatley,
  can look squashed; menus, loading screens and the pause menu show on a flat screen.
