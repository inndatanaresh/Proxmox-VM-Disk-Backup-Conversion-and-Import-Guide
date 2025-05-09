# Proxmox VM Disk Backup Conversion and Import Guide

This document provides a step-by-step guide to converting and importing Proxmox VM disk backups from Ceph RBD format to VDI format (commonly used in Ubuntu environments).

---

## Step 1: Convert Ceph RBD Disk to QCOW2 Format

1. **Identify the storage pool type**:

   ```bash
   rados lspools
   ```

   This will help you determine the pool type (e.g., `hdd-pool` or `ssd-pool`).

2. **Convert the RBD disk to QCOW2 format**:

   ```bash
   qemu-img convert -f raw -O qcow2 rbd:<pool-type>/vm-<vmid>-disk-0 <diskname>.qcow2
   ```

   Replace:

   * `<pool-type>` with your actual pool name (`hdd-pool` or `ssd-pool`)
   * `<vmid>` with your VM ID
   * `<diskname>` with your desired output filename

---

## Step 2: Convert QCOW2 to VDI Format

Convert the previously created QCOW2 file to VDI:

```bash
qemu-img convert -f qcow2 -O raw <diskname>.qcow2 <diskname>.vdi
```

Replace `<diskname>` with your actual disk file name.

---

## Step 3: Import VDI Disk Back to Proxmox VM

Use the following command to import the converted disk to your VM:

```bash
qm importdisk <vmid> /root/<diskname>.vdi <target-storage> --format raw
```

Replace:

* `<vmid>` with your VM ID
* `<diskname>` with your disk file name
* `<target-storage>` with the name of your Proxmox storage (e.g., `ceph-hdd`)

> **Note**: Ensure your VM is powered off before importing the disk.

---

## Step 4: Attach the Imported Disk to VM

1. In the Proxmox UI, select the VM.
2. Go to the **Hardware** tab.
3. Locate the **Unused Disk 0** at the bottom.
4. Detach any existing disk if needed.
5. Edit the unused disk and attach it.
6. Go to the **Options** tab and enable **SCSI controller** for the secondary disk.
7. Save and start the VM.
8. Verify that your files are present inside the VM.

---

## Common Errors and Troubleshooting

* **Disk not recognized**: If you didn’t select the correct disk format (e.g., `raw`, `qcow2`, or `vdi`) during conversion/import, the VM may not boot.
* **No IP address**: If the VM boots but has no IP address, recheck the VM settings or recreate the VM with correct configuration as per the above steps.

---

## Final Notes

* Always ensure the VM is shut down before performing disk import or modifications.
* Take a backup before modifying or replacing VM disks.
* Use consistent naming conventions to avoid confusion.

---

End of Document
