# Dokumen Perancangan Solusi Wi-Fi Lantai 2 Gedung F FILKOM

## 1. Tujuan

Dokumen ini menerjemahkan hasil analisis site survey lantai 2 Gedung F FILKOM
menjadi rancangan perbaikan yang dapat diterapkan dan diuji. Fokus rancangan
adalah menjaga cakupan Wi-Fi, mengurangi interferensi dan kepadatan kanal,
memulihkan konektivitas di F2.8, serta menurunkan latensi dan jitter pada area
yang masih memiliki RSSI kuat tetapi kualitas layanannya belum baik.

Rancangan ini menggunakan data pada:

- `analisis-l2/output/L2_ringkasan.txt`;
- `analisis-l2/output/L2_titik.csv`;
- `analisis-l2/output/L2_area_lemah.csv`;
- `analisis-l2/output/L2_roaming_transitions.csv`;
- hasil active test pada `analisis-l2/input/active_L2.csv`; dan
- heatmap Cambium pada folder `cambium-heatmap`.

## 2. Ringkasan kondisi saat ini

### 2.1 Temuan kuantitatif

| Indikator | Hasil | Makna perancangan |
| --- | ---: | --- |
| Titik yang dianalisis | 9 titik | Sampel cukup untuk diagnosis awal, tetapi belum menjadi desain final |
| RSSI AP terkuat >= -67 dBm | 100% | Tidak ada bukti kebutuhan penambahan AP hanya karena blank spot |
| AP cadangan >= -75 dBm | 100% | Dasar redundansi dan roaming sudah tersedia |
| CCI > 2 radio | 44,4% | Perlu optimasi kanal, lebar kanal, dan daya pancar |
| RSSI median / minimum | -42,7 / -58,0 dBm | Cakupan radio relatif kuat |
| Status baik | 0 titik | RSSI kuat belum menjamin kualitas layanan |
| Status perhatian | 8 titik | Masalah dominan berupa interferensi, utilisasi, atau latensi |
| Status buruk | 1 titik | F2.8 membutuhkan investigasi dan perbaikan prioritas |
| Area lemah terdeteksi | F2.8 | Active test mengalami packet loss 100% |

### 2.2 Masalah yang harus diselesaikan

1. **F2.8 tidak dapat memberikan layanan secara konsisten.** Passive scan
   masih melihat radio dengan RSSI -58 dBm, tetapi active test mencatat loss
   100%. Ini lebih mengarah ke masalah akses, asosiasi, konfigurasi, kanal,
   atau gangguan lokal daripada sekadar kekurangan daya sinyal.
2. **Interferensi co-channel terjadi di beberapa lokasi.** CCI tercatat
   sampai 7 radio pada L2-F2 dan 9 radio pada L2-F2.2. Pada F2.8 terdapat
   4 radio yang berbagi/berdekatan pada kondisi pengukuran.
3. **Kanal dan utilisasi radio padat.** L2-000 mencatat utilisasi sekitar 55%,
   sedangkan L2-F2.2 sekitar 63%. Data controller historis untuk AP F2.6
   juga menunjukkan utilisasi 5 GHz sekitar 92%; angka ini harus dikonfirmasi
   kembali karena waktunya berbeda dari survei.
4. **Latensi dan jitter tinggi walaupun RSSI baik.** Active test menunjukkan
   RTT dan jitter tinggi terutama pada F2.9, F2.1, F2.2, F2.4, dan F2.5.
   Hal ini dapat berasal dari airtime contention, antrean pada AP atau
   uplink, beban controller, jalur jaringan, atau metode pengujian.
5. **Roaming terjadi, tetapi data timeout belum cukup untuk menyimpulkan
   roaming gagal.** Terdapat 8 perpindahan BSSID dan tidak ada timeout ping
   yang tercatat pada jendela yang tersedia. Namun, RSSI roaming adalah
   aproksimasi Windows dan sampel ping pada berkas transisi tidak memadai.

## 3. Prinsip desain

Rancangan perbaikan mengikuti prinsip berikut:

- **Perbaiki akar masalah sebelum menambah perangkat.** Penambahan AP di
  area yang sudah memiliki RSSI kuat dapat memperburuk CCI dan airtime.
- **Prioritaskan kualitas airtime, bukan hanya bar RSSI.** Keputusan
  kanal, lebar kanal, daya pancar, dan jumlah klien harus dilihat bersama
  dengan utilisasi, retry, latency, dan packet loss.
- **Gunakan kanal 20 MHz sebagai baseline stabilitas.** Lebar 40/80 MHz
  hanya digunakan bila hasil validasi menunjukkan spektrum dan kapasitas
  benar-benar mendukungnya.
- **Pertahankan desain 5 GHz sebagai band utama.** Band 2,4 GHz digunakan
  untuk kompatibilitas perangkat lama atau kebutuhan jangkauan, bukan sebagai
  solusi otomatis untuk semua masalah 5 GHz.
- **Setiap perubahan harus dapat dibalik dan dibandingkan.** Konfigurasi
  awal, hasil pengukuran sebelum perubahan, dan hasil sesudah perubahan
  disimpan sebagai baseline.

## 4. Rancangan solusi

### 4.1 Prioritas P0: pemulihan layanan F2.8

F2.8 harus diperiksa terlebih dahulu sebelum perubahan massal. Urutan
diagnosis dan tindakan yang disarankan:

1. Ulangi passive scan dan active test di F2.8 menggunakan minimal dua
   perangkat, pada dua waktu berbeda. Catat BSSID yang terasosiasi, kanal,
   lebar kanal, RSSI, retry, tx rate, DHCP, gateway, dan DNS.
2. Dari controller Cambium, cocokkan AP fisik dan BSSID yang terlihat di F2.8.
   Pastikan AP tidak berada dalam kondisi down, terisolasi, overload, atau
   gagal meneruskan trafik ke jaringan kabel.
3. Verifikasi VLAN, ACL, DHCP scope, captive portal, dan kebijakan akses
   untuk klien yang terhubung melalui AP tersebut. Packet loss 100% dengan
   RSSI -58 dBm tidak boleh langsung dianggap sebagai coverage gap.
4. Periksa kanal 157 dan radio sekitar F2.8, termasuk radar/DFS event,
   retry rate, noise floor pada controller, dan perubahan kanal otomatis.
5. Bila AP yang melayani F2.8 terlalu jauh atau terhalang struktur bangunan,
   lakukan uji sementara dengan daya/kanal yang terkontrol. Penambahan AP
   permanen hanya dipertimbangkan bila loss tetap terjadi setelah jalur
   jaringan dan konfigurasi dinyatakan normal.

**Kriteria selesai P0:**

- packet loss ke gateway dan endpoint pengujian < 2% pada tiga pengujian;
- klien dapat memperoleh DHCP dan akses ke gateway secara konsisten;
- tidak ada asosiasi yang berpindah-pindah tanpa sebab selama pengujian;
- RSSI minimal tetap >= -67 dBm atau tersedia AP cadangan >= -75 dBm; dan
- hasil dikonfirmasi pada jam sepi dan jam sibuk.

### 4.2 Prioritas P1: rencana kanal dan pengurangan CCI

Buat profil kanal per radio berdasarkan data controller terbaru, bukan hanya
hasil scan satu waktu. Untuk lantai 2, gunakan prinsip berikut:

- mulai dengan lebar kanal 20 MHz pada 5 GHz;
- susun kanal antar-AP bertetangga agar tidak menggunakan kanal yang sama
  secara berulang;
- hindari memaksa semua radio ke kanal yang sama hanya karena kanal tersebut
  terlihat paling bersih pada satu titik;
- evaluasi kanal DFS berdasarkan stabilitas dan riwayat perpindahan kanal;
- aktifkan auto-channel hanya bila batas kanal, daya, dan perilaku perpindahan
  sudah dikendalikan; dan
- untuk 2,4 GHz, batasi rencana pada kanal tidak tumpang tindih 1, 6, dan 11
  dengan lebar 20 MHz.

Lokasi yang menjadi prioritas audit kanal adalah:

| Lokasi | Bukti | Tindakan |
| --- | --- | --- |
| L2-F2 | CCI 7 radio, RTT rata-rata sekitar 63 ms | Audit kanal radio yang dominan dan kurangi overlap |
| L2-F2.2 | CCI 9 radio, utilisasi sekitar 63%, RTT sekitar 55 ms | Prioritas tertinggi untuk channel plan dan airtime |
| L2-F2.8 | CCI 4 radio dan packet loss 100% | Gabungkan audit kanal dengan diagnosis P0 |
| L2-017 | CCI 3 radio, RTT sekitar 60 ms | Audit kanal 157 dan AP yang melayaninya |
| L2-000 | Utilisasi sekitar 55% | Pantau setelah perubahan kanal dan daya |

### 4.3 Prioritas P1: pengaturan daya pancar dan sel

RSSI kuat di sekitar AP dan keberadaan AP cadangan di seluruh titik
menunjukkan bahwa daya pancar tidak boleh dinaikkan secara seragam. Lakukan
penyetelan bertahap:

1. Petakan RSSI tiap AP pada titik survei setelah identitas AP fisik dan BSSID
   dikonfirmasi.
2. Kurangi daya pada radio yang terlalu dominan dan menyebabkan klien tetap
   menempel pada AP lama ketika bergerak.
3. Hindari perbedaan daya yang terlalu besar antar-AP bertetangga.
4. Validasi ulang area tepi F2.8, F2.1, dan koridor setelah setiap perubahan.
5. Jangan mengubah daya dan kanal seluruh AP sekaligus; ubah satu kelompok
   radio yang berdekatan agar penyebab perubahan dapat ditelusuri.

Tujuan pengaturan daya adalah membentuk sel yang saling beririsan secukupnya
untuk roaming, bukan membuat setiap AP terdengar maksimum di seluruh lantai.

### 4.4 Prioritas P1: penurunan latensi dan jitter

Karena RSSI mayoritas baik, latensi harus dipisahkan menjadi tiga segmen:

1. **Klien ke AP:** ukur retry, airtime, MCS, tx rate, dan antrean radio.
2. **AP ke gateway:** lakukan ping dari sisi kabel/controller bila tersedia
   untuk membedakan masalah radio dari masalah uplink.
3. **Gateway ke endpoint:** bandingkan RTT ke gateway, DNS, dan endpoint
   `tcp443`; jangan menggunakan satu endpoint internet sebagai satu-satunya
   indikator kesehatan Wi-Fi.

Tindakan teknis:

- turunkan kepadatan airtime melalui channel plan 20 MHz dan pengurangan CCI;
- periksa uplink AP, error interface, duplex, PoE, dan kapasitas switch;
- periksa utilisasi CPU/memori AP/controller serta jumlah klien per radio;
- aktifkan fitur steering/band steering hanya setelah cakupan 5 GHz stabil;
- gunakan QoS/WMM sesuai kebutuhan aplikasi, tanpa menjadikan QoS sebagai
  pengganti perbaikan interferensi; dan
- ulangi pengujian dengan durasi dan ukuran sampel yang sama agar RTT antar
  lokasi dapat dibandingkan.

Target operasional awal yang disarankan adalah RTT ke gateway <= 20 ms pada
jam normal, jitter <= 10 ms, dan packet loss < 2%. Target ini perlu
disesuaikan dengan topologi jaringan kampus setelah baseline gateway tersedia.

### 4.5 Prioritas P2: optimasi roaming

Delapan transisi BSSID menunjukkan bahwa klien dapat berpindah AP, tetapi
belum membuktikan kualitas handoff. Setelah kanal dan daya stabil:

- tetapkan SSID, keamanan, VLAN, dan kebijakan akses yang konsisten pada AP
  lantai 2;
- gunakan 802.11k/v bila didukung dan kompatibel dengan controller serta
  perangkat klien;
- uji 802.11r secara bertahap pada SSID uji atau kelompok perangkat karena
  kompatibilitas klien harus dipastikan;
- lakukan walk test pada lintasan yang sama dengan pencatatan BSSID, RSSI,
  waktu perpindahan, packet loss, dan RTT; dan
- pastikan klien tidak melakukan sticky roaming karena AP lama masih terlalu
  kuat dibanding AP tujuan.

Roaming dianggap berhasil bila perpindahan tidak menyebabkan packet loss
yang terlihat oleh aplikasi, tidak ada putus sesi yang tidak semestinya, dan
waktu handoff konsisten pada beberapa percobaan.

## 5. Urutan implementasi

| Tahap | Kegiatan | Output |
| --- | --- | --- |
| 0. Baseline | Simpan konfigurasi controller, mapping BSSID-AP, utilisasi, retry, dan hasil uji | Baseline yang dapat dibandingkan |
| 1. F2.8 | Validasi radio, VLAN, DHCP, uplink, dan kanal | Layanan F2.8 pulih atau akar masalah terisolasi |
| 2. Kanal | Terapkan channel plan 20 MHz pada radio prioritas | CCI dan utilisasi menurun |
| 3. Daya | Seimbangkan sel AP bertetangga | Roaming dan cakupan tepi membaik |
| 4. Latensi | Validasi radio, uplink, gateway, dan endpoint | Lokasi bottleneck teridentifikasi |
| 5. Roaming | Uji 802.11k/v/r dan walk test | Handoff tervalidasi |
| 6. Evaluasi | Jalankan ulang survei seluruh titik | Keputusan final penambahan AP atau optimasi lanjutan |

Setiap tahap harus memiliki backup konfigurasi dan jadwal perubahan yang
meminimalkan gangguan layanan.

## 6. Rencana validasi pascaperubahan

Gunakan titik dan metode yang sama agar hasil sebelum dan sesudah dapat
dibandingkan. Tambahkan pengukuran di sekitar F2.8 karena satu titik saja
belum cukup untuk menggambarkan batas area masalah.

### Metrik penerimaan

| Metrik | Kondisi awal | Target awal |
| --- | --- | --- |
| Packet loss F2.8 | 100% | < 2% pada tiga pengujian |
| Titik RSSI AP utama >= -67 dBm | 100% | Dipertahankan >= 95% |
| Titik AP cadangan >= -75 dBm | 100% | Dipertahankan >= 95% |
| CCI > 2 radio | 44,4% | Turun secara terukur, target <= 10% |
| Utilisasi radio padat | 55-63%; historis F2.6 92% | < 50% pada jam normal, diverifikasi jam sibuk |
| RTT ke gateway | Belum dipisahkan dari endpoint | <= 20 ms pada jam normal |
| Jitter ke gateway | Tinggi pada beberapa titik | <= 10 ms |
| Roaming | 8 transisi, timeout tidak tervalidasi | Handoff konsisten tanpa putus sesi |

Target adalah sasaran awal desain, bukan klaim bahwa data saat ini sudah
mengukur seluruh metrik tersebut. Bila target tidak tercapai, ulangi diagnosis
berdasarkan segmen klien-AP, AP-gateway, dan gateway-endpoint.

## 7. Keputusan penambahan AP

Penambahan AP **belum menjadi rekomendasi utama** berdasarkan data saat ini.
Seluruh titik memiliki RSSI dan AP cadangan yang memenuhi ambang dasar, sehingga
menambah AP sebelum mengatasi CCI dan F2.8 berisiko meningkatkan interferensi.

Penambahan AP baru hanya disetujui apabila setelah P0-P2:

1. F2.8 masih mengalami packet loss atau kapasitas tidak mencukupi;
2. hasil pengukuran tambahan menunjukkan area fisik yang konsisten berada di
   bawah target cakupan; atau
3. utilisasi airtime tetap tinggi pada jam sibuk setelah channel plan,
   pengaturan daya, dan optimasi uplink dilakukan.

Jika penambahan AP diperlukan, lakukan predictive survey dan uji penempatan
sementara terlebih dahulu. AP baru harus memiliki kanal, daya, VLAN,
keamanan, dan kebijakan roaming yang konsisten dengan desain lantai 2.

## 8. Batasan dan kebutuhan data lanjutan

- Identitas AP fisik belum dipetakan secara pasti ke BSSID controller.
- Koordinat titik dan estimasi posisi AP masih berupa aproksimasi piksel.
- Noise floor tidak tersedia, sehingga SNR dan kualitas spektrum tidak dapat
  disimpulkan secara langsung.
- Utilisasi F2.6 berasal dari snapshot historis dan perlu dikonfirmasi pada
  waktu survei yang sebanding.
- Pengujian throughput belum tersedia dan perlu ditambahkan bila kapasitas
  pengguna menjadi sasaran desain.
- Data roaming tidak memiliki sampel ping yang cukup pada setiap transisi.
- Hasil satu endpoint `tcp443` tidak cukup untuk menentukan lokasi bottleneck.

Dengan batasan tersebut, dokumen ini merupakan rancangan remediasi dan
validasi, bukan pengesahan konfigurasi produksi final. Keputusan akhir harus
dibuat setelah baseline controller, pemetaan BSSID-AP, dan pengukuran ulang
tersedia.

