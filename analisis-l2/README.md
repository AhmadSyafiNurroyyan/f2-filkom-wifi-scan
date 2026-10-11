# Analisis Wi-Fi Lantai 2

## Input dan asumsi

- Sumber passive scan: tiga CSV di `../hasil-scan/point/`, digabung menjadi `input/passive_L2.csv`.
- Sumber active test: tiga CSV di `../hasil-scan/point/`, digabung menjadi `input/active_L2.csv`.
- SSID: `WiFi-UB.x`, band 5 GHz.
- Titik `L2-017` dipetakan ke titik denah `F2.6` berdasarkan klarifikasi survei.
- `L2-F2` dipetakan ke titik hijau tanpa label di sisi kiri denah.
- `L2-000` dipertahankan sebagai titik kalibrasi dan ditempatkan pada lokasi F2.6.
- `L2-FKoridor` tidak diberi koordinat karena pada denah jalur kuning adalah lintasan roaming, bukan titik ukur.
- Koordinat dibuat dari pratinjau denah yang diperkecil agar dapat diproses tanpa kehabisan memori. Posisi bersifat aproksimasi piksel, bukan hasil klik `mark_points.py`.
- Lima AP fisik dibuat dari lima simbol AP biru pada denah. Identitas AP fisik belum dikaitkan secara pasti dengan BSSID controller.

## Menjalankan ulang

```powershell
.\.venv\Scripts\python.exe analyze.py `
  --passive analisis-l2\input\passive_L2.csv `
  --titik analisis-l2\titik_L2.csv `
  --ssid "WiFi-UB.x" --band 5 `
  --denah mapping\denah-preview.png --scale 46.4 `
  --out analisis-l2\output\L2 `
  --active analisis-l2\input\active_L2.csv `
  --ap-fisik analisis-l2\ap_fisik_L2.csv
```

## Keluaran

- `output/L2_titik.csv`: metrik per titik, status, dan penyebab.
- `output/L2_area_lemah.csv`: kelompok area lemah.
- `output/L2_ap.csv`: estimasi posisi radio.
- `output/L2_heatmap.png`: interpolasi RSSI.
- `output/L2_status.png`: status titik, estimasi AP, dan AP fisik.
- `output/L2_kanal.png`: distribusi kanal.
- `output/L2_ringkasan.txt`: ringkasan angka.
- `output/L2_roaming_transitions.csv`: perpindahan BSSID dan indikasi timeout ping dalam jendela +/- 5 detik.

## Batasan interpretasi

Data Windows tidak menyediakan noise floor, sehingga SNR tidak dapat dihitung. RSSI pada log roaming adalah aproksimasi dari kualitas sinyal Windows. Hasil koordinat dan estimasi AP harus divalidasi dengan sketsa/foto lapangan sebelum digunakan sebagai ukuran presisi.
