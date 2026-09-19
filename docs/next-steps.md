# What to try next

## Lowest-risk next steps

1. Repeat the fastboot trigger and collect a fresh, redacted `getvar all` transcript.
2. Capture the exact Guide screen shown after `fastboot reboot recovery`; document whether it says “No command,” shows a recovery menu, or simply boots Android.
3. Test the physical camera privacy slider and microphone switch independently, with the USB monitor running, to narrow the actual boot-mode trigger.
4. Obtain a known-good Peloton Guide/RE01 remote or a BLE capture from one. That is the most direct way to learn the expected pairing and report protocol.

## EDL: what it would and would not provide

Qualcomm Emergency Download Mode is implemented below Android and the normal bootloader. A device normally appears as Qualcomm HS-USB QDLoader `9008`. A host then uses the Sahara protocol to transfer a device-specific, OEM-signed programmer; that programmer implements Firehose storage commands.

EDL could potentially allow:

- Identifying the Qualcomm target and storage layout.
- Reading partitions, if the programmer permits reads.
- Restoring exact stock partitions, if an authentic Peloton package and authorized programmer are available.
- Diagnosing a boot-chain failure below fastboot.

EDL would not automatically provide:

- An Android shell.
- ADB.
- A generic way around secure boot.
- A safe way to flash arbitrary images.

The Guide currently reports a secure, locked user build and `QCS EMMC`. The critical prerequisite is therefore not merely entering EDL; it is finding the exact signed programmer and matching PT01 firmware. Qualcomm's [QIL guide](https://github.com/qualcomm/qcom-image-loader/blob/main/doc/User_Guide.md) and [QDL documentation](https://github.com/96boards/documentation/blob/master/consumer/guides/qdl.md) describe these requirements.

## EDL risk boundary

Entering EDL and only checking USB enumeration is a probe. Sending a programmer or Firehose commands is a separate risk. Do not erase or write GPT, bootloader, `persist`, modem, Bluetooth, or calibration partitions without a verified device-specific recovery package and a backup plan.

No EDL entry, programmer transfer, erase, or flash was performed during this investigation.

## Avoid these assumptions

- A `tiger` USB node does not imply ADB.
- Fastboot availability does not imply bootloader unlock availability.
- `fastboot reboot recovery` being accepted does not prove that recovery ADB is running.
- A standard BLE keyboard/consumer HID profile is not proof of compatibility with the Peloton remote.
- A nearby name beginning with `PLTN-` is not proof that it is this Guide or its remote.
