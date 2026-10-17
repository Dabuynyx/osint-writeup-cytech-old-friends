# Old Friends — Cytech CTF Write-Up

<div align="center">

![Category](https://img.shields.io/badge/Category-OSINT%20%2F%20Digital%20Forensics-blue?style=for-the-badge&logo=googlemaps)
![Status](https://img.shields.io/badge/Status-Solved-success?style=for-the-badge&logo=checkmarx)
![CTF](https://img.shields.io/badge/CTF-Cytech-orange?style=for-the-badge)
![Flag](https://img.shields.io/badge/Flag-Cryptix%7Bstonehenge%7D-red?style=for-the-badge)

</div>

---

## 📋 Challenge Information

| Field | Detail |
| :--- | :--- |
| **Challenge Name** | Old Friends |
| **Category** | OSINT / Digital Forensics / Web Archive Analysis |
| **Artifacts** | `deleted_post.html`, `image_4_0_next_build_a_11_replica.jpg` |
| **Target Identity** | `Cr4t3K1ng` (`cr4t3k1ng.site`) |
| **Final Flag** | `Cryptix{stonehenge}` |

---

## 🎯 Challenge Overview

The challenge provides an HTML file representing historical web snapshots: `deleted_post.html`. The author, a Minecraft creator named **Cr4t3K1ng**, teased an upcoming 1:1 real-world replica build with a reference photograph taken on a personal trip. Although he later deleted the photo and post claiming it gave away too much, the image was captured in an earlier snapshot.

**Flag Format:**
```text
Cryptix{place_name}
```

---

## 🗺️ Investigation Flowchart

```mermaid
flowchart TD
    A["📄 deleted_post.html"] --> B["🕰️ Audit 4 Archived Snapshots"]
    B --> C["Snapshot 1 (02/04/2026): Intro"]
    B --> D["Snapshot 2 (11/06/2026): Teaser + Embedded Base64 JPEG"]
    B --> E["Snapshot 3 (19/07/2026): Post edited, image deleted"]
    B --> F["Snapshot 4 (03/09/2026): Site cleaned up"]
    
    D -->|Extract data:image/jpeg;base64,...| G["🖼️ Recover image_4_0_next_build_a_11_replica.jpg (49,432 bytes)"]
    G -->|Run exiftool| H["📍 EXIF GPS: 51° 10' 43.98\" N, 1° 49' 34.37\" W"]
    H -->|Convert to Decimal| I["🌐 Coordinates: 51.178883, -1.826214"]
    I -->|Map Geolocation| J["🏛️ Stonehenge (Salisbury, UK)"]
    J --> K["🚩 Flag: Cryptix{stonehenge}"]

    style K fill:#22c55e,stroke:#15803d,stroke-width:2px,color:#fff
```

---

## 🔍 Step-by-Step Investigation Walkthrough

### 1. Checking the Archived Page (`deleted_post.html`)

Opened the file and analyzed the 4 distinct snapshots of `cr4t3k1ng.site`:

#### Snapshot 1 — 02/04/2026
* Posts:
  * `mine B redesign`
  * `hey, i'm Cr4t3K1ng`

#### Snapshot 2 — 11/06/2026
* New post: `next build: a 1:1 replica`
* Text:
  > *“next project is a 1:1 replica of a real place i visited. here’s my reference pic from the trip. guess where 👀”*
* Embedded in the page was a Base64-encoded JPEG image.

#### Snapshot 3 — 19/07/2026
* Photo was deleted.
* Post said:
  > *(edit: took the pic down, gave away too much lol) [image removed by author]*

#### Snapshot 4 — 03/09/2026
* Post renamed: `builds coming soon`
* Text:
  > *“cleaning up the site. the replica is still happening, it's a surprise now.”*

---

### 2. Extracting the Deleted Image

Navigated back to the **11/06/2026** snapshot in `deleted_post.html`:
```html
data:image/jpeg;base64,/9j/4AAQSkZJRg...
```

#### Extraction Commands:

Using Python or Bash to extract and decode the Base64 image payload:

```bash
# Using grep + base64 CLI
grep -o 'data:image/jpeg;base64,[^"]*' deleted_post.html | head -n 1 | cut -d',' -f2 | base64 -d > image_4_0_next_build_a_11_replica.jpg

# Verify the extracted image file size
ls -la image_4_0_next_build_a_11_replica.jpg
# Size: 49,432 bytes
```

---

### 3. Forensic Analysis: Checking EXIF Metadata

Ran `exiftool` on the recovered image:

```bash
exiftool image_4_0_next_build_a_11_replica.jpg
```

**Output Excerpt:**
```text
Camera Model Name               : Pixelfone PF-2 Pro
Date/Time Original              : 2026:06:09 14:22:05
GPS Latitude                    : 51 deg 10' 43.98" N
GPS Longitude                   : 1 deg 49' 34.37" W
GPS Position                    : 51 deg 10' 43.98" N, 1 deg 49' 34.37" W
```

> [!NOTE]
> The GPS coordinate metadata was completely intact inside the deleted photograph.

---

### 4. Converting the Coordinates

Convert DMS (Degrees, Minutes, Seconds) to Decimal Degrees (DD):

```text
Latitude:
51 + (10 / 60) + (43.98 / 3600) = 51.178883

Longitude:
-(1 + (49 / 60) + (34.37 / 3600)) = -1.826214
```

#### Quick Python Verification:
```bash
python3 -c '
lat = 51 + (10/60) + (43.98/3600)
lon = -(1 + (49/60) + (34.37/3600))
print(f"Coordinates: {lat:.6f}, {lon:.6f}")
'
# Output: Coordinates: 51.178883, -1.826214
```

**Coordinates:**
```text
51.178883, -1.826214
```

---

### 5. Identifying the Landmark

Lookup on mapping services (Google Maps / OpenStreetMap):
* **Location:** Salisbury, Wiltshire, United Kingdom
* **Site:** Stonehenge

---

## 🚩 Final Flag

```text
Cryptix{stonehenge}
```

---

## 💡 Summary & Key Takeaways

```text
Cr4t3K1ng
└── deleted post
    └── old website snapshot (11/06/2026)
        └── Base64 image
            └── extract image (49,432 bytes)
                └── EXIF metadata (exiftool)
                    └── GPS coordinates: 51.178883, -1.826214
                        └── Stonehenge
                            └── Cryptix{stonehenge}
```

> [!IMPORTANT]
> **Main lesson:** Even though the author deleted the post and removed the image from the latest revision of the website, the historical snapshot still archived the Base64 image payload, and the unstripped EXIF metadata leaked the exact real-world GPS location.
