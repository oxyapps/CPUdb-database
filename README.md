# CPUdb database

The processor database behind [CPUdb](https://cpudb.app), the free, ad-free Android app and website with the
specifications of released CPUs and SoCs: Intel, AMD, Apple, Qualcomm, MediaTek, Samsung, Google, Cyrix, VIA and
more. About 6,700 processors.

This repository holds the published datasets and is deployed as-is to **https://update.cpudb.app/**, where the app
checks for database updates once a day and the website loads its data.

## Found a mistake or a missing CPU?

[Open an issue](https://github.com/oxyapps/CPUdb-database/issues/new/choose). Name the processor, the field, the
value you expect and, if you can, a source (the manufacturer's spec page is best). Corrections go into the next
dataset release, and the app picks it up on its own.

Bugs and ideas for the app or the website are welcome here too.

## Files

```
public/manifest.json              the current dataset: version, file name, SHA-256, size, CPU count
public/cpus-<dataVersion>.json.gz the datasets, one per release (gzipped JSON, never changed once published)
public/_headers                   Cloudflare cache rules
public/index.html                 the page at https://update.cpudb.app/
```

`manifest.json`:

```json
{
  "schemaVersion": 1,
  "dataVersion": 2026111302,
  "url": "cpus-2026111302.json.gz",
  "sha256": "0f06615f…",
  "sizeBytes": 243468,
  "cpuCount": 6740
}
```

- `dataVersion` is `YYYYMMDDNN` (date plus build number of that day). Higher is newer.
- `url` is relative to the manifest.
- `schemaVersion` changes only when the format changes incompatibly (a field renamed or removed). New optional
  fields don't change it; readers should ignore fields they don't know.

To read the current dataset:

```sh
curl -s https://update.cpudb.app/$(curl -s https://update.cpudb.app/manifest.json | python3 -c 'import json,sys; print(json.load(sys.stdin)["url"])') \
  | gunzip | python3 -m json.tool | less
```

## Dataset format (schema 1)

```json
{
  "schemaVersion": 1,
  "dataVersion": 2026111302,
  "generatedAt": "2026-11-13T14:43:00+00:00",
  "license": "GPL-3.0",
  "attribution": "…",
  "cpus": [
    {
      "id": "intel-core-i7-4770k", "manufacturer": "Intel", "family": "Core i7", "model": "Core i7-4770K",
      "codename": "Haswell-DT", "microarchitecture": null, "segment": "DESKTOP", "isa": "x86-64",
      "releaseDate": "2013-06", "releaseYear": 2013, "socket": "LGA 1150", "memoryType": "DDR3-1600, DDR3L-1600",
      "cores": 4, "threads": 8, "performanceCores": null, "efficiencyCores": null, "lowPowerCores": null,
      "coreLayout": null, "baseClockMHz": 3500, "boostClockMHz": 3900, "l2KB": null, "l3KB": 8192,
      "tdpW": 84.0, "tdpMinW": null, "tdpMaxW": null, "processNm": 22, "transistorsMillions": null,
      "launchPriceUsd": 350, "gpu": "HD 4600",
      "instructionSets": "MMX, SSE, SSE2, SSE3, SSSE3, SSE4.1, SSE4.2, AES, AVX, F16C, FMA3, AVX2",
      "specIntRate2026": null, "specIntRate2026Chips": null, "specIntRate2026Url": null,
      "specFpRate2026": null, "specFpRate2026Chips": null, "specFpRate2026Url": null,
      "specIntRate2017": null, "specIntRate2017Chips": null, "specIntRate2017Url": null,
      "specFpRate2017": null, "specFpRate2017Chips": null, "specFpRate2017Url": null,
      "specRetrieved": null,
      "sourceUrl": "https://en.wikipedia.org/wiki/List_of_Intel_Core_desktop_processors",
      "productUrl": "https://www.intel.com/content/www/us/en/products/sku/75123.html"
    }
  ]
}
```

Every field except `id`, `manufacturer`, `model` and `segment` may be `null`.

| Field | Meaning |
|---|---|
| `id` | Stable `manufacturer-model` slug. Use it when reporting a correction. |
| `segment` | `DESKTOP`, `LAPTOP`, `SERVER`, `EMBEDDED` or `MOBILE`. |
| `releaseDate` | As precise as the sources are: `2013`, `2013-06` or `2013-06-02`. A future date means the CPU is announced. |
| `cores`, `threads` | Totals for the whole chip. |
| `performanceCores`, `efficiencyCores`, `lowPowerCores`, `coreLayout` | The split on CPUs with more than one kind of core (Intel P/E/LP-E, AMD Zen 5/Zen 5c, Apple, Arm big.LITTLE), set only when it adds up to `cores`. `coreLayout` names it the manufacturer's way: `6P + 8E + 2LP-E`, `4× Zen 5 + 8× Zen 5c`. On hybrids the clocks are the performance cores'. |
| `baseClockMHz`, `boostClockMHz` | Base and maximum boost clock. |
| `l2KB`, `l3KB` | Total cache of the chip, not per core. |
| `tdpW`, `tdpMinW`, `tdpMaxW` | Default TDP (AMD "Default TDP", Intel "Processor Base Power") and the configurable range (cTDP), when there is one. |
| `memoryType` | Each supported type with its fastest data rate: `DDR4-3200, DDR5-4800`. |
| `instructionSets` | x86 extensions, oldest first. Derived from the design generation when no source lists them. |
| `spec…Rate…` | Best published SPECrate base result on a one-chip system, with its result page in `…Url`. `…Chips` is 2 when only a two-chip result exists. `specRetrieved` is the day the results were downloaded. SPEC CPU 2017 is retired and its scores can't be compared with CPU 2026. |
| `sourceUrl` | Where the row comes from. |
| `productUrl` | The manufacturer's page for the CPU. |

If you show SPEC scores, follow [SPEC's fair use rule](https://www.spec.org/products/fairuse/): show each score's
link and date, say that CPU 2017 is retired, and don't present anything computed from them (averages, ratios) as
a SPEC metric.

## Sources

The dataset is built from:

- the processor lists on English Wikipedia ([CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/));
- AMD's processor specifications ([client](https://www.amd.com/en/products/specifications/processors.html),
  [server](https://www.amd.com/en/products/specifications/server-processor.html));
- the "intel-processors" dataset collected from Intel ARK (GPL-3.0) and Intel ARK's series pages;
- the SPEC CPU results published at [spec.org](https://www.spec.org);
- hand-made corrections, many of them reported here.

SPEC, SPECrate and SPEC CPU are registered trademarks of the Standard Performance Evaluation Corporation.

## License

The data is licensed under the [GNU General Public License v3.0](LICENSE). Wikipedia's CC BY-SA 4.0 is one-way
compatible with GPLv3. Each dataset file carries its attribution in the `attribution` field. Keep it when you
redistribute the data.
