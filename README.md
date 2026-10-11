## 8. Prosedur Lapangan (Pertemuan 1)

### 8.1 Tahap 1: Sketsa denah dan titik ukur

**Acuan: peta jalur evakuasi Gedung F** yang terpasang di setiap lantai. Foto peta itu, tetapi **tetap gambar denah sendiri** dan ukur ulang di lapangan. Hal-hal yang terbaca dari peta lantai 3:
- Ukuran lantai **36 × 36 m**, dengan grid kolom **7,2 m**: sumbu **A–F** (arah horizontal) dan **1–6** (arah vertikal).
- **Garis jalur evakuasi = koridor.** Koridor berbentuk **cincin** yang mengelilingi blok ruang di tengah, dengan **persimpangan (junction) di keempat sudutnya**.
- **Kotak-kotak = ruang kelas**, yaitu blok tengah (±6 kelas) serta deretan kelas di sisi atas dan bawah denah.
- Lift, toilet, dan tangga ada di sisi sumbu F. Tangga lain ada di dekat sumbu C–B (atas dan bawah). Hall terbuka ada di sisi sumbu F dan sumbu A–B.
- **Perhatikan mata angin:** pada peta evakuasi, **Utara ada di bawah** gambar. Pakai **nama sumbu grid** (mis. "koridor sumbu 5 antara E–D"), jangan kiri/kanan.

**Sketsa (di kertas A3/milimeter blok, pensil):**
- Gambar dinding luar, koridor, ruang (tulis nomor/nama ruang), pintu, tangga, lift, dan toilet.
- **Ukur dan tulis angka dimensi** pada sketsa: panjang dan lebar koridor serta lebar setiap ruang. Gunakan meteran/laser, atau **hitung ubin lantai** (ukur satu ubin, mis. 60 × 60 cm), atau langkah kaki yang sudah dikalibrasi.
- Tandai **titik acuan (0,0) = perpotongan sumbu F dan 6** (sudut gedung di sisi toilet) dan panah arah utara. Titik ini sama di ketiga lantai.
- Beri keterangan material dinding penting: beton, bata, kaca, partisi gipsum, pintu/lemari logam.
- Tandai **setiap AP yang terlihat** (plafon/dinding) dengan ■ dan kode `AP-L{n}-01`, `AP-L{n}-02`, …, lalu **foto** setiap AP. Jika label atau model AP terbaca dari bawah, catat juga.

**Titik ukur (target 25–35 titik per lantai), gambar di sketsa dan beri nomor `L{n}-001`, `L{n}-002`, …:**
- Koridor cincin: 1 titik setiap **± 5 m**, **ditambah 1 titik di setiap persimpangan/sudut koridor**, karena di sudut sinyal sering terhalang dinding dari dua arah.
- Setiap **ruang kelas** yang bisa diakses: minimal **1 titik di tengah**. Kelas besar: tambah 1 titik di **pojok terjauh dari AP**. Titik di dalam kelas penting karena mahasiswa memakai Wi-Fi di kelas, bukan di koridor.
- Area khusus: tangga, depan lift, lounge mahasiswa, ujung koridor, dan dekat jendela/balkon.
- 1 **titik kalibrasi** `L{n}-000` di area terbuka dekat AP.
- **Foto sketsa** setelah selesai dan unggah ke folder bersama. Foto ini menjadi acuan digitalisasi.

### 8.2 Tahap 2: Kalibrasi

Di titik `L{n}-000`, semua laptop melakukan 5 scan bersamaan, dan ponsel mencatat RSSI SSID kampus dari WiFiAnalyzer:

```bat
:: Windows
python scan_point.py --titik L3-000 --lantai 3 --slot T1 --device laptopA --out data\passive_L3_A.csv --n 5 --ssid "WiFi-UB.x"
```
```bash
# Ubuntu/Linux dan macOS
python scan_point.py --titik L3-000 --lantai 3 --slot T1 \
     --device laptopA --out data/passive_L3_A.csv --n 5 --ssid "WiFi-UB.x"
```

Catat selisih RSSI antar-perangkat untuk AP yang sama (**offset perangkat**) di Lembar Kerja. Offset ini dibahas di laporan dan **tidak** dipakai untuk mengoreksi data secara otomatis.

### 8.3 Tahap 3: Survei pasif

> Koordinat titik **belum diperlukan** saat survei. Cukup nomor titik dari sketsa. Koordinat disambungkan saat analisis (`analyze.py --titik`).

Di setiap titik:
1. Berdiri tepat di titik sesuai sketsa. Pegang laptop setinggi **± 1 m** dengan layar menghadap **arah yang sama** di semua titik. Surveyor tidak berdiri di antara laptop dan arah AP terdekat.
2. Jalankan (3 scan, ± 20–25 detik):
   ```bat
   :: Windows
   python scan_point.py --titik L3-017 --lantai 3 --slot T1 --device laptopA --out data\passive_L3_A.csv --ssid "WiFi-UB.x" --catatan "kelas kosong, pintu tertutup"
   ```
   ```bash
   # Ubuntu/Linux dan macOS
   python scan_point.py --titik L3-017 --lantai 3 --slot T1 \
        --device laptopA --out data/passive_L3_A.csv --ssid "WiFi-UB.x" --catatan "kelas kosong"
   ```
3. **Tulis RSSI terkuat SSID kampus** yang tampil di layar ke sketsa, di samping nomor titik. Angka ini dipakai untuk memilih titik uji aktif (§8.4) dan membantu presentasi.
4. Secara bersamaan, anggota lain mengambil screenshot **WiFiAnalyzer** (tab *Access Points*, filter 5 GHz) dengan nama file `L3-017.png`.
5. Isi Lembar Kerja: centang titik, perkiraan jumlah orang, dan pintu terbuka/tertutup.
6. Pindah ke titik berikutnya. Target: ± 2 menit per titik.

**Jalur darurat (jika laptop bermasalah):** catat manual dari WiFiAnalyzer ke CSV dengan kolom `titik_id,ssid,bssid,channel,rssi_dbm`, **satu baris untuk setiap BSSID 5 GHz yang terlihat** (bukan hanya yang terkuat). Karena SSID-nya sama di semua AP, BSSID adalah satu-satunya pembeda AP. Catat juga baris eduroam/FILKOM EVENT bila terlihat. `analyze.py` tetap bisa memprosesnya.

### 8.4 Tahap 4: Survei aktif dan roaming

**Pilih 6–8 titik** dari RSSI yang tertulis di sketsa:
- 3 titik dengan **RSSI terendah atau tidak terdeteksi**,
- 2 titik dengan **RSSI terbaik** (pembanding),
- 1–3 titik yang **RSSI-nya bagus tetapi dicurigai**, misalnya area padat mahasiswa atau banyak AP terdengar.

Hubungkan laptop ke **`WiFi-UB.x`**, lalu di setiap titik jalankan:
```bat
:: Windows
python active_point.py --titik L3-017 --out data\active_L3.csv
```
```bash
# Ubuntu/Linux dan macOS
python active_point.py --titik L3-017 --out data/active_L3.csv
```

> **Target latensi default adalah `filkom.ub.ac.id`** (server kampus). **Gateway jaringan UB tidak membalas ping**, karena ICMP diblokir. Ini sudah diuji langsung di lantai 3, jadi jangan memakai gateway sebagai target. Jika target sama sekali tidak membalas ping, skrip otomatis mengukur latensi dengan **koneksi TCP port 443**, dan metodenya dicatat di kolom `metode_rtt`. Throughput iperf3 bersifat **opsional**: tambahkan `--iperf <IP_SERVER>` hanya jika ada server iperf3.

**Uji roaming (1 lintasan koridor, ± 5 menit):** berjalan pelan dari ujung ke ujung koridor sambil menjalankan dua terminal:
```bat
:: Terminal 1 (Windows) - BSSID dan sinyal setiap detik
python roam_log.py --out raw\roaming_L3.csv
:: Terminal 2 (PowerShell) - ping dengan cap waktu
ping -t filkom.ub.ac.id | ForEach-Object { "{0} {1}" -f (Get-Date -Format HH:mm:ss), $_ } | Tee-Object raw\roaming_ping_L3.txt
```
```bash
# Ubuntu/Linux dan macOS
python roam_log.py --out raw/roaming_L3.csv                          # terminal 1
ping -i 0.2 filkom.ub.ac.id | while read l; do echo "$(date +%T) $l"; done | tee raw/roaming_ping_L3.txt   # terminal 2
```
Tandai di sketsa lokasi saat log menampilkan **"PINDAH AP"**. Catat **sinyal sesaat sebelum pindah** dan **ping yang hilang** di sekitar waktu tersebut. Jika klien tetap menempel ke AP jauh sampai sinyal < −75 dBm, itu tanda **sticky client** atau kurangnya overlap antar-AP. Karena SSID-nya sama di semua lantai, perhatikan juga apakah klien **berpindah ke AP lantai lain**. Cocokkan BSSID di log dengan daftar AP per lantai hasil pleno (§9.5).

> Pada Windows, sinyal di `roam_log.py` adalah **aproksimasi** dari persentase kualitas sinyal ((persen/2) − 100), karena pembacaan dBm yang akurat memerlukan scan yang akan mengganggu uji roaming. Sebutkan hal ini di laporan.

### 8.5 Penutup

Gabungkan data tim A dan B, lalu unggah seluruh folder ke folder bersama:
```bash
# Linux/macOS
head -1 data/passive_L3_A.csv > data/passive_L3.csv
tail -q -n +2 data/passive_L3_A.csv data/passive_L3_B.csv >> data/passive_L3.csv
```
```bat
:: Windows (PowerShell)
Import-Csv data\passive_L3_A.csv, data\passive_L3_B.csv -Encoding UTF8 | Export-Csv data\passive_L3.csv -NoTypeInformation -Encoding UTF8
```

---

## 9. Pengolahan dan Analisis (selama seminggu)

### 9.1 Digitalisasi denah

**Tool:** draw.io/diagrams.net (disarankan), Inkscape, atau LibreCAD.

| Aturan | Ketentuan |
|---|---|
| Skala | **1 meter = 20 piksel**. Di draw.io, aktifkan grid 20 px, sehingga 1 kotak = 1 m |
| Titik acuan (0,0) | **Perpotongan sumbu F dan 6** diletakkan di piksel **(100, 100)** pada ketiga denah, supaya ketiga lantai bisa ditumpuk |
| Ukuran kanvas | **920 × 920 px** untuk ketiga lantai (36 m × 20 px = 720 px, ditambah margin 100 px di setiap sisi) |
| Isi | Dinding, ruang + nomor, pintu, tangga, lift, toilet, material penting, AP fisik (■ + kode), panah utara |
| Ekspor | `denah_L{n}.png` tanpa margin + file sumber `.drawio`/`.svg` |

Setelah itu:
```bash
python mark_points.py --denah denah/denah_L3.png --lantai 3 --out data/titik_L3.csv
```
Klik dua ujung koridor dan masukkan panjang sebenarnya. Skala yang tercetak harus **≈ 20 px/m**; jika jauh berbeda, periksa kembali denahnya. Lalu klik titik ukur **berurutan sesuai nomor di sketsa**. Buat juga `ap_fisik_L{n}.csv`:
```csv
ap_id,x_px,y_px,keterangan
AP-L3-01,340,180,plafon koridor depan R.3.x
```

### 9.2 Menjalankan analisis

```bash
python analyze.py --passive data/passive_L3.csv --titik data/titik_L3.csv --ssid "WiFi-UB.x" \
       --denah denah/denah_L3.png --scale 20 --out output/L3 \
       --active data/active_L3.csv --ap-fisik data/ap_fisik_L3.csv
```

| File keluaran | Isi | Dipakai untuk |
|---|---|---|
| `L3_heatmap.png` | Heatmap RSSI AP terkuat + garis kontur −67 dBm (hijau) dan −75 dBm (merah) | Peta cakupan |
| `L3_status.png` | Titik berwarna menurut status, area lemah A1, A2, … (arsir merah), AP estimasi (▲ penuh = di lantai ini, △ kosong = mungkin tembus dari lantai lain), dan AP fisik (□) | **Peta Kualitas Sinyal dan Penempatan AP** |
| `L3_titik.csv` | Metrik per titik: rssi1, rssi2, kanal, lebar, CCI, SNR, util, status, penyebab | Tabel lampiran |
| `L3_area_lemah.csv` | Daftar area lemah + penyebab dominan | Inti analisis |
| `L3_ap.csv` | Estimasi posisi setiap radio AP (weighted centroid dari 3 titik terkuat), kanal, keyakinan (tinggi: RSSI maks ≥ −55 dBm; sedang: −65 s.d. −55; rendah: < −65) | Pemetaan AP |
| `L3_kanal.png` | Jumlah radio per kanal 5 GHz (SSID kampus vs lainnya) | Analisis channel plan |
| `L3_ringkasan.txt` | Persentase cakupan dan jumlah status | Angka kunci laporan |

Contoh keluaran dari **data sintetis** (bukan data FILKOM) ada di `scripts_survey_wifi/contoh_output/`.

> **Keterbatasan yang wajib disadari:**
> (a) Interpolasi heatmap hanya diisi sampai **6 m dari titik ukur terdekat**. Contohnya, blok kelas di tengah koridor cincin tidak akan "dikarang" nilainya jika tidak ada titik di dalamnya. Jika semua titik berada pada satu garis, skrip otomatis beralih ke IDW (radius 5 m). Metode yang dipakai tercantum di judul heatmap.
> (b) Estimasi posisi AP adalah **perkiraan**. Keyakinan "sedang" (△) berarti AP kemungkinan tembus dari lantai atas/bawah atau berada di ruang tertutup. Keyakinan "rendah" tidak digambar, tetapi tetap tercantum di `L{n}_ap.csv`. Konfirmasi lantai asal AP saat pleno (`gabung_lantai.py`).
> (c) BSSID digabung menjadi satu radio hanya jika **kanalnya sama, SSID-nya berbeda, MAC-nya hanya beda 1 oktet, dan RSSI-nya hampir identik** (median selisih ≤ 4 dB) di titik yang sama. Contohnya WiFi-UB.x, eduroam, dan FILKOM EVENT dari satu AP. BSSID ber-SSID sama tidak pernah digabung, sehingga dua AP WiFi-UB.x ber-MAC berurutan tetap terhitung terpisah. Skrip menampilkan baris `Info:` saat penggabungan terjadi.
> (d) Survei dilakukan pada satu rentang waktu. Kondisi beban pada jam lain bisa berbeda.

### 9.3 Langkah interpretasi

1. **Cakupan:** berapa % titik yang BAIK (≥ −67 dBm)? Di ruang mana blank spot berada?
2. **Pemetaan AP:** bandingkan ▲ (estimasi) dengan □ (fisik).
   - AP fisik tanpa ▲ di dekatnya → AP mati, memakai SSID lain, atau hanya 2,4 GHz.
   - ▲ tanpa □ → AP tersembunyi di dalam ruangan atau berasal dari lantai lain.
3. **Kanal:** apakah AP bertetangga memakai kanal yang sama? Berapa lebar kanalnya? Apakah kanal DFS dimanfaatkan?
4. **Area lemah:** untuk setiap area A1, A2, …, tentukan akar masalah dengan tabel §9.4.
5. **Verifikasi aktif:** apakah titik yang statusnya buruk memang menunjukkan throughput rendah, loss, atau RTT tinggi? Apakah ada titik "RSSI bagus tetapi lambat"?
6. **Roaming:** di mana perpindahan AP terjadi, dan apakah ada ping yang hilang?

### 9.4 Tabel diagnosis: gejala → akar masalah → rekomendasi

| Gejala di data | Kemungkinan akar masalah | Contoh rekomendasi |
|---|---|---|
| RSSI < −75, AP terdekat jauh (> 15–20 m) | **Coverage gap**: jumlah AP kurang | Tambah AP, atau pindahkan AP ke titik tengah area |
| RSSI rendah padahal AP dekat, terhalang dinding beton/ruang tertutup | **Atenuasi material**: AP di koridor melayani ruang kelas | Tempatkan AP di dalam ruang kelas; perhatikan orientasi AP |
| RSSI baik, tetapi throughput rendah atau loss tinggi, CCI > 2 | **Co-channel interference** | Susun ulang kanal, turunkan lebar kanal ke 40/20 MHz, manfaatkan kanal DFS, atur daya pancar |
| RSSI baik, utilisasi kanal > 50%, banyak stasiun | **Beban/kepadatan** | Tambah AP kapasitas (daya rendah), band steering, batasi data rate rendah |
| AP terkuat berasal dari lantai lain | **Floor bleed**: desain vertikal tidak diperhitungkan | Kurangi daya, atur penggunaan kanal secara vertikal, tambah AP di lantai sendiri |
| Tidak ada AP cadangan ≥ −75 dBm, roaming disertai ping hilang | **Overlap antar-AP kurang** | Rapatkan jarak AP (overlap sel 15–20%), aktifkan 802.11k/v/r |
| SNR rendah padahal RSSI cukup | **Noise/interferensi non-Wi-Fi** | Identifikasi sumber interferensi, pindahkan kanal |

### 9.5 Analisis lintas lantai (Pertemuan 2)

Setelah ketiga kelompok mengunggah `L{n}_ap.csv` dan `L{n}_ringkasan.txt`, asisten menjalankan:
```bash
python gabung_lantai.py --ap 2:output/L2_ap.csv 3:output/L3_ap.csv 4:output/L4_ap.csv \
       --titik 2:output/L2_titik.csv 3:output/L3_titik.csv 4:output/L4_titik.csv \
       --ringkasan output/L2_ringkasan.txt output/L3_ringkasan.txt output/L4_ringkasan.txt \
       --out output/gedung
```
Hasilnya menunjukkan:
- **Lantai asal setiap AP** (berdasarkan BSSID), sehingga daftar AP per lantai bisa disusun meskipun SSID-nya sama.
- **Titik yang AP terkuatnya berasal dari lantai lain** (`gedung_titik_lantai_lain.csv`). Di titik ini, klien kemungkinan besar tersambung ke AP lantai atas/bawah. Ini indikasi kuat kurangnya AP di lantai tersebut.
- Selisih RSSI antar-lantai, yang menjadi **estimasi kasar atenuasi pelat lantai**.

 Selisih ini juga dipengaruhi jarak horizontal, jadi yang paling bermakna adalah titik yang berada tepat di atas atau di bawah AP. Hal ini menjadi bahan diskusi.
