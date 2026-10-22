# homelab-nextcloud-pi4
Self-hosted home server infrastructure on Raspberry Pi 4 (Arch Linux ARM) featuring Nextcloud, Paperless-ngx, Tailscale, Uptime Kuma, and Docker Compose. Includes RCA and troubleshooting for USB storage mounting.
## 🛠️ Infrastructure & Service Stack

All services are containerized using **Docker** & **Docker Compose** with persistence mapped to an external 1TB HDD (`/mnt/nas`):

* **Storage & Productivity:**
  * **Nextcloud:** Primary cloud storage, file sync, and collaboration platform.
  * **Paperless-ngx:** Automated document indexing, OCR, and document management system.
* **Networking & Monitoring:**
  * **Tailscale:** Secure mesh VPN (Overlay network) for encrypted remote access without exposing ports.
  * **Uptime Kuma:** Real-time infrastructure monitoring and uptime metrics.
  * **Telegram Bot:** Automated alerting and notification system integrated with monitoring services.
* **Container Orchestration:**
  * **Docker & Docker Compose:** Modular microservices management.

---

## ⚠️ Incident Report & Root Cause Analysis (RCA)

Following an abrupt power interruption, Nextcloud and secondary services lost connection to the primary storage volume. Diagnostic analysis revealed two interrelated issues:

1. **Power Instability (Under-Voltage Event):**
   Interrogating the SoC power management state via `vcgencmd get_throttled` returned the bitmask **`0x50000`**. This confirmed a historical under-voltage event triggered by the peak current draw of the external mechanical drive during boot.

2. **Dynamic Device Re-assignment & Mount Drift:**
   When power to the USB bus restored, the kernel re-indexed the storage block device from `/dev/sda1` to `/dev/sdb1`. Consequently, static path references to `/mnt/nas` failed, leaving Docker containers disconnected from persistent data.

---

## 🛠️ Remediation & Implementation

### Phase 1: Storage Re-attachment & Integrity Verification
Forced an unmount of orphaned handles and manually remounted the block device to verify data integrity across all service directories:
```bash
sudo umount /dev/sda1
sudo mount /dev/sda1 /mnt/nas
ls -la /mnt/nas
