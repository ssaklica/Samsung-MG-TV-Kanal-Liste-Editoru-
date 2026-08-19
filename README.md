# SSAKLICA — Multi Generation Samsung Channel Editor

**Version:** v0.6.3 · Multi-Generation  
**Brand:** SSAKLICA  
**Application type:** Browser-based Samsung channel list editor  
**Installation:** Not required

SSAKLICA — Multi Generation Samsung Channel Editor is a lightweight web application for opening, inspecting, reordering and editing Samsung TV channel-list exports directly in a modern browser.

The application is designed around a **multi-generation format engine**. It can automatically distinguish between supported modern Samsung SQLite/TKGS exports and supported legacy Samsung SCM archives, then load the appropriate parser/writer without asking the user to select a TV generation manually.

> The channel-list file is processed locally in the browser. The selected Samsung export is not uploaded to an application server.

---

## What it does

- Opens Samsung channel-list exports from USB.
- Detects the supported Samsung export format automatically.
- Displays separate channel sources/lists when the export contains more than one list.
- Reorders channels by entering the desired destination number.
- Automatically shifts the surrounding channels when a channel is moved.
- Renames channels.
- Rewrites channel numbering as `1..N` when requested.
- Exports a new Samsung-compatible ZIP or SCM file.
- Verifies the generated database/binary file before allowing export.
- Preserves unknown/unmodified data as much as possible.
- Runs in the browser without a traditional installation package.

### Modern Samsung format functions

For supported SQLite/TKGS exports:

- TKGS and DVB-S lists are shown separately.
- Channel ordering can be changed.
- Channel names can be edited.
- Channels can be marked hidden/restored where supported by the current parser.
- System/data services are protected from normal reordering.
- SQLite output is reopened and validated before the final archive is produced.

### Legacy Samsung SCM functions

For supported SCM profiles:

- Multiple `map-*` channel lists can be detected and shown as separate tabs.
- `map-CableD` and `map-SateD` are currently supported for the validated profiles listed below.
- Channel ordering can be changed.
- Channel names can be edited.
- Record checksums are recalculated after supported changes.
- Unknown bytes in legacy channel records are intentionally left untouched.
- The generated SCM archive is reopened and verified before download.

---

## Compatibility

Compatibility is determined primarily by the **export-file format**, not only by the TV model name.

### Confirmed / tested

| TV / family | Export format | Status | Notes |
|---|---|---:|---|
| **Samsung QE55LS03DAUXTK** | Modern ZIP / SQLite / TKGS | ✅ Tested | Modern Samsung channel export containing `dvbs`, `tkgs`, `sat`, `metadata.xml` and `db-tvs-chms-schema.sei`. |
| **Samsung UE46F6400** | Legacy `.scm` | ✅ Tested | Legacy multi-list SCM archive. `map-CableD` 320-byte and `map-SateD` 168-byte records validated. |

### Format-level support

#### Modern Samsung SQLite / TKGS

The application recognizes the modern format when the archive contains the expected database structure, such as:

```text
dvbs
tkgs
sat
metadata.xml
db-tvs-chms-schema.sei
```

This format is used by newer Samsung generations, including the tested Tizen-based model above. **Not every Samsung/Tizen model has been individually validated**, so compatibility should be considered format-based rather than guaranteed for every model year.

#### Legacy Samsung SCM

The application recognizes legacy SCM archives containing Samsung `map-*` files and `CloneInfo`.

Current validated record profiles:

- `map-CableD` — **320 bytes/record**
- `map-SateD` — **168 bytes/record**

These profiles are associated with older Samsung generations, including the tested **F Series UE46F6400**. Similar E/F/H and some later legacy SCM exports may use related layouts, but unsupported record sizes are intentionally not guessed or modified.

### Important compatibility principle

If the application does not recognize a record/database layout with sufficient confidence, it should not silently write unknown data. This is intentional: preserving the original Samsung export is more important than pretending that an unverified model is supported.

---

## How it works

### 1. No installation

There is no MSI/EXE setup package. Extract the project files and open:

```text
index.html
```

with a modern version of:

- Microsoft Edge
- Google Chrome

### 2. Browser-side processing

The Samsung ZIP/SCM file is opened and processed in browser memory.

For archive handling the application uses **JSZip**. Modern SQLite-based channel databases are handled with **sql.js**.

The current build loads these JavaScript libraries from a CDN. Therefore an internet connection can be required on first load. A browser may subsequently reuse cached copies, but fully offline availability depends on the browser cache. A future build can bundle these dependencies locally for guaranteed offline operation.

### 3. Automatic format detection

Simplified flow:

```text
Samsung channel export
        │
        ▼
Archive inspection
        │
        ├── dvbs + tkgs + sat
        │       ▼
        │   Modern SQLite/TKGS adapter
        │
        └── map-* + CloneInfo
                ▼
            Legacy SCM adapter
```

The user does not need to choose “old Samsung” or “new Samsung” manually.

### 4. Shared editor UI

Both generations are normalized into a common channel model. The interface can therefore provide the same basic editing workflow while each adapter writes changes back using the rules of its own Samsung format.

---

## Legacy SCM profiles currently implemented

### `map-CableD` — 320-byte record

Validated with the supplied Samsung UE46F6400 SCM export.

Known fields used by the editor include:

- Program number: offset `0`
- Service ID: offset `6`
- Service type: offset `15`
- Encrypted flag: offset `24`
- Lock flag: offset `31`
- ONID: offset `32`
- TSID: offset `48`
- Channel name: offset `64`, 100 bytes, UTF-16BE
- Checksum: offset `319`

### `map-SateD` — 168-byte record

Validated with the supplied Samsung UE46F6400 SCM export.

Known fields used by the editor include:

- Program number: offset `0`
- In-use flag: offset `7`, bit `0`
- Service type: offset `14`
- SID: offset `16`
- TSID: offset `24`
- ONID: offset `28`
- Channel name: offset `36`, 100 bytes, UTF-16BE
- Encrypted flag: offset `136`, bit `0`
- Checksum: offset `167`

The writer changes only verified fields and recalculates the record checksum after supported modifications.

---

## Modern Samsung data model

The modern format uses multiple SQLite databases. The most important channel/service data is stored in tables such as:

```text
CHNL
SRV
SRV_DVB
SRV_SO
SRV_FAV
PROV
```

TKGS and normal DVB-S are treated as separate channel-list contexts.

The editor deliberately protects system/data services from normal user reordering.

---

## Usage

1. Extract the application folder.
2. Open `index.html` in Edge or Chrome.
3. Click **Kanal Listesi Aç**.
4. Select the Samsung `.zip` or `.scm` export from the USB drive.
5. Check the detected TV/model, format and channel-list tabs.
6. Enter a desired number in the **No** field to move a channel directly to that position.
7. Edit the channel name where required.
8. Optionally use **Tüm Listeyi 1..N Olarak Yaz** to normalize numbering.
9. Click **Doğrula ve Yeni Dosya Oluştur**.
10. Import the newly generated file back into the Samsung TV.

It is strongly recommended to keep the original TV export unchanged as a backup.

---

## Privacy and safety design

- The selected channel-list file is processed locally in the browser.
- The application itself does not upload the TV export to a backend service.
- Legacy writers modify only known fields.
- Modern SQLite outputs are reopened and validated after modification.
- Legacy SCM outputs are reopened and checked after modification.
- Unknown record data is preserved rather than rewritten unnecessarily.
- Original files should always be retained as a recovery copy.

---

## Current limitations

- **Legacy SCM channel deletion is supported for the verified map-CableD/map-SateD profiles. Modern SQLite/TKGS permanent deletion is not currently implemented; hiding remains experimental.**
- TKGS may restore channels depending on TV/TKGS operating mode and Samsung firmware behavior.
- Legacy SCM record layouts vary across generations; only explicitly recognized profiles are written.
- Not every Samsung model or firmware version has been tested.
- Modern dependencies are currently loaded from a CDN rather than bundled locally.
- This is an independent utility and is not an official Samsung application.

---

## Project structure

```text
SamsungChannelEditor-Web-v0.6.3/
├── index.html
├── app.js
├── styles.css
└── README.md
```

The application architecture is logically divided into:

```text
Editor UI
   │
   ├── Modern Samsung SQLite/TKGS adapter
   │
   └── Legacy Samsung SCM adapter
          ├── map-CableD parser/writer
          └── map-SateD parser/writer
```

---

## Version

**v0.6.3 · Multi-Generation**

Key milestone of this release:

- Modern Samsung SQLite/TKGS support retained.
- Legacy Samsung SCM support introduced.
- Multi-list legacy channel detection added.
- `map-CableD` and `map-SateD` use independent validated profiles.
- Automatic format detection allows old and new Samsung generations to share one editor UI.

---

## Branding / credits

**SSAKLICA — Multi Generation Samsung Channel Editor**

Designed as a practical, privacy-conscious browser utility for editing Samsung USB channel-list exports across multiple Samsung TV generations.

This project is independent and is not affiliated with, endorsed by, or distributed by Samsung Electronics.



## v0.6.3 additions

- Legacy SCM multi-select channel deletion. Deletion uses the verified SCM status fields: `map-CableD` sets the Deleted bit, while `map-SateD` clears the InUse bit. Records stay in place and checksums are recalculated.
- Legacy record checksums are recalculated/verified after ordering or rename changes.
- Separate **Tip** column for TV / HD / Radyo / Data values.
- Channel-type filter in the toolbar.
- Provider/source presentation split: Legacy lists show source + record index, while Modern SQLite lists continue to show provider names.
- Export verification reopens the generated archive and validates deletion compaction, order, names, checksums and untouched archive entries.

See `RELEASE_NOTES.md` for the detailed release note.
