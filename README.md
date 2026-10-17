# Old Friends — Cytech CTF Write-Up

| Challenge | Category | Final Flag |
| :--- | :--- | :--- |
| **Old Friends** | OSINT / Digital Forensics / Web Archive Analysis | `Cryptix{stonehenge}` |

---

## 📌 Challenge Overview

The challenge presents an archived web file `deleted_post.html` simulating historical snapshots of a personal blog owned by a Minecraft player named **Cr4t3K1ng** (`cr4t3k1ng.site`). Although the user deleted a teaser photo of an upcoming real-life landmark build, traces remained preserved across historical timeline captures.

**Goal:** Uncover the deleted photo from the web archive snapshots, extract its embedded metadata, and identify the physical landmark location.

**Flag Format:**
```text
Cryptix{place_name}
```

---

## 🔍 Investigation Walkthrough

### 1. Analyzing the Archived Page (`deleted_post.html`)

Inspecting `deleted_post.html` showed 4 chronological snapshots of the site:

1. **Snapshot 1 — 02/04/2026:**
   * Initial introduction post (`hey, i'm Cr4t3K1ng`) and Minecraft build updates (`mine B redesign`).
2. **Snapshot 2 — 11/06/2026:**
   * New teaser post: *"next build: a 1:1 replica"*.
   * Post content:
     > *"next project is a 1:1 replica of a real place i visited. here’s my reference pic from the trip. guess where 👀"*
   * An embedded Base64 data URI image was present.
3. **Snapshot 3 — 19/07/2026:**
   * The image was taken down:
     > *(edit: took the pic down, gave away too much lol) [image removed by author]*
4. **Snapshot 4 — 03/09/2026:**
   * Post retitled: *"builds coming soon"*. The author scrubbed references to the trip.

---

### 2. Extracting the Deleted Image

Navigating back to Snapshot 2 (11/06/2026) in the HTML source revealed the embedded JPEG image in Base64:

```html
<img src="data:image/jpeg;base64,/9j/4AAQSkZJRg..." />
```

Extracting the Base64 string and decoding it into a file:

```bash
# Decoded and saved as:
image_4_0_next_build_a_11_replica.jpg (49,432 bytes)
```

---

### 3. EXIF Metadata Forensics

Running `exiftool` against the recovered image revealed intact camera and GPS EXIF metadata:

```bash
exiftool image_4_0_next_build_a_11_replica.jpg
```

**Key EXIF Fields:**
* **Camera Model:** Pixelfone PF-2 Pro
* **Date/Time:** 2026:06:09 14:22:05
* **GPS Latitude:** `51 deg 10' 43.98" N`
* **GPS Longitude:** `1 deg 49' 34.37" W`

---

### 4. Coordinate Conversion & Geolocation

Converting degrees, minutes, seconds (DMS) to Decimal Degrees:

$$\text{Latitude} = 51 + \frac{10}{60} + \frac{43.98}{3600} = 51.178883^\circ\text{ N}$$

$$\text{Longitude} = -\left(1 + \frac{49}{60} + \frac{34.37}{3600}\right) = -1.826214^\circ\text{ W}$$

**Decimal Coordinates:**
```text
51.178883, -1.826214
```

Looking up the coordinates on Google Maps / OpenStreetMap:
* **Location:** Stonehenge, Salisbury, Wiltshire, United Kingdom
* **Landmark:** Stonehenge

---

## 🚩 Flag

```text
Cryptix{stonehenge}
```

---

## 💡 Summary & Key Takeaways

1. **Wayback / Archive Forensics:** Deleting content from a live website does not eliminate earlier captures stored in web archives or snapshot mirrors.
2. **EXIF Hygiene:** Photos taken with smartphones typically embed exact GPS coordinates in EXIF tags unless sanitized or stripped before uploading.
3. Attack path:
   ```text
   cr4t3k1ng.site archive (deleted_post.html)
   └── Snapshot 2 (11/06/2026)
       └── Base64 JPEG
           └── EXIF Metadata (exiftool)
               └── 51.178883, -1.826214
                   └── Stonehenge
                       └── Cryptix{stonehenge}
   ```
