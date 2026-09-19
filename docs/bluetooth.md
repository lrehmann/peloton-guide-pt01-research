# Bluetooth remote investigation

## Screen evidence

The Guide showed:

> Press any button on the remote
>
> To reconnect, press and hold Up and Menu for 4 seconds

The same screen also showed the Guide offline and offered **Open Network Settings**. The screen is a reconnect state, not a visible Android Settings/ADB surface.

Peloton's Guide manual describes the remote as Bluetooth Low Energy and gives the Up + Menu hold as the pairing/reconnect gesture. The manual does not document a generic keyboard profile or a way to pair from a Mac without the remote.

## Generic emulator test

The open-source [darwin-bt-remote](https://github.com/jqssun/darwin-bt-remote) project was cloned to a temporary directory, generated with XcodeGen, and built successfully as a macOS app.

Observed setup:

- App: \`BTRemote.app\`
- Advertised name: \`BTRemote\`
- Low Energy mode: advertising started
- Generic controls: Up, Down, Left, Right, Enter, keyboard keys, and consumer controls
- One generic Up action was sent
- No Guide connection or visible Guide reaction was observed

The app's Low Energy UI says its compatibility is intended for Apple devices/Windows and advises Classic mode for other devices. Classic mode was selected as a second experiment, but its pairing dialog did not show the Guide as a pairable device and no paired device was available.

## Nearby inquiry

The Mac's Bluetooth inquiry found a nearby name resembling a Peloton device:

~~~
PLTN-TCAV1
~~~

This was not proven to be the Guide, was not paired, and was not treated as evidence of the remote protocol.

## Interpretation

The generic emulator failed to provide useful control for at least three possible reasons:

1. The Guide expects the original remote's proprietary identity, advertising data, or GATT layout.
2. The Guide may be bonded to a specific remote and waiting for a reconnect gesture.
3. The standard HID transport/profile exposed by the emulator may not match the Peloton remote implementation.

No exact RE01 protocol was recovered. A compatible replacement remote or a BLE capture made with a real remote would provide substantially better evidence than guessing button reports.
