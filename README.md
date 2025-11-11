# Komdat-Jarkom-Subnetting

# Tugas 1 Jarkom - Desain Jaringan Yayasan ARA (VLSM & Routing)

Halo guys! Ini laporan tugas jaringan kami yang bakal ngebahas gimana cara bikin jaringan buat Yayasan Pendidikan ARA pake Cisco Packet Tracer. Simak ya!

- **Mata Kuliah:** Komunikasi Data dan Jaringan Komputer  
- **Nama:** Muhammad Khairul Yahya  
- **NRP:** 5027241092  

---

## 1. Perhitungan Pembagian IP (VLSM) 🧮

**Kenapa sih harus pake VLSM?**  
Soalnya yayasan ini punya 7 bagian dengan kebutuhan host yang beda-beda banget - ada yang butuh 380 host (Sekretariat), ada yang cuma butuh 2 host (Link WAN). Kalau pake subnet biasa bakal boros banget IP-nya.

**Gimana solusinya?**  
Kita pake **VLSM** biar lebih efisien. Kita mulai dari network dasar **`10.132.0.0`** (sesuai aturan NRP), terus kita potong-potong sesuai kebutuhan, dari yang terbesar dulu.

**Hasil Akhirnya:**  
Ini nih pembagian IP yang udah kami hitung:

### Tabel Pembagian IP (VLSM)

| Bagian Jaringan | Butuh Berapa Host | Network | Prefix | Subnet Mask | Range IP | Broadcast | Gateway |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Sekretariat | 380 | `10.132.0.0` | /23 | `255.255.254.0` | `10.132.0.1` - `10.132.1.254` | `10.132.1.255` | `10.132.0.1` |
| Kurikulum | 220 | `10.132.2.0` | /24 | `255.255.255.0` | `10.132.2.1` - `10.132.2.254` | `10.132.2.255` | `10.132.2.1` |
| Guru & Tendik | 95 | `10.132.3.0` | /25 | `255.255.255.128` | `10.132.3.1` - `10.132.3.126` | `10.132.3.127` | `10.132.3.1` |
| Sarpras | 45 | `10.132.3.128` | /26 | `255.255.255.192` | `10.132.3.129` - `10.132.3.190` | `10.132.3.191` | `10.132.3.129` |
| Pengawas | 18 | `10.132.3.192` | /27 | `255.255.255.224` | `10.132.3.193` - `10.132.3.222` | `10.132.3.223` | `10.132.3.193` |
| Server & Admin | 6 | `10.132.3.224` | /29 | `255.255.255.248` | `10.132.3.225` - `10.132.3.230` | `10.132.3.231` | `10.132.3.225` |
| Link WAN | 2 | `10.132.3.232` | /30 | `255.255.255.252` | `10.132.3.233` - `10.132.3.234` | `10.132.3.235` | - |

---

## 2. Desain Jaringan & Setting IP 🖥️

**Topologi Jaringan:**  
Kita bikin dua lokasi:  
* **`Router_Pusat`**: Nganggurin 5 jaringan (Sekretariat, Kurikulum, Guru, Sarpras, Server)  
* **`Router_Cabang`**: Nganggurin 1 jaringan (Pengawas)  
* **Link WAN**: Nyambungin kedua router  
* **Switch & PC**: Tiap jaringan ada switch dan PC contoh  

![Topologi Jaringan](link-screenshot-kalian-disini)

**Cara Setting IP:**  
Semua device (PC dan Router) kita set manual **sesuai tabel VLSM** di atas.  
* Tiap PC dapet IP dari range yang udah ditentuin  
* Gatewaynya juga sesuai  
* Hasilnya? Bisa ping semua! 🎉  

---

## 3. Masalah yang Ditemuin & Solusinya 🔧

Waktu setup, kami nemuin beberapa kendala teknis, terutama di `Router_Pusat`.

### 3.1. Masalah: Port Router Pusat Gak Bisa Diisi IP

**Masalahnya:**  
Pas mau kasih IP di port `Router_Pusat` (misal `GigabitEthernet0/1/0`), router nya nolak dan keluar error. Ternyata modul yang kita tambahin itu **modul Switch**, jadi portnya gak bisa langsung dikasih IP.

**Solusinya:**  
Kita pake **Router-on-a-Stick** pake VLAN:  
1. **Buat "Kamar" Virtual:** Kita bikin interface virtual buat tiap LAN (misal `interface Vlan10` buat Guru)  
2. **Kasih IP ke "Kamar":** Gateway (misal `10.132.3.1`) kita kasih ke interface virtual ini  
3. **Masukin Port ke "Kamar":** Port fisik dimasukin ke VLAN yang sesuai  

**Hasilnya:**  
Ini alasan kenapa di `Router_Pusat` pake `interface Vlan...` buat semua jaringan, termasuk Link WAN (`Vlan99`).

### 3.2. Solusi: Router Cabang Lebih Gampang

**Bedanya:**  
`Router_Cabang` punya port router asli yang bisa langsung dikasih IP  

**Hasilnya:**  
Settingnya lebih simpel - langsung kasih IP (`10.132.3.234...` dan `10.132.3.193...`) ke port fisik tanpa perlu ribet pake VLAN

---

## 4. Routing & Penggabungan Jaringan (CIDR) 🗺️

**Kenapa perlu Routing?**  
Setelah semua device bisa komunikasi di jaringan lokalnya sendiri, kedua router masih belum tahu jalan ke jaringan seberang. Jadi kita perlu kasih "peta" atau **Static Route**.

**Cara Ngasih Tahu Router:**  
* **Di `Router_Pusat`:** Kita kasih tau cara ke jaringan "Pengawas":  
    ```bash
    ip route 10.132.3.192 255.255.255.224 10.132.3.234
    ```

* **Di `Router_Cabang`:** Daripada buat 5 rute terpisah (satu-satu buat tiap LAN di pusat), kita **gabung jadi satu** pake CIDR.

### Cara Ngegabung Jaringan (Supernetting)

Kita liat di Tabel VLSM dan nemu bahwa semua 7 jaringan ini ternyata berurutan dari `10.132.0.0` sampai `10.132.3.255`.

| Blok Network | Range IP | Dipake Buat Apa |
| :--- | :--- | :--- |
| `10.132.0.0/24` | `10.132.0.0` - `10.132.0.255` | **Sekretariat** |
| `10.132.1.0/24` | `10.132.1.0` - `10.132.1.255` | **Sekretariat** |
| `10.132.2.0/24` | `10.132.2.0` - `10.132.2.255` | **Kurikulum** |
| `10.132.3.0/24` | `10.132.3.0` - `10.132.3.255` | **Guru, Sarpras, Pengawas, Server, WAN** |

### Hasil Penggabungan (CIDR)

Dari tabel di atas, semua jaringan bisa digabung jadi satu Supernet: **`10.132.0.0/22`**.

| Jaringan Gabungan | Prefix | Mask |
| :--- | :--- | :--- |
| `10.132.0.0` | `/22` | `255.255.252.0` |

Jadi di `Router_Cabang` cuma perlu **satu perintah** doang:  
```bash
ip route 10.132.0.0 255.255.252.0 10.132.3.233
