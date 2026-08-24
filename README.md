# library-wazuh-single

Sandbox definition CyberRangeCZ untuk 1 VM dengan Wazuh all-in-one
(Indexer + Server + Dashboard) terinstall otomatis.

## Spesifikasi

- 1 host: `wazuh-server`
- Image: `ubuntu-noble-x86_64`
- Flavor: `wazuh.custom` (4 vCPU, 16GB RAM, 60GB disk) — flavor custom, lihat langkah pembuatannya di bawah
- Network: `wazuh-net` (10.1.60.0/24), langsung accessible oleh user (tanpa router)

Flavor ini didesain supaya RAM tetap besar (Wazuh butuh RAM cukup banyak
untuk Indexer/Elasticsearch-nya), tapi disk dikecilkan dari `standard.large`
bawaan (80GB) ke 60GB — supaya 2 pool sekaligus bisa muat di kapasitas
disk host yang terbatas, dengan margin aman.

## Cara pakai

1. **Buat flavor custom dulu** di OpenStack (sekali saja, tidak perlu diulang
   tiap build pool):
   ```bash
   openstack flavor create \
     --ram 16384 \
     --disk 60 \
     --vcpus 4 \
     wazuh.custom
   ```
2. Push repo ini ke GitHub kamu sendiri.
3. Di web UI CyberRangeCZ: **Sandbox Definitions → Create**, masukkan URL git repo ini.
4. Buat **Pool** dari definition ini, lalu **Allocate**.
4. Tunggu provisioning selesai (~15-20 menit, karena instalasi Wazuh cukup lama).
5. Akses VM lewat SSH (management access atau user access sesuai konfigurasi),
   lalu buka dashboard Wazuh di browser: `https://<ip-wazuh-server>` (port 443).
6. Login dashboard: username `admin`, password ada di
   `/root/wazuh-install-output.log` pada VM (cari baris
   "The generated password for admin is").

## Catatan resource

Wazuh all-in-one cukup berat untuk RAM. Kalau di flavor `standard.medium` (4GB)
terasa lambat atau service Wazuh Indexer gagal start, naikkan flavor node
`wazuh-server` di `topology.yml` ke `standard.large` (16GB RAM) — asalkan
kapasitas host mengizinkan.

## Menambahkan agent Wazuh (opsional, pengembangan lanjutan)

Untuk skenario yang lebih lengkap (monitoring endpoint lain), tambahkan host
baru di `topology.yml` sebagai target monitoring, lalu tambahkan role baru
di `provisioning/roles/` untuk install Wazuh agent dan mendaftarkannya ke
`wazuh-server` ini sebagai manager.
