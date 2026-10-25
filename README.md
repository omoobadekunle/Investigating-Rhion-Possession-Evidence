# Investigating Rhino Possession Evidence

### SBT-DF204 · Computer Forensics Case Studies 4 · Lab 4

**Individual Recovered Media and Network-Traffic Investigation: Digital Forensics Investigation Report**

![Course](https://img.shields.io/badge/Course-SBT--DF204-blue)
![Field](https://img.shields.io/badge/Field-Digital%20Forensics-informational)
![Evidence](https://img.shields.io/badge/Evidence-USB%20%7C%20FTP%20%7C%20HTTP-orange)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

> **Disclaimer:** This repository documents an academic exercise using training evidence supplied for the course. No real hosts, people, accounts or services were contacted. The recovered executable was **examined but never executed**.

---

## Table of Contents

- [Executive Summary](#executive-summary)
- [Overview](#overview)
- [Course Information](#course-information)
- [Objectives](#objectives)
- [Evidence and Integrity](#evidence-and-integrity)
- [Tools Used](#tools-used)
- [Methodology](#methodology)
- [Key Findings](#key-findings)
  - [1. Evidence Preservation](#1-evidence-preservation)
  - [2. USB Image Analysis](#2-usb-image-analysis)
  - [3. File Recovery and Carving](#3-file-recovery-and-carving)
  - [4. Hidden Content Checks](#4-hidden-content-checks)
  - [5. FTP Traffic Analysis](#5-ftp-traffic-analysis)
  - [6. Protected ZIP Archive](#6-protected-zip-archive)
  - [7. HTTP Traffic Analysis](#7-http-traffic-analysis)
- [Correlated UTC Timeline](#correlated-utc-timeline)
- [Evidence Log](#evidence-log)
- [Conclusion](#conclusion)
- [Limitations](#limitations)
- [Repository Structure](#repository-structure)
- [Reproducing the Analysis](#reproducing-the-analysis)
- [Author](#author)

---

## Executive Summary

In this investigation, I examined a USB disk image and three network capture files supplied for SBT-DF204 Case Study 4. The purpose was to examine the USB file system, recover available files, check selected images for possible hidden data, investigate FTP file transfers, examine a protected ZIP archive, and review HTTP requests and responses.

I first created working copies of the supplied evidence and compared their SHA-256 hashes with those of the originals. The hashes matched, showing the working copies were identical to the corresponding files at the time of comparison.

- **USB image:** identified as a FAT16 file system with two allocated text files, `gumbo1.txt` and `gumbo2.txt`. I recovered both and used Foremost to carve seven JPEG images; two of the carved JPEGs had identical contents.
- **Hidden content:** Binwalk reported no additional embedded file signatures in the two JPEGs examined, and Steghide could not extract data with an empty passphrase. These results did not confirm hidden content, but they do not rule out other hiding methods or a different passphrase.
- **FTP capture:** showed upload requests for `rhino1.jpg`, `rhino3.jpg` and `contraband.zip` from client `137.30.122.253` to server `137.30.120.40`, with server responses reporting successful completion of the relevant binary-mode transfers.
- **ZIP archive:** contained one encrypted entry, `rhino2.jpg`. I could not decrypt it, so its contents are unconfirmed.
- **HTTP captures:** showed requests for `rhino4.jpg`, `rhino5.gif` and `rhino.exe`. The `rhino.exe` request was followed by an HTTP 200 response reporting 145,920 bytes of binary data. The recovered file was identified as a Windows PE32 executable, and I did not execute it.

The evidence supports the technical observations about the USB file system, recovered files and recorded network activity. It does not independently establish the identity of the person operating the client, prove that the recovered executable was run, or confirm the contents of the encrypted image.

---

## Overview

This project is a full digital forensics investigation of a **USB disk image** and **three network capture files**. The investigation covers:

- File-system examination and file recovery from the USB image
- Carving of JPEG images and checks for hidden data (steganography)
- FTP upload analysis, including an encrypted ZIP archive
- HTTP request/response analysis, including a downloaded Windows executable
- A correlated UTC timeline and a structured evidence log

The work was carried out offline on working copies of the supplied evidence, with SHA-256 hashing used to record integrity.

---

## Course Information

| Item | Details |
| --- | --- |
| **Student** | Adekunle Ogunyemi |
| **Course Code** | SBT-DF204 |
| **Course Title** | Computer Forensics Case Studies 4 |
| **Lab** | Lab 4: Investigating Rhino Possession Evidence |
| **Scheduled Date** | October 25, 2026 |
| **Institution** | International Cybersecurity and Digital Forensics |

---

## Objectives

- Preserve the supplied evidence and record its integrity
- Examine the USB image before recovery
- Recover and validate available files from the USB image
- Check selected images for possible hidden content
- Identify relevant FTP commands and server responses
- Examine the protected ZIP archive and document its limitations
- Analyse HTTP requests and responses and validate reconstructed objects
- Create a UTC timeline and state a conclusion supported by the evidence

---

## Evidence and Integrity

Original files were kept in `original_evidence/` and all analysis was performed on working copies in `working/`. SHA-256 hashes of the working copies matched the originals.

| Evidence Item | Working Copy | Size (bytes) | SHA-256 (working copy) |
| --- | --- | ---: | --- |
| USB image | `working_RHINOUSB.dd` | 259,506,176 | `ce550424200a997c61b413941c8ef4df9619a2f96579674952294a176a32be65` |
| FTP capture | `working_rhino.log` | 3,187,907 | `64e6d55b76660eb3aaa41572c1d04e4452510a343bbf42e844424827dedfddb2` |
| HTTP capture 1 | `working_rhino2.log` | 292,604 | `41939d5de0556b70279056572dee44b6fd84cd05b0788cfd0b6f52f37b161dde` |
| HTTP capture 2 | `working_rhino3.log` | 226,094 | `7b0304f5e88a30c305a99b5a1e2977bced5b9c94da6f458f887bb39966bbfc46` |

> A hash match shows the working copies are identical to the compared originals. It does not prove the originals were unaltered before they were supplied.

<img width="1535" height="883" alt="image" src="https://github.com/user-attachments/assets/6b6a54ca-0f8e-43e7-984a-98b14829d884" />  
*Figure 1: SHA-256 hash comparison*


<img width="1657" height="782" alt="image" src="https://github.com/user-attachments/assets/c4bcc287-0b6c-4d99-8e9e-493b62d46e2a" />
*Figure 2: Evidence inventory*


---

## Tools Used

| Tool | Version | Purpose |
| --- | --- | --- |
| Sleuth Kit (`mmls`, `fsstat`, `fls`, `istat`) | 4.14.0 | File-system examination and recovery |
| Foremost | 1.5.7 | JPEG carving from the USB image |
| TShark / Wireshark | 4.6.6 | FTP and HTTP traffic analysis |
| ExifTool | 13.55 | Image and file metadata |
| Binwalk | n/a | Embedded file signature checks |
| Steghide | n/a | Steganography extraction tests |
| SHA-256 utilities, `file`, `unzip`, `zipinfo` | n/a | Hashing, file identification, archive inspection |

---

## Methodology

1. **Preserve:** record sizes and SHA-256 hashes, then work only on copies.
2. **Examine the USB image:** identify the file system and list allocated and deleted entries with Sleuth Kit.
3. **Recover:** recover files with Sleuth Kit and carve JPEGs with Foremost; identify duplicates by hash.
4. **Check for hidden data:** run Binwalk, ExifTool and Steghide against selected JPEGs.
5. **Analyse network traffic:** extract FTP commands/responses and HTTP requests/responses with TShark and Wireshark, recording frame numbers, timestamps, IPs, object names, sizes and hashes.
6. **Inspect the ZIP archive:** use listing and metadata tools only, since it was not decrypted.
7. **Correlate:** combine FTP, HTTP and USB metadata events into a single UTC timeline.

---

## Key Findings

### 1. Evidence Preservation

Original and working copies were separated, and file sizes and SHA-256 hashes were recorded in `reports/evidence_integrity.txt`. The working copies matched the supplied originals at the time of comparison.


<img width="975" height="522" alt="image" src="https://github.com/user-attachments/assets/e2a77770-73f0-497a-9705-0ff3c15df03c" />
*Figure 3: Terminal output showing the recorded file sizes and SHA-256 hashes for the supplied evidence and working copies*


### 2. USB Image Analysis

| Property | Value |
| --- | --- |
| File system | FAT16 |
| Sector / cluster size | 512 bytes / 4,096 bytes |
| Total sector range | 0–506847 |
| First / second FAT | Sectors 1–248 / 249–496 |
| Root directory | Sectors 497–528 |
| Cluster area | Sectors 529–506840 |
| Partition table (`mmls`) | None displayed (FAT begins at sector 0) |
| Deleted entries | None returned by the listing |


<img width="975" height="562" alt="image" src="https://github.com/user-attachments/assets/82244f04-242e-413d-a5f1-bf6bf543517a" />
*Figure 4: Sleuth Kit `fsstat` output*  

  


Two allocated text files were found:

| File | Inode | Size | Metadata timestamp (UTC) |
| --- | ---: | ---: | --- |
| `gumbo1.txt` | 4 | 2,815 bytes | 2004-04-30 17:11:20 |
| `gumbo2.txt` | 6 | 1,293 bytes | 2004-04-30 17:11:24 |


<img width="975" height="567" alt="image" src="https://github.com/user-attachments/assets/4421c3d3-5b11-4002-84ac-fc83713cc43c" />

*Figure 5: Sleuth Kit directory listing*


<img width="975" height="297" alt="image" src="https://github.com/user-attachments/assets/b479d678-d43b-495a-ae19-cbb6ed699c33" />  
*Figure 6a: `istat` output showing the sizes, allocation status and recorded timestamps of `gumbo1.txt` and `gumbo2.txt` before recovery*

<img width="975" height="364" alt="image" src="https://github.com/user-attachments/assets/069bcc97-1740-4024-b576-a5052852a1c2" />  
*Figure 6b: `istat` output showing the sizes, allocation status and recorded timestamps of `gumbo1.txt` and `gumbo2.txt` before recovery*


### 3. File Recovery and Carving

- `gumbo1.txt` and `gumbo2.txt` were recovered and validated by SHA-256. Because both were allocated, they are described as *recovered copies*, not deleted files.
- **Foremost carved 7 JPEG files:** `00104057.jpg`, `00104249.jpg`, `00105065.jpg`, `00105873.jpg`, `00106393.jpg`, `00106409.jpg`, `00335081.jpg`.
- `00105873.jpg` and `00335081.jpg` are **identical** (same SHA-256):
  `f92654d9ee17ab6b684b09de01cf0bc4076383c007964946d3f31577447596fb`


<img width="975" height="130" alt="image" src="https://github.com/user-attachments/assets/cd5cda64-1e74-4475-bfe0-f25169cf65e5" />

*Figure 7: Recovery output*

<img width="975" height="401" alt="image" src="https://github.com/user-attachments/assets/d963da34-6295-434b-af80-1add4044e1ed" />

*Figure 8: Foremost audit output listing the seven JPEG files carved from the USB image, including offsets and file sizes*

<img width="975" height="190" alt="image" src="https://github.com/user-attachments/assets/fcef90e1-364e-4bea-9ca7-7ae6961de506" />

*Figure 9: ExifTool metadata and SHA-256 results for the carved JPEGs, showing `00105873.jpg` and `00335081.jpg` had identical contents*  

  

### 4. Hidden Content Checks

Files tested: `00106409.jpg` and `rhino4.jpg`.

| Test | Result |
| --- | --- |
| Binwalk | JPEG/JFIF at offset 0 only; no additional embedded signatures |
| ExifTool | Standard JPEG/JFIF metadata only |
| Steghide (empty passphrase) | Extraction failed for both files |

Hidden content was **not confirmed**. This does not rule out a different passphrase or another hiding method.

<img width="975" height="196" alt="image" src="https://github.com/user-attachments/assets/f3f5a375-51fa-4bcc-8695-3968040cefae" />

*Figure 10: Steghide output showing the JPEG format and estimated capacity of `00106409.jpg`*

<img width="975" height="182" alt="image" src="https://github.com/user-attachments/assets/9bd79e23-891b-402b-b2c0-5b56123d249c" />

*Figure 11: Steghide output for `rhino4.jpg`*

<img width="975" height="610" alt="image" src="https://github.com/user-attachments/assets/1409f3dd-5922-4e0e-a2b8-8e5a6b28439b" />

*Figure 12: Binwalk and ExifTool results for the selected JPEGs*


### 5. FTP Traffic Analysis

Client `137.30.122.253` sent upload requests to server `137.30.120.40`.

| File | Request (frame) | Server response | Notes |
| --- | --- | --- | --- |
| `rhino1.jpg` | `STOR` (1546) | `226 Transfer complete` (1612) | Binary mode |
| `rhino3.jpg` | `STOR` (1649, then 1763) | `226 Transfer complete` (1851) | First attempt in ASCII mode (321 bare linefeeds warning, frame 1737); second attempt in binary mode |
| `contraband.zip` | `STOR` (5647) | `226 Transfer complete` (5839) | Binary mode |

<img width="975" height="597" alt="image" src="https://github.com/user-attachments/assets/7f6eefbe-dc7e-4ef0-9b28-1134c488249d" />

*Figure 13: Wireshark FTP control traffic*

<img width="938" height="571" alt="image" src="https://github.com/user-attachments/assets/da3479e3-d416-4e68-ba0f-2a73683f1bc6" />

*Figure 14: FTP server responses*


| Exported Object | Size | SHA-256 |
| --- | ---: | --- |
| `rhino1.jpg` | 65,703 bytes | See `reports/` |
| `rhino3.jpg` | 96,899 bytes | `a85aea8e47d9f5425456f3bafb9c67201c8c2b2b2cec80154c83102e802240c5` |
| `contraband.zip` | 230,566 bytes | `b7091a0b4295a561bf76969073ede126a5ca75899bb115447e67272f95d5a29f` |

<img width="975" height="404" alt="image" src="https://github.com/user-attachments/assets/cdffd27e-f22f-44fd-a506-75e08d1d20c5" />

*Figure 15: Exported FTP objects with their recorded file sizes and SHA-256 hashes*


Captured credentials are intentionally **not** reproduced in this repository.

### 6. Protected ZIP Archive

- `contraband.zip` contains **one encrypted entry**: `rhino2.jpg` (uncompressed size 230,665 bytes, CRC-32 `936ebe65`).
- The archive was **not decrypted**, so the image contents are unknown.
- The entry timestamp (2004-04-26 17:00:12) has no established timezone and is excluded from the main timeline.

<img width="975" height="131" alt="image" src="https://github.com/user-attachments/assets/81db2641-3dca-45a0-8247-988899e19e9d" />

*Figure 16: Archive listing*

<img width="975" height="618" alt="image" src="https://github.com/user-attachments/assets/675fcc51-b1c6-47d3-a4a8-7ab1fd6be471" />

*Figure 17: `zipinfo` output*


### 7. HTTP Traffic Analysis

Client `137.30.123.234` generated the following activity.

| Capture | Request | Frame | Time (UTC) |
| --- | --- | ---: | --- |
| `rhino2.log` | `www.cs.uno.edu/~gnome/rhino4.jpg` | 49 | 2004-04-28 21:08:38 |
| `rhino2.log` | `www.cs.uno.edu/~gnome/rhino5.gif` | 217 | 2004-04-28 21:08:44 |
| `rhino3.log` | Google search for `rhino.exe` | 10 | 2004-04-28 21:15:48 |
| `rhino3.log` | `www.cs.uno.edu/~gnome/rhino.exe` | 110 | 2004-04-28 21:16:23 |

<img width="975" height="610" alt="image" src="https://github.com/user-attachments/assets/7e7bdf28-f0dc-454b-87c1-2279f6a39275" />

*Figure 18: Wireshark HTTP request records for `rhino4.jpg` and `rhino5.gif`*


**`rhino.exe`**

- HTTP 200 response (frame 274) from `137.30.120.37`, `application/octet-stream`, **145,920 bytes**
- Identified as a **PE32 Windows console executable (Intel 80386)**
- SHA-256: `93e70049b60bf5691b43d9e256ca8f459f6bfac832097985bcc6a9040b5ca555`
- Metadata: version 1.0.3.1, internal name `Diskpart`, build timestamp 2001-08-17
- **Not executed.** Analysis was limited to static metadata and file identification.

<img width="975" height="611" alt="image" src="https://github.com/user-attachments/assets/f63215dc-7485-4aaf-bd49-35f40aaf948f" />

*Figure 19: HTTP request and response evidence for `rhino.exe`*

<img width="975" height="618" alt="image" src="https://github.com/user-attachments/assets/ccd97dc8-38b4-464a-8adf-9b4715dfe388" />

*Figure 20: File identification, metadata and SHA-256 results for the recovered `rhino.exe` (examined but not run)*


Recovered `rhino4.jpg` is 153,191 bytes (1531 × 2286) and `rhino5.gif` is 85,137 bytes (400 × 275).

---

## Correlated UTC Timeline

Full timeline: [`reports/timeline_utc.csv`](reports/timeline_utc.csv)

| UTC Timestamp | Source | Event |
| --- | --- | --- |
| 2004-04-26 22:21:49.954 | FTP | `STOR rhino1.jpg` |
| 2004-04-26 22:21:50.180 | FTP | rhino1.jpg transfer complete |
| 2004-04-26 22:22:08.906 | FTP | `STOR rhino3.jpg` (ASCII mode) |
| 2004-04-26 22:22:09.040 | FTP | Warning: 321 bare linefeeds |
| 2004-04-26 22:22:16.091 | FTP | Second `STOR rhino3.jpg` (binary mode) |
| 2004-04-26 22:22:16.263 | FTP | rhino3.jpg transfer complete |
| 2004-04-26 22:26:46.672 | FTP | `STOR contraband.zip` |
| 2004-04-26 22:26:46.877 | FTP | contraband.zip transfer complete |
| 2004-04-28 21:15:48.235 | HTTP | Google search for `rhino.exe` |
| 2004-04-28 21:16:23.343 | HTTP | Request for `rhino.exe` |
| 2004-04-28 21:16:24.214 | HTTP | HTTP 200, 145,920-byte response |
| 2004-04-30 17:11:20 | USB metadata | `gumbo1.txt` written/created time |
| 2004-04-30 17:11:24 | USB metadata | `gumbo2.txt` written/created time |

<img width="975" height="162" alt="image" src="https://github.com/user-attachments/assets/74073581-b531-43b1-86bc-c4b8111f04f0" />

*Figure 21: Correlated timeline showing FTP events, HTTP request/response events, and USB metadata timestamps converted to UTC*


These events come from different sources and do not, by themselves, show that one person was responsible for all of them.

---

## Evidence Log

Detailed log: [`reports/evidence_log.tsv`](reports/evidence_log.tsv)

| ID | Item | Location | Result |
| --- | --- | --- | --- |
| E01 | USB disk image | `working/working_RHINOUSB.dd` | FAT16; two allocated text files; seven JPEGs carved |
| E02 | FTP capture | `working/working_rhino.log` | Upload requests and responses for three file names |
| E03 | HTTP capture | `working/working_rhino2.log` | Requests for `rhino4.jpg` and `rhino5.gif` |
| E04 | HTTP capture | `working/working_rhino3.log` | Request for `rhino.exe`; HTTP 200 response |
| E05 | Recovered text files | `recovered/gumbo1.txt`, `recovered/gumbo2.txt` | Hashes matched TSK recovery copies |
| E06 | Carved JPEGs | `recovered/foremost_usb/jpg/` | Seven JPEGs; two identical; hidden content not confirmed |
| E07 | FTP archive | `recovered/ftp_objects/contraband.zip` | One encrypted entry; not decrypted |
| E08 | Recovered executable | `recovered/http_rhino3/rhino.exe` | PE32 Windows executable; not executed |

<img width="975" height="121" alt="image" src="https://github.com/user-attachments/assets/0f8a9334-ea14-4d60-bf27-1d97c7456dbd" />

*Figure 22: Evidence log showing evidence identifiers, file locations, examination methods and recorded results*


---

## Conclusion

The evidence supports the following technical observations:

- The USB image is FAT16 with two allocated text files and seven carvable JPEGs, two of which are identical.
- No hidden data was confirmed in the two JPEGs tested.
- The FTP capture records upload requests for `rhino1.jpg`, `rhino3.jpg` and `contraband.zip`, each followed by a server completion response.
- `contraband.zip` holds one encrypted entry, `rhino2.jpg`, whose contents remain unknown.
- The HTTP captures record requests for `rhino4.jpg`, `rhino5.gif` and `rhino.exe`, with a 145,920-byte response for the executable.

---

## Limitations

- IP addresses do not identify the person operating a client.
- File-system timestamps do not prove a person accessed or modified files at those times.
- Failed Steghide extraction and clean Binwalk results do not rule out every form of hidden data.
- The contents of the encrypted archive are unknown.
- The HTTP response does not prove the executable was run.
- The evidence alone cannot prove that a named person possessed the files, intended a particular use, or carried out every recorded action.

---

## Repository Structure

```text
.
├── README.md
├── screenshots/                 # Figures 1-22 used in this README
├── Case_Study_4.docx            # Full investigation report
├── original_evidence/           # Supplied evidence (not modified)
│   ├── RHINOUSB.dd
│   ├── rhino.log
│   ├── rhino2.log
│   └── rhino3.log
├── working/                     # Working copies used for analysis
├── recovered/
│   ├── gumbo1.txt
│   ├── gumbo2.txt
│   ├── foremost_usb/            # Carved JPEGs and audit.txt
│   ├── ftp_objects/             # rhino1.jpg, rhino3.jpg, contraband.zip
│   ├── http_rhino2/             # rhino4.jpg, rhino5.gif
│   └── http_rhino3/             # rhino.exe (never executed)
└── reports/
    ├── evidence_integrity.txt
    ├── usb_findings.txt
    ├── usb_hashes_and_types.txt
    ├── ftp_commands_responses.tsv
    ├── http_requests_responses.tsv
    ├── timeline_utc.csv
    └── evidence_log.tsv
```

> **Note:** Large evidence files and the recovered executable should be excluded from version control (for example with `.gitignore` or Git LFS). Share hashes instead of binaries where possible.

---

## Reproducing the Analysis

```bash
# 1. Verify integrity of working copies
sha256sum original_evidence/* working/*

# 2. Examine the USB image
mmls working/working_RHINOUSB.dd
fsstat working/working_RHINOUSB.dd
fls working/working_RHINOUSB.dd
istat working/working_RHINOUSB.dd 4

# 3. Carve JPEGs
foremost -t jpg -i working/working_RHINOUSB.dd -o recovered/foremost_usb

# 4. Check for hidden content
binwalk recovered/foremost_usb/jpg/00106409.jpg
exiftool recovered/foremost_usb/jpg/00106409.jpg
steghide info recovered/foremost_usb/jpg/00106409.jpg

# 5. Inspect the archive without decrypting it
unzip -l recovered/ftp_objects/contraband.zip
zipinfo recovered/ftp_objects/contraband.zip

# 6. Review FTP and HTTP traffic
tshark -r working/working_rhino.log -Y "ftp"
tshark -r working/working_rhino3.log -Y "http.request"
```

> Never execute recovered binaries on a host system. Use static analysis only, or an isolated sandbox.

---

## Author

**Adekunle Ogunyemi**  
Student, SBT-DF204 · International Cybersecurity and Digital Forensics

---

## Author

*This project is for educational purposes only.*
