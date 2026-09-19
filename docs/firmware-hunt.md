# PT01 firmware hunt

Search date: 2026-09-18.

## Scope

The search looked for a package that could be matched to the observed unit:

- Peloton Guide model `PT01`.
- USB product `tiger`, VID `0x317e`, PID `0x4ee1`.
- Qualcomm `QCS EMMC` fastboot variant.
- A signed Qualcomm programmer or Firehose loader.
- An OTA or factory image containing matching boot, system, vendor, modem,
  bootloader, or device-specific partitions.

The local search covered the user's Downloads directory and this repository.
The public search covered GitHub repository/code results and public web results
for the exact model, product name, observed identifiers, and related Qualcomm
loader terms.

## Results

No exact PT01/`tiger` firmware image, OTA package, signed programmer, or
Firehose loader was found in those sources. This does not establish that no
such package exists; it only records that none was publicly discoverable in
this bounded search.

Two useful public references were checked:

- [FCC ID 2AA3N-PT01](https://fccid.io/2AA3N-PT01) — confirms the PT01 hardware
  identity and provides regulatory/internal-photo material, but not a firmware
  download.
- [OpenPelo](https://github.com/doudar/Openpelo) — community Android/ADB,
  wireless-debugging, and app-management tooling. It is useful if debugging is
  already enabled, but it is not a PT01 firmware bundle; its source did not
  contain `tiger`, PT01, Firehose, or a matching firmware package.

No firmware was downloaded, executed as a device programmer, or flashed.

## Best next sources

The highest-value sources are an OTA capture from a working Guide, a package
obtained from another matching PT01 unit, or a vendor-supported recovery image.
Until one of those is available, EDL/programmer work would have no verified
device-specific payload and carries a substantial brick risk.
