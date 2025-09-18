## QNAP TS-133 NAS - QSWAP Manager alkalmazás
---

Ez a **QSWAP Manager** (QPKG csomag) lehetővé teszi, hogy egy **USB 3.2** interfészen keresztül csatlakoztatott meghajtót használjunk swap (lapozási) területként a **[QNAP TS-133](https://www.qnap.com/hu-hu/product/ts-133)** NAS eszközön. Ez gyorsabb, kíméli a HDD-t, és növeli a rendszer stabilitását. Ez a megoldás különösen hasznos kis mennyiségű RAM-mal rendelkező modelleknél (≤ 2GB RAM). Alapértelmezés szerint a QNAP NAS a beépített HDD-n hozza létre a swap területet (`/share/CACHEDEV1_DATA/.swap/qnap_swap`), ami lassú, a rendszerlemezt terheli, írás intenzív műveletek esetén hátrányos lehet és kevésbé ideális, ha nagy mennyiségű RAM-ot helyettesítünk.

---
## Figyelmeztetés
---

- A program használata saját felelősségre történik
- Javasolt jó minőségű, gyors USB 3.2 meghajtót használni (pl. NVME, SSD)
- Nem hivatalos QPKG – fejlesztői mód szükséges a telepítéshez
- [x] Az érvényes digitális aláírás nélküli alkalmazások telepítésének engedélyezése
- Ez a csomag nem hivatalosan támogatott. A használatából eredő bármilyen kárért a fejlesztő nem vállal felelősséget
- A program eltávolítása utána újra kell indítani a NAS készüléket

---
## Előkészület
---

- [x] Minőségi NVME/SSD meghajtó használata (nem pendrive)
- ![](/img/7.png)
- [x] Kompakt méretű NVME/SSD fém burkolatú ház
- ![](/img/8.png)
- [x] Az érvényes digitális aláírás nélküli alkalmazások telepítésének engedélyezése (App Center)
- ![](/img/1.png)
- [x] OS: ≥ QTS 5.2.6
- [x] Egyéb, nem rendszerszintű programok leállítása (App Center)
- ![](/img/2.png)

---
## Előnyök
---

- Gyorsabb memória kezelés (NVMe SSD akár 5× gyorsabb, mint a HDD)
- 8GB swap fájl írás eredménye: `8589934592 bytes (8.0GB) copied, 52.075203 seconds, 157.3MB/s`
- 16GB swap fájl írás eredménye: `17179869184 bytes (16.0GB) copied, 99.273097 seconds, 165.0MB/s`
- Nagyobb rendelkezésre álló "virtuális memória"
- Kevesebb lefagyás, OOM (Out of Memory) hiba
- Csendesebb működés (kevesebb HDD használat)
- Csökkentett írási terhelés a fő lemezen
- Plex-et vagy más médialejátszót futtatsz
- Docker-t vagy Container Station-t használsz
- Több fájlátvitel vagy torrent kliens aktív
- Sok QNAP szolgáltatás (pl. indexelés, AI album, Qsirch) fut a háttérben
- HDD életciklusának védelme

---
## Kiadás: V0.1
---

- [x] Első kiadás
- [ ] Ismert problémák

---
## QSWAP telepítése és beállítása
---

- Formatáljuk EXT4 fájlrendszerként a(z) SSD/NVME meghajtót
- Csatlakoztassuk az USB 3.2 interfészen keresztül a(z) SSD/NVME meghajtót
- Az App Centerben manuálisan telepítsük a QSWAP programot

![](/img/3.png)

---
## QSWAP működése és tulajdonsága
---

- **Változatok:** 8GB (8GB_QSWAP_0.1.qpkg), és 16GB (16GB_QSWAP_0.1.qpkg) swap tárterület beállítással érhető el
- **swap_check.sh:** 5 percenként ellenőrzi (`crontab`), hogy a swap tárterület be van-e állítva. Esetleges meghajtó cserekor automatikusan beállítja.
- **swap_setup.sh:** Fő program. Az App Centerből irányítható. **start-stop** funkció, valamint CLI-n keresztül a **restore** funkció is aktiválható.
- **Start funkció:** QNAP swap meghajtó leállítása és törlése (`/share/CACHEDEV1_DATA/.swap/qnap_swap`) ---> USB 3.2 interfészen csatlakoztatott meghajtón `swapfile` beállítása ---> Lock fájl (szkript ellenőrzés) létrehozása (`/var/lock/swap_setup.lck`) ---> egyéb finomhangolások elvgézése (`vm.swappiness=10, vm.vfs_cache_pressure=50, vm.dirty_ratio=10, vm.dirty_background_ratio=5, vm.min_free_kbytes=65536`) ---> Crontab beállítása ---> naplófájl kiíratása hangjelzéssel (`/var/log/swap_setup_log.txt`)
- **Stop funkció:** USB 3.2 interfészen csatlakoztatott meghajtón `swapfile` törlése ---> Lock fájl (szkript ellenőrzés) törlése (`/var/lock/swap_setup.lck`) ---> Crontab visszaállítása ---> naplófájl kiíratása hangjelzéssel (`/var/log/swap_setup_log.txt`)
- **Restore funkció:** Mindent visszaállít az eredeti állapotba (újraindításig)
- **Napló fájl:** `/var/log/swap_setup_log.txt`
- **Crontab fájl:** `/etc/config/crontab`
- **Új swap fájl (USB 3.2):** `/share/external/DEV3302_1/swapfile`
- **Régi swap fájl (HDD):** `/share/CACHEDEV1_DATA/.swap/qnap_swap`
- **Lock fájl:** `/var/lock/swap_setup.lck`
- **QSWAP program:** `/share/CACHEDEV1_DATA/.qpkg/QSWAP`

![](/img/5.png)

---
## Tipp
---

**Ellenőrzés (CLI):**

- `free -m`
- `cat /proc/swaps`
- `ls /share/CACHEDEV1_DATA/.swap`
- `dd if=/dev/zero of=/share/external/DEV3302_1/test bs=1M count=8192`
- `cat /etc/os-release`
- `ls /var/lock`
- `cat -n /etc/config/crontab`
- `ls /share/CACHEDEV1_DATA/.qpkg/QSWAP `
- `cat /var/log/swap_setup_log.txt `
- `cat /etc/config/qpkg.conf`
- `sysctl -a`
- `ls /etc/config/`
- `watch -n 2 free -m`
- `df -h`
- `fdisk -l`
- `lsusb`
- `udevadm info --query=all --name=/dev/sdb`

**Ellenőrzés (Web UI):**

![](/img/4.png)

![](/img/6.png)

**Forrás:**

- QPKG Development Guidelines: [LINK](https://wiki.qnap.com/wiki/QPKG_Development_Guidelines)
- App Center: [LINK](https://www.xeams.com/remove-apps-qnap.htm)
- Scripting: [LINK](https://forum.qnapclub.de/thread/45028-script-autorun-sh-erstellen-von-start-und-stopp-scripten/)
- Crontab: [LINK](https://www.qnap.com/en/how-to/faq/article/how-to-add-jobs-to-crontab-to-schedule-a-job)
