# DF-EXPERIMENTS

# Experiment 1: Evidence Acquisition Using AccessData FTK Imager

## Aim

To acquire forensic evidence (volatile memory and disk image) and verify integrity using MD5 and SHA-1 hash values — performed on Kali Linux (ARM64) using `dd`, `dc3dd`, and `Guymager` as Linux equivalents of FTK Imager.

## Software Used

- Kali Linux (ARM64)
- `dd` — built-in Linux disk imager
- `dc3dd` — forensic version of dd by US DoD Cyber Crime Center
- `Guymager` — graphical forensic imager (GUI equivalent of FTK Imager)

## Introduction

FTK Imager is a computer forensics software product made by AccessData. Since FTK Imager runs only on Microsoft Windows, this experiment was performed on a Kali Linux (ARM64) workstation using Linux equivalents — `dd` and `dc3dd` for command-line acquisition, and `Guymager` for graphical acquisition. Guymager offers the same facilities as FTK Imager: source device selection, image format choice, case detail entry, fragment size, and MD5/SHA-1 hash verification after acquisition.

FTK Imager can acquire:

- Volatile memory (RAM)
- Non-volatile memory (Hard disk)
- Physical drives
- Logical drives (partitions)
- Image files
- Contents of folders
- CDs/DVDs

---

## Procedure

### Part 1 — Acquiring Volatile Memory

#### Step 1: Create Output Directory

```bash
mkdir -p ~/forensics_output
ls ~/
```
<img width="824" height="547" alt="Screenshot 2026-08-24 at 3 45 06 PM" src="https://github.com/user-attachments/assets/33d9a1f7-39d5-4798-8ecc-34e32d5ba2b5" />

#### Step 2: Capture Volatile Memory using dd

The Linux kernel exposes live physical memory through `/proc/kcore`. This was captured using `dd` — equivalent to FTK Imager's "Capture Memory" button. The output file uses the `.mem` extension, exactly as FTK Imager does.

```bash
sudo dd if=/proc/kcore \
        of=~/forensics_output/memdump.mem \
        bs=1MB count=100 \
        status=progress
```

<img width="824" height="547" alt="Screenshot 2026-08-24 at 3 45 40 PM" src="https://github.com/user-attachments/assets/53da9f11-b3f7-444a-8755-9145f7d462ac" />


**Note:** 100 records were read and written — 100,000,000 bytes (100 MB, 95 MiB) copied at 1.9 GB/s. Just as FTK Imager saves the volatile memory with extension `.mem`, the destination folder `~/forensics_output` now holds `memdump.mem`.

#### Step 3: Verify the Memory Image

```bash
ls -lh ~/forensics_output/memdump.mem
file ~/forensics_output/memdump.mem
```
<img width="1648" height="1094" alt="image" src="https://github.com/user-attachments/assets/998a423e-1444-458d-98ec-a4aea4d160c3" />


The dump is 96 MB and identified as an ELF 64-bit LSB core file for the ARM aarch64 architecture, confirming a genuine memory image was captured.

#### Step 4: Compute and Store Hash Values

To preserve the integrity of volatile evidence, MD5 and SHA-1 hashes were calculated immediately after acquisition.

```bash
md5sum  ~/forensics_output/memdump.mem | tee ~/forensics_output/mem_md5.txt
sha1sum ~/forensics_output/memdump.mem | tee ~/forensics_output/mem_sha1.txt
cat ~/forensics_output/mem_md5.txt
cat ~/forensics_output/mem_sha1.txt
```
<img width="1728" height="1094" alt="image" src="https://github.com/user-attachments/assets/57bb5639-f0c0-4ccf-b87a-05ffe77f3f05" />


**Recorded Hash Values — Volatile Memory:**

| Algorithm | Hash Value |
|-----------|------------|
| MD5 | `719e173f0a65336bf0a5d7b3069a4327` |
| SHA1 | `b736363b7c14f28f8ef42907fa25bbd5854024a8` |

---

### Part 2 — Acquiring Non-Volatile Memory (Disk Image)

#### Step 5: Create a Virtual Evidence Disk

A 100 MB virtual disk was prepared as the evidence source so the acquisition could be demonstrated safely without altering the real disk of the machine.

```bash
sudo dd if=/dev/zero \
        of=~/forensics_output/virtual_disk.img \
        bs=1MB count=100 status=progress
ls -lh ~/forensics_output/virtual_disk.img
```

<img width="1728" height="1094" alt="image" src="https://github.com/user-attachments/assets/24382d5f-a58a-47bd-a9bd-dc619440697f" />


#### Step 6: Format the Virtual Disk with ext4

The container was formatted with an ext4 file system so it behaves like a real evidence partition.

```bash
sudo mkfs.ext4 ~/forensics_output/virtual_disk.img
file ~/forensics_output/virtual_disk.img
```

<img width="1728" height="1094" alt="image" src="https://github.com/user-attachments/assets/5385fd13-3821-48b8-8761-6c78ebe6b134" />

The new file system was created with 97,656 blocks and 24,384 inodes, assigned UUID `71e5f665-c69f-4d77-8239-acf0586673f9`.

#### Step 7: Mount Disk and Add Evidence Files

The disk was mounted through a loop device and sample files were placed on it to represent the evidence that will later be recovered from the image.

```bash
sudo mkdir -p /mnt/virtual_disk
sudo mount -o loop ~/forensics_output/virtual_disk.img /mnt/virtual_disk
sudo bash -c 'echo "Case Number: 2024-DF-001"      > /mnt/virtual_disk/case_info.txt'
sudo bash -c 'echo "Evidence: Test forensics data" > /mnt/virtual_disk/evidence1.txt'
sudo bash -c 'echo "Examiner: Sandeep Chakka"      > /mnt/virtual_disk/evidence2.txt'
ls -la /mnt/virtual_disk/
```

<img width="1728" height="1094" alt="image" src="https://github.com/user-attachments/assets/8fd6ea34-78db-427e-b975-42dce56ab05b" />


#### Step 8: Unmount the Disk (Write Blocker Equivalent)

Before imaging, the disk is unmounted so the acquisition reads a stable device with no pending writes — this is the software equivalent of working through a write blocker.

```bash
sudo umount /mnt/virtual_disk
echo "Disk unmounted successfully"
```

<img width="1728" height="1094" alt="image" src="https://github.com/user-attachments/assets/f2c63b3d-15e0-4a35-9551-3ee8c84fa8a0" />


#### Step 9: Record Case Details

FTK Imager collects case details through its "Evidence Item Information" dialog. Here, the same information is recorded in a case details file stored with the evidence.

```bash
cat ~/forensics_output/case_details.txt
```

<img width="1728" height="1094" alt="image" src="https://github.com/user-attachments/assets/b8390e9b-bc53-40ac-82b2-866c250f6fc7" />


| Field | Value |
|-------|-------|
| Case Number | 2024-DF-001 |
| Evidence Number | EV-001 |
| Examiner Name | Sandeep Chakka |
| Description | Digital Forensics Lab – Ex No 1 |
| Source | Virtual Disk Image |
| Destination | `~/forensics_output/` |
| Institution | KARE SoC CSE 2024-2025 |

#### Step 10: Acquire the Disk Image using dc3dd

`dc3dd` is the forensic version of `dd` developed by the US Department of Defense Cyber Crime Center. It performs acquisition and simultaneously computes MD5 and SHA-1 hashes, and writes a full acquisition log — equivalent to FTK Imager's "Verify images after they are created" option.

```bash
sudo dc3dd if=~/forensics_output/virtual_disk.img \
           of=~/forensics_output/disk_image.dd \
           hash=md5 hash=sha1 \
           log=~/forensics_output/acquisition_log.txt
```

<img width="1728" height="1094" alt="image" src="https://github.com/user-attachments/assets/f70ae133-ea1a-4eb5-b26b-5dd8ebb18836" />

The tool reports 195,312 sectors + 256 bytes in and the same out — copy is complete and nothing was lost.

```bash
ls -lh ~/forensics_output/disk_image.dd
```

<img width="1728" height="1094" alt="image" src="https://github.com/user-attachments/assets/dd903f32-599d-468b-9023-55505315cbbe" />


#### Step 11: Verify the Acquired Image

Hash values of the finished image were recomputed and compared with the values dc3dd calculated during acquisition — the same purpose as FTK Imager's hash verification step.

```bash
md5sum  ~/forensics_output/disk_image.dd | tee ~/forensics_output/disk_md5.txt
sha1sum ~/forensics_output/disk_image.dd | tee ~/forensics_output/disk_sha1.txt
cat ~/forensics_output/disk_md5.txt
cat ~/forensics_output/disk_sha1.txt
```

<img width="1728" height="1094" alt="image" src="https://github.com/user-attachments/assets/b77f6c51-2a55-4464-8d59-ed8b8028a1e1" />

The values are identical to those dc3dd produced during acquisition. The stored checksum files were then re-checked with the `-c` option — the final integrity test.

```bash
md5sum  -c ~/forensics_output/disk_md5.txt
sha1sum -c ~/forensics_output/disk_sha1.txt
```

<img width="1728" height="1094" alt="image" src="https://github.com/user-attachments/assets/c0300d1f-ea09-4975-93db-5fd71d58e4b6" />

Both checks report **OK**.

**Recorded Hash Values — Disk Image:**

| Algorithm | Hash Value |
|-----------|------------|
| MD5 | `610d3e6bdd4718c27a29ed417b30cf86` |
| SHA1 | `07b410aba4fa6400bdc3dc68a2099e6f967cc8bd` |

#### Step 12: Generate Acquisition Summary Report

```bash
cat ~/forensics_output/image_summary.txt
```

<img width="2880" height="1800" alt="image" src="https://github.com/user-attachments/assets/732d87bd-f91a-4471-b59e-f38cb58986ee" />


#### Step 13: List All Acquired Evidence Files

```bash
ls -lh ~/forensics_output/
echo "Hash Values are Matched"
echo "Acquisition Complete"
```

<img width="1728" height="1094" alt="image" src="https://github.com/user-attachments/assets/f5756584-33e9-4013-ad88-bcd31177d7ff" />

---

### Part 3 — Graphical Acquisition using Guymager

Guymager is the graphical forensic imager supplied with Kali Linux. It lists every attached device, and right-clicking a device and choosing "Acquire image" opens a dialog that corresponds one-to-one with the FTK Imager "Create Disk Image" wizard.

The dialog has the same three sections as FTK Imager:
- **File format** — "Linux dd raw image (.dd)" = Raw (dd); "Expert Witness Format (.Exx)" = E01/EnCase
- **Case details** — Case number, evidence number, examiner, description, notes
- **Hash calculation / verification** — Calculate MD5, SHA-1, and "Verify image after acquisition"

#### Step 14: Open Guymager and Configure Acquisition

The dialog was filled with: Linux dd raw image format, Case Number `2024-DF-001`, Evidence Number `EV-001`, Examiner `Sandeep Chakka`, image directory `/home/sandeep/forensics_output/`, image filename `disk_image`, with Calculate MD5, Calculate SHA-1, and "Verify image after acquisition" all selected.

<img width="1796" height="1382" alt="image" src="https://github.com/user-attachments/assets/9f90c3eb-4162-4018-8cf5-1050d3681f50" />


<img width="1796" height="1382" alt="image" src="https://github.com/user-attachments/assets/64695dc3-2cbe-48ca-89d0-90d64c338038" />


#### Step 15: Start Acquisition — Running

Clicking Start begins the acquisition. The main window shows the progress of the running job.

<img width="2880" height="1800" alt="image" src="https://github.com/user-attachments/assets/3fdc356e-4235-4928-a04e-cd25e24327dd" />


#### Step 16: Acquisition Complete — Finished

<img width="2880" height="1800" alt="image" src="https://github.com/user-attachments/assets/40eff934-4b1f-4d94-9a01-182f018ead38" />


The loop device `/dev/loop0` holding `virtual_disk.img` (100 MB) was acquired completely at an average speed of 190.73 MB/s with 0 bad sectors, producing `guymager_image.dd` and its info file `guymager_image.info`. The image was verified after acquisition and hash values matched.

---

## Result

Volatile memory (RAM) and a virtual disk image were successfully acquired on **Kali Linux (ARM64)** using `dd`, `dc3dd`, and `Guymager` — the Linux equivalents of AccessData FTK Imager. Both MD5 and SHA-1 hash values were verified and confirmed to match, proving the integrity of all acquired forensic evidence.

## Verification Results

| Parameter | Volatile Memory | Disk Image |
|-----------|----------------|------------|
| Tool | `dd` | `dc3dd` + `Guymager` |
| Output File | `memdump.mem` | `disk_image.dd` |
| Size | 96 MB | 96 MB |
| MD5 Verification | ✅ Match | ✅ Match |
| SHA1 Verification | ✅ Match | ✅ Match |
| Bad Blocks | None | None |
| Status | **Complete** | **Complete** |
