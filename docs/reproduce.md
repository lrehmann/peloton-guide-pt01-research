# Reproduction runbook

This runbook starts with read-only checks and records state transitions. Replace the placeholder serial with the value printed by the connected device; do not publish a device serial in issue reports unless it is redacted.

## Host prerequisites

The original investigation used:

- macOS 26.3, build 25D125
- Apple Silicon / arm64
- Android platform-tools 37.0.1-15733141
- `adb` at `/opt/homebrew/bin/adb`
- `fastboot` from the same platform-tools installation

## Normal USB and ADB check

~~~
adb version
adb devices -l
fastboot devices -l

# macOS USB descriptors
ioreg -p IOUSB -l -w 0 | rg -A35 -B5 'USB Product Name" = "tiger'
system_profiler SPUSBDataType
~~~

Expected ordinary-state result from this unit:

~~~
adb devices -l
List of devices attached
~~~

Do not interpret an empty ADB list as a cable failure if macOS still sees the `tiger` USB device.

## Continuous state monitor

This prints only transitions among USB presence, ADB, and fastboot:

~~~
last_state=''
while true; do
  adb_state=$(adb devices -l 2>/dev/null | awk 'NR > 1 && NF {print}' | tr '\\n' ';')
  [[ -n "$adb_state" ]] || adb_state='none'

  fastboot_state=$(fastboot devices 2>/dev/null | awk 'NF {print}' | tr '\\n' ';')
  [[ -n "$fastboot_state" ]] || fastboot_state='none'

  if ioreg -p IOUSB -l -w 0 2>/dev/null | rg -q '"USB Product Name" = "tiger"'; then
    usb_state='tiger-present'
  else
    usb_state='tiger-absent'
  fi

  state="USB=$usb_state ADB=$adb_state FASTBOOT=$fastboot_state"
  if [[ "$state" != "$last_state" ]]; then
    printf '[%s] %s\\n' "$(date '+%Y-%m-%d %H:%M:%S')" "$state"
    last_state="$state"
  fi
  sleep 2
done
~~~

## Reproduce the fastboot foothold

The exact physical sequence is not confirmed. The observed lead was:

1. Keep HDMI connected so the Guide's display remains visible.
2. Start the monitor above.
3. Toggle the physical microphone switch while plugging the Guide into USB.
4. Watch for the purple/bootloader state.
5. Confirm with:

~~~
fastboot devices -l
~~~

The original unit then exposed a `tiger` fastboot device.

## Read-only fastboot collection

Once exactly one expected device is present:

~~~
fastboot devices -l
fastboot getvar product 2>&1
fastboot getvar unlocked 2>&1
fastboot getvar current-slot 2>&1
fastboot flashing get_unlock_ability 2>&1
fastboot oem device-info 2>&1
fastboot getvar all 2>&1
~~~

Save output locally, then redact serials, debug tokens, and other identifiers before publishing.

## Commands with side effects

These are not part of the read-only baseline:

~~~
# May factory-reset if the bootloader permits it.
fastboot flashing unlock

# Reboots the device; does not itself flash data.
fastboot reboot recovery

# Potentially enters Qualcomm EDL; no EDL test was performed in this report.
fastboot reboot edl
~~~

Before any write, verify the fastboot serial again and understand whether the command can wipe or overwrite storage.
