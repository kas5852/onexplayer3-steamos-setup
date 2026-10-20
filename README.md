# ONEXPLAYER 3 game pack: test guide

Faster Final Fantasy VII Rebirth, Cyberpunk 2077 and Resident Evil 4 on the ONEXPLAYER 3 under SteamOS or another Linux. This is a test build. Please try it and report your numbers in the Discord with the `--check` output from step 4.

Measured on one ONEXPLAYER 3 at 30 W, same scene and game settings before and after:

| Game | Before | With the pack | Scene |
|---|---|---|---|
| Final Fantasy VII Rebirth | 52.6 fps, 1% low 39.7 | **72.7 fps, 1% low 58.6** | Gongaga field, 1080p, TAA. A ROG Xbox Ally X (Z2 Extreme) does 55.5 fps here at 30 W |
| Cyberpunk 2077 | 59.0 fps | **79.3 fps** | Built-in benchmark, Steam Deck preset |
| Resident Evil 4 | 89.8 fps | **95.4 fps, 1% low 70.5** | Chapter 6, needs the power profile in step 3.4 |

The pack is patched Intel Vulkan driver builds, a one-line vkd3d-proton fix and per-game settings that keep the games off the four LP E-cores without L3 cache. It installs into your home folder without sudo, changes no system files and `--revert` removes it.

**Where to start:**
- **Still on Windows:** start at step 1.
- **Already on SteamOS:** start at step 2. If you already ran the fix pack, re-run it anyway to get the latest version.
- **Another distribution** (Bazzite, CachyOS, Arch, ...) **with native Steam:** skip to step 3. You need `python3` and a TDP tool. Flatpak Steam is untested and the installer refuses it.

> **Tip: let an AI agent do the typing.** I highly recommend having an AI coding agent on your PC walk you through this by SSH-ing into the OXP3 for you. That's how this pack was built and tested. It can run the steps, read the `--check` output, fix what breaks and run the benchmarks while you hold the device. To let it in, set a password on the OXP3 (step 1.12 or `passwd` in Konsole), then in Konsole run `sudo systemctl enable --now sshd` and `ip -4 addr` to get its address. Your agent connects with `ssh deck@<that address>` from a PC on the same Wi-Fi. When you're done, `sudo systemctl disable --now sshd` turns SSH off again.

## Step 1 - Install SteamOS (skip if you already run SteamOS or another Linux)

This is the recovery-image method that works on this machine. You need a Windows PC with Rufus, a USB drive, and later a temporary network connection such as USB tethering from a phone.

> **Warning:** `repair_device.sh all` reinstalls SteamOS on the internal drive and erases what is on it. Back up first.

> **Why the TTY:** the graphics driver in the recovery image does not work properly on this machine. Unless you switch to the TTY, the screen is flooded with error messages continuously. That is why the extra kernel parameters are added below and why several steps are done from the TTY.

**Prepare the USB drive**

1. Download `steamdeck-oobe-repair-20260707.10-3.8.14.img.bz2` from https://steamdeck-images.steamos.cloud/recovery/.
2. Flash the image to a USB drive with Rufus.
3. When flashing is complete, open `/EFI/steamos/grub.cfg` on the USB drive.
4. At the end of line 117, append these parameters: `nomodeset 3 pci=noaer modprobe.blacklist=rtsx_pci`

   It should look something like this:

   ```
   steamenv_boot linux /boot/vmlinuz-linux-neptune-616 console=tty1 rd.luks=0 rd.lvm=0 rd.md=0 rd.dm=0 rd.systemd.gpt_auto=no log_buf_len=4M amd_iommu=off amdgpu.lockup_timeout=5000,10000,10000,5000 ttm.pages_min=2097152 amdgpu.sched_hw_submission=4 amdgpu.dcdebugmask=0x20000 audit=0 fbcon=vc:4-6 fsck.mode=auto fsck.repair=preen  loglevel=3 quiet splash plymouth.ignore-serial-consoles nomodeset 3 pci=noaer modprobe.blacklist=rtsx_pci
   ```

**Install**

5. Insert the USB drive and boot from it. The system boots into the TTY interface automatically.
6. Type `deck` to log in.
7. Enter the following commands:

   ```bash
   cd ./tools
   sed -i 's/zenity/true/g' ./repair_device.sh
   sed -i 's/kdialog/true/g' ./repair_device.sh
   sudo ./repair_device.sh all
   ```

   **Installing to the second SSD (Mini SSD) instead.** `repair_device.sh` always installs to the disk named in its `DISK=` line near the top (`nvme0n1`) and erases it. To install to the other SSD, run `lsblk -o NAME,SIZE,MODEL` first and note the name of the target disk, then before `sudo ./repair_device.sh all` change that line, for example `sed -i 's|nvme0n1|nvme1n1|g' ./repair_device.sh`, and check it with `grep -n nvme ./repair_device.sh`. The disk numbering can differ between boots and machines, so go by size and model, not by the number. (Tip from issue #13.)

8. The installation begins. After a moment the machine shows the OneXPlayer logo and appears to be completely frozen. This state lasts a long time. Do **not** touch or operate the machine. Wait until the USB drive's read/write indicator light turns off completely; this took about half an hour.
9. Once the light is off, press Ctrl+Alt+F4 to return to the TTY and type `reboot`.
10. You may see a massive amount of error logs flooding the screen. Do not panic, just press and hold the power button to force a shutdown.
11. Remove the USB drive and power on the device. Wait a moment at the OneXPlayer logo screen.

**First boot**

12. Press Ctrl+Alt+F4 to enter the TTY, type `deck` to log in, then type `passwd` to set your password.
13. Connect to the internet, for example with USB tethering from your phone. You can ping google.com to confirm the connection is active.
14. Run:

    ```bash
    sudo steamos-select-branch main
    sudo steamos-update
    ```

15. Wait for the system update to complete, then reboot. The first boot takes a while.
16. You should hear the familiar Steam startup chime. Connect an external monitor and you will enter the system.

## Step 2 - Run the fix pack (SteamOS)

The fix pack repairs what doesn't work out of the box on SteamOS (suspend, the OLED panel, keys, audio, the GPU power limit and more). Its TDP sync is part of the numbers above. In Desktop Mode open Konsole and run:

```bash
cd ~ && curl -fL https://github.com/HHHHanasak1/onexplayer3-steamos-setup/releases/latest/download/oxp3-fix.tar.gz | tar -xz
~/oxp3-fix/oxp3-apply-fixes.sh
```

It asks for your sudo password and tells you if a reboot is needed. Answer **yes** when it asks about SimpleDeckyTDP-Intel (needs [Decky Loader](https://decky.xyz) installed first) since step 3 uses it for the per-game power profiles. Re-run it after every SteamOS update. Everything it does and fixes is listed in the [main repository](https://github.com/HHHHanasak1/onexplayer3-steamos-setup).

## Step 3 - Install the game pack

Set the three games to **Proton Experimental** first (game > ⚙ > Properties > Compatibility). That's what the numbers used.

**1. Download and run the installer.** In Desktop Mode open Konsole and run:

```bash
cd ~ && curl -fL https://github.com/kas5852/onexplayer3-steamos-setup/releases/download/game-pack-20261020/oxp3-game-pack.tar.gz | tar -xz
~/oxp3-game-pack/oxp3-game-pack.sh
```

This creates `~/oxp3-game-pack`. Keep that folder since the games load the driver from it. The installer tests each driver build and only uses the ones that load. It writes the per-game settings to `~/.config/oxp3-game-pack/` and copies two vkd3d-proton DLLs into the Cyberpunk and RE4 folders, keeping any file it replaces as `*.oxp3-pack-bak`.

**2. Paste the launch options.** Steam > game > ⚙ > Properties > General > Launch Options:

| Game | Launch option |
|---|---|
| Cyberpunk 2077 | `~/oxp3-game-pack/oxp3-run %command% --launcher-skip -skipStartScreen` |
| Resident Evil 4 | `~/oxp3-game-pack/oxp3-run %command%` |
| Final Fantasy VII Rebirth | `~/oxp3-game-pack/oxp3-run %command%` |

If a game already has launch options, put your own variables in front, for example `MANGOHUD=1 ~/oxp3-game-pack/oxp3-run %command%`.

**3. Switch back to Game Mode.**

**4. Set a per-game power profile.** Start the game, open SimpleDeckyTDP, turn on the per-game profile and set:

| Game | TDP | GPU frequency | EPP | CPU Boost |
|---|---|---|---|---|
| Resident Evil 4 | 30 W | **Range, 650-2200 MHz** | **performance** | Smart |
| Final Fantasy VII Rebirth | 30 W | default | **performance** | default |
| Cyberpunk 2077 | 30 W | default | balance_performance | default |

EPP is in the plugin's advanced options. RE4's GPU cap matters most. Without it the faster driver runs the GPU into the power limit and the 1% low drops from about 70 to 58.

**5. Play.** The first launch of each game rebuilds its shaders with the new driver. Expect a longer load and some stutter for the first few minutes before you judge anything.

## Step 4 - Check it and report back

```bash
~/oxp3-game-pack/oxp3-game-pack.sh --check
```

This prints the device, BIOS, OS, which driver builds load, the TDP sync state and for each game whether the settings, DLLs and launch option are in place. `pack=on` after the last launch means the game ran with the pack. Paste this output with your numbers (scene, TDP, in-game preset) when you report results.

To compare against stock, put `OXP3_PACK=0` at the front of a game's launch option and run the same scene again. The easiest comparison is Cyberpunk's built-in benchmark: Steam Deck preset, 30 W, performance overlay off, ignore the first run after a change.

## Updates, problems and removal

- **SteamOS update:** usually nothing changes. If a driver build stops loading, that game falls back to the system driver on its own and `--check` shows the driver as `FAILED`. Re-run the fix pack as usual.
- **Game update or Verify integrity** can remove the DLLs. `--check` shows them as missing and re-running the installer puts them back.
- **A game won't start:** set its launch option to just `%command%`, then post the `--check` output.
- **Missing grass, flicker or black patches in FF7R:** post a screenshot and the spot. Put `OXP3_PACK=0` in front of the launch option to keep playing meanwhile.
- **Remove it:** run `~/oxp3-game-pack/oxp3-game-pack.sh --revert`, which puts back any DLL it replaced and deletes the settings and the pack's shader caches. Then delete the `oxp3-run` part from each launch option.

## More detail

Where each gain comes from, the per-game settings, XeSS on the XMX units, troubleshooting, the build sources and the patches: [game-pack/README.md](game-pack/README.md).
