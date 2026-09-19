# Findings log

Observation date: 2026-09-18. Host: Apple Silicon Mac running macOS 26.3, build 25D125.

## 1. Hardware identity

The product label identifies the unit as:

- Peloton Guide
- Model: `PT01`
- Made in Taiwan
- The label serial was correlated with the USB serial. The full serial is visible in the committed label image by explicit owner request.

The device photograph is [evidence/guide-device.png](../evidence/guide-device.png), and the committed label is [evidence/offline-remote-reconnect.jpeg](../evidence/offline-remote-reconnect.jpeg). The filename is retained so the existing public URL continues to work, although the image now shows the label rather than the earlier screen photograph.

## 2. USB enumeration

With the Guide connected over USB, macOS saw:

| Field | Observed value |
| --- | --- |
| USB vendor | Peloton Interactive LLC |
| USB product | `tiger` |
| Vendor ID | `0x317e` (decimal 12670) |
| Product ID | `0x4ee1` (decimal 20193) |
| USB speed | 480 Mbps |
| Device class | `0` |
| Configurations | `1` |
| Serial | redacted in this public report |

The Mac USB host and cable path are therefore working. In the ordinary Android state, the USB node did not expose an ADB interface and `adb devices -l` remained empty.

## 3. ADB status

ADB was already installed on the Mac:

~~~
Android Debug Bridge version 1.0.41
Version 37.0.1-15733141
~~~

Repeated checks produced no device:

~~~
$ adb devices -l
List of devices attached
~~~

No `device`, `unauthorized`, `recovery`, or `sideload` state was observed. This means the limiting factor was the Guide's current USB/debug configuration, not the Mac's ADB installation.

## 4. Fastboot discovery

During a controlled experiment, the microphone switch was toggled while plugging in USB. The Guide showed a purple/bootloader state and fastboot appeared:

~~~
QATCAS...       fastboot usb:17825792X
~~~

The full hardware serial is intentionally shortened above. Use `fastboot devices -l` on the actual unit rather than copying a public identifier.

Confirmed read-only values:

~~~
product: tiger
unlocked: no
current-slot: a
slot-count: 2
secure: yes
variant: QCS EMMC
kernel: uefi
max-download-size: 805306368
~~~

A fresh read-only pass on 2026-09-18 re-confirmed the fastboot target and the
same key values. The target serial was used locally to avoid addressing the
wrong device, but is redacted from this report. No write, erase, unlock, or
flash command was issued during this pass.

The partition table exposed A/B `boot`, `system`, `vendor`, `vbmeta`, `dtbo`, `abl`, Qualcomm firmware, and other partitions. The boot and vendor/system partitions are large enough that blind flashing would be especially risky.

## 5. Unlock and recovery tests

The bootloader reported:

~~~
(bootloader) get_unlock_ability: 0
(bootloader) Verity mode: true
(bootloader) Device unlocked: false
(bootloader) Device critical unlocked: false
(bootloader) Factory Mode: No
(bootloader) Has Debug block: No
~~~

An unlock request was then made only after explicit authorization, using the observed fastboot serial as the target:

~~~
FAILED (remote: 'unlock is not allowed for user build')
fastboot: error: Command failed
~~~

The device remained in fastboot, still reported `unlocked: no`, and no wipe occurred.

This command was also tested:

~~~
fastboot reboot recovery
~~~

Fastboot accepted the request, but the device later followed the normal Android boot path. No recovery ADB endpoint appeared. The device does not expose a conventional separate `recovery` partition in the observed fastboot partition list, so this result is consistent with recovery being integrated into another boot image or the target being ignored by this bootloader.

Fastboot's `fetch` command was tested read-only against `boot_a`; the bootloader rejected it:

~~~
fastboot: error: Unable to get max-fetch-size. Device does not support fetch command.
~~~

No image was retrieved or written.

## 6. Qualcomm EDL probe

Two non-writing software entry attempts were made while the verified Guide was in fastboot:

```text
$ fastboot reboot edl
fastboot: usage: unknown reboot target edl

$ fastboot oem edl
FAILED (remote: 'unknown command')
fastboot: error: Command failed
```

The first command was rejected by the host fastboot client before transmission. The second reached the bootloader and was rejected by the device. After both tests, the Guide remained in fastboot, no Qualcomm `9008`/QDLoader markers appeared in macOS USB enumeration, and `adb devices -l` remained empty.

No EDL loader, Sahara handshake, Firehose command, partition read, partition write, erase, or flash was attempted.

## 7. Screen state

The Guide displayed two separate states in the supplied photograph:

~~~
Press any button on the remote
To reconnect, press and hold [Up] and [Menu] for 4 seconds

You're Offline
Please check your connection and try again.
OPEN NETWORK SETTINGS
~~~

This is a remote-reconnect overlay plus an offline-network message. It is not evidence that ADB is enabled, and it is not the same thing as a generic Android Bluetooth pairing dialog.

The manual's stated remote sequence is an instruction for a compatible Peloton remote; it was not treated as proof that a generic keyboard/HID emulator would work.

## 8. Bluetooth experiments

The Mac had Bluetooth enabled. A nearby device named `PLTN-TCAV1` was seen by a local Bluetooth inquiry, but its identity was not proven and it was not paired.

A generic macOS Bluetooth HID emulator was built from the open-source `darwin-bt-remote` project. It advertised a standard HOGP-style keyboard/consumer-control profile as `BTRemote` in Low Energy mode. No Guide connection or subscription was observed, and sending a generic Up button produced no visible Guide response.

The emulator's Classic mode was also selected. Its pairing dialog did not show the Guide as a pairable target, and no paired devices were available. The emulator's own UI says Classic mode is for Android/Linux/ChromeOS targets, while the Peloton manual describes the Guide remote as Bluetooth Low Energy. This mismatch is unresolved.

No exact Peloton RE01 GATT/advertising protocol was recovered. A standard HID profile is therefore not enough evidence to emulate the original remote.

## 9. What was not done

- No partition was erased.
- No partition was flashed.
- No bootloader unlock succeeded.
- No EDL transition was observed; the two software probes were rejected.
- No custom boot image was supplied or booted.
- No factory reset was intentionally initiated.
- No microphone, camera, or user audio was recorded.

## 10. Firmware and programmer hunt

A second-pass search was performed on 2026-09-18. It covered the local
Downloads directory, this repository, public GitHub repository and code search,
and public web results for PT01, `tiger`, Peloton Guide firmware/OTA packages,
Qualcomm Firehose programmers, and the observed identifiers.

No exact PT01/`tiger` firmware image, OTA package, signed programmer, or
Firehose loader was located. This is a bounded negative result, not proof that
Peloton has never distributed one privately or through an update session.

The [FCC PT01 filing](https://fccid.io/2AA3N-PT01) is useful for hardware identity
and regulatory/internal-photo references, but did not provide a firmware
package. The community [OpenPelo project](https://github.com/doudar/Openpelo)
provides Android/ADB, wireless-debugging, and app-management guidance for
devices where debugging can already be enabled; it is not an exact PT01
firmware or recovery bundle, and its source did not contain a `tiger`, PT01,
Firehose, or matching firmware package.

Nothing was downloaded, executed as a device programmer, or flashed as part of
this search. The most promising firmware sources remain an OTA capture from a
working Guide or a package obtained from another matching PT01 unit.
