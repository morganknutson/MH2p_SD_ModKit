# MH2p PCM Modding Project Handoff

Last updated: 2026-08-29 (America/Los_Angeles)

This document preserves the complete working context from the initial investigation so the project can be resumed on another computer without repeating the discovery work.

## Project goal

The immediate goals are:

1. Enable full-screen Apple CarPlay on a Porsche PCM running MH2p.
2. Extract the exact stock Porsche PCM user-interface resources and relevant compiled HMI code.
3. Replace or redesign stock Porsche button backgrounds and other UI surfaces. Replacing CarPlay icons is not the goal, and replacing all Porsche icons is no longer the primary goal.
4. Build a local, interactive PCM UI simulator/theme previewer using the exact extracted resources.
5. Package any eventual theme as a reversible, firmware-specific ModKit component with backups and an uninstaller.

The next concrete task is to build a **read-only PCM UI dump mod** before building the simulator or modifying any UI assets.

## Repository and GitHub state

Original repository:

- `https://github.com/LawPaul/MH2p_SD_ModKit`

Working fork:

- `https://github.com/morganknutson/MH2p_SD_ModKit`

Local remotes are configured as:

- `origin` -> `https://github.com/morganknutson/MH2p_SD_ModKit.git`
- `upstream` -> `https://github.com/LawPaul/MH2p_SD_ModKit`
- Local `main` tracks `origin/main`.

At handoff time there was a pre-existing untracked `.gitignore`. It was deliberately left untouched and must not accidentally be included in unrelated commits.

## Exact vehicle/PCM information

The vehicle's Version Information screen was photographed and confirms:

- Software string: `MH2p_US_POG35_P2656`
- Hardware version: `H056`
- Display/software family: `G35`
- Region: North America / `NAR`
- Media driver: `DEV_MMX2P_PAG_NAR_G35_093PROD-1`
- Phone driver: `43.600.648`
- Navigation database: `V03959804JH_P0326_NA_2019.3`
- Gracenote: `Region=NORTH_AMERICA - Version=8`
- Station logo database: not available
- Owner's Manual: not available

This exact software string matters. Do not use a JAR or patch intended for a merely similar release when a firmware-specific file exists.

## Full-screen CarPlay mod bundle

A downloaded bundle was previously inspected at:

`/Users/morgan/Downloads/MH2p_ModKit_Mods_Bundle`

That path is local to the original computer and will not exist on the new machine unless copied separately.

The bundle was byte-for-byte identical to the cloned ModKit repository except for one added component:

`Mods/Phone_FullScreen`

All 21 JARs and the component's install/uninstall scripts matched the current `LawPaul/MH2p_CarPlay_FullScreen` repository by Git blob SHA at the time of inspection:

- `https://github.com/LawPaul/MH2p_CarPlay_FullScreen`

For this PCM, the exact full-screen file is:

`fc-full-POG35_P2656.jar`

The installer derives a filename in this form:

`fc-full-PO<TYPE>_<RELEASE_TYPE><SOFTWARE_VERSION>.jar`

It installs the selected JAR as:

`/mnt/app/eso/hmi/lsd/jars/fc.jar`

The uninstaller removes that `fc.jar`.

The firmware-specific JAR overrides compiled Java HMI classes, commonly including:

- `de.audi.app.terminalmode.adi.ADITMConfiguration`
- `de.audi.tghu.terminalmode.hmi.pag2pg35.TerminalModeScreenBag1`
- `de.esolutions.hmi.widgets.pgen2.utils.WidgetUtilsPGen2`

The full-screen JAR itself does not contain image assets. It changes layout/behavior through compiled classes.

The downloaded component did not contain `uninstall.txt`, so the ModKit would treat it as an install operation.

## How this ModKit works

This repository is a QNX/PCM5 modification framework, not the full-screen patch itself.

High-level execution flow:

1. The update manifests copy the ModKit payload into `/mnt/persist_new`.
2. `modkit.sh` locates the update media, mounts `/mnt/app`, displays a splash screen, temporarily installs `fecswap`, and invokes `modkit_install.sh`.
3. The installer discovers components under `Mods/` and runs their `Update`, `Post`, and `Persist` scripts when present.
4. Persistence is achieved by replacing `servicemgrmibhigh` with a wrapper and renaming the original executable to `servicemgrmibhigh0`.
5. `modkit_persist.sh` handles the failsafe and runs the `Post` and `Persist` stages.

Important risk notes:

- Mod components execute shell scripts as root on the PCM.
- The persistence mechanism changes a boot-time service executable.
- The framework does not provide a simple self-uninstaller for all of its own persistent changes.
- `fecswap` is a stripped ARM binary and is therefore opaque without deeper reverse engineering.
- Firmware-specific JAR selection is safety-critical. A generic fallback should not be preferred over the exact `POG35_P2656` build.
- A likely shell bug was noticed around `modkit_install.sh:83`: a wildcard appears to be quoted, which may prevent expansion depending on the exact shell expression. Re-inspect before relying on that path.

## What the photographs established about the UI

Fifteen photographs were supplied showing the stock PCM UI across home, radio, vehicle, sound, devices, climate, applications, settings, Sport Chrono, and CarPlay-overlay screens.

The UI characteristics visible in the photographs include:

- Wide G35 display layout.
- Persistent top status bar and left-side navigation rail.
- Large, flat, translucent gray button/tile surfaces.
- Brighter gray selected states.
- Red/pink underline accents for selected controls.
- White icons and typography.
- Vehicle-specific 3D/background images.
- Multiple UI states for selected, unselected, available, disabled, and active controls.
- Stock Porsche widgets can remain visible beside the partial-width CarPlay projection, which is exactly what the full-screen patch changes.

The user's design intent evolved during the conversation:

- Initially: investigate replacing the stock Porsche icon set.
- Current intent: focus primarily on changing button backgrounds, tiles, panels, selection treatments, and other UI surfaces; icons may remain stock.

The original photograph files were temporary chat attachments under `/var/folders/.../addy-chat-...jpg`. They are not part of this repository and will not transfer to the new machine. Export or reattach them if photographic references are still needed.

## Likely location and format of stock UI resources

The most relevant resource root is expected to be:

`/mnt/app/eso/hmi/lsd/Resources/`

Likely skin-specific files and directories include:

- `skin*/images.mcf`
- `skin*/images.res`
- `skin*/ambienceColorMap.res`
- `skin*/info.txt`
- `skin*/viewhandler.zip`
- View definitions and possibly fonts or palette/configuration files located alongside them.

Related VW Group tooling supports this hypothesis. The M.I.B. ImageMod uses:

`IMGDIR="/net/mmx/mnt/app/eso/hmi/lsd/Resources"`

and works with `skin*/images.mcf`, `images.res`, `viewhandler.zip`, and palettes:

- `https://github.com/Mr-MIBonk/M.I.B._More-Incredible-Bash/blob/main/apps/ImageMod`

Potential MCF extraction/recompression tooling exists in:

- `https://github.com/jilleb/mib2-toolbox`
- Notable scripts: `extract-mcf.py` and `compress-mcf.py`

Do not assume those scripts support this exact MH2p MCF variant until they have been tested against a copied stock file. Preserve stock hashes and never experiment on the only copy.

## Read-only PCM dump: required first step

Build a ModKit component, tentatively named `00_Dump_PCM_UI`, that runs before any modifying components. It should only read from the PCM and write to the SD card.

It should collect at least:

- `/mnt/app/eso/hmi/lsd/Resources/`
- `/mnt/app/eso/hmi/lsd/jars/`
- `/mnt/app/eso/hmi/lsd/lsd.jar`
- Relevant launcher/classpath scripts and HMI configuration files.
- A recursive directory/file manifest.
- File sizes and timestamps where available.
- SHA-1 and/or SHA-256 hashes when tools are available on the PCM; otherwise calculate hashes immediately on the computer after retrieval.
- The exact detected release string and hardware/display identifiers.

Suggested output layout on the SD card:

```text
PCM_Dump/
  MH2p_US_POG35_P2656/
    Resources/
    jars/
    lsd.jar
    manifests/
    system-info.txt
```

Dump component constraints:

- Abort clearly if the detected release is not `MH2p_US_POG35_P2656`, unless the mismatch is intentionally acknowledged later.
- Do not write, rename, delete, or patch anything under `/mnt/app`.
- Do not install a persistent hook for the dump itself.
- Run during the `Update` phase because the media path is available there.
- Use conservative QNX-compatible shell commands.
- Detect hash utilities instead of assuming GNU tooling exists.
- Use a lexically early component name so it captures stock files before another mod can change them.
- Record failures and missing paths in the dump output rather than silently continuing.
- Estimate required space before copying if practical.
- Keep the first untouched dump permanently and make additional working copies for extraction.

The dump should happen before installing full-screen CarPlay or any theme modification. The CarPlay mod only adds `fc.jar`, but capturing a pristine baseline avoids ambiguity.

## Local emulator/simulator conclusion

A useful local PCM **UI simulator/theme previewer** is feasible and worth building after the dump.

It can:

- Use the exact G35 dimensions discovered from PCM configuration rather than estimating from photographs.
- Load extracted stock assets.
- Recreate the navigation rail, top bar, panels, buttons, and major screens.
- Preview normal, selected, pressed, disabled, and active states.
- Compare original and modified themes side-by-side.
- Perform pixel/screenshot comparisons.
- Export correctly sized replacement resources and eventually a reversible ModKit theme package.

A true full-system PCM emulator is a different and much harder problem. The production software depends on:

- QNX Neutrino on an ARM target.
- Proprietary graphics/runtime libraries.
- PCM hardware and drivers.
- Vehicle services such as CAN and MOST.
- Potentially licensed QNX components.

QEMU may be useful for isolated research if a complete legal firmware image and boot configuration are available, but the actual Porsche HMI is likely to fail when hardware-backed services are absent. Therefore, build the visual/behavioral simulator first. A small JVM harness could potentially execute isolated pure-Java classes with stubs, but should not be treated as a complete PCM emulator.

The simulator is a theme-development and regression-testing environment, not a substitute for final in-car testing.

## Proposed development sequence

1. Implement and review `00_Dump_PCM_UI`.
2. On an unmanaged computer, prepare compatible FAT32 media and copy the ModKit plus dump component.
3. Run the read-only dump on the PCM before installing other mods.
4. Copy the resulting dump to the development machine twice: one immutable archive and one working copy.
5. Hash and inventory every extracted file.
6. Test MCF extraction against copies and build an asset catalog with dimensions, formats, transparency, and probable UI use.
7. Decompile relevant JARs and inspect `viewhandler.zip`/configuration to map assets to screens and states.
8. Build a browser-based G35 UI simulator with exact dimensions and stock resources.
9. Implement custom button/panel treatments in the simulator and create visual regression snapshots.
10. Build a reversible theme installer that backs up originals and has an explicit uninstaller.
11. Review every PCM write operation and test only the exact `MH2p_US_POG35_P2656` package in the vehicle.

## SD-card problem and blocker

The original computer could not retain a newly inserted 256 GB SDXC card. The card appeared briefly in Disk Utility and was immediately ejected. Hardware enumeration showed the built-in reader/card as `LSUSD00`, manufacturer `0xad`, and media type SDXC, so it was not simply an absent reader. Changing macOS accessory-authorization settings did not resolve the final behavior.

System logs showed `SystemUIServer` requesting unmount/eject. The root cause was an organization-managed macOS profile, not the ModKit and not necessarily the card:

- Mac is managed through Rippling/Hadrius MDM.
- Installed profile: `Hadrius - Block Removable Media`
- Profile identifier: `com.hadrius.mdm.blockmedia`
- A `com.apple.systemuiserver` mount-controls policy denies/ejects external media.

Do not attempt to bypass company MDM. Safe options are:

- Use a personal/unmanaged computer.
- Ask IT for an authorized temporary exception.

The 256 GB SDXC card likely ships as exFAT. The PCM/ModKit is expected to require FAT32-compatible media, so the card must be deliberately formatted on the unmanaged computer. Confirm partition scheme and PCM compatibility before use. A smaller known-good SD card may reduce compatibility risk.

## Safety and recovery principles

- Treat every PCM modification as potentially capable of making the infotainment system unbootable.
- Keep an immutable, hashed stock dump.
- Never replace a stock file without preserving its exact original and permissions.
- Make every theme installer firmware-specific and reversible.
- Provide a tested uninstaller before installing the corresponding customization.
- Change as few files as possible.
- Do not mix the read-only dump operation with an install operation.
- Maintain stable vehicle power while the PCM is applying an update.
- Do not interrupt an update or remove the card while files are being written.
- The first in-car custom-theme test should change a small, non-critical visual resource rather than an entire skin.

## Important source links

- ModKit site: `https://lawpaul.github.io/MH2p_SD_ModKit_Site/`
- Original ModKit repository: `https://github.com/LawPaul/MH2p_SD_ModKit`
- This fork: `https://github.com/morganknutson/MH2p_SD_ModKit`
- Full-screen CarPlay mod: `https://github.com/LawPaul/MH2p_CarPlay_FullScreen`
- M.I.B. ImageMod reference: `https://github.com/Mr-MIBonk/M.I.B._More-Incredible-Bash/blob/main/apps/ImageMod`
- MIB2 Toolbox / MCF tools: `https://github.com/jilleb/mib2-toolbox`
- Apple accessory security information used during SD troubleshooting: `https://support.apple.com/en-us/102282`

## Resume checklist for the other machine

```sh
git clone https://github.com/morganknutson/MH2p_SD_ModKit.git
cd MH2p_SD_ModKit
git remote add upstream https://github.com/LawPaul/MH2p_SD_ModKit.git
git fetch --all
```

Then:

1. Read this file completely.
2. Obtain or copy the full-screen mod bundle if needed; it is not yet committed here.
3. Confirm the card can be mounted and formatted as FAT32 on the unmanaged computer.
4. Implement the read-only dump component before beginning simulator or theme work.
5. Preserve the exact release constraint: `MH2p_US_POG35_P2656`.

## Current implementation state

Completed:

- Repository architecture and execution flow investigated.
- Downloaded full-screen bundle compared against upstream.
- Exact PCM software/hardware strings captured.
- Exact full-screen CarPlay JAR identified.
- Likely UI resource paths and existing tooling identified.
- SD-card failure diagnosed as MDM policy.
- Project fork created and local remotes configured.
- Strategy selected: dump first, then simulator, then reversible theme package.

Not yet implemented:

- Read-only PCM dump component.
- Full-screen component copied into this fork.
- Extracted PCM resource archive.
- MCF extraction validation.
- UI asset catalog.
- Local simulator.
- Custom button/background theme.
- Theme installer/uninstaller.

The next coding action should be the smallest possible, read-only `00_Dump_PCM_UI` component, followed by a careful code review before placing it on vehicle media.
