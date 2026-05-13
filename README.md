# MAP1202 SSD Identity Modifier

A tool and guide for modifying the Serial Number, Model Number, and Firmware Version of NVMe SSDs equipped with a **Maxio MAP1202** controller, using the `MXMPTool_MAP1202_USB_V0_01_009d.exe` utility.

> **Warning:** Modifying your SSD voids the manufacturer warranty and may damage the drive. Proceed at your own risk.

---

## Requirements

- An NVMe SSD with a **Maxio MAP1202** controller (see compatible drives below)
- An **M.2 to USB adapter** to connect the SSD externally
- A Windows PC
- `MXMPTool_MAP1202_USB_V0_01_009d.exe` (included in this repository)
- *(Optional)* CrystalDiskInfo to verify changes

---

## Compatible Drives

The following drives use the Maxio MAP1202 controller and have been confirmed or reported to work:

| Drive | Capacity | Tested |
|---|---|---|
| fanxiang S500 Pro 2TB | 2048 GB | Yes |
| fanxiang S500 Pro 1TB | 1024 GB | Reported |
| Fikwot FN501 Pro 2TB | 2048 GB | Reported |
| Fikwot FN501 Pro 1TB | 1024 GB | Reported |
| TeamGroup MP33 Pro 1TB | 1024 GB | Reported |
| Verbatim Vi3000 1TB | 1024 GB | Reported |
| Origin Inception TLC 830 Pro 2TB | 2048 GB | Reported |

To find additional compatible drives, visit [ssd-tester.de](https://ssd-tester.de/top_ssd.php) and filter by controller. Note: the controller column is only visible in the German language version of the site.

---

## Usage

### Step 1 — Connect the drive

Use an M.2 to USB adapter to connect the target SSD to your PC. It is recommended to perform this on a secondary machine rather than your main system.

### Step 2 — Open MXMPTool

Launch `MXMPTool_MAP1202_USB_V0_01_009d.exe`. If the drive is detected correctly, the first device entry will highlight in yellow and display the drive's capacity.

### Step 3 — Configure Test Items

Navigate to the **Test Items** tab and enable the following options:

- [x] Modify Serial Number
- [x] Modify Model Number
- [x] Modify FW Version
- [x] Modify Device Settings

### Step 4 — Set Device Settings

Navigate to the **Device Settings** tab and enter your desired values:

| Field | Constraints |
|---|---|
| Firmware Version | Numbers only, max. 8 characters |
| Model Number | Letters only, max. 20 characters |
| Serial Number | Alphanumeric, max. 13 characters |

### Step 5 — Apply

Return to the **Main Page** and press **Start**. A successful operation will turn the device entry green with a status of **Pass**.

Unplug the drive and reconnect it. Open CrystalDiskInfo to verify the updated device information.

---

## Clearing the Windows WMIC / WMI Cache

After spoofing the drive, Windows may still report the old serial number through WMIC or other system queries because device identifiers are cached in the WMI repository and the Plug-and-Play registry. A full Windows reinstallation is one solution, but the following methods avoid that.

### Method 1 — Reset the WMI Repository (recommended)

Run the following commands in an elevated Command Prompt (`cmd` as Administrator):

```cmd
net stop winmgmt /y
winmgmt /resetrepository
net start winmgmt
```

This rebuilds the WMI database from scratch. After a reboot, WMIC will re-enumerate all hardware and pick up the new serial number.

If `winmgmt /resetrepository` fails (common on Windows 10/11), use the salvage variant:

```cmd
net stop winmgmt /y
winmgmt /salvagerepository
net start winmgmt
```

### Method 2 — Remove the Device from the PnP Registry

Windows stores per-device entries under `HKLM\SYSTEM\CurrentControlSet\Enum\NVMe\`. Deleting the stale entry forces re-enumeration on next boot.

1. Open `regedit` as Administrator.
2. Navigate to:
   ```
   HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\NVMe\
   ```
3. Locate the sub-key matching your drive's old model/serial (the key name includes the original identifiers).
4. Delete the entire sub-key for that device.
5. Reboot — Windows will re-detect the drive using the new spoofed values.

> **Caution:** Only delete the key for the target drive. Removing unrelated entries can destabilize the system.

### Method 3 — Uninstall via Device Manager

1. Open **Device Manager** (`devmgmt.msc`).
2. Expand **Disk drives**, right-click the target SSD, and select **Uninstall device**.
3. Check **Delete the driver software for this device** if the option appears.
4. Unplug the drive, reboot, then reconnect it.

Windows will re-enumerate the drive on plug-in and register the new serial.

### Verification

After applying any of the above methods, confirm the cache is cleared:

```cmd
wmic diskdrive get model,serialnumber
```

The output should reflect the new values set with MXMPTool.

---

## Troubleshooting

| Issue | Solution |
|---|---|
| Drive not recognized / program freezes | Run the tool as Administrator; try a different USB port or slot |
| "Device modify failed" error | Try a different USB slot; erase the drive beforehand; adjust the Device Settings values |

---

## Credits

Guide by **iThaClan**, **Fendix**
