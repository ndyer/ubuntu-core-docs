---
myst:
  html_meta:
    description: Troubleshoot common Ubuntu Core issues. Find solutions for TPM, installation, and configuration problems.
---

(how-to-guides-manage-ubuntu-core-troubleshooting)=
# Troubleshooting

Ubuntu Core runs on, and can be built for, a diverse and constantly evolving set of {ref}`platforms and devices <reference-system-requirements>`.

The majority of our users and developers experience very few issues, but any technology this complex and diverse will likely encounter some issues and incompatibilities.

This page attempts to guide users to either an appropriate solution to their issues, or the correct forum/thread where they can get help. 

## Ubuntu Core install error: TPM is in DA Lockout Mode

Installing Ubuntu Core on a device with a TPM (such as an Intel NUC, or QEMU with emulated TPM) can sometimes result in a stalled installation and a **TPM is in DA Lockout Mode** error, as shown in the following example install log:

```bash
ubuntu snapd[15531]: handlers install.go:254:
   make system runnable
ubuntu snapd[115531]: secboot_tpm.go:483: 
   TPM provisioning error: the TPM is in DA lockout mode
ubuntu snapd[115531]: taskrunner.go:271:
   [change 2 "Setup system for run mode" task] failed: 
   cannot make system runnable: cannot seal the encryption keys:
   cannot provision TPM: the TPM is in DA lockout mode
ubuntu snapd[15531]: secboot_tpm.go:483: TPM provisioning error:
   the TPM is in DA lockout mode
ubuntu snapd[15531]: taskrunner.go:271:
   [change 2 "Setup system for run mode" task] failed:
   cannot make system runnable:
   cannot seal the encryption keys:
   cannot provision TPM:
   the TPM is in DA lockout mode 
```

This error typically means the TPM has been locked to protect the system against potential dictionary attacks (DA) and the TPM needs to be cleared before the Ubuntu Core installation will proceed.

To clear the TPM on hardware, boot a classic Ubuntu system (such as a live version of [Ubuntu 20.04 LTS](https://releases.ubuntu.com/20.04/) from USB storage) and run the following command from a terminal:

```bash
echo 5 | sudo tee /sys/class/tpm/tpm0/ppi/request
```

To clear a software TPM, such as the [test-snapd-swtpm](https://snapcraft.io/test-snapd-swtpm) snap, remove it and re-install it again:

```bash
snap remove test-snapd-swtpm --purge; snap install test-snapd-swtpm
```

Now reboot the problematic system and re-attempt the Ubuntu Core installation, which should  continue without error.

## Console-conf shows no-ip

During a {ref}`snap refresh <explanation-refresh-control>`, _console-conf_ may display an `no-ip` message.

Despite the _no-ip_ message, you should still be able to connect to the device using SSH if you actually know the IP.

The [snap changes](https://snapcraft.io/docs/keeping-snaps-up-to-date#heading--changes) command will show that one or more snaps are being updated and the device may need to reboot.

The solution to the _no-ip_ error is to simply wait for any updates to complete.

## Ubuntu Core boot asking for recovery key

When using {ref}`Full Disk Encryption <explanation-full-disk-encryption>`, a device’s Trusted Platform Module (TPM) stores the encryption keys necessary to decrypt and boot the device.

If an encrypted drive is detected, but the TPM does not contain a valid key, the Ubuntu Core boot process will prompt for a recovery key. 

```bash
🔐 Please enter the recovery key for disk /dev/disk/by-partuuid/c7f7971b: (press TAB for no echo)
```

To progress from this point, you will need to enter a  previously retrieved recovery key for the device.

See {ref}`Using recovery keys <how-to-guides-manage-ubuntu-core-use-a-recovery-mode>` for further details.

(troubleshooting-no-sealed-keys)=
## No sealed keys

`error: no sealed keys`

The device's `ubuntu-data` partition is encrypted but no sealed key blob was found on `ubuntu-seed`. This usually means the installation did not complete sealing, or the seed partition has been wiped or corrupted.

Recovery steps:

1. Boot into {ref}`recover mode <how-to-guides-manage-ubuntu-core-use-a-recovery-mode>` from the recovery system on `ubuntu-seed`.
2. Verify that `/run/mnt/ubuntu-seed/device/fde/` contains files matching `*.sealed-key`.
3. If the files are missing, the device must be re-installed from a recovery system. The encrypted data partition cannot be recovered without either a sealed key or the recovery key.

(troubleshooting-fde-unlock-failed)=
## FDE unlock failed

`error: kernel key not found`, or `error: cannot find supported FDE key protector`.

The boot process found a sealed key but could not use it to unlock the encrypted partition. Causes include a damaged TPM, a measured-boot policy change (e.g. firmware update that invalidates PCR values), or a system where the FDE key protector hook is missing.

Recovery steps:

1. The most reliable next step is to enter the **recovery key** when prompted (see {ref}`Ubuntu Core boot asking for recovery key <troubleshooting-no-sealed-keys>` above).
2. If the cause was a firmware/kernel update, re-sealing the keys against the new measurements may resolve the issue.
3. If the device repeatedly fails to unlock with both the sealed key and the recovery key, suspect TPM lockout (see {ref}`Ubuntu Core install error: TPM is in DA Lockout Mode <how-to-guides-manage-ubuntu-core-troubleshooting>` above).

(troubleshooting-keyslot-missing)=
## Keyslot missing

`error: key slot reference ... not found`

A LUKS2 keyslot expected by snapd is not present on the encrypted volume. This typically follows a partial re-key operation that was interrupted, or manual LUKS administration that removed a slot snapd was still tracking.

Recovery steps:

1. Boot into recover mode and inspect the volume with `cryptsetup luksDump /dev/<part>`.
2. If the recovery-key slot is still present, unlock with that and re-add the run-time keyslot via the snapd FDE state API.
3. If no usable slot remains, the data is lost and the device must be re-installed.

(troubleshooting-no-recovery-system)=
## No recovery system

`error: recovery system does not exist`, or `error: no systems seeds`

The bootloader was asked to enter a recovery system label that is not present on `ubuntu-seed`, or no recovery systems are seeded at all.

Recovery steps:

1. Check the list of available recovery systems at the bootloader menu (typically by booting and selecting **Recover**).
2. If the desired recovery system is missing, it may have been removed; pick a different one.
3. If *no* recovery systems are present, the seed partition is likely corrupted and the device must be re-imaged.

(troubleshooting-recovery-mode-unsupported)=
## Recovery mode unsupported

`error: system mode is unsupported`

The device is running an Ubuntu Core release older than UC20 and does not support the modern recovery-system workflow. The QR code on a pre-UC20 device should never link here; if it does, the device firmware or snapd version is unexpected.

Recovery steps:

1. Confirm the Ubuntu Core version with `snap version`.
2. Upgrade to a supported UC20+ release using the {ref}`upgrade guide <how-to-guides-manage-ubuntu-core-upgrade-ubuntu-core>`.

(troubleshooting-auth-quality)=
## Auth quality

`error: calculated entropy (... bits) is less than the required minimum entropy ...`

A passphrase or PIN supplied for FDE volume authentication did not meet the entropy minimum. The volume is not unlocked.

Recovery steps:

1. Re-enter authentication with a longer or more complex value.
2. If using a numeric PIN, ensure it has at least 6–8 digits.
3. If you have lost the passphrase, fall back to the recovery key.

